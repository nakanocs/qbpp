---
layout: default
nav_exclude: true
title: "MIS Problem"
nav_order: 50
---
# Maximum Independent Set (MIS) Problem

An independent set of an undirected graph $G=(V,E)$ is a subset of vertices $S\subseteq V$ such that no two vertices in $S$ are connected by an edge in $E$.
The Maximum Independent Set (MIS) problem asks for an independent set with maximum cardinality.

The MIS problem can be formulated as a QUBO as follows.
Assume that $G$ has $n$ vertices indexed from $0$ to $n-1$.
We introduce $n$ binary variables $x_i$ $(0\le i\le n-1)$, where $x_i=1$ if and only if vertex $i$ is included in $S$.
Since we want to maximize $\|S\|=\sum_{i=0}^{n-1}x_i$, we minimize the following objective:

$$
\begin{aligned}
\text{objective} = -\sum_{i=0}^{n-1} x_i .
\end{aligned}
$$

To enforce independence, for every edge $(i,j)\in E$ we must not select both endpoints simultaneously.
This can be penalized by

$$
\begin{aligned}
\text{constraint} = \sum_{(i,j)\in E} x_i x_j .
\end{aligned}
$$

Combining the objective and the penalty yields the QUBO function

$$
\begin{aligned}
f = \text{objective} + 2\times\text{constraint}.
\end{aligned}
$$

The penalty coefficient $2$ is sufficient to prioritize feasibility over increasing the set size.

## PyQBPP Program for the MIS Problem
Based on the QUBO formulation of the MIS problem described above, the following PyQBPP program solves an instance with 16 nodes:
```python
from pyqbpp import var, expr, sum, toExpr, ExhaustiveSolver

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),
    (3, 7),  (3, 13), (4, 6),  (4, 7),  (5, 8),  (6, 8),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),
    (10, 12),(11, 13),(12, 14),(13, 15),(14, 15)]

x = var("x", N)

objective = -sum(x)
constraint = expr()
for u, v in edges:
    constraint += x[u] * x[v]
f = objective + constraint * 2
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol.eval(objective)}")
print(f"constraint = {sol.eval(constraint)}")

print("Selected nodes:", end="")
for i in range(N):
    if sol.get(x[i]) == 1:
        print(f" {i}", end="")
print()
```
For a vector `x` of `N = 16` binary variables, the expressions `objective`, `constraint`, and `f` are constructed according to the above QUBO formulation.
The Exhaustive Solver is then used to find an optimal solution for `f`, which is stored in `sol`. The values of `objective` and `constraint` evaluated at `sol` are printed.

This program produces the following output:
```
objective = -7
constraint = 0
```
This implies that the obtained solution selects 7 nodes and satisfies all constraints.

## Visualization using matplotlib
The following code visualizes the MIS solution using `matplotlib` and `networkx`:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

colors = ["#e74c3c" if sol.get(x[i]) == 1 else "#d5dbdb" for i in range(N)]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color="#888888", width=1.2)
plt.title("Maximum Independent Set")
plt.savefig("mis.png", dpi=150, bbox_inches="tight")
plt.show()
```

Selected nodes are shown in red, and unselected nodes are shown in gray.
