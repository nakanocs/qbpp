---
layout: default
nav_exclude: true
title: "3-Digit Math"
nav_order: 45
---
# 3-Digit Math Problem

Let us solve the following math problem using PyQBPP.

> **Math Problem**:
> Find all three-digit odd integers whose **product of digits** is **252**.

Let $x$, $y$, and $z$ be the hundreds, tens, and ones digits of the integer, respectively.
More specifically:
- $x$ is an integer in $[1, 9]$,
- $y$ is an integer in $[0, 9]$,
- $t$ is an integer in $[0, 4]$,
- $z = 2t + 1$ (so $z$ is odd).

Then the value $v$ of the three-digit integer $xyz$ is

$$
\begin{aligned}
v&=100x+10y+z
\end{aligned}
$$


We find all solutions satisfying:

$$
\begin{aligned}
xyz &= 252
\end{aligned}
$$

## PyQBPP program
The following PyQBPP program finds all solutions:
```python
from pyqbpp import var_int, between, ExhaustiveSolver

x = between(var_int("x"), 1, 9)
y = between(var_int("y"), 0, 9)
t = between(var_int("t"), 0, 4)
z = 2 * t + 1
v = x * 100 + y * 10 + z

f = x * y * z == 252

f.simplify_as_binary()
solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
results = set()
for sol in sols:
    results.add(sol.eval(v))
for val in sorted(results):
    print(val, end=" ")
print()
```
In this program, **`x`**, **`y`**, and **`t`** are defined as integer variables with the ranges above.
Then **`z`**, **`v`**, and **`f`** are defined as expressions.
We create an Exhaustive Solver instance for `f` and store all optimal solutions in `sols`.

Because `x`, `y`, and `t` are encoded by multiple binary variables, different binary assignments can represent the same integer values.
As a result, the same digit triple (`x`,`y`,`z`) may appear multiple times in `sols`.
Therefore, we use a `set` to remove duplicates by collecting only the resulting integer values `v`.

The integers are printed as follows:
```
479 497 667 749 947
```
