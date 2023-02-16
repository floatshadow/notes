commit: 3fa8a5a864c48a490625648387a86be3eb7c2c06
看样例 `01-vector-add`
```python
import torch

import triton
import triton.language as tl


@triton.jit
def add_kernel(
    x_ptr,  # *Pointer* to first input vector.
    y_ptr,  # *Pointer* to second input vector.
    output_ptr,  # *Pointer* to output vector.
    n_elements,  # Size of the vector.
    BLOCK_SIZE: tl.constexpr,  # Number of elements each program should process.
                 # NOTE: `constexpr` so it can be used as a shape value.
):
    # There are multiple 'programs' processing different data. We identify which program
    # we are here:
    pid = tl.program_id(axis=0)  # We use a 1D launch grid so axis is 0.
    # This program will process inputs that are offset from the initial data.
    # For instance, if you had a vector of length 256 and block_size of 64, the programs
    # would each access the elements [0:64, 64:128, 128:192, 192:256].
    # Note that offsets is a list of pointers:
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
    # Create a mask to guard memory operations against out-of-bounds accesses.
    mask = offsets < n_elements
    # Load x and y from DRAM, masking out any extra elements in case the input is not a
    # multiple of the block size.
    x = tl.load(x_ptr + offsets, mask=mask)
    y = tl.load(y_ptr + offsets, mask=mask)
    output = x + y
    # Write x + y back to DRAM.
    tl.store(output_ptr + offsets, output, mask=mask)

```

首先看 decorator `@triton.jit`
位于 `python/triton/runtime/jit.py`
```python
def jit(
    fn: Optional[T] = None,
    *,
    version=None,
    do_not_specialize: Optional[Iterable[int]] = None,
) -> Union[JITFunction[T], Callable[[T], JITFunction[T]]]:
    """
    Decorator for JIT-compiling a function using the Triton compiler.

    :note: When a jit'd function is called, :code:`torch.tensor` arguments are
        implicitly converted to pointers using the :code:`.data_ptr()` method.

    :note: This function will be compiled and run on the GPU. It will only have access to:

           * python primitives,
           * builtins within the triton package,
           * arguments to this function,
           * other jit'd functions

    :param fn: the function to be jit-compiled
    :type fn: Callable
    """

    def decorator(fn: T) -> JITFunction[T]:
        assert callable(fn)
        return JITFunction(
            fn,
            version=version,
            do_not_specialize=do_not_specialize,
        )

    if fn is not None:
        return decorator(fn)

    else:
        return decorator
```

对于 JIT 的函数, 它的具体内部实现是 JITFunction, 继承于 KeernelInterface[T]
```python
class KernelInterface(Generic[T]):
    run: T

    def __getitem__(self, grid) -> T:
        """
        A JIT function is launched with: fn[grid](*args, **kwargs).
        Hence JITFunction.__getitem__ returns a callable proxy that
        memorizes the grid.
        """
        return cast(T, functools.partial(cast(Callable, self.run), grid=grid))

```
在 `01-vector-add` 中 kernel 的启动也是通过类似的语法
```python
def add(x: torch.Tensor, y: torch.Tensor):
    ...
    # NOTE:
    #  - Each torch.tensor object is implicitly converted into a pointer to its first element.
    #  - `triton.jit`'ed functions can be indexed with a launch grid to obtain a callable GPU kernel.
    #  - Don't forget to pass meta-parameters as keywords arguments.
    add_kernel[grid](x, y, output, n_elements, BLOCK_SIZE=1024)
```

接下来我们回过头来看 JITFunction 这个重要的类
先看构造的 `__init__`
获取 kernel 的源码以及 lancher
```python
    # function source code (without decorators)
    self.src = textwrap.dedent(inspect.getsource(fn))
    self.src = self.src[self.src.find("def"):]
    ...
    # launcher
    self.run = self._make_launcher()
```

`__make_laucher` 是个很有意思的的方法,主要分为几个部分:
1. 处理常规参数和 constexpr 参数, specializtion 参数, 和启动 grid kernel 的参数
2. 用暴力的方法嵌入模板然后让 Python 执行, 后端 C++ Module 执行之后会返回一个 JIT 完成的 binary 并放入 cache, 这个过程会在调用 Kernel 时完成

下面具体看这段看起来就非常暴力的代码
```python
        src = f"""
def {self.fn.__name__}({', '.join(self.arg_names)}, grid, num_warps=4, num_stages=3, extern_libs=None, stream=None, warmup=False):
    sig_key =  {sig_keys},
    constexpr_key = {f'{constexpr_keys},' if len(constexpr_keys) > 0 else ()}
    spec_key = {f'{spec_keys},' if len(spec_keys) > 0 else ()}
    key = (version_key, sig_key, constexpr_key, spec_key)
    if not extern_libs is None:
      key = (key, tuple(extern_libs.items()))
    assert num_warps > 0 and (num_warps & (num_warps - 1)) == 0, "num_warps must be a power of 2"
    if callable(grid):
        grid = grid({{{grid_args}}})
    grid_size = len(grid)
    grid_0 = grid[0]
    grid_1 = grid[1] if grid_size > 1 else 1
    grid_2 = grid[2] if grid_size > 2 else 1
    device = torch.cuda.current_device()
    torch.cuda.set_device(device)
    if stream is None and not warmup:
      stream = get_cuda_stream(device)
    ...
"""
```
第一部分主要还是一些参数的获取, 这里值得一提是 grid, 在样例中 grid 是一个 lambda, 它希望获取一个参数列表, 然后从中获取启动的 grid size tuple, 否则可以给他一个固定的 grid size.
```python
    # The SPMD launch grid denotes the number of kernel instances that run in parallel.
    # It is analogous to CUDA launch grids. It can be either Tuple[int], or Callable(metaparameters) -> Tuple[int].
    # In this case, we use a 1D grid where the size is the number of blocks:
    grid = lambda meta: (triton.cdiv(n_elements, meta['BLOCK_SIZE']),)
```

下面这部分主要涉及到 JIT kernel 的 cache, 如果 kernel 已经被 cache, 那就直接取出, 否则就进行 JIT (triton.compile 和 bin.c_wrapper).
```python
"""
    ...
    try:
      bin = cache[device][key]
      if not warmup:
          bin.c_wrapper(grid_0, grid_1, grid_2, bin.num_warps, bin.shared, stream, bin.cu_function, triton.compiler.CompiledKernel.launch_enter_hook, triton.compiler.CompiledKernel.launch_exit_hook, bin, {args})
      return bin
    # kernel not cached -- compile
    except KeyError:
      # build dict of constant values
      args = [{args}]
      all_args = {', '.join([f'{arg}' for arg in self.arg_names])},
      configs = self._get_config(*all_args),
      constants = self._make_constants(constexpr_key)
      constants.update({{i: None for i, arg in enumerate(all_args) if arg is None}})
      constants.update({{i: 1 for i in configs[0].equal_to_1}})
      # build kernel signature -- doesn't include specialized arguments
      signature = {{ i: self._type_of(_key_of(arg)) for i, arg in enumerate(all_args) if i not in self.constexprs }}
      # build stub signature -- includes arguments that are specialized
      for i, arg in constants.items():
        if callable(arg):
          raise TypeError(f"Callable constexpr at index {{i}} is not supported")
      if not self._call_hook(key, signature, device, constants, num_warps, num_stages, extern_libs, configs):
        bin = triton.compile(self, signature=signature, device=device, constants=constants, num_warps=num_warps, num_stages=num_stages, extern_libs=extern_libs, configs=configs)
        if not warmup:
            bin.c_wrapper(grid_0, grid_1, grid_2, bin.num_warps, bin.shared, stream, bin.cu_function, triton.compiler.CompiledKernel.launch_enter_hook, triton.compiler.CompiledKernel.launch_exit_hook, bin, *args)
        self.cache[device][key] = bin
        return bin
      return None
"""
    
        scope = {"version_key": version_key(), "get_cuda_stream": get_cuda_stream,
                 "self": self, "_spec_of": self._spec_of, "_key_of": self._key_of,
                 "cache": self.cache, "triton": triton, "torch": torch}
        exec(src, scope)
        return scope[self.fn.__name__]
```

JIT 真正执行的部分在 `python/triton/compiler.py`
compile 函数接受一个 fn, 可以是一个 JITFunction, 也可以是一个 File Path, 这部分就有很多以 C++ Module 实现
```python
    # build compilation stages
    stages = {
        "ast": (lambda path: fn, None),
        "ttir": (lambda path: parse_mlir_module(path, context),
                 lambda src: ast_to_ttir(src, signature, configs[0], constants)),
        "ttgir": (lambda path: parse_mlir_module(path, context),
                  lambda src: ttir_to_ttgir(src, num_warps, num_stages, capability)),
        "llir": (lambda path: Path(path).read_text(),
                 lambda src: ttgir_to_llir(src, extern_libs, capability)),
        "ptx": (lambda path: Path(path).read_text(),
                lambda src: llir_to_ptx(src, capability)),
        "cubin": (lambda path: Path(path).read_bytes(),
                  lambda src: ptx_to_cubin(src, capability))
    }
```
首先将整个 compile 过程分成了若干个 stage, 根据不同的输入类型选择从不同的 stage 开始 (由 first stage 控制). 
对于 JITFunction, ast 是 first stage. 

在 compile 过程中, 由 CacheManager 负责生成的中间临时文件的管理, 默认路径在 `${HOME}/.triton/cache` 下.
我们测试了教程的 `01-vector-add` 得到 cache 下的两个目录:
```plaintext
|-- b79152443f57e5e0a6b86bd77483d8e2
|  |-- add_kernel.cubin
|  |-- add_kernel.llir
|  |-- add_kernel.ttgir
|  |-- add_kernel.ttir
|  |-- add_kernel.ptx
|  |-- add_kernel.json
|  |-- lock
|
|-- ecf4a1c73b5e4a41b5bfa6a132fac94f
   |-- add_kernel.so
   |-- lock
```
其中, ttir 和 tgir 都是基于 mlir 的, llir 基于 llvm, 最后由 llvm 后端编译到 ptx, 再由 NVIDIA 的 ptxas 编译到 cubin 进行测试, JSON 文件主要存储一些 metadata

下面我们详细来看编译的 Pipeline:
```python
    first_stage = list(stages.keys()).index(ext)
    asm = dict()
    module = fn
    # run compilation pipeline  and populate metadata
    for ir, (parse, compile) in list(stages.items())[first_stage:]:
        path = fn_cache_manager._make_path(f"{name}.{ir}")
        if ir == ext:
            next_module = parse(fn)
        elif os.path.exists(path) and\
                ir in metadata["ctime"] and\
                os.path.getctime(path) == metadata["ctime"][ir]:
            next_module = parse(path)
        else:
            next_module = compile(module)
            fn_cache_manager.put(next_module, f"{name}.{ir}")
        if os.path.exists(path):
            metadata["ctime"][ir] = os.path.getctime(path)
        asm[ir] = next_module if ir == "cubin" else str(next_module)
        if ir == "llir" and "shared" not in metadata:
            metadata["shared"] = _triton.get_shared_memory_size(module)
        if ir == "ptx":
            metadata["name"] = ptx_get_kernel_name(next_module)
        module = next_module
    # write-back metadata
    fn_cache_manager.put(json.dumps(metadata), f"{name}.json", binary=False)
    # return handle to compiled kernel
    return CompiledKernel(so_path, metadata, asm)
```

第一部分编译到 mlir, 先 parse 后 transform
```python
def parse_mlir_module(path, context):
    module = _triton.ir.parse_mlir_module(path, context)
    # module takes ownership of the context
    module.context = context
    return module

def ast_to_ttir(fn, signature, specialization, constants):
    mod, _ = build_triton_ir(fn, signature, specialization, constants)
    return optimize_triton_ir(mod)
```

`build_triton_ir` 这个函数从 JITFunction 存储的源代码 parse 成 Python AST, 然后实现了一个类 CodeGenerator (继承自 ast.NodeVisitor) 生成 triton_ir, 然后进行一些优化:
```python
def optimize_triton_ir(mod):
    pm = _triton.ir.pass_manager(mod.context)
    pm.enable_debug()
    pm.add_inliner_pass()
    pm.add_triton_combine_pass()
    pm.add_canonicalizer_pass()
    pm.add_cse_pass()
    pm.add_licm_pass()
    pm.run(mod)
    return mod
```

在我们具体分析之前, 先来看一下 C++ extension 实现 pybind 的接口, 我们能发现 `python/triton/_C/libtrition.so`.
根据 CMakeLists.txt, 它由 `python/src` 下的 `main.cc` 和 `triton.cc` 两个文件编译产生, `main.cc` 主要进行了 PYBIND11_MODULE 的定义, 随后调用 `triton,cc:init_triton` 进行具体类的绑定, IR Module 由 `init_triton_ir` 完成, 随后我们找到了具体的绑定:
```c++
  m.def(
      "parse_mlir_module",
      [](const std::string &inputFilename, mlir::MLIRContext &context) {
        // initialize registry
        // note: we initialize llvm for undef
        mlir::DialectRegistry registry;
        registry.insert<mlir::triton::TritonDialect,
                        mlir::triton::gpu::TritonGPUDialect,
                        mlir::math::MathDialect, mlir::arith::ArithmeticDialect,
                        mlir::StandardOpsDialect, mlir::scf::SCFDialect>();
        context.appendDialectRegistry(registry);
        context.loadAllAvailableDialects();

        // parse module
        mlir::OwningOpRef<mlir::ModuleOp> module(
            mlir::parseSourceFile(inputFilename, &context));
        // locations are incompatible with ptx < 7.5 !
        module->walk([](mlir::Operation *op) {
          op->setLoc(mlir::UnknownLoc::get(op->getContext()));
        });
        if (!module)
          throw std::runtime_error("Parse MLIR file failed.");

        return module->clone();
      },
      ret::take_ownership);
```