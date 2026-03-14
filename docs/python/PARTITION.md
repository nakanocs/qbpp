---
section: python
layout: default
title: "Solving Partitioning Problem Using Vector of variables (Python)"
---


# Solving Partitioning Problem Using Vector of variables

## Partitioning problem
Let $w=(w_0, w_1, \ldots, w_{n-1})$ be $n$ positive numbers.
The **partitioning problem** is to partition these numbers into two sets $P$ and $Q$ ($=\overline{P}$) such that the sums of the elements in the two sets are as close as possible.
More specifically, the problem is to find a subset $L \subseteq \lbrace 0,1,\ldots, n-1\rbrace$ that minimizes:

$$
\begin{aligned}
P(L) &= \sum_{i\in L}w_i \\
Q(L) &= \sum_{i\not\in L}w_i \\
f(L) &= \left| P(L)-Q(L) \right|
\end{aligned}
$$

This problem can be formulated as a QUBO problem.
Let $x=(x_0, x_1, \ldots, x_{n-1})$ be binary variables representing the set $L$,
that is, $i\in L$ if and only if $x_i=1$.
We can rewrite $P(L)$, $Q(L)$ and $f(L)$ using $x$ as follows:

$$
\begin{aligned}
P(x) &= \sum_{i=0}^{n-1} w_ix_i \\
Q(x) &= \sum_{i=0}^{n-1} w_i (1-x_i) \\
f(x)    &= \left( P(x)-Q(x) \right)^2
\end{aligned}
$$

Clearly, $f(x)=f(L)^2$ holds.
The function $f(x)$ is a quadratic expression of $x$, and an optimal solution that minimizes $f(x)$ also gives an optimal solution to the original partitioning problem.

## PyQBPP program for the partitioning problem
The following program creates the QUBO formulation of the partitioning problem for a fixed set of 8 numbers and finds a solution using the Exhaustive Solver.

```python
from pyqbpp import var, expr, sqr, ExhaustiveSolver

w = [64, 27, 47, 74, 12, 83, 63, 40]

x = var("x", len(w))
p = expr()
q = expr()
for i in range(len(w)):
    p += w[i] * x[i]
    q += w[i] * (1 - x[i])
f = sqr(p - q)
print("f =", f.simplify_as_binary())

solver = ExhaustiveSolver(f)
sol = solver.search()

print("Solution:", sol)
print("f(sol) =", sol.eval(f))
print("p(sol) =", sol.eval(p))
print("q(sol) =", sol.eval(q))

print("P :", end="")
for i in range(len(w)):
    if sol.get(x[i]) == 1:
        print(f" {w[i]}", end="")
print()

print("Q :", end="")
for i in range(len(w)):
    if sol.get(x[i]) == 0:
        print(f" {w[i]}", end="")
print()
```

In this program, **`w`** is a Python list with 8 numbers.
A vector **`x`** of `len(w)=8` binary variables is defined.
Two `Expr` objects **`p`** and **`q`** are defined, and the expressions for $P(x)$ and $Q(x)$
are constructed in the for-loop.
An `Expr` object **`f`** stores the expression for $f(x)$.

An Exhaustive Solver object **`solver`** for `f` is created
and the solution **`sol`** (a `Sol` object) is obtained by calling its `search()` method.

The values of $f(x)$, $P(x)$, and $Q(x)$ are evaluated by calling **`sol.eval(f)`**, **`sol.eval(p)`** and **`sol.eval(q)`**, respectively.
The numbers in the sets $L$ and $\overline{L}$ are displayed using the for loops.
In these loops, **`sol.get(x[i])`** returns the value of `x[i]` in `sol`.

This program outputs:
```
f = 168100 -88576*x[0] ...
Solution: Sol(energy=0, x[0]=0, x[1]=0, x[2]=1, x[3]=0, x[4]=1, x[5]=1, x[6]=1, x[7]=0)
f(sol) = 0
p(sol) = 205
q(sol) = 205
P : 47 12 83 63
Q : 64 27 74 40
```

> **NOTE**
> For a `Sol` object `sol` and an `Expr` object `f`, **`sol.eval(f)`** returns the evaluated value of `f` on `sol`.
> For a `Var` object `a`, **`sol.get(a)`** returns the value of `a` in the solution `sol`.

## PyQBPP program using vector operations
PyQBPP has rich vector operations that can simplify the code.
Since the overloaded operator `*` for `Vector` performs element-wise multiplication,
**`sum(Vector(w) * x)`** returns the `Expr` object representing $P(L)$.
Furthermore, the overloaded operator `-` for a scalar and a `Vector` object returns
a `Vector` object whose components are the scalar minus each element of the vector.
Thus, **`sum(Vector(w) * (1 - x))`** returns an `Expr` object storing $Q(L)$.

```python
from pyqbpp import var, sum, sqr, Vector

w = Vector([64, 27, 47, 74, 12, 83, 63, 40])
x = var("x", len(w))
p = sum(w * x)
q = sum(w * (1 - x))
f = sqr(p - q)
```

PyQBPP programs can be simplified by using these vector operations.

> **NOTE**
> The operators `+`, `-`, and `*` are overloaded both for two `Vector` objects and for a scalar and a `Vector` object.
> For two `Vector` objects, the overloaded operators perform element-wise operations.
> For a scalar and a `Vector` object, the overloaded operators apply the scalar operation to each element of the vector.
