---
layout: default
nav_exclude: true
title: "Maximum Clique"
nav_order: 53
---
# Maximum Clique Problem

Given an undirected graph $G=(V,E)$, the Maximum Clique problem aims to find a largest subset
$S\subseteq V$ such that every pair of distinct vertices in $S$ is connected by an edge in $E$.

Assume that the vertices are labeled $0,1,\ldots,n−1$.
We introduce $n$ binary variables $x_0, x_1, \ldots, x_{n-1}$,
where $x_i=1$ if and only if node $i$ belongs to $S$ ($0\leq i\leq n−1$).
Then, the size of $S$ is given by

$$
\begin{aligned}
\text{objective} &= \sum_{i=0}^{n-1}x_i.
\end{aligned}
$$

For $S$ to be a clique, every pair of selected nodes must be connected by an edge.
Equivalently, for every pair of nodes $i$ and $j$ such that
$(i,j)\not\in E$ we cannot select both $i$ and $j$.
This can be expressed by the constraint:

$$
\begin{aligned}
\text{constraint} &= \sum_{(i,j)\not\in E}x_ix_j
\end{aligned}
$$

A feasible clique satisfies $constraint=0$.
Thus, we obtain the following QUBO formulation $f$ (to be minimized):

$$
\begin{aligned}
f &= -\text{objective}+2\times \text{constraint}
\end{aligned}
$$

## PyQBPP program for the maximum clique problem
```python
from pyqbpp import var, sum, Expr, ExhaustiveSolver

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),
    (3, 7),  (3, 13), (4, 6),  (4, 7),  (4, 12), (4, 14),
    (5, 8),  (6, 8),  (6, 12), (6, 14), (7, 14), (7, 15),
    (8, 9),  (9, 10), (9, 12), (10, 11),(10, 12),(11, 13),
    (11, 15),(12, 14),(12, 15),(13, 15),(14, 15)]

adj = [[False] * N for _ in range(N)]
for u, v in edges:
    adj[u][v] = adj[v][u] = True

x = var("x", N)

objective = sum(x)

constraint = Expr(0)
for i in range(N):
    for j in range(i + 1, N):
        if not adj[i][j]:
            constraint += x[i] * x[j]

f = -objective + N * constraint
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol.eval(objective)}")
print(f"constraint = {sol.eval(constraint)}")
print("Clique nodes:", end="")
for i in range(N):
    if sol.get(x[i]) == 1:
        print(f" {i}", end="")
print()
```
From the edge list `edges`, we build an adjacency matrix `adj`, which allows us to test whether a given pair of nodes forms an edge in the graph.
For the vector `x` of `N = 16` binary variables, the expressions `objective`, `constraint`, and `f` are constructed according to the QUBO formulation above.

This program produces the following output:
```
objective = 4
constraint = 0
```
From this output, we obtain a maximum clique of 4 nodes without violating the constraint.

## Visualization using matplotlib
The following code visualizes the Maximum Clique solution:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

colors = ["#e74c3c" if sol.get(x[i]) == 1 else "#d5dbdb" for i in range(N)]
edge_colors = ["#e74c3c" if sol.get(x[u]) == 1 and sol.get(x[v]) == 1
               else "#cccccc" for u, v in edges]
edge_widths = [2.5 if sol.get(x[u]) == 1 and sol.get(x[v]) == 1
               else 1.0 for u, v in edges]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color=edge_colors, width=edge_widths)
plt.title("Maximum Clique")
plt.savefig("max_clique.png", dpi=150, bbox_inches="tight")
plt.show()
```

Clique nodes are shown in red, and edges within the clique are highlighted.
