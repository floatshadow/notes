## PLDI'21 Alive2: Bounded Translation Validation for LLVM

### Preface

General intra-procedural passess, e.g. remove redundant pure function call, aggressive optimization with undefine behavior, are hard to verifiy.
Each functional change to LLVM requires regression tests (usually a function in LLVM IR as a unit), these changes may modify CFG, and not _expression level_.

### Contribution

Extend Alive to general intra-procedural optimization, formalize _observation behavior_, e.g. memory, UB.

### Framwork

#### Verify

Alive2 correctness criteria for transform $A \Rightarrow B$ (_Oberservational Semantics_):
- Return value
- State of memory
- IO behavior
- Undefine Behavior

#### Some Other Details 

Z3 Tactics in Alive2
- bit-vectors
- arrays
- lambdas
- floating-point numbers
- EUF
- quantifiers

Pipeline
```plaintext
frontend            LLVM2Alive              static unroll               Semantic Encoder & Correctness Constraint
--------->  LLVM IR ------------> Alive2 IR ---------------> Alive2 IR------->
              | tested IR transform                                           |
              |                                                               |
             LLVM IR -----------> Alive2 IR ---------------> Alive2 IR -------> SMT formula ----> Z3 (UNSAT or partial mode)

```
