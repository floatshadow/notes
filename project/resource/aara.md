
## Examples


resource parametric (polymorphism) that leads to exponential explosion of linear constraint sets.

```ocaml tilte="dup.raml"
let rec dup l =
  match l with
    | [] -> 
      let _ = Raml.tick(1.) in
      []
    | hd::tl ->
      let _ = Raml.tick(2.) in
      hd::hd::(dup tl)

let dup2 l = dup (dup l)
let dup3 l = dup2 (dup2 l)
let dup4 l = dup3 (dup3 l)
let dup5 l = dup4 (dup4 l)
let dup6 l = dup5 (dup5 l)
let dup7 l = dup6 (dup6 l)
let dup8 l = dup7 (dup7 l)
let dup9 l = dup8 (dup8 l)
;;

(*
 * Analyzing function dup ...
 * 
 *   Trying degree: 2
 * 
 * == dup :
 * 
 *   'a list -> 'a list
 * 
 *   Non-zero annotations of the argument:
 *          2  <--  [::(*)]
 *          1  <--  []
 * 
 *   Non-zero annotations of result:
 * 
 *   Simplified bound:
 *      1 + 2*M
 *    where
 *      M is the number of ::-nodes of the argument
 * 
 * --
 *   Mode:          upper
 *   Metric:        ticks
 *   Degree:        2
 *   Run time:      0.05 seconds
 *   #Constraints:  52
 * 
 * ====
 * 
 * Analyzing function dup2 ...
 * 
 *   Trying degree: 2
 * 
 * == dup2 :
 * 
 *   'a list -> 'a list
 * 
 *   Non-zero annotations of the argument:
 *          6  <--  [::(*)]
 *          2  <--  []
 * 
 *   Non-zero annotations of result:
 * 
 *   Simplified bound:
 *      2 + 6*M
 *    where
 *      M is the number of ::-nodes of the argument
 *)
--

```

closure capture breaks linearity

```ocaml title="closure_capture.raml"
let rec append l1 l2 =
  match l1 with
  | [] -> l2
  | x :: xs -> 
  	let _ = Raml.tick(1.) in
    x :: (append xs l2)



let cap = [1;2;3;4;5;6;7;8;9;10]

let _ =
  let cap_app = append cap in
  let cap_app_map = map cap_app in
  let x = [[1;2;3;1;3;3];[1;2;3;1;3;3];[1;2;3;1;3;3];[1;2;3;1;3;3]] in
  cap_app_map x

(*
 * Resource Aware ML, Version 1.5.0, June 2020
 *
 * Typechecking expression ...
 *  Typecheck successful.
 *  Stack-based typecheck successful.
 *
 * Analyzing expression ...
 *
 *   Trying degree: 2
 *
 *   No bound could be derived. The linear program is infeasible.
 *
 *   Mode:          upper
 *   Metric:        ticks
 *   Degree:        2
 *   Run time:      0.06 seconds
 *   #Constraints:  1080
 *)
```
