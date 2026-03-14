---
layout: default
nav_exclude: true
title: "Vertex Cover"
nav_order: 54
---
# Minimum Vertex Cover Problem
A **vertex cover** of an undirected graph $G=(V,E)$ is a subset
$S\subseteq V$ such that, for every edge $(u,v)\in E$, at least one of its endpoints belongs to $S$.
The **minimum vertex cover problem** is to find a vertex cover with minimum cardinality.

We can formulate this problem as a QUBO expression.
For an $n$-node graph $G=(V,E)$ whose nodes are labeled $0,1,\ldots,n−1$,
we introduce $n$ binary variables $x_0,x_1,\ldots, x_{n-1}$, where $x_i=1$ if and only if node
$i$ is selected (i.e., $i\in S$).

Using negated variables $(1-x_i)$ (where $(1-x_i)=1$ iff $x_i=0$),
we define the following penalty term, which becomes 0 if and only if every edge is covered:

$$
\begin{aligned}
\text{constraint} &= \sum_{(i,j)\in E} (1-x_i)(1-x_j)
\end{aligned}
$$

The objective is to minimize the number of selected vertices:

$$
\begin{aligned}
\text{objective} &= \sum_{i=0}^{n-1}x_i
\end{aligned}
$$

Finally, the QUBO expression $f$ is given by:

$$
\begin{aligned}
f &= \text{objective} + 2\times \text{constraint}
\end{aligned}
$$

## PyQBPP program for the minimum vertex cover problem
```python
from pyqbpp import var, sum, toExpr, ExhaustiveSolver

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),
    (3, 7),  (3, 13), (4, 6),  (4, 7),  (5, 8),  (6, 8),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),
    (10, 12),(11, 13),(12, 14),(13, 15),(14, 15)]

x = var("x", N)

objective = sum(x)
constraint = toExpr(0)
for u, v in edges:
    constraint += (1 - x[u]) * (1 - x[v])
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
In this program, `objective`, `constraint`, and `f` are constructed according to the formulation above.
The Exhaustive Solver is applied to `f` to search for an optimal solution.

This program prints the following output:
```
objective = 9
constraint = 0
```
An optimal solution with objective value 9 and constraint value 0 is obtained.

### Comparison with C++ QUBO++

| C++ QUBO++                   | PyQBPP                              |
|------------------------------|---------------------------------------|
| `~x[e.first] * ~x[e.second]`| `(1 - x[u]) * (1 - x[v])`            |

## Visualization using matplotlib
The following code visualizes the Vertex Cover solution:
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
plt.title("Minimum Vertex Cover")
plt.savefig("vertex_cover.png", dpi=150, bbox_inches="tight")
plt.show()
```

Cover vertices are shown in red. Every edge has at least one red endpoint.
