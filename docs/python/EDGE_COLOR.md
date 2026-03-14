---
layout: default
nav_exclude: true
title: "Edge Coloring"
nav_order: 60
---
# Graph Edge Coloring Problem
Given an undirected graph $G=(V,E)$, the **graph edge coloring problem** aims to assign a color to each edge so that no two edges of the same color share a common endpoint.

More specifically, for a set $C$ of colors, the goal is to find an assignment
$\sigma:E\rightarrow C$ such that for any two distinct edges $e$ and $e'$ incident to the same node, we have $\sigma(e)\neq \sigma(e')$.

Let $V=\lbrace 0,1,\ldots,n−1\rbrace$ and $C=\lbrace 0,1,\ldots,m−1\rbrace$.
We assign unique IDs $0,1,\ldots,s−1$ to the $s=|E|$ edges.

We introduce an $s\times m$ matrix $X=(x_{i,j})$ of binary variables, where
$x_{i,j}=1$ if and only if edge $i$ is assigned color $j$.

### One-hot constraint
Each row of $X$ must be one-hot:

$$
\begin{aligned}
  \text{onehot}&= \sum_{i=0}^{s-1}\bigr(\sum_{j=0}^{m-1}x_{i,j}==1\bigl)
\end{aligned}
$$

### Incident edges must differ

$$
\begin{aligned}
  \text{different}&= \sum_{u\in V}\sum_{\substack{i<k\\ i,k\in I(u)}}\sum_{j=0}^{m-1}x_{i,j}x_{k,j}
\end{aligned}
$$

where $I(u)$ denotes the set of edge IDs incident to node $u$.

## PyQBPP program
It is known that the edge chromatic number of a simple graph is either $\Delta$ or $\Delta+1$, where $\Delta$ is the maximum degree. The following program attempts an edge coloring with $m=\Delta$ colors:
```python
from pyqbpp import var, sum, Expr, vector_sum, EasySolver

n = 16
edges = [
    (0, 1),  (0, 2),  (0, 4),  (1, 3),  (1, 4),  (1, 7),  (2, 5),
    (2, 6),  (3, 7),  (3, 13), (3, 15), (4, 6),  (4, 7),  (4, 14),
    (5, 8),  (6, 8),  (6, 14), (7, 14), (7, 15), (8, 9),  (8, 12),
    (9, 10), (9, 11), (9, 12), (10, 11),(10, 12),(10, 13),(10, 14),
    (10, 15),(11, 13),(12, 14),(13, 15),(14, 15)]

adj = [[] for _ in range(n)]
for i, (u, v) in enumerate(edges):
    adj[u].append(i)
    adj[v].append(i)

max_degree = max(len(neighbors) for neighbors in adj)
m = max_degree

s = len(edges)
x = var("x", s, m)

onehot = sum(vector_sum(x) == 1)
different = Expr(0)
for i in range(n):
    for u in adj[i]:
        for v in adj[i]:
            if u < v:
                different += sum(x[u] * x[v])

f = onehot + different

f.simplify_as_binary()
solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print(f"colors = {m}")
print(f"onehot = {sol.eval(onehot)}")
print(f"different = {sol.eval(different)}")

# Extract edge colors
for i in range(s):
    for j in range(m):
        if sol.get(x[i][j]) == 1:
            print(f"Edge {edges[i]}: color {j}")
            break
```
In this program, we first build the incidence list `adj`, where `adj[i]` stores the indices of edges incident to node `i`.
We then compute the maximum degree $\Delta$ and set `m=`$\Delta$.

This program produces the following output:
```
colors = 6
onehot = 0
different = 0
```
Therefore, a valid edge coloring using `m = 6` colors is found.

## Visualization using matplotlib
The following code visualizes the Edge Coloring solution:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(n))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

palette = ["#e74c3c", "#3498db", "#2ecc71", "#f39c12", "#9b59b6", "#1abc9c",
           "#e67e22", "#2c3e50"]
edge_color_idx = [next((k for k in range(m) if sol.get(x[i][k]) == 1), 0)
                  for i in range(len(edges))]
edge_colors = [palette[c % len(palette)] for c in edge_color_idx]
nx.draw(G, pos, with_labels=True, node_color="#d5dbdb", node_size=400,
        font_size=9, edge_color=edge_colors, width=2.5)
plt.title(f"Edge Coloring ({m} colors)")
plt.savefig("edge_color.png", dpi=150, bbox_inches="tight")
plt.show()
```

Each edge is colored according to its assigned color.
