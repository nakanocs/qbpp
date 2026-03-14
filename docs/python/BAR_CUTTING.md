---
layout: default
nav_exclude: true
title: "Bar Cutting"
nav_order: 73
---
# Bar Cutting Problem
Suppose that we are given $M$ identical bars of fixed length $L$, and a set of $N$ orders specified by pairs $(l_j,c_j)$
($0\leq j\leq N−1$), where $l_j$ is the required length and $c_j$ is the required quantity of order $j$.
The **bar cutting problem** aims to determine how the $M$ bars can be cut to satisfy all orders.

### Order Constraint:
For each order $j$, the total number of pieces assigned across all bars must equal $c_j$:

$$
\begin{aligned}
 \sum_{i=0}^{M-1}x_{i,j} &= c_j & &(0\leq j\leq N-1)
\end{aligned}
$$

### Bar Constraint
For each bar $i$, the total length of the assigned pieces must not exceed $L$:

$$
\begin{aligned}
 \sum_{j=0}^{N-1}l_jx_{i,j} &\leq  L & &(0\leq i\leq M-1)
\end{aligned}
$$

## PyQBPP program
The following PyQBPP program finds a feasible cutting plan using
$M=6$ bars of length $L=60$ and $N=4$ orders:

| Order $j$ | 0 | 1 | 2 | 3|
|---|---|---|---|---|
| Length $l_j$ | 13 | 23 | 8 | 11 |
| Quantity $c_j$ | 10 | 4 | 8 | 6 |

```python
from pyqbpp import var_int, between, sum, toExpr, expr, EasySolver

L = 60
l = [13, 23, 8, 11]
c = [10, 4, 8, 6]
N = len(l)
M = 6

# Create integer variables x[i][j] for pieces of order j cut from bar i
x = [[between(var_int(), 0, c[j]) for j in range(N)] for i in range(M)]

# Order constraint: total pieces for each order must equal c[j]
order_constraint = toExpr(0)
for j in range(N):
    col_sum = toExpr(0)
    for i in range(M):
        col_sum += x[i][j]
    order_constraint += col_sum == c[j]

# Bar constraint: total length used in each bar must not exceed L
bar_length_used = []
bar_constraint = toExpr(0)
for i in range(M):
    used = toExpr(0)
    for j in range(N):
        used += x[i][j] * l[j]
    bar_length_used.append(used)
    bar_constraint += between(used, 0, L)

f = order_constraint + bar_constraint
f.simplify_as_binary()

solver = EasySolver(f)
solver.time_limit(10.0)
solver.target_energy(0)
sol = solver.search()

for i in range(M):
    pieces = "  ".join(str(sol.eval(x[i][j])) for j in range(N))
    used = sol.eval(bar_length_used[i])
    print(f"Bar {i}:  {pieces}   used = {used}, waste = {L - used}")

for j in range(N):
    fulfilled = 0
    for i in range(M):
        fulfilled += sol.eval(x[i][j])
    print(f"Order {j} fulfilled = {fulfilled}, required = {c[j]}")
```
The following output is an example feasible solution:
```
Bar 0:  2  0  0  3   used = 59, waste = 1
Bar 1:  4  0  1  0   used = 60, waste = 0
Bar 2:  1  1  3  0   used = 60, waste = 0
Bar 3:  0  0  4  2   used = 54, waste = 6
Bar 4:  2  1  0  1   used = 60, waste = 0
Bar 5:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 10, required = 10
Order 1 fulfilled = 4, required = 4
Order 2 fulfilled = 8, required = 8
Order 3 fulfilled = 6, required = 6
```
