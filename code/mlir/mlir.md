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