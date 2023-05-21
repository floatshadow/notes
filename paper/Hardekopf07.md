## PLDI'07 The Ant and the Grasshopper: Fast and Accurate Pointer Analysis for Millions of Lines of Code

### Contribution

This paper proposes a new optimization (Lazy Cycle Detection) to inclusion-based pointer analysis (i.e. Anderson style). 
Given that inclusion pointer analysis has a complexity of $O(n^3)$, the primary method for reducing $n$ is to looks for cycles in constraint graph (i.e. find strong connected component).
The trigger of computing scc is if the point-to set of two variable is the same.


### LCD

The core idea and algorithm of this paper. 
we check to see if two conditions are met: 
1. the points-to sets are identical
2. we haven’t triggered a search on this edge previously. (This restriction for optimization implies that Lazy Cycle Detection is incomplete — it is not guaranteed to find all cycles in the constraint graph.)

```plaintext
let G =< V,E >
R ← ∅
W ← V
while W != ∅ do
    n ← SELECT-FROM(W )
    for each v ∈ pts(n) do
        for each constraint a ⊇ ∗n do
            if v → a /∈ E then
                E ← E ∪ {v → a}
                W ← W ∪ {v}
        for each constraint ∗n ⊇ b do
            if b → v /∈ E then
                E ← E ∪ {b → v}
                W ← W ∪ {b}
    for each n → z ∈ E do
        if pts(z) = pts(n) ∧ n → z /∈ R then
              DETECT-AND-COLLAPSE-CYCLES(z)
              R ← R ∪ {n → z}
        pts(z) ← pts(z) ∪ pts(n)
        if pts(z) changed then
              W ← W ∪ {z}
```


### HCD

Combines preprocess of online (new edges) and offline (initial constraint graph).

In offine cycle detection, the author propose a special __ref__ node to deal with complex contraint type.

| Constriant Type | Program Code | Constriant |  Meaning | Offine Constraint Graph |
| --- | --- | --- | --- | --- |
| Base | $a = \\& b$ | $a \supseteq \\{b\\}$ | $loc(b) \in pts(a)$ |  ignore |
| Simple | $a = b$ | $a \supseteq b$ | $pts(a) \supseteq pts(b)$ | $b \to a$ | 
| Complexity | $a = \*b$ | $a \supseteq \*b$ | $forall v \in pts(b) : pts(a) \supseteq pts(v)$ | $\*b \to a$ | 
| Complexity | $\*a = b$ | $\*a \supseteq b$ | $forall v \in pts(a) : pts(v) \supseteq pts(b)$ | $b \to \*a$ | 

For SCC with no ref node, we can do callapse safely, with SCC with ref node a and non-ref node b, create tuple (a, b) in a list $L$.
This part will first check potential collapse opportunity and then add new edges as base inclusion-style pointer analysis.

```plaintext
let G =< V,E >
W ← V
while W != ∅ do
    n ← SELECT-FROM(W )
    if (n, a) ∈ L then
        for each v ∈ pts(n) do
            COLLAPSE(v,a)
            W ← W ∪ {a}
    for each v ∈ pts(n) do
        for each constraint a ⊇ ∗n do
            if v → a /∈ E then
                E ← E ∪ {v → a}
                W ← W ∪ {v}
        for each constraint ∗n ⊇ b do
            if b → v /∈ E then
                E ← E ∪ {b → v}
                W ← W ∪ {b}
    for each n → z ∈ E do
        pts(z) ← pts(z) ∪ pts(n)
        if pts(z) changed then
            W ← W ∪ {z}
```
