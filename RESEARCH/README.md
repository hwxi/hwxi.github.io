# Research

I enjoy programming.

My primary research interest lies in programming language design and
implementation. In particular, I have long been applying type-based
approaches to static debugging, that is, detecting potential
programming errors before run-time. Recently, I have also been
studying template-based programing (TBP), which advocates using
function templates in place of functions to achieve a great deal more
degree of code sharing.

The initial idea of the Applied Type System framework (for studying
type theory immediately applicable to practical programming) came to
me around the end of 2002.  I have since been continually working on
the design and implmentation of [ATS](http://www.ats-lang.org), a
programming language of a functional core that is equipped with
support for advanced types such as linear types and dependent types.
As of now, I am actively working on
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

## Programming with Theorem-Proving

ATS advocates a programming paradigm in which programs and proofs can
be constructed in a syntactically intertwined manner. This paradigm is
often referred to as programming with theorem-proving (PwTP), and it
plays a central indispensible role in the development of ATS. Let us
now see a simple and concrete example that clearly illustrates PwTP as
is supported in ATS.

A function `fib` can be specified as follows for computing Fibonacci numbers:

* fib(0)   = 0
* fib(1)   = 1
* fib(n+2) = fib(n) + fib(n+1) for n >= 0

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
time-complexity. An implementation of fib in C is given as follows
that is of linear time-complexity:

## Types for Verification and Productivity
