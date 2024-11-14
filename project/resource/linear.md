

## Materials

- Linear Logic
  + Linear Types Can Change the World! by Phillip Wadler, 1990

- Unqiueness Type
  + Uniqueness Typing Simplified (IFL '08), Clean language，Clean 的[文档](https://clean.cs.ru.nl/download/html_report/CleanRep.2.2_1.htm)

- Quantitative Type
  + Idris 2: Quantitative Type Theory in Practice (ECOOP '21), Idris 的 [multiplicitis](https://idris2.readthedocs.io/en/latest/tutorial/multiplicities.html#multiplicities) 文档
  + Resource-Guided Program Synthesis (PLDI '19) & A unifying type-theory for higher-order (amortized)
cost analysis (POPL '21) 采用类似的方法来处理 closure capture 问题，对于 m 重的 $m \cdot (A \stackrel{p / q}{\rightarrow} B)$，我们需要 $m \cdot \Gamma$ 的势能环境

但是，三者似乎都要求“同一个函数写 unique 和 shared 两种版本”？

- Graded Type
  + A graded dependent type system with a usage-aware semantics (POPL '21)

- Borrow
  + Borrow 本身并不依赖于 ownership 机制（譬如 Rust），在 FBIP/FIP 中，只作为单纯的性能改进提示（减少 Rc 操作）
  + Borrow 推断：Counting immutable beans: reference counting optimized for purely functional programming (IFL '19)，不过被指出，该 borrow inference 结果指导的 reuse 不是 space-safe 的
  + Borrow 似乎能借助 effect/contexutual type 描述，Lionel 的一个 [talk](https://hkuplg.github.io/2024/11/05/lionel/)
  + Borrow 似乎对改进 AARA 对 closure/high-order 的处理有所帮助，譬如这段来自 koka 的[样例](https://github.com/koka-lang/koka/blob/dev/samples/learn/fip.kk)，有没有可能推断出类似 $\text{append} :: \& List(int^1) \rightarrow \& List(int^0) \rightarrow List(int^0)$

  ```koka
  // Unfortunately, we cannot quite check a recursive polymorphic fip version of `insert` yet
  // since we cannot (yet) express second-rank borrow information where the compare
  // function does not only need to be borrowed, but borrow its arguments as well
  // (e.g. we need `^?cmp : (^a,^a) -> order`).
  //
  // The `insert-poly` actually _does_ execute in-place at runtime like a `fip(1)`
  // function, we just can't check it statically (at this time).
  fun insert-poly( t : tree<a>, k : a, ^?cmp : (a,a) -> order ) : tree<a>
  match t
      Bin(l,x,r) -> if (x==k) then Bin(l,k,r)
                  elif (x < k) then Bin(l,x,insert-poly(r,k))
                  else Bin(insert-poly(l,k),x,r)
      Tip        -> Bin(Tip,k,Tip)

  // We can get around the previous restriction by using the `:order2<a>`
  // (value) type which contains not only the `:order`, but also the elements
  // themselves in ascending order.
  // Instead of a bare comparison, we use an `order2` function that returns
  // such an ordered pair -- this way we can keep using the elements `k` and `x` in
  // a linear way while being polymorphic over the elements.
  // A subtle point here is that `order2` is passed borrowed and as such can itself
  // allocate/deallocate; the `fip` annotation only checks if a function is intrinsically
  // fully in-place -- modulo the borrowed function parameters. (Consider the `map` function for example).
  fip(1) fun insert-order( t : tree<a>, k : a, ^?order2 : (a,a) -> order2<a> ) : tree<a>
  match t
      Bin(l,x,r) -> match order2(x,k)
                      Eq2(kx)     -> Bin(l,kx,r)
                      Lt2(kx,kk)  -> Bin(l,kx,insert-order(r,kk))
                      Gt2(kk,kx)  -> Bin(insert-order(l,kk),kx,r)
      Tip        -> Bin(Tip,k,Tip)
    ```


## Miscellaneous

- FIP 虽然很有用，但是使用的是 reuse credit，而不是可加性的 space credit，这主要是出于 reuse 要求 cell 的大小是匹配的，而堆空间一般是离散的；这一结果导致 reuse analysis 是基于语法的，在 mergesort 等例子上需要一些技巧（辅助的 ADT）进行改写，balanced tree 需要 zipper ("one-hole context") 等。如果采用 compact memory representantion (例如 LoCal)，是否可能推广 reuse 的使用范围
