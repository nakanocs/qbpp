---
layout: default
nav_exclude: true
title: "Knapsack"
nav_order: 70
---
# Knapsack Problem
Given a set of items, each with a weight and a value, and a knapsack with a limited weight capacity, **the knapsack problem** aims to select a subset of items that maximizes the total value while keeping the total weight within the capacity.

Let $w_i$ and $v_i$ ($0\leq i\leq n-1$) denote the weight and value of item
$i$, respectively.
Let $S\in \lbrace 0, 1, \ldots n-1\rbrace$ be the set of selected items.

$$
\begin{aligned}
\text{Maximize:} & \sum_{i\in S} v_i \\
\text{Subject to:} & \sum_{i\in S} w_i \leq W
\end{aligned}
$$

where $W$ is the weight capacity of the knapsack.

## QUBO formulation
We introduce $n$ binary variables $x_i\in\lbrace 0,1\rbrace$ ($0\leq i\leq n-1$),
where item $i$ is selected if and only if $x_i=1$.
The QUBO objective function is:

$$
\begin{aligned}
f(X) &= -\sum_{i=0}^{n-1} v_ix_i + P\times (0\leq \sum_{i=0}^{n-1} w_ix_i \leq W)
\end{aligned}
$$

## PyQBPP program
```python
from pyqbpp import var, sum, between, Vector, ExhaustiveSolver

w = Vector([10, 20, 30, 5, 8, 15, 12, 7, 17, 18])
v = Vector([60, 100, 120, 60, 80, 150, 110, 70, 150, 160])
capacity = 50

x = var("x", len(w))

constraint = between(sum(w * x), 0, capacity)
objective = sum(v * x)

f = -objective + 1000 * constraint
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for idx, sol in enumerate(sols):
    print(f"[Solution {idx}]")
    print(f"Energy = {sol.energy()}")
    print(f"Constraint = {sol.eval(constraint.body)}")
    print(f"Objective = {sol.eval(objective)}")
    for j in range(len(w)):
        if sol.get(x[j]) == 1:
            print(f"Item {j}: weight = {w[j]}, value = {v[j]}")
```
The expressions `constraint` and `objective` are constructed separately and combined into the final QUBO expression `f`.
The Exhaustive Solver is then applied to enumerate all optimal solutions.

The following output shows the optimal solutions:
```
[Solution 0]
Energy = -480
Constraint = 50
Objective = 480
Item 3: weight = 5, value = 60
Item 5: weight = 15, value = 150
Item 6: weight = 12, value = 110
Item 9: weight = 18, value = 160
[Solution 1]
Energy = -480
Constraint = 50
Objective = 480
Item 3: weight = 5, value = 60
Item 4: weight = 8, value = 80
Item 6: weight = 12, value = 110
Item 7: weight = 7, value = 70
Item 9: weight = 18, value = 160
```

### Comparison with C++ QUBO++

| C++ QUBO++                   | PyQBPP                              |
|------------------------------|---------------------------------------|
| `0 <= sum(w * x) <= capacity`| `between(sum(w * x), 0, capacity)`   |
| `sol(*constraint)`           | `sol.eval(constraint.body)`           |
