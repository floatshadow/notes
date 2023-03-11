
## SAT
### Algorithm

Brute force, DPLL, CDCL, DPLL(T)

### Optimizations

- 冲突检测：不等全部赋值完，发现有冲突的就 UNSAT
- 赋值推导：根据 CNF 形式和已有的赋值推导字句里其他变量的值。
标准推导方法有：
    - Unit Propagation
    - Unate Propagation
    - Pure literal elimination
高级推导方法：
    - Probing：如果 x=0 或者 x=1 都能推导出 y=0，则推导出 y=0
    - Equivalence classes：消除等价的子句
- 变量选择（Branching Heuristics），例如基于子句测，优先选择最短子句变量、最常出现变量，基于历史的，优先选择之前导致冲突的变量。
例如 VSIDS，首先按变量出现次数给所有变量打分，添加新子句的时候给子句中的变量加分，每隔一段时间把所有变量的分数除以一个常量。
- 冲突导向字句学习（CDCL） 基本思想：在搜索过程中学习问题的性质，加入约束集合。
也即 Lazy Style。

## SMT

Satisfiability Modulo Theories，例如 Equality with Uninterpreted Functions（EUF） 
