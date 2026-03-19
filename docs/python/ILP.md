---
layout: default
nav_exclude: true
title: "Integer Linear Programming (PyQBPP)"
nav_order: 74
---
<div class="lang-en" markdown="1">
# Integer Linear Programming (ILP)
**Integer Linear Programming (ILP)** can be converted into a QUBO expression using PyQBPP.
As an example, consider the following ILP:

$$
\begin{aligned}
\text{Maximize:} && 2x_0 +5x_1+5x_2\\
\text{Subject to:} && x_0 + 3 x_1 + x_2 &\leq 12 \\
                &&  x_0 + 2x_2 &\leq 5\\
                && x_1 + x_2 &\leq 4;
\end{aligned}
$$

## PyQBPP program
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x", 3), 0, 5)
objective = 2 * x[0] + 5 * x[1] + 5 * x[2]
c1 = between(x[0] + 3 * x[1] + x[2], 0, 12)
c2 = between(x[0] + 2 * x[2], 0, 5)
c3 = between(x[1] + x[2], 0, 4)

f = -objective + 100 * (c1 + c2 + c3)
f.simplify_as_binary()
solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()
print(f"x0 = {sol(x[0])}, x1 = {sol(x[1])}, x2 = {sol(x[2])}")
print(f"objective = {sol(objective)}")
print(f"c1 = {sol(c1.body)}, c2 = {sol(c2.body)}, c3 = {sol(c3.body)}")
```
In this program, `x` is a vector of three integer variables, each taking a value in the range $[0, 5]$.
The objective function and the three constraints are represented by `objective`, `c1`, `c2`, and `c3`.

The obtained solution is printed as follows:
```
x0 = 2, x1 = 3, x2 = 1
objective = 24
c1 = 12, c2 = 4, c3 = 4
```

### Comparison with C++ QUBO++

| C++ QUBO++                        | PyQBPP                              |
|------------------------------------|---------------------------------------|
| `0 <= x[0] + 3 * x[1] + x[2] <= 12` | `between(x[0] + 3 * x[1] + x[2], 0, 12)` |
| `sol(*c1)`                         | `sol(c1.body)`                  |
</div>

<div class="lang-ja" markdown="1">
# 整数線形計画法（ILP）
**整数線形計画法（ILP）** は、PyQBPPを使用してQUBO式に変換できます。
例として、以下のILPを考えます:

$$
\begin{aligned}
\text{Maximize:} && 2x_0 +5x_1+5x_2\\
\text{Subject to:} && x_0 + 3 x_1 + x_2 &\leq 12 \\
                &&  x_0 + 2x_2 &\leq 5\\
                && x_1 + x_2 &\leq 4;
\end{aligned}
$$

## PyQBPPプログラム
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x", 3), 0, 5)
objective = 2 * x[0] + 5 * x[1] + 5 * x[2]
c1 = between(x[0] + 3 * x[1] + x[2], 0, 12)
c2 = between(x[0] + 2 * x[2], 0, 5)
c3 = between(x[1] + x[2], 0, 4)

f = -objective + 100 * (c1 + c2 + c3)
f.simplify_as_binary()
solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()
print(f"x0 = {sol(x[0])}, x1 = {sol(x[1])}, x2 = {sol(x[2])}")
print(f"objective = {sol(objective)}")
print(f"c1 = {sol(c1.body)}, c2 = {sol(c2.body)}, c3 = {sol(c3.body)}")
```
このプログラムでは、`x` は3つの整数変数のベクトルで、それぞれ $[0, 5]$ の範囲の値をとります。
目的関数と3つの制約は `objective`、`c1`、`c2`、`c3` で表現されています。

得られた解は以下のように表示されます:
```
x0 = 2, x1 = 3, x2 = 1
objective = 24
c1 = 12, c2 = 4, c3 = 4
```

### C++ QUBO++との比較

| C++ QUBO++                        | PyQBPP                              |
|------------------------------------|---------------------------------------|
| `0 <= x[0] + 3 * x[1] + x[2] <= 12` | `between(x[0] + 3 * x[1] + x[2], 0, 12)` |
| `sol(*c1)`                         | `sol(c1.body)`                  |
</div>
