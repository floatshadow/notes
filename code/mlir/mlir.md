## Toy Tutorial

Dialect 注册 `toyc.cpp:dumpMLIR`
```cpp
  // Load our Dialect in this MLIR Context.
  context.getOrLoadDialect<mlir::toy::ToyDialect>();
```

OpBuilder `toyc.cpp:dumpMLIR` `MLIRGen.cpp:MLIRGenImpl`
```cpp
  mlir::MLIRContext context;
  ...
  /// The builder is a helper class to create IR inside a function. The builder
  /// is stateful, in particular it keeps an "insertion point": this is where
  /// the next operations will be introduced.
  mlir::OpBuilder builder;
```

管理 Pass `toyc.cpp:dumpMLIR`
```cpp
  mlir::PassManager pm(&context);
  // Apply any generic pass manager command line options and run the pipeline.
  applyPassManagerCLOptions(pm);

  // Add a run of the canonicalizer to optimize the mlir module.
  pm.addNestedPass<mlir::toy::FuncOp>(mlir::createCanonicalizerPass());
```

样例
```plaintext
# RUN: toyc-ch2 %s -emit=mlir 2>&1 | FileCheck %s

# User defined generic function that operates on unknown shaped arguments
def multiply_transpose(a, b) {
  return transpose(a) * transpose(b);
}

def main() {
  var a<2, 3> = [[1, 2, 3], [4, 5, 6]];
  var b<2, 3> = [1, 2, 3, 4, 5, 6];
  var c = multiply_transpose(a, b);
  var d = multiply_transpose(b, a);
  print(d);
}

# CHECK-LABEL: toy.func @multiply_transpose(
# CHECK-SAME:                               [[VAL_0:%.*]]: tensor<*xf64>, [[VAL_1:%.*]]: tensor<*xf64>) -> tensor<*xf64>
# CHECK:         [[VAL_2:%.*]] = toy.transpose([[VAL_0]] : tensor<*xf64>) to tensor<*xf64>
# CHECK-NEXT:    [[VAL_3:%.*]] = toy.transpose([[VAL_1]] : tensor<*xf64>) to tensor<*xf64>
# CHECK-NEXT:    [[VAL_4:%.*]] = toy.mul [[VAL_2]], [[VAL_3]] :  tensor<*xf64>
# CHECK-NEXT:    toy.return [[VAL_4]] : tensor<*xf64>

# CHECK-LABEL: toy.func @main()
# CHECK-NEXT:    [[VAL_5:%.*]] = toy.constant dense<{{\[\[}}1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64>
# CHECK-NEXT:    [[VAL_6:%.*]] = toy.reshape([[VAL_5]] : tensor<2x3xf64>) to tensor<2x3xf64>
# CHECK-NEXT:    [[VAL_7:%.*]] = toy.constant dense<[1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00]> : tensor<6xf64>
# CHECK-NEXT:    [[VAL_8:%.*]] = toy.reshape([[VAL_7]] : tensor<6xf64>) to tensor<2x3xf64>
# CHECK-NEXT:    [[VAL_9:%.*]] = toy.generic_call @multiply_transpose([[VAL_6]], [[VAL_8]]) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>
# CHECK-NEXT:    [[VAL_10:%.*]] = toy.generic_call @multiply_transpose([[VAL_8]], [[VAL_6]]) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>
# CHECK-NEXT:    toy.print [[VAL_10]] : tensor<*xf64>
# CHECK-NEXT:    toy.return

```

由于 `toyc.cpp: main` 中注册了 mlir 的 [debug 选项](https://mlir.llvm.org/getting_started/Debugging/)
```cpp
  // Register any command line options.
  mlir::registerAsmPrinterCLOptions();
  mlir::registerMLIRContextCLOptions();
  cl::ParseCommandLineOptions(argc, argv, "toy compiler\n");
```
因此我们可以打印出更为详细的信息, 例如使用 `--mlir-print-debuginfo` 得到 IR 的 location:
```mlir
module {
  toy.func @multiply_transpose(%arg0: tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":4:1), %arg1: tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":4:1)) -> tensor<*xf64> {
    %0 = toy.transpose(%arg0 : tensor<*xf64>) to tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":5:10)
    %1 = toy.transpose(%arg1 : tensor<*xf64>) to tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":5:25)
    %2 = toy.mul %0, %1 : tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":5:25)
    toy.return %2 : tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":5:3)
  } loc("Examples/Toy/Ch2/codegen.toy":4:1)
  toy.func @main() {
    %0 = toy.constant dense<[[1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64> loc("Examples/Toy/Ch2/codegen.toy":9:17)
    %1 = toy.reshape(%0 : tensor<2x3xf64>) to tensor<2x3xf64> loc("Examples/Toy/Ch2/codegen.toy":9:3)
    %2 = toy.constant dense<[1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00]> : tensor<6xf64> loc("Examples/Toy/Ch2/codegen.toy":10:17)
    %3 = toy.reshape(%2 : tensor<6xf64>) to tensor<2x3xf64> loc("Examples/Toy/Ch2/codegen.toy":10:3)
    %4 = toy.generic_call @multiply_transpose(%1, %3) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":11:11)
    %5 = toy.generic_call @multiply_transpose(%3, %1) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":12:11)
    toy.print %5 : tensor<*xf64> loc("Examples/Toy/Ch2/codegen.toy":13:3)
    toy.return loc("Examples/Toy/Ch2/codegen.toy":8:1)
  } loc("Examples/Toy/Ch2/codegen.toy":8:1)
} loc(unknown)

```

## Elements

参考 [Understanding the IR Structure](https://mlir.llvm.org/docs/Tutorials/UnderstandingTheIRStructure/) 和 MLIR Doxygen.


### Operation

乍一看类似于 `llvm::Instruction`, 有 RAUW, 链表操作, 获取 Operand Value 等操作, 不同的是:
- 一整个编译单元是一个 `ModuleOp`, 而在 LLVM 中是 `llvm::Module`.
- 每个 Operation 定义零个或多个 `mlir::Value` (即 Operation Results) 以及零个或多个操作数 `mlir::Value`, 注意 Doxygen 中提到一点定义的 Value 是按照逆序排在 Operation 地址前面, 例如 [Result2, Result1, Result0, *Operation*].
- mlir 中使用对应的 Block 和 Region 的概念. IR 是递归定义的, 一个 Operation 可以有多个内嵌的 Region, 每个 Region 是一个 Block 的链表, 每个 Block 又包含一串 Opeation, 相应的 Operation 也就支持一系列 get 方法. 
Block 有点类似于 `llvm::Function`.

`Operation` 和 `Op` 的区别:
- `Operation` 是 opaque 的, 只提供 generic API, `Op` 的子类是所有实际的 Operation 类型
- `Op` 及其子类使用起来类似于 `Operation *` 的智能指针, 因此 Op 某种意义上是值传递的

使用 llvm::dyn_cast 或 llvm::cast 做 Downcast, `getOperation()` 做 Upcast
```cpp
void processConstantOp(mlir::Operation *operation) {
  ConstantOp op = llvm::dyn_cast<ConstantOp>(operation);

  // This operation is not an instance of `ConstantOp`.
  if (!op)
    return;

  // Get the internal operation instance wrapped by the smart pointer.
  mlir::Operation *internalOperation = op.getOperation();
  assert(internalOperation == operation &&
         "these operation instances are the same");
}
```

Operation 的 Attribute:
```cpp
IntegerAttr opsetAttr = op->getAttrOfType<::mlir::Attribute>("onnx_opset").dyn_cast_or_null<IntegerAttr>();
if (opsetAttr)
  opset = opsetAttr.getValue().getSExtValue();
// getAttr
mlir::Type targetType = op->getAttr("to").cast<::mlir::TypeAttr>().getValue();
```

使用 ODS (tablegen) 指定 Operation 的属性

首先定义基类 
```tablegen
// Base class for toy dialect operations. This operation inherits from the base
// `Op` class in OpBase.td, and provides:
//   * The parent dialect of the operation.
//   * The mnemonic for the operation, or the name without the dialect prefix.
//   * A list of traits for the operation.
class Toy_Op<string mnemonic, list<Trait> traits = []> :
    Op<Toy_Dialect, mnemonic, traits>;
```

下面具体看一个例子:
```tablegen
def ConstantOp : Toy_Op<"constant"> {
  // The constant operation takes an attribute as the only input.
  // `F64ElementsAttr` corresponds to a 64-bit floating-point ElementsAttr.
  let arguments = (ins F64ElementsAttr:$value);

  // The constant operation returns a single value of TensorType.
  // F64Tensor corresponds to a 64-bit floating-point TensorType.
  let results = (outs F64Tensor);
}
```
`Toy_Op<"constant">` 将用于匹配 `ConstantOp::getOperationName`.
这里 ODS 可以从 arguments 和 results 中自动推导出 trait, 也即:

```cpp
class ConstantOp : public mlir::Op<
                     /// `mlir::Op` is a CRTP class, meaning that we provide the
                     /// derived class as a template parameter.
                     ConstantOp,
                     /// The ConstantOp takes zero input operands.
                     mlir::OpTrait::ZeroOperands,
                     /// The ConstantOp returns a single result.
                     mlir::OpTrait::OneResult,
                     /// We also provide a utility `getType` accessor that
                     /// returns the TensorType of the single result.
                     mlir::OpTraits::OneTypedResult<TensorType>::Impl> 
```

当然, 也可以手动设置 trait, 这样能给 Optimizer 更多的信息.
例如, 对于无副作用的 `toy.transpose` 我们可以设置为 `NoSideEffect`:
```tablegen
def TransposeOp : Toy_Op<"transpose", [NoSideEffect]> {...}
```
这样 Optimizer 能够消除掉那些 Dead Transpose

使用 custom 的 dump 格式, 需要重载 `void PrintOp::print(mlir::OpAsmPrinter &printer)` 和 `mlir::ParseResult PrintOp::parse(mlir::OpAsmParser &parser, mlir::OperationState &result)`:
```tablegen
/// Consider a stripped definition of `toy.print` here.
def PrintOp : Toy_Op<"print"> {
  let arguments = (ins F64Tensor:$input);

  // Divert the printer and parser to `parse` and `print` methods on our operation,
  // to be implemented in the .cpp file. More details on these methods is shown below.
  let hasCustomAssemblyFormat = 1;
}
```
或者使用 declarative format:
```tablegen
/// Consider a stripped definition of `toy.print` here.
def PrintOp : Toy_Op<"print"> {
  let arguments = (ins F64Tensor:$input);

  // In the following format we have two directives, `attr-dict` and `type`.
  // These correspond to the attribute dictionary and the type of a given
  // variable represectively.
  let assemblyFormat = "$input attr-dict `:` type($input)";
}
```

从上面的例子, 我们看到几个 Operation 相关的重要元素 `Op`, `OpTrait`, `ins/outs`, `TypeConstraint` 和 `AttrConstraint`.
这些具体的约束, 可参考文档 `Operation Definition Specification (ODS)`.


#### Filtered Iterator
对于 Block, 可以用 `getOps<OpTy>` 过滤子 Operations, 类似的 Region 也可以过滤子 Block
```cpp
  auto varOps = entryBlock.getOps<spirv::GlobalVariableOp>();
  for (spirv::GlobalVariableOp gvOp : varOps) {
     // process each GlobalVariable Operation in the block.
     ...
  }
```

#### Walk
只访问 linalgOp
```cpp
  getFunction().walk([](LinalgOp linalgOp) {
    // process LinalgOp `linalgOp`.
  });
```

### Value

类似 `llvm::Value`, Value 的子类是 SSA IR 中的基本操作数, 例如 `mlir::BlockArgument` (Block 的参数), `mlir::OpResult` (Operation 的结果, 在 LLVM 中, Instruction 本身就是结果)
例如: 从 Value 获取定义他的 Operation, 这和 LLVM 有一些不同, 
```cpp
Operation * operation = value.getDefiningOp();
toy::MulOp op = value.getDefiningOp<toy::MulOp>();
// get Value from operation
Value inner_value = operation->getResult(0);
assert(inner_value == value);
```

### Type

类似 `llvm::Type`, 类型的基类, 例如子类 `TensorType` 的 `tensor<*xf64>`, `tensor<2x3xf64>`.

#### ShapedType

分 ranked 和 unranked, ranked 还分 static 和 dynamic:
- [*] if it is an unranked shape
- [?, 2] if a rank 2 tensor with one unknown dimension
- [3, 4] is a rank 2 static tensor
- [] is a scalar
- [1] is a rank 1 tensor with 1 element
- [invalid] for an invalid shape

ShapeType 的 Trait:
```cpp
class ShapedType : public ::mlir::TypeInterface<ShapedType, detail::ShapedTypeInterfaceTraits> 
```
可以由 Dialect 的 ODS 生成:
```tablegen
def ShapedTypeInterface : TypeInterface<"ShapedType"> {
  let cppNamespace = "::mlir";
  let description = [{
    ...
  }]
  ...
}
```

## Dialect

Implementation:
- 实现 Dialect -> C++ Dialect 或者 ODS 定义 (Operation, Attribute, Type, Constraint, Interface, Trait)
- Dialect 内部 Operation 转换 -> Dialect Transformation (C++/DRR实现)
- Dialect 之间 相互转换 -> DialectConversion (C++/DRR 实现)
- Pass 优化 (C++/DRR 实现)

在 Tot Tutorial 中在 Ops.td 定义了 Dialect, 使用 `mlir-tblgen` 生成头文件和 cpp 文件, 在默认情况下 `MLIRContext` 只会 Load 默认的 Builtin Dialect, 需要手动 Load

在 `examples/toy/Ch2/include/toy` 下的 CMakeLists 中, 如此生成文件
```cmake
set(LLVM_TARGET_DEFINITIONS Ops.td)
mlir_tablegen(Ops.h.inc -gen-op-decls)
mlir_tablegen(Ops.cpp.inc -gen-op-defs)
mlir_tablegen(Dialect.h.inc -gen-dialect-decls)
mlir_tablegen(Dialect.cpp.inc -gen-dialect-defs)
add_public_tablegen_target(ToyCh2OpsIncGen)
```

## Pass and Rewrite

首先需要明确 [Canonicalization](https://sunfishcode.github.io/blog/2018/10/22/Canonicalization.html) 及其意义.
mlir 官方文档也有一篇专门讲 [Operation Canonicalization](https://mlir.llvm.org/docs/Canonicalization/).

在 LLVM 中, InstCombine 和 DAGCombine 就是我们熟知的 canonicalization pass.
主要目的是为了方便后续的优化和分析可以针对一种统一的形式.

mlir 的典范化是一个 Pass, 通过递归地匹配所有 Dialect 里的典范化 pattern 做贪心的 rewrite.
需要注意的是, 这个过程是重复直到达到不动点或者次数上限.

在 Toy 中, 为了消除 transpose(transpose(x)) 这儿冗余的操作, 我们把 rewrite 嵌入 cannibalization 的 pipeline 里.

首先在 ODS 中, 可以添加如下声明, 会生成 `getCanonicalizationPatterns` 方法的声明:
```tablegen
def MyOp : ... {
  // I want to define a fully general set of patterns for this op.
  let hasCanonicalizer = 1;
}

def OtherOp : ... {
  // A single "matchAndRewrite" style RewritePattern implemented as a method
  // is good enough for me.
  let hasCanonicalizeMethod = 1;
}
```

在源代码中对应的实现:
```cpp
void MyOp::getCanonicalizationPatterns(RewritePatternSet &patterns,
                                       MLIRContext *context) {
  patterns.add<...>(...);
}

// For Toy TransposeOp
// Register our patterns for rewrite by the Canonicalization framework.
void TransposeOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<SimplifyRedundantTranspose>(context);
}

LogicalResult OtherOp::canonicalize(OtherOp op, PatternRewriter &rewriter) {
  // patterns and rewrites go here.
  return failure();
}

// For Toy TransposeOp
/// This method is attempting to match a pattern and rewrite it. The rewriter
/// argument is the orchestrator of the sequence of rewrites. It is expected
/// to interact with it to perform any changes to the IR from here.
mlir::LogicalResult
matchAndRewrite(TransposeOp op,
                mlir::PatternRewriter &rewriter) const override {
  // Look through the input of the current transpose.
  mlir::Value transposeInput = op.getOperand();
  TransposeOp transposeInputOp = transposeInput.getDefiningOp<TransposeOp>();

  // Input defined by another transpose? If not, no match.
  if (!transposeInputOp)
    return failure();

  // Otherwise, we have a redundant transpose. Use the rewriter.
  rewriter.replaceOp(op, {transposeInputOp.getOperand()});
  return success();
}
```

同样的, 使用 tablegen 的声明式 pattern match and rewrite 系统 (DRR).
先插入提一句 tablegen 的 dag 类型格式, 见 [Tablegen Programmar Reference](https://llvm.org/docs/TableGen/ProgRef.html).
dag node 由一个 operator 以及零个或多个 argument 组成

( operator argument1, argument2, … ) 


| Format	| Meaning | 
| ---     | ---     |
| value	| argument value |
| value:name	| argument value and associated name | 
| name	| argument name with unset (uninitialized) value |

value 是任意的 TableGen Value.

在 DDR 中, 在 `mlir/IR/PatternBase.td` 中定义了 DDR 的 pattenr 格式:
```tablegen
class Pattern<
    dag sourcePattern, list<dag> resultPatterns,
    list<dag> additionalConstraints = [],
    dag benefitsAdded = (addBenefit 0)>;
```
以及 Toy 的样例
```tablegen
def FoldConstantReshapeOptPattern : Pat<
  (ReshapeOp:$res (ConstantOp $arg)),
  (ConstantOp (ReshapeConstant $arg, $res))>;
```
