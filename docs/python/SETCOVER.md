---
layout: default
nav_exclude: true
title: "Set Cover"
nav_order: 57
---
# Minimum Set Cover Problem
Let $U$ be a universe set, and let ${\cal 𝐹}=\lbrace S_0, S_1, \ldots S_{m-1}\rbrace$ be a family of subsets of $U$.
A subfamily $\cal S\subseteq \cal F$ is called a **set cover** if it covers all elements of $U$, i.e.,

$$
\begin{aligned}
\bigcup_{S_j\subseteq \cal S}S_j &= U.
\end{aligned}
$$

The **minimum set cover problem** is to find a set cover
$\cal S$ with the minimum cardinality.
Here, we consider the weighted version, where each subset
$S_j$ has a weight $w_j$, and the goal is to find a **set cover with the minimum total weight**.

## HUBO formulation of the minimum set cover problem
We formulate this problem as a HUBO problem.
Assume that $U=\lbrace 0,1,\ldots, n-1\rbrace$, and $m$ subsets $S_0, S_1, \ldots, S_{m-1}$ are given.
We introduce $m$ binary variables $x_0, x_1, \ldots, x_{m-1}$,
where $x_j=1$ if and only if $S_j\in\cal S$.

For each element $i\in U$, define the following expression:

$$
\begin{aligned}
c_i &=\prod_{j: i\in S_j}(1-x_j) && (0\leq i\leq n-1)
\end{aligned}
$$

The following **constraint** becomes 0 if and only if all elements are covered:

$$
\begin{aligned}
\text{constraint} &=\sum_{i=0}^{n-1}c_i
\end{aligned}
$$

The **objective** is to minimize the total weight of the selected subsets:

$$
\begin{aligned}
\text{objective} &=\sum_{j=0}^{m-1}w_jx_j
\end{aligned}
$$

## PyQBPP program for the minimum set cover problem
```python
from pyqbpp import var, expr, sum, toExpr, Vector, ExhaustiveSolver

n = 10
cover = [
    [0, 1, 2], [2, 3, 4],       [4, 5, 6],    [6, 7, 8],
    [9, 0, 1], [1, 3, 5, 7, 9], [0, 3, 6, 9], [1, 4, 7, 8]]
cost = [3, 4, 3, 2, 3, 4, 3, 3]
m = len(cover)

x = var("x", m)

c = [toExpr(1) for _ in range(n)]
for i in range(m):
    for j in cover[i]:
        c[j] *= (1 - x[i])

objective = toExpr(0)
for i in range(m):
    objective += cost[i] * x[i]

constraint = toExpr(0)
for j in range(n):
    constraint += c[j]

f = objective + 1000 * constraint

f.simplify_as_binary()
solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol.eval(objective)}")
print(f"constraint = {sol.eval(constraint)}")

for i in range(m):
    if sol.get(x[i]) == 1:
        print(f"Set {i}: {cover[i]} cost = {cost[i]}")
```
This program defines a vector `x` of $m=8$ binary variables and constructs a list `c` of $n=10$ expressions.
Each expression `c[j]` corresponds to an element $j\in U$ and is initialized to 1.
For every subset $S_i$, and for every element $j\in S_i$, we multiply `c[j]` by
`(1 − x[i])`.

The Exhaustive Solver is then used to find an optimal solution. For example, the output is:
```
objective = 11
constraint = 0
Set 0: [0, 1, 2] cost = 3
Set 2: [4, 5, 6] cost = 3
Set 3: [6, 7, 8] cost = 2
Set 6: [0, 3, 6, 9] cost = 3
```
This output indicates that a feasible set cover with total cost 11 is obtained.

## QUBO formulation for the minimum set cover problem
To obtain a QUBO formulation, we rewrite the covering constraint.
For each element $i\in U$, define $c_i =\sum_{j: i\in S_j} x_j$.
If $c_i\geq 1$, then at least one selected subset covers $i$.

The constraint in PyQBPP can be modified as follows:
```python
from pyqbpp import between

c = [toExpr(0) for _ in range(n)]
for i in range(m):
    for j in cover[i]:
        c[j] += x[i]

constraint = toExpr(0)
for j in range(n):
    constraint += between(c[j], 1, m)
```

### Comparison with C++ QUBO++

| C++ QUBO++                   | PyQBPP                              |
|------------------------------|---------------------------------------|
| `1 <= c[j] <= +qbpp::inf`   | `between(c[j], 1, m)`                |

With this modification, the program produces the same optimal solution as the HUBO version.
