# Reading List

A list of courses, projects and papers.

## Theoretical Computer Science

### Computation Complexity

- Cook-Levin Theorem [The Complexity of Theorem-Proving Procedures](https://www.inf.unibz.it/~calvanese/teaching/11-12-tc/material/cook-1971-NP-completeness-of-SAT.pdf) by Stephen A. Cook., STOC 1971
- One of Karp's 21 NP-complete problems [Reducibility Among Combinatorial Problems](https://cgi.di.uoa.gr/~sgk/teaching/grad/handouts/karp.pdf) by Richard M. Karp., 1972
- 3-Color is NP-complete [Coverings and colorings of hypergraphs](https://web.cs.elte.hu/~lovasz/scans/covercolor.pdf) by Lovász., 1973
- AKS primality test [PRIMES is in P](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/primality_journal.pdf) by Manindra Agrawal et al., Annals of Mathematics 2004
- Relation between class PP and gap function [Gap-definable counting classes](https://www.sciencedirect.com/science/article/pii/S0022000005800248) by Stephen A.Fenner et al., JCSS 1994
- Rice's theorem [Classes of recursively enumerable sets and their decision problems](https://www.ams.org/journals/tran/1953-074-02/S0002-9947-1953-0053041-6/home.html) by Rice, H. G., TAMS 1953

### Quantum Computing

- [Local Hamiltionian](http://theory.caltech.edu/~preskill/ph219/20_local_Hamiltonian_problem.pdf) by John Preskill in course [ph219](http://theory.caltech.edu/~preskill/ph219/)
- [How is the ground state of a Hamiltonian defined?](https://quantumcomputing.stackexchange.com/questions/11544/how-is-the-ground-state-of-a-hamiltonian-defined) by StackExchange

## Compiler

### LLVM

- [2019 EuroLLVM Developers’ Meeting: V. Bridgers & F. Piovezan “LLVM IR Tutorial - Phis, GEPs ...](https://www.youtube.com/watch?v=m8G_S5LwlTo&t=249s)
- mem2reg [调试LLVM如何生成SSA](https://blog.csdn.net/dashuniuniu/article/details/103389157#alloca_593)

### Analysis & Optimizations

- [Combating Control Flow Linearization](https://hal.inria.fr/hal-01649008/document) by Julian Kirsch et al. 2017
- [Branching in Data-Parallel Languages using Predication with LLVM](https://llvm.org/devmtg/2014-04/PDFs/Talks/pred_presentation.pdf) by Marcello Maggioni et al. 2014

### Instruction Selection

- LLVM DAG Selection [Generalized Instruction Selection using SSA-Graphs](https://llvm.org/pubs/2008-06-LCTES-ISelUsingSSAGraphs.pdf) by Dietmar Ebner et al., 2008

### Register Allocation

- Greedy Allocator [Greedy Register Allocation in LLVM 3.0](https://blog.llvm.org/2011/09/greedy-register-allocation-in-llvm-30.html)
- Register Hint [Optimized Interval Splitting
in a Linear Scan Register Allocator](https://www.usenix.org/legacy/events/vee05/full_papers/p132-wimmer.pdf) by Christian Wimmer et al. 2005

### Linking

- [ELF loading and dynamic linking](https://www.gabriel.urdhr.fr/2015/01/22/elf-linking/)
- [MaskRay's Blog](https://maskray.me/blog/) 
- [HiFive Blog, All Aboard](https://www.sifive.com/blog/all-aboard-part-3-linker-relaxation-in-riscv-toolchain)
### Blogs

- A compiler framework written in Rust [Cranelift Wasmtime Backend](https://cfallin.org/blog/)

### Other Practical Materials

- Microsoft [Overview of ARM32 ABI Conventions](https://docs.microsoft.com/en-us/cpp/build/overview-of-arm-abi-conventions?view=msvc-170)
- ARM-software doc [Procedure Call Standard for the Arm® Architecture](https://github.com/ARM-software/abi-aa/blob/60a8eb8c55e999d74dac5e368fc9d7e36e38dda4/aapcs32/aapcs32.rst)
- ARM-software doc [Application Binary Interface for the Arm® Architecture – The Base Standard](https://github.com/ARM-software/abi-aa/blob/60a8eb8c55e999d74dac5e368fc9d7e36e38dda4/bsabi32/bsabi32.rst)
- Floating Point Register Compiler Docs [VMOV #imm](https://developer.arm.com/documentation/ka001136/latest), [VMOV between Regs](https://developer.arm.com/documentation/dui0473/m/vfp-instructions/vmov--between-one-arm-register-and-single-precision-vfp-)
- Assembly Learning Manuals [Linux Kernel Module Cheat](https://cirosantilli.com/linux-kernel-module-cheat/)  **Warning: Political Bias included**
- Intel DPC++ Multi-Tile Card [Intel LLVM Docs](https://intel.github.io/llvm-docs/MultiTileCardWithLevelZero.html)

## Computer Graphics

### PBR

- pbr book [Physically Based Rendering:From Theory To Implementation](https://www.pbr-book.org/)

## Network Models

### Convolutional Network

- [Gradient-based learning applied to document recognition](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf) by Lecun. Y et al., PIEEE 1998
- [ImageNet Classification with Deep Convolutional Neural Networks](https://proceedings.neurips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html) by Alex Krizhevsky et al., NIPS 2012
- [Very Deep Convolutional Networks for Large-Scale Image Recognition](https://arxiv.org/abs/1409.1556) by Karen Simonyan et al., ICLR 2015
- [You Only Look Once: Unified, Real-Time Object Detection](https://arxiv.org/abs/1506.02640) by Joseph Redmon et al., arXiv 2015 
- [Deep Residual Learning for Image Recognition](https://www.computer.org/csdl/proceedings-article/cvpr/2016/8851a770/12OmNxvwoXv) by Kaiming He, et al., CVPR 2016
- [Densely Connected Convolutional Networks](https://arxiv.org/abs/1608.06993) by Gao Huang et al., CVPR 2017
- [MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications](https://arxiv.org/abs/1704.04861) by Andrew G. Howard et al., arXiv 2017
- [MobileNetV2: Inverted Residuals and Linear Bottlenecks](https://arxiv.org/abs/1801.04381) by Mark Sandler et al., arXiv 2018

### Recurrent Network

- [Neural networks and physical systems with emergent collective computational abilities](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC346238/) by John Hopfield.,  PNAS 1982
- [Turing Machines are Recurrent Neural Networks](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.49.5161) by Heikki Hyotyniemi., FCAI 1996
- [Long Short-Term Memory](https://www.researchgate.net/publication/13853244_Long_Short-term_Memory) by Sepp Hochreiter et al., 1998
- [Efficient Estimation of Word Representations in Vector Space](https://arxiv.org/abs/1301.3781) by Tomas Mikolov et al., arXiv 2013
- [Distributed Representations of Words and Phrases and their Compositionality](https://arxiv.org/abs/1310.4546) Tomas Mikolov et al., arXiv 2013
- [On the Properties of Neural Machine Translation: Encoder-Decoder Approaches](https://arxiv.org/abs/1409.1259) by Kyunghyun Cho et al., 2014

### Transformer

- [Attention Is All You Need](https://proceedings.neurips.cc/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf) by Ashish Vaswani et al., NIPS 2017
- [BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/abs/1810.04805) by Jacob Devlin et al., arXiv 2018
- [Language models are unsupervised multitask learners](https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf) by Radford A, Wu J, Child R, et al., OpenAI blog 2019
- [An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale](https://arxiv.org/abs/2010.11929) by Alexey Dosovitskiy et al., arXiv 2020

### Layers

- [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167) by Sergey loffe et al., arXiv 2015
- [Layer Normalization](https://arxiv.org/abs/1607.06450) by Jimmy Lei Ba et al., arXiv 2016

## Machine Learning System

### Parallelism

- [GPipe: Efficient Training of Giant Neural Networks using Pipeline Parallelism](https://arxiv.org/abs/1811.06965) by Yanping Huang et al., arXiv 2018
- [Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism](https://arxiv.org/abs/1909.08053) by Mohammand Shoeybi et al., arXiv 2019
- [ZeRO: Memory Optimizations Toward Training Trillion Parameter Models](https://arxiv.org/abs/1910.02054) by Samyam Rajbhandari et al., arXiv 2019
- [ZeRO-Infinity: Breaking the GPU Memory Wall for Extreme Scale Deep Learnin](https://arxiv.org/abs/2104.07857) by Samyam Rajbhandari et al., arXiv 2021

### Others

- [1-bit Adam: Communication Efficient Large-Scale Training with Adam's Convergence Speed](https://arxiv.org/abs/2102.02888) by Hanlin Tang et al., arXiv 2021
