## CGO'20 Testing Static Analyses for Precision and Soundness

### Preface

In the background part of this paper, the author mentioned LLVM value analysis.
- Known bits: a forward analysis attempting to prove that individual bits of a value are always either zero or one.
- Demanded bits: a backward analysis attempting to prove that individual bits of a value are "not demanded": their value is irrelevant.
- Integer ranges: a forwrd analysis attempting to prove that an integer-typed value lies within a sub-range.
- A collection of forward analyses determining Boolean properties of a value, such as whether it is provably non-zero or a power of two

I got to know `computeKnownBits` when I sumbitted a InstCombine patch.
And by the way, many dirty code and bug came from handling GEP instruction, as GEP has no "canonicalization".
Some RFC propose replace GEP with pure `ptr+offset` style.
