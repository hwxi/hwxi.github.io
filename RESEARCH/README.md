# Research

Programming really means programming productively.

Let us first remember the following quote of HARLAN MILLS
on productivity:

*Mankind, under the grace of God, hungers for spiritual peace,
esthetic achievements, family security, justice, and liberty,
none directly satisfied by industrial productivity. But productivity
allows the sharing of the plentiful rather than fighting over scarcity;
it provides time for spiritual, esthetic, and family matters. It allows
society to delegate special skills to institutions of religion, justice,
and the preservation of liberty.*

I am passionate about programming.
My primary research interest lies in programming language design and
implementation. In particular, I have long been studying type-based
approaches and applying them to static debugging, that is, detecting
potential programming errors before run-time. Recently, I have also
been studying template-based programing (TBP), which advocates using
function templates in place of functions to achieve a great deal more
degree of code sharing.

The initial idea of the Applied Type System framework (for formulating
type theory on practical programming) came to me around the end of
year 2002. I have since been continually working on the design and
implmentation of [ATS](http://www.ats-lang.org), a programming
language of a functional core that is equipped with support for
advanced types such as linear types and dependent types.  As of now, I
am actively working on
[ATS/Xanadu](https://github.com/githwxi/ATS-Xanadu), which is an
implementation for ATS3, the third edition of ATS. My expectation is
for ATS to turn from a programming language for research into one that
is of production strength.

## Being Precise

How many times have you heard someone saying "I know the answer but I
can't explain it"? A lot, I bet. I can easily tell the image of a cat
from that of a dog, but I cannot readily explain how I do it. In
programming, it is often easier for someone to write code than to
explain what the code actually does. This is very worrisome,
especially, if we want to build high-quality software systems.

In order to be precise, we need to be able to state clearly what we want.
Let me use the following elementary problem as an example.

Alice and Bob ate 10 apples together. Alice ate 3 apples. What is the
number of apples eaten by Bob?

My daughter as a first grader had no difficulty telling me immediately that
the answer is 7. I, however, wanted her to give me a more formal
specification of the problem. We essentially came up with the following one
at the end:

Let A and B stand for the numbers of apples eaten by Alice and Bob,
respectively. It is given that A+B=10 and A=3. What is the value of B?

With this specification, we could readily and formally derive that (A+B)-A
= 10-3.  Addition being both commutative and associative, we derived
(A+B)-A = B+(A-A) = B+0 = B. Therefore, B = 7. So we not only found that 7 is an
answer but also proved that 7 is the unique answer.

While I might have been a bit overly pedantic about the above
elementary problem, I do hope that you can see my point of being
precise here. I strive to be precise in programming and have a passion
for *effectively* enforcing precision in realistic software
development.
  
## Being Precise Is Not Enough

Programming languages are tools and programming language design and
implementation should focus on increasing a programmer's
productivity. Ideally, a programming language should be simple and
general, and it should permit extensive error checking, facilitate
proofs of program properties and possess a correct and efficient
implementation.  Invariably, there will be conflicts among these
goals, which must be resolved with trade-offs being carefully made. In
order to make significant progress, I firmly believe the necessity to
adopt approaches that can scale well.

## Types for Productivity

Traditionally, types are touted for helping debug programs.  As
debugging is an essential part of programming, such help from types
can undoubtedly result in increased programming productivity. This
kind of indirect support of types for increased productivity is
already well-known. What is much less well-known is that types can
also directly result in increased programming productivity by
facilitating code reuse at compile-time. For instance, the feature of
type classes in Haskell makes essential use of types in choosing type
class instances. Moreover, types play an even more conspicuous role in
template-based programming, a defining feature of ATS that is partly
inspired by type classes.

## Template-Based Programming

The notion of *late-binding* is prevalent in programming language
design and implementation. For instance, linking the name of a
function to the actual code implementing the function can be seen as a
form of late-binding (at link-time). More conspicuously, method
dispatching in object-oriented programming (OOP) is also a well-known
form of late-binding (at run-time). As a form of late-binding at
compile-time, one can think of overloading supported by type classes
in Haskell.

Template-Based Programming (TBP) advocates the use of (function)
templates in place of functions, which can be regarded as a form of
late-binding at compile-time. Intuitively, one may think of templates
as functions containing placeholders inside their bodies that can be
replaced with code obtained *contextually* at compile-time. It is
important to emphasize that the mentioned code replacement is
inherently of a recursive nature as code obtained contextually may
itself contain templates.

There are plenty of motivating examples for templates. For the moment,
let us take a look at equality types in Standard ML (SML), which can
be said to have directly motivated the addition of type classes into
Haskell.

Various aspects of templates have already appeared in languages such
as LISP (macros), Haskell (type classes), Scala (implicits), etc. One
may also see great similarity between resolving a template call (in
TBP) at compile-time and dispatching a method call (in OOP) at
run-time.

Basically, an equality type in SML is one that supports polymorphic
equality (=), which is given the type `''a * ''a -> bool` (instead of
`'a * 'a -> bool`). In other words, a type variable `''a` can only be
instantiated with an equality type (where a special function is
designated for testing equality on elements of this type).
  
There is *no* polymorphic equality in ATS3. In order to test whether two
given lists (of the same generic type) are equal, one could try to
implement a function template of the following interface:

```ats
fun
<a:type>
list_fequal
( xs: list(a)
, ys: list(a)
, eq: (a, a) -> bool): bool
```

This is a workable solution based on the notion of higher-order
function but it suffers from the requirement that each caller of
`list_fequal` must pass to it *explicitly* a function argument (for
testing equality on elements in the two given lists).

Let us see a template-based solution to implementing equality test on
lists. In ATS3, the name `g_equal` refers to a template of the
following interface:

```ats
fun
<a:type>
g_equal(a, a): bool
```

A function `list_equal` can be defined as follows to test whether two
given lists are equal by calling `g_equal` to test equality on elements:

```ats

#extern
fun
<a:type>
list_equal
( xs: list(a)
, ys: list(a)): bool

(* ****** ****** *)

impltmp
<a>(*tmp*)
list_equal
  (xs, ys) =
( loop(xs, ys) ) where
{
//
fun
loop
( xs: list(a)
, ys: list(a) ): bool =
(
case+ xs of
| list_nil() =>
  (
  case+ ys of
  | list_nil() => true
  | list_cons _ => false
  )
| list_cons(x0, xs) =>
  (
  case+ ys of
  | list_nil() => true
  | list_cons(y0, ys) =>
    let
      val ans =
      g_equal<a>(x0, y0)
    in
      if ans then loop(xs, ys) else false
    end
  )
) (* end of [loop] *)
//
} (* end of [list_equal] *)

```

Note that `g_equal` is already given a standard implementation on
basic types like int, bool, char, string, etc. If `list_equal` is
called on two lists of the type `list(int)`, the compiler can
automatically find based on contextual information an implementation
of `g_equal<int>` needed for compiling `list_equal<int>`. Moreover,
one can direct the compiler to locate another implementation of
`g_equal<int>` if needed. It is in this sense of directing the
compiler to establish a link between a call to `g_equal<int>`
and an implementation of `g_equal<int>` that TBP can be regarded
as a form of late-binding at compile-time.

Templates in ATS3 are embeddable for they can be implemented in the
body of other templates. As an example, the previous higher-order
template `list_fequal` can be given an implementation based on
`list_equal` as follows:


```ats
impltmp
<a>(*tmp*)
list_fequal
  (xs, ys, eq) =
(
  list_equal<a>(xs, ys)
) where
{
  impltmp g_equal<a>(x, y) = eq(x, y)
}
```

where an implementation of `g_equal<a>` is given in the body of the
implementation of `list_fequal<a>`.  It will become clear soon that
the embeddability of templates is a feature that can be made of use
to greatly simplify the way in which a program is structured.

With the above simple example, I have demonstrated a bit of TBP where
templates are employed to replace higher-order functions.  While this
is a typical entry point for TBP, there are many more aspects of TBP
that are yet to be explored.

## Programming with Theorem-Proving

ATS advocates a programming paradigm in which programs and proofs can
be constructed in a syntactically intertwined manner. This paradigm is
often referred to as programming with theorem-proving (PwTP), and it
plays a central indispensible role in the development of ATS. Let us
now see a simple and concrete example that clearly illustrates PwTP as
is supported in ATS.

A function `fib` can be specified as follows for computing Fibonacci numbers:

* `fib(0) = 0`
* `fib(1) = 1`
* `fib(n+2) = fib(n) + fib(n+1) for n >= 0`

Following is a direct implementation of this specified function in ATS:

```ats
fun
fib (
  n: int
) : int =
  if n >= 2 then fib(n-2) + fib(n-1) else n
// end of [fib]
```

Clearly, this is a terribly inefficient implementation of exponential
time-complexity. An implementation of `fib` in C is given as follows
that is of linear time-complexity:

```c
int
fibc(int n)
{
  int tmp, f0 = 0, f1 = 1 ;
  while (n-- > 0) { tmp = f1 ; f1 = f0 + f1 ; f0 = tmp ; } ; return f0 ;
} // end of [fibc]
```

If translated into ATS, the function `fibc` can essentially be implemented as follows:

```ats
fun
fibc (
  n: int
) : int = let
//
  fun
  loop(n: int, f0: int, f1: int): int =
    if n > 0 then loop(n-1, f1, f0+f1) else f0
  // end of [loop]
//
in
  loop(n, 0, 1)
end // end of [fibc]
```

There is obviously a logic gap between the defintion of `fib` and its
implementation as is embodied in `fibc`. In ATS, an implementation of
`fib` can be given that completely bridges this gap. First, the
specification of `fib` needs to be encoded into ATS, which is fulfilled
by the declaration of the following dataprop:

```ats
dataprop
FIB(int, int) =
| FIB0(0, 0) of ()
| FIB1(1, 1) of ()
| {n:nat}{r0,r1:int}
  FIB2( n+2, r0+r1 ) of (FIB(n, r0), FIB(n+1, r1))
```

This declaration introduces a type `FIB` for proofs, and such a type is
referred to as a prop in ATS. Intuitively, if a proof can be assgined
the type `FIB(n,r)` for some integers `n` and `r`, then `fib(n)` equals `r`.
In other words, `FIB(n,r)` encodes the relation `fib(n)=r`. There are three
constructors `FIB0`, `FIB1` and `FIB2` associated with `FIB`, which are given
the following types corresponding to the three equations in the definition of
`fib`:

* `FIB0 : () -> FIB(0, 0)`
* `FIB1 : () -> FIB(1, 1)`
* `FIB2 : {n:nat}{r0,r1:int} (FIB(n, r0), FIB(n+1, r1)) -> FIB(n+2, r0+r1)`

Note that {...} is the concrete syntax in ATS for universal
quantification. For instance, `FIB2(FIB0(), FIB1())` is a term of the
type `FIB(2,1)`, attesting to `fib(2)=1`.

A fully verified implementaion of the `fib` function in ATS can now be
given as follows:

```ats
fun
fibats
{n:nat}
( n
: int(n))
: [r:int] (FIB (n, r) | int r) = let
  //
  fun
  loop
  {i:nat|i <= n}{r0,r1:int}
  ( pf0: FIB(i, r0)
  , pf1: FIB(i+1, r1)
  | n_i: int(n-i)
  , r0: int r0, r1: int r1
  ) : [r:int] (FIB(n, r) | int(r)) =
  (
    if
    (n_i > 0)
    then
    loop{i+1}
    (
      pf1, FIB2(pf0, pf1) | n_i-1, r1, r0+r1
    ) (* then *)
    else (pf0 | r0)
  ) (* end of [loop] *)
in
  loop{0}(FIB0(*void*), FIB1(*void*) | n, 0, 1)
end // end of [fibats]
```

Note that `fibats` is given the following declaration:

```
fun fibats : {n:nat} int(n) -> [r:int] (FIB(n,r) | int(r))
```

where the concrete syntax [...] is for existential quantification and
the bar symbol (|) is just a separator (like a comma) for separating
proofs from values. For each integer `I`, `int(I)` is a singleton type
for the only integer whose value equals `I`. When `fibats` is applied to
an integer of value `n`, it returns a pair consisting of a proof and an
integer value `r` such that the proof, which is of the type `FIB(n,r)`,
asserts `fib(n)=r`. Therefore, `fibats` is a verified implementation of
`fib` as is encoded by `FIB`. Note that the inner function `loop` directly
corresponds to the while-loop in the body of the function `fibc`
(written in C).

Lastly, it should be emphasized that proofs are completely erased
after they pass typechecking. In particular, there is no proof
construction at run-time.

## Building a Code-Sharing Ecosystem

## Multirole Logic and Multiparty Channels
