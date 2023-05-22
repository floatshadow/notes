## PLDI'23 Don’t Look UB: Exposing Sanitizer-Eliding Compiler Optimizations

### Contribution

The author present the _sanitizer-eliding optimization_. It means valid compiler optimization may optimize away UB which sanitizers want to expose. 
This paper proposes __LookUB__ to find such kind of optimizations. 

### Useful Background

Sanitizers:
- ASan: buffer overflows, use-after-free, use-after-scope, memory leak, invalid argument to libc
- UBSan: generic undefined behavior, e.g. integer overflow (LLVM has a work-around in ConstraintElimination pass to make sanitizer happy), dereferencing null pointer
- MSan: uninitialized memory uses

Sanitizer pipeline in Clang:
```plaintext
                                                                                 Asan enabled
                                                                            --------> Asan Pass ----------->
                                                                            |        no sanitizer enabled   |
--> Clang frontend ------> Initial LLVM IR ---> Opt passes ----> Transformed LLVM IR --------------------> Backend
    |                          |                                            |                               |
    --> UBSan ----------------->                                            --------> MSan Pass --> Opt passes
                                                                                 MSan enabled
```

### Framework

Generally, it is a differential-testing based fuzzing pipeline.

```plaintext
  (mutated program P', fitness score)           mutated program P'               
       <-------------- Fitness function <----------------------------
       |                                                            | if invariant not violated
       |                                                            |
   Scheduler -----------> Program mutator -------------------> Test Oracle -----------------------> Test cases
              program P                    mutated program(s) P'             if invariant violated
```

#### Scheduler

Program priority worklist. 
The selection criteria is _fitness score_ (likelihood of sanitizer-eliding optimization)

#### Mutator

Randomly mutate a program $P$ into $P'$. 

#### Test Oracle

Check if mutated program $P'$ has optimized bugs that should be detected by sanitizer, i.e. _failure perservation_.
