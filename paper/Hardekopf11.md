## CGO'11 Flow Sensitve Pointer Analysis for Millions of Lines of Code

### Contribution

This paper proposes staged pointer analysis, which means we do a fast **AUX** analysis (usually flow- and context-insensitive), and then use **AUX** to accelerate expensive pointer analysis. The technique this paper constructs is somehow similar to MemorySSA (and in later SVF), i.e. build a sparse representation of memory operation. GCC and LLVM adopt partial SSA form for their IR. For top-level variables, flow information and use-def chains are encoded in SSA, but address-taken variables have to use load/store pair which encode indirect use-def chains.


### Framework

#### AUX

Fast and Conservative, flow-insensitive, context-insensitive pointer analysis, get __point-to sets__

#### Main Sparse Analysis

TODO

### Annotation

On CFG (focus on memory operations):
- store: $v_{k+1} = \chi(v_{k})$, store **may** clobber the memory location v (MemoryDef)
- load:  $\mu(v_{k})$, load **may** use the memory location v (MemoryUse)
- phi: insert Phi nodes for MemoryDefs

We see that is basically the same as today's LLVM MemorySSA. And **may** result is conservative, which can be signified. For the store instruction:
```c
*x = y;
```
We have: 
- x might not point to v in flow-sensitive results: $v_{k+1}$ is a copy of $v_{k}$, y dependent
- x might only point to v in flow-sensitive results: strong update to $v_{k}$, y dependent, $v_{k}$ in-dependent
- x might point to v and other variable in flow-sensitive results: weak update


### Def-Use Graph (DUG)

Similar to MemorySSA use-def graph. But do what the author call **Access Equivalence**. AE represents whether two address-taken variable has the same load/store pattern in **AUX** results (thus make further DUG simplification).

## Algorithm Description

TODO
