---
layout: default
title: "Comparison operators"
---

# Comparison Operators
QUBO++ supports two types of operators for creating constraints:

- **Equality operator**: $f=n$, where $f$ is an expression and $n$ is an integer.
- **Range operator**: $l\leq f\leq u$, where $f$ is an expression and 
$l$ and $u$ ($l\leq u$) are integers.

These operators return an expression that attains the minimum value of 0 if and only if the corresponding constraints are satisfied.

## Equality operator
The equality operator $f=n$ creates the following expression:
$$
(f−n)^2
$$
This expression attains the minimum value of 0 if and only if the equality $f=n$ is satisfied.

The following QUBO++ program searches for all solutions satisfying
$a+2b+3c=3$ using the Exhaustive Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = a + 2 * b + 3 * c == 3;
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  std::cout << "*f = " << *f << std::endl;

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << "a = " << a(sol) << ", b = " << b(sol) << ", c = " << c(sol)
              << ", f = " << f(sol) << ", *f = " << (*f)(sol) << std::endl;
  }
}
```
In this program, `f` internally holds two qbpp::Expr objects:
- `f`: $(a+2b+3c−3)^2$, which attains the minimum value of 0 if the equality $a+2b+3c=3$ is satisfied.
- `*f`: the left-hand side of the equality, $a+2b+3c$.

Using the Exhaustive Solver object created for `f`, all optimal solutions are stored in `sols`.
By iterating over `sols`, all solutions and the values of `f` and `*f` are printed as follows:
```
f = 9 -5*a -8*b -9*c +4*a*b +6*a*c +12*b*c
*f = a +2*b +3*c
a = 0, b = 0, c = 1, f = 0, *f = 3
a = 1, b = 1, c = 0, f = 0, *f = 3
```
These results confirm that two optimal solutions attain `f = 0` and satisfy `*f = 3`.

Note that QUBO++ supports the euqlity operator in the form of 
`expression == integer` only.
Neigher `integer == expression` nor `expression == expression` is not supported.
Instead of `expression == expression`, we can use `expression - expression == 0`.

## Notes on Supported Equality Forms
QUBO++ supports the equality operator only in the following form:
- `expression == integer`.

The following forms are not supported:
- `integer == expression`
- `expression1 == expression2`

Instead of `expression1 == expression2`, you can rewrite the constraint as:
- `expression1 - expression2 == 0`

which is fully supported.


## Range operator
The range operator in the form of $l\leq f\leq u$, ($l\leq u), which
creates an expression that attains the minimum value of 0 if and only if the constraints is satisfied.

We consider the following cases in terms of the value of $l$ and $u$:
- Case 1: $u=l$
- Case 2: $u=l+1$
- Case 3: $u=l+2$
- Case 4: $u\geq l+3$.

### Case 1: $u=l$
It is easy to see that if $l=u$, it can be implemented as the equality operator $f=l$.

### Case 2: $u=l+1$
If $u=l+1$ then the following expression is created:

$$
 (f-l)(f-u)
$$

Since no integer between $l$ and $u$ (exclusive), this expression attains a minimum value of 0
if and only if $f=l$ or $f=u$.

### Case 3: $u=l+2$.
We introduce an auxiary binary variable $a$ taking integer 0 or 1
and use the following expression:

$$
\begin{aligned}
(f-l-a)(f-l-(a+1))
\end{aligned}
$$

This expresion takes the following expressions of binary variable $a$ for $f=l$, $l+1$, and $l+2$

$$
\begin{aligned}
(f-l-a)(f-l-(a+1)) &= (-a)(-(a+1)) & {\rm if\,\,} f=l \\
                   &= (1-a)(-a) & {\rm if\,\,} f=l+1 \\
                   &=(2-a)(1-a)  & {\rm if\,\,} f=l+2
\end{aligned}
$$

Since all these expressions take a minimum value of 0,
$(f-l-a)(f-l-(a+1))$ take a minimum value of 0 if $l\leq f\leq u$ is satisfied.

Let $g = f-l-a$ be an integer.
We have,

$$
\begin{aligned}
(f-l-a)(f-l-(a+1)) &= g(g-1)
\end{aligned}
$$

which cannot take a negative value.
Thus,  $(f-l-a)(f-l-(a+1))$ takes a minimum value of 0 if and only if  $l\leq f\leq u$ is satisfied.

## Case 4: $u\geq l+3$
We introduce an auxiary integer variable $a$ taking integer in the range $[l,u-1]$.
Such integer variable can be defined using multiple binary variable as shown in [Integer Variables and Solving Simultaneous Equations](INTEGER).

The expression for thie case is as follows:

$$
\begin{aligned}
(f-a)(f-(a+1))
\end{aligned}
$$

Siminarly, we can prove that $(f-l-a)(f-l-(a+1))\geq 0$ always holds.

Suppose that $f$ takes an integer value in the range $[l,u]$.
When $a=f$, then we have

$$
\begin{aligned}
f-a &= 0 & {\rm if\,\,} f\in [l,u-1]\\
f-(a+1) &= 0& {\rm if\,\,} f\in [l+1,u]
\end{aligned}
$$

Thus, either $f-a$=0 or $f-(a+1)=0$ if $f\in [l,u]$ and so
$(f-a)(f-(a+1))$ takes a minimum value of 0 if and only if $l\leq f \leq u$.

## Reducing the number of binary variables
In [Integer Variables and Solving Simultaneous Equations](INTEGER),
$n$ binary variables $x_0, x_1, \ldots, x_{n-1}$ are used to represent integer $a$ in rage $[l,u-1]$
using the following linear expression:

$$
\begin{aligned}
a & = l+2^0x_0+2^1x_1+\cdots +2^{n-2}x_{n-2}+dx_{n-1}
\end{aligned}
$$

This expression can represent all integers from $l$ to $l+2^{n-1}+d-1$, and so
we can select $n$ and $d$ so that $u-1=l+2^{n-1}+d-1$.

For Case 4, QUBO++ uses the following linear expression of $n-1$ binary variables $x_1, \ldots, x_{n-1}$ instead:
$$
\begin{aligned}
a &= l+2^1x_1+\cdots +2^{n-2}x_{n-2}+dx_{n-1}
\end{aligned}
$$

This expression can represent integers from $l$ to $l+2^{n-1}+d-2$, and so
we select $n$ and $d$ so that $u-1=l+2^{n-1}+d-2$.
We call such binary variable $a$, **unit-gap integer variable** which takes
integer value in $[l,u-1]$, but some values in it cannot take.
However, if $a$ cannot take the value $k\in [l,u-1]$,
then $a$ can take $k-1$.
Thus, either $a$ or $a+1$ can take any value in $[k,u-1]$.

## QUBO++ program for the range operator

```cpp
#include "qbpp.hpp"

int main() {
  auto f = qbpp::toExpr(qbpp::var("f"));
  auto f1 = 1 <= f <= 1;
  auto f2 = 1 <= f <= 2;
  auto f3 = 1 <= f <= 3;
  auto f4 = 1 <= f <= 5;
  std::cout << "f1 = " << f1.simplify() << std::endl;
  std::cout << "f2 = " << f2.simplify() << std::endl;
  std::cout << "f3 = " << f3.simplify() << std::endl;
  std::cout << "f4 = " << f4.simplify() << std::endl;
}
```
