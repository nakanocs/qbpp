---
layout: default
nav_exclude: true
title: "Dominating Set"
nav_order: 55
---
# Minimum Dominating Set Problem
A dominating set of an undirected graph $G=(V,E)$ is a subset $S\subseteq V$ such that every node
$u\in V$ is either in $S$ or adjacent to a vertex in $S$.

Let $N(u)=\lbrace v\in V\mid (u,v)\in E\rbrace$ be the set of neighbors of $u\in V$, and let
$N[u]=\lbrace u\rbrace\cup N(u)$ be the closed neighborhood of $u$.
Then $S$ is a dominating set if and only if

$$
\begin{aligned}
V = \bigcup_{u\in V} N[u].
\end{aligned}
$$

The minimum dominating set problem aims to find the dominating set with the minimum cardinality.

We will show two formulations:
- **HUBO formulation**: the expression may include higher-degree terms.
- **QUBO formulation**: the expression is quadratic, but auxiliary variables are used.

## HUBO formulation of the minimum dominating set problem

For each node $i\in V$, node $i$ is NOT dominated only when $x_j=0$ for all $j\in N[i]$, i.e., $\prod_{j\in N[i]}(1-x_j)=1$.
Thus, we define the constraint as:

$$
\begin{aligned}
\text{constraint} = \sum_{i=0}^{n-1} \prod_{j\in N[i]}(1-x_j)
\end{aligned}
$$

The objective is to minimize the number of selected nodes:

$$
\begin{aligned}
\text{objective} = \sum_{i=0}^{n-1} x_i
\end{aligned}
$$

Finally, the expression $f$:

$$
\begin{aligned}
f &= \text{objective} + (n+1)\times \text{constraint}
\end{aligned}
$$

## PyQBPP program for the HUBO formulation
```python
from pyqbpp import var, sum, toExpr, EasySolver

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),
    (3, 7),  (3, 13), (4, 6),  (4, 7),  (5, 8),  (6, 8),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),
    (10, 12),(11, 13),(12, 14),(13, 15),(14, 15)]

adj = [[] for _ in range(N)]
for u, v in edges:
    adj[u].append(v)
    adj[v].append(u)

x = var("x", N)

objective = sum(x)

constraint = toExpr(0)
for i in range(N):
    t = toExpr(1 - x[i])
    for j in adj[i]:
        t *= (1 - x[j])
    constraint += t

f = objective + (N + 1) * constraint
f.simplify_as_binary()

solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

print(f"objective = {sol.eval(objective)}")
print(f"constraint = {sol.eval(constraint)}")
print("Dominating set:", end="")
for i in range(N):
    if sol.get(x[i]) == 1:
        print(f" {i}", end="")
print()
```
This program first builds the adjacency list `adj` from the edge list `edges`.
It then constructs `constraint`, `objective`, and `f` according to the HUBO formulation.

This program produces the following output:
```
objective = 5
constraint = 0
```

## QUBO formulation and the PyQBPP program
A node $i$ is dominated if $N[i]\cap S$ is not empty.
This condition is equivalent to the following inequality:

$$
\begin{aligned}
\sum_{j\in N[i]}x_j &\geq 1
\end{aligned}
$$

The constraint can be described in PyQBPP as follows:
```python
from pyqbpp import between

constraint = toExpr(0)
for i in range(N):
    t = toExpr(x[i])
    for j in adj[i]:
        t += x[j]
    constraint += between(t, 1, len(adj[i]) + 1)
```
In this code, `t` stores the expression $\sum_{j\in N[i]}x_j$ and the `between()` function creates a penalty expression for $1\leq \sum_{j\in N[i]}x_j \leq |N[i]|+1$, which takes the minimum value 0 if and only if the inequality is satisfied.

### Comparison with C++ QUBO++

| C++ QUBO++                  | PyQBPP                                |
|-----------------------------|----------------------------------------|
| `~x[i]`                    | `(1 - x[i])`                          |
| `1 <= t <= +qbpp::inf`     | `between(t, 1, upper_bound)`          |

## Visualization using matplotlib
The following code visualizes the Dominating Set solution:
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
plt.title("Minimum Dominating Set")
plt.savefig("dominating_set.png", dpi=150, bbox_inches="tight")
plt.show()
```

Dominating set vertices are shown in red. Every gray node is adjacent to at least one red node.
