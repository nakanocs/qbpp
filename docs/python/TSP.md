---
layout: default
nav_exclude: true
title: "TSP"
nav_order: 62
---
# Traveling Salesman Problem
The Traveling Salesman Problem (TSP) asks for the shortest tour that visits every node exactly once and returns to the start.
We assume that the nodes are placed on a plane and that the tour length is measured using the Euclidean distance.

<p align="center">
  <img src="../images/tsp_solution.svg" alt="An example of nodes and the TSP solution" width="50%">
</p>


## QUBO formulation of the TSP
A tour can be represented by a permutation of the nodes.
Accordingly, we use a [permutation matrix](PERMUTATION) to encode a TSP solution.

Let $X=(x_{i,j})$ ($0\leq i,j\leq n-1$) be a matrix of $n\times n$ binary values.
We interpret  $x_{k,i}$ as "the $k$-th position in the tour is node $i$".
Therefore, every row and every column of $X$ must be one-hot:

$$
\begin{aligned}
{\rm row}:& \sum_{j=0}^{n-1}x_{i,j}=1 & (0\leq i\leq n-1)\\
{\rm column}:& \sum_{i=0}^{n-1}x_{i,j}=1 & (0\leq j\leq n-1)
\end{aligned}
$$

Let $d_{i,j}$ denote the distance between nodes $i$ and $j$.
Then the tour length for a permutation matrix $X$ can be written as:

$$
\begin{aligned}
{\rm objective}: &\sum_{k=0}^{k-1} d_{i,j}x_{k,i}x_{(k+1)\bmod n,j}
\end{aligned}
$$

## PyQBPP program for TSP
```python
import math
from pyqbpp import var, expr, sum, vector_sum, EasySolver

nodes = [(10, 12),  (33, 125),  (12, 226),
         (121, 11), (108, 142), (111, 243),
         (220, 4),  (210, 113), (211, 233)]

def dist(i, j):
    dx = nodes[i][0] - nodes[j][0]
    dy = nodes[i][1] - nodes[j][1]
    return round(math.sqrt(dx * dx + dy * dy))

n = len(nodes)
x = var("x", n, n)

constraint = sum(vector_sum(x, 1) == 1) + sum(vector_sum(x, 0) == 1)

objective = expr()
for i in range(n):
    next_i = (i + 1) % n
    for j in range(n):
        for k in range(n):
            if k != j:
                objective += dist(j, k) * x[i][j] * x[next_i][k]

f = objective + constraint * 1000
f.simplify_as_binary()

solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

# Extract tour from permutation matrix
tour = []
for i in range(n):
    for j in range(n):
        if sol.get(x[i][j]) == 1:
            tour.append(j)
            break
print(f"Tour: {tour}")
```
In this program, the coordinates of nodes are stored in a list.
We create a 2D array `x` of binary variables and construct the one-hot constraints together with the tour-length objective.

This program produces the following output:
```
Tour: [7, 8, 5, 2, 4, 1, 0, 3, 6]
```

## Fixing the first node
Without loss of generality, we can assume that node 0 is the starting node of the tour.
By fixing the start node, we can reduce the number of binary variables in the QUBO expression.

```python
from pyqbpp import MapList, replace, Sol

ml = MapList()
ml.add(x[0][0], 1)
for i in range(1, n):
    ml.add(x[i][0], 0)
    ml.add(x[0][i], 0)

g = replace(f, ml)
g.simplify_as_binary()

solver = EasySolver(g)
solver.time_limit(1.0)
sol = solver.search()

full_sol = Sol(f)
full_sol.set(sol)
full_sol.set(ml)

# Extract tour
tour = []
for i in range(n):
    for j in range(n):
        if full_sol.get(x[i][j]) == 1:
            tour.append(j)
            break
print(f"Tour: {tour}")
```
First, we create a `MapList` object `ml`, which stores fixed assignments of variables.
Next, we call `replace(f, ml)`, which returns a new expression obtained by substituting the fixed values.
Since `sol` corresponds to the reduced problem, we create a `Sol` object for `f` and set both the solver output `sol` and the fixed assignments `ml`.

This program produces the following tour starting from node 0:
```
Tour: [0, 3, 6, 7, 8, 5, 2, 1, 4]
```

### Comparison with C++ QUBO++

| C++ QUBO++                        | PyQBPP                            |
|------------------------------------|------------------------------------|
| `qbpp::onehot_to_int(sol(x))`    | Manual loop over `sol.get(x[i][j])` |
| `qbpp::MapList ml;`              | `ml = MapList()`                   |
| `ml.push_back({x[0][0], 1})`    | `ml.add(x[0][0], 1)`              |
| `qbpp::replace(f, ml)`          | `replace(f, ml)`                   |
| `qbpp::Sol(f).set(sol).set(ml)` | `Sol(f); full_sol.set(sol); full_sol.set(ml)` |

## Visualization using matplotlib
The following code visualizes the TSP solution:
```python
import matplotlib.pyplot as plt

plt.figure(figsize=(6, 6))
for i, (nx_, ny) in enumerate(nodes):
    plt.plot(nx_, ny, "ko", markersize=8)
    plt.annotate(str(i), (nx_, ny), textcoords="offset points", xytext=(5, 5))

for i in range(n):
    fr = tour[i]
    to = tour[(i + 1) % n]
    plt.annotate("", xy=(nodes[to][0], nodes[to][1]),
                 xytext=(nodes[fr][0], nodes[fr][1]),
                 arrowprops=dict(arrowstyle="->", color="#e74c3c", lw=2))
plt.title("TSP Tour")
plt.savefig("tsp.png", dpi=150, bbox_inches="tight")
plt.show()
```

The tour is shown as directed red arrows connecting the nodes.
