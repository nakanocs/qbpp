---
layout: default
nav_exclude: true
title: "Maximum Matching"
nav_order: 52
---
# Maximum Matching Problem

A **matching** in an undirected graph is a set of edges such that no two edges share a common node.
Given an undirected graph $G=(V,E)$, the **Maximum Matching** problem aims to find a matching $S \subseteq E$ that contains the maximum number of edges.

Assume that the graph has $n$ vertices and $m$ edges, and that the edges are labeled $0,1,\ldots,m-1$.
We introduce $m$ binary variables $x_0, x_1, \ldots, x_{m-1}$, where
$x_i=1$ if and only if edge $i$ is selected (i.e., belongs to $S$) ($0\le i\le m-1$).
The objective is to maximize the number of selected edges:

$$
\begin{aligned}
\text{objective} &= \sum_{i=0}^{m-1} x_i .
\end{aligned}
$$

To enforce the matching condition, we penalize any pair of selected edges that share a node.
Let $\mathcal{P}$ be the set of unordered pairs $(e_1,e_2)$ of distinct edges that share a common endpoint.
Then the following penalty takes value $0$ if and only if the selected edges form a matching:

$$
\begin{aligned}
\text{constraint} &= \sum_{\{e_1,e_2\}\in \mathcal{P}} x_{e_1}x_{e_2}.
\end{aligned}
$$

We construct a QUBO expression $f$ by combining the objective and the penalty as follows:

$$
\begin{aligned}
f &= -\text{objective} + 2 \times \text{constraint}.
\end{aligned}
$$

## PyQBPP program for the maximum matching
```python
from pyqbpp import var, sum, toExpr, ExhaustiveSolver

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),  (3, 7),
    (3, 13), (4, 6),  (4, 7),  (4, 14), (5, 8),  (6, 8),  (6, 12),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),(10, 12),
    (11, 13),(11, 15),(12, 14),(12, 15),(13, 15),(14, 15)]
M = len(edges)

x = var("x", M)

objective = sum(x)

constraint = toExpr(0)
for i in range(M):
    for j in range(i + 1, M):
        if (edges[i][0] == edges[j][0] or edges[i][0] == edges[j][1] or
            edges[i][1] == edges[j][0] or edges[i][1] == edges[j][1]):
            constraint += x[i] * x[j]

f = -objective + 2 * constraint
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol.eval(objective)}")
print("Selected edges:", end="")
for i in range(M):
    if sol.get(x[i]) == 1:
        print(f" {edges[i]}", end="")
print()
```
This program creates the expressions `objective`, `constraint`, and `f`, where `f` is the negated `objective` plus a penalty term.
The Exhaustive Solver minimizes `f`, and an optimal assignment is stored in `sol`.

## Visualization using matplotlib
The following code visualizes the Maximum Matching solution:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

edge_colors = ["#e74c3c" if sol.get(x[i]) == 1 else "#cccccc"
               for i in range(M)]
edge_widths = [2.5 if sol.get(x[i]) == 1 else 1.0 for i in range(M)]
nx.draw(G, pos, with_labels=True, node_color="#d5dbdb", node_size=400,
        font_size=9, edge_color=edge_colors, width=edge_widths)
plt.title("Maximum Matching")
plt.savefig("max_matching.png", dpi=150, bbox_inches="tight")
plt.show()
```

Selected matching edges are shown in red.
