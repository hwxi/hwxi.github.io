# Research

I enjoy programming.

Programming means programming productively.
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
class instances.

## Template-Based Programming

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
