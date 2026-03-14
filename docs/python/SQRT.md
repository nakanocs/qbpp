---
layout: default
nav_exclude: true
title: "Square Root"
nav_order: 42
---
# Square Root

This example demonstrates how to compute the square root of
$c=2$ using large integers.
Let $s = 10 ^{10}$ be a fixed integer.
Since PyQBPP cannot handle real numbers directly, we compute  $\sqrt{cs^2}$ instead of $\sqrt{c}$.
From the following relation,

$$
\begin{aligned}
\sqrt{c} &= \sqrt{cs^2}/s
\end{aligned}
$$

we can obtain an approximation of $\sqrt{c}$ with 10 decimal-digit precision.

## HUBO formulation of the square root computation
We define an integer variable $x$ that takes values in the range $[s, 2s]$.
We then formulate the problem using the following equation:

$$
\begin{aligned}
x ^ 2 &= cs ^ 2
\end{aligned}
$$

In PyQBPP, this equality constraint is converted into the following HUBO expression:

$$
(x ^ 2 -cs^2)^2
$$

By finding the value of $x$ that minimizes this expression,
we obtain an approximation of the square root of $c$ with
10 decimal-digit precision.
Since $x$ is internally represented as a linear expression of binary variables, this objective function becomes quartic in those binary variables.

## PyQBPP program
The following PyQBPP program constructs a HUBO expression based on the above idea and solves it using the Easy Solver:
```python
from pyqbpp import var_int, between, EasySolver

c = 2
s = 10**10
x = between(var_int("x"), s, c * s)
f = x * x == c * s * s
f.simplify_as_binary()
solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()
print(f"Energy = {sol.energy()}")
print(f"x = {sol.eval(x)}")
```

Since Python integers have unlimited precision, there is no need to specify special integer types (unlike the C++ version which requires `COEFF_TYPE=cpp_int`).
The constant `s`, the integer variable `x`, and the HUBO expression `f` are defined according to the formulation described above.
The Easy Solver is executed with a time limit of 1.0 second.

This program produces the following output:
```
Energy = 57910111919782629376
x = 14142135624
```
We can confirm that the Easy Solver outputs the correct approximation:

$$
 \sqrt{2\times 10^{20}}\approx 14142135624
$$

Note that the reported energy value is not zero, and the equality constraint is not satisfied exactly.
This is simply because there is no exact integer solution to the equality.
Instead, the solver finds a solution that minimizes the error of the equality constraint.
The energy value shown in the output corresponds to the square of this error.
Since the error is minimized, the resulting value of $x$ represents an approximation of the square root.
