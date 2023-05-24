## PLDI'15 Provably Correct Peephole Optimizations with Alive

### Preface

Alive2 has become an very important tool for LLVM middle-end developers who work on IR & Optimization.
Compared to expensive and large scale analysis and transforms passes, many other passes heavily based on _peephole style_ optimization, e.g. InstCombine, InstSimplify.
When we submit patch to the community, an Alive2 proof is also required for the soundness.
So we came back to Alive in PLDI'15 and see its developing pathways.

The main problem is LLVM IR’s semantics have no formal spec and described in English, (LLVM IR at that time has introduced _undef_ and _poison_).
This ambiguous of IR leads to implementation inconsistency in Clang/LLVM.

Alive and later similar works (transform validation in LLVM) has similar pattern:
- Determine the research scope
- Formalize the IR semantics
- Use oracle to verify the functionality

### Contribution

This paper presents Alive for proving transform is correct or giving counterexample, i.e. try to find a case where the given transform fails.
Alive mainly focus on InstCombine pass, i.e. peephole optimzation and expression level.

### Framwork

### Formalization

Alive define IR semantics based on LLVM IR reference and handle undefined behavior with _undef_ and _poison_.
In this paper, Alive presents arithematic and memory UB.

#### Constraint Solving
verifying transform $A \Rightarrow B$ is equivalent (soundness criteria):
1. A and B are purely arithmetically equivalent.
2. Considering undefine behavior.
    - When A is well-defined, B should be well-defined.
    - When A is undefined, B should also preserve this.

Give Z3 the negation and check satisfiability and produce counterexmaple. 

#### Some Other Details

- Memory instruction constraint encoding, SMT array theory.
- Memory operation defineness propagation?
- Other memory models?

  > Hence, we explore encoding memory operations without arrays. This is usually accomplished with Ackermann’s expansion

### Remark

- Focus on _integer_ operations, but we should admit it's the major op in InstCombine.
- Does not support vector type and CFG, today's Alive2 is still useful in SECV and Vectorization.
- Predicat in example can be express more elegant in Alive2 with llvm intrinsics
