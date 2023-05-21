## POPL'09 Semi-Sparse Flow-Sensitive Pointer Analysis

### Contribution

The major background prerequisites are covered by SSA book.


### Semi-Sparse Analysis

Data-Flow Graph (DFG) for partial SSA form:
- For top-level varibale, SSA form encodes use-def chains and data flow path
- For address-taken variables, SEG form (i.e. CFG of defines and uses) encodes data flow path

|  Type |  Example |  Def-Use Info | Remark
| --- | --- | --- | --- |
| ALLOC |  x = ALLOCi |  DEFtop |  | 
| COPY  | x = y z | DEFtop, USEtop | | 
| LOAD  | x = \*y  | DEFtop, USEtop, USEadr | | 
| STORE  | \*x = y  | USEtop, DEFadr, USEadr | USEadr because weak updates require the updated variable’s previous points-to set. |
| CALL  | x = foo(y) |  DEFtop, USEtop, DEFadr, USEadr | DEFadr because they can modify address-taken variables via the callee function |
| RET  | return x  | USEtop, USEadr | USEadr because they need to pass the address-taken pointer information to/from the callee function.  |


### Unification Optimization

TODO
