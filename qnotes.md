
## Compiler

- Cold/Hot Splitting (Ruijie Fang, GSoC'20, LLVM-CGO workshop 21)
- Classical Loop Nest Transformation Framework on MLIR (Vinay M, Ranjith Kumar H, Siddharth Tiwary and Prashantha Nr, LLVM-CGO workshop 21)
- LTO and Data Layout Optimisations in MLIR (Prashantha Nr and Ranjith Kumar, LLVM-CGO workshop 21)
- Performance Tuning: Future Compiler Improvements (Denis Bakhvalov, LLVM-CGO workshop 21)
- Latest Advancements in Automatic Vectorization Research (Stefanos Baziotis, LLVM-CGO workshop 21)
- Tail Call Optimization in Interpreters (no JIT): https://blog.reverberate.org/2021/04/21/musttail-efficient-interpreters.html
- LLVM GlobalISel
  - [2016 LLVM Developers’ Meeting: A. Bougacha & Q. Colombet & T. Northover “Global Instr.."](https://www.youtube.com/watch?v=6tfb344A7w8&t=11s)
  - [2017 LLVM Developers’ Meeting: “GlobalISel: Past, Present, and Future ”](https://www.youtube.com/watch?v=McByO0QgqCY&t=845s)
  - [2017 LLVM Developers’ Meeting: J. Bogner & A. Nandakumar & D. Sanders “Tutorial: GlobalISel ”](https://www.youtube.com/watch?v=6tfb344A7w8&t=11s)
  - [2019 LLVM Developers’ Meeting: V. Keles & D. Sanders “Generating Optimized Code with GlobalISel”](https://www.youtube.com/watch?v=8427bl_7k1g)
  - Discourse [GlobalISel design update and goals](https://discourse.llvm.org/t/globalisel-design-update-and-goals/49276)
- Dominator Tree
  - [2017 LLVM Developers’ Meeting: J. Kuderski “Dominator Trees and incremental updates that ... ”](https://www.youtube.com/watch?v=bNV18Wy-J0U) and LLVM [slides](https://llvm.org/devmtg/2017-10/slides/Kuderski-Dominator_Trees.pdf)
  - [编译器中的图论算法](https://zhuanlan.zhihu.com/p/365912693)
  - Blog https://blog.csdn.net/dashuniuniu/article/details/103462147
  - [A Simple, Fast Dominance Algorithm](https://www.cs.rice.edu/~keith/EMBED/dom.pdf) by Keith D. Cooper, Timothy J. Harvey, and Ken Kennedy
  - [An Experimental Study of Dynamic Dominators](https://arxiv.org/pdf/1604.02711.pdf) by Loukas Georgiadis, Giuseppe F. Italiano, Luigi Laura, Federico Santaroni. And corresponding practices [Finding Dominators in Practice](https://renatowerneck.files.wordpress.com/2016/06/gwtta04-dominators.pdf)
- Pointer Analysis
  - [Alias Analysis in LLVM](https://www.cse.psu.edu/~gxt29/papers/ShengHsiuLin_thesis.pdf) by Sheng-Hsiu Lin, a brief introduction to classification of pointer analysis and implementation in LLVM
  - Points-to Analysis using BDDs by Marc Berndl, et al. PLDI 03
  - The Ant and the Grasshopper: Fast and Accurate Pointer Analysis for Millions of Lines of Code by Hardekopf and Lin, PLDI 07, lazy detecting cycles
  - Flow-sensitive pointer analysis for millions of lines of code by Hardekopf B, Lin C. CGO 11, partial SSA form
  - CFLA & TBAA
  - [Memory SSA- A Unified Approach for Sparsely Representing Memory Operations](https://www.airs.com/dnovillo/Papers/mem-ssa.pdf) by Novillo D , Canada R H
  - LLVM example memory SSA code, find the nearest load/store to the same memory given a store inst.
  ```cpp
  llvm::MemorySSA *MSSA = &getAnalysis<MemorySSAWrapperPass>(F).getMSSA();
  llvm::StoreInst *SI = ...;
  MemoryAccess *storeMA = MSSA->getMemoryAccess(SI);
  const llvm::Instruction *immediate_access_inst = nullptr;
  for (const llvm::Insturction *I = getNextNonDebugInstruction(SI); I; I = getNextNonDebugInstruction(I)) {
    MemoryAccess *ma = MSSA->getMemoryAccess(I);
    if (ma) {
      MemoryUseOrDef *ud = dyn_cast<MemoryUseOrDef>(ma);
      if (ud && ud->getDefiningAccess() == storeMA) {
        immediate_access_inst = ud->getMemoryInst();
        break;
      }
    }
  }
  ```

## Formal Language

- Interchange Lemma for Context-Free Language
- DFA Minimization
- Omega-Automaton
- Tree Automaton
- Brzozowski derivative
- Unique Game Conjucture
- NC Complexity
- Rational Set
- E-graph
- Conway Semiring
- Curry–Howard correspondence
- Chomsky–Schützenberger representation theorem
- Stone Duality

## Algebra

- Metacategory 
- Slice Category
- Monoidal Category, (Free) Monoid
- Center (category theory)
- Functor, Natual Transformation and Abelianization
- 2-morphism
- Homotopy Type Theory, HoTT Book, EPIT Spring School on Homotopy Type Theory

## Other

- Condensed Mathematics
- Reverse Mathematics (Kohlenberg;2005)
- Proximal Policy Optimization, https://openai.com/research/openai-baselines-ppo
- Lambada Calculus, category view: https://arxiv.org/abs/1202.4678
- Complexity of Reachability Problems in Neural Networks: https://arxiv.org/abs/2306.05818 (p.s. can generator/fuzzer enumerate in a countable domain $D$? )
- Three Candidate Plurality is Stablest for Correlations at most 1/11, KKMO conjecture: https://arxiv.org/abs/2306.03312 (correctness not sure)
- Computational Intractability: A Guide to Algorithmic Lower Bounds by Demanine: https://hardness.mit.edu/ and http://erikdemaine.org/
- CCC conference: https://computationalcomplexity.org/
- Daniela Petrisan / Compositionality of Effects in Semantics and Automata Theory: https://www.youtube.com/watch?v=uAfoCc1XXCA
- Takako Nemoto: On a decomposition of WKL!!: https://www.youtube.com/watch?v=sIbERCBMrro
- A mathematical theory of true randomness (Part 2): https://www.youtube.com/watch?v=SQkv4v3qqPU
- Internal Set Theory
- Reverse Mathematics, IST axiom, WKL, WWKL, textbooks by Dzhaferov: https://link.springer.com/book/10.1007/978-3-031-11367-3
- Quantum system is hard to learn: https://arxiv.org/abs/2306.04731
- Quantum system verification： https://arxiv.org/abs/2306.04843
- AdS/CFT duality: https://journals.aps.org/prd/abstract/10.1103/PhysRevD.97.086015
- ECCC: https://eccc.weizmann.ac.il/
- Model Checking Quantum System: https://academic.oup.com/nsr/article/6/1/28/5128515
- LEO: A Programming Language for Formally Verified, Zero-Knowledge Applications: https://eprint.iacr.org/2021/651.pdf
