---
layout: default
nav_exclude: true
title: "Min-Max Matching"
nav_order: 58
---
<div class="lang-en" markdown="1">
# Minimum Maximal Matching Problem

A **matching** in an undirected graph is a set of edges such that no two edges share a common endpoint.
A **maximal matching** is a matching to which no additional edge can be added without violating the matching condition.
Given an undirected graph $G=(V,E)$, the minimum maximal matching problem asks for a maximal matching $S \subseteq E$ with the minimum number of edges.

The maximality condition of a matching can be described in a compact way.
For a node $u\in V$, let $N(u)$ denote the set of edges in $S$ incident to $u$.
Then $S$ is a maximal matching if and only if, for every edge $(u,v)\in E$, the following condition holds:

$$
 1 \leq |N(u)|+|N(v)| \leq 2
$$

> **Reference**:
> Nakahara, Y., Tsukiyama, S., Nakano, K., Parque, V., & Ito, Y. (2025). **A penalty-free QUBO formulation for the minimum maximal matching problem**. International Journal of Parallel, Emergent and Distributed Systems, 1--19. https://doi.org/10.1080/17445760.2025.2579546


## QUBO formulation for the minimum maximal matching
We introduce $m$ binary variables $x_0, x_1, \ldots, x_{m-1}$, where $x_i=1$ if and only if edge $i$ is selected.
The objective is to minimize the number of selected edges:

$$
\begin{aligned}
\text{objective} &= \sum_{i=0}^{m-1} x_i .
\end{aligned}
$$

To enforce the maximality condition:

$$
\begin{aligned}
\text{constraint} &= \sum_{(u,v)\in E} (1 \leq |N(u)|+|N(v)| \leq 2)
\end{aligned}
$$

## PyQBPP program
```python
import pyqbpp as qbpp

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),  (3, 7),
    (3, 13), (4, 6),  (4, 7),  (4, 14), (5, 8),  (6, 8),  (6, 12),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),(10, 12),
    (11, 13),(11, 15),(12, 14),(12, 15),(13, 15),(14, 15)]
M = len(edges)

adj = [[] for _ in range(N)]
for i in range(M):
    adj[edges[i][0]].append(i)
    adj[edges[i][1]].append(i)

x = qbpp.var("x", M)

objective = qbpp.sum(x)

constraint = 0
for u, v in edges:
    t = 0
    for idx in adj[u]:
        t += x[idx]
    for idx in adj[v]:
        t += x[idx]
    constraint += qbpp.between(t, 1, 2)

f = objective + constraint

f.simplify_as_binary()
solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol(objective)}")
print(f"constraint = {sol(constraint)}")
print("Selected edges:", end="")
for i in range(M):
    if sol(x[i]) == 1:
        print(f" {edges[i]}", end="")
print()
```
We first define a vector `x` of `M` binary variables, and then define `objective`, `constraint`, and `f` based on the formulation above.
The Exhaustive Solver is used to find an optimal solution of `f`.

This program produces the following output:
```
objective = 6
constraint = 0
```
Thus, $S$ contains 6 edges, forming a minimum maximal matching.

### Comparison with C++ QUBO++

| C++ QUBO++                   | PyQBPP                              |
|------------------------------|---------------------------------------|
| `1 <= t <= 2`                | `between(t, 1, 2)`                   |

## Visualization using matplotlib
The following code visualizes the Min-Max Matching solution:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

matched_nodes = set()
for i in range(M):
    if sol(x[i]) == 1:
        matched_nodes.add(edges[i][0])
        matched_nodes.add(edges[i][1])
colors = ["#e74c3c" if i in matched_nodes else "#d5dbdb" for i in range(N)]
edge_colors = ["#e74c3c" if sol(x[i]) == 1 else "#cccccc"
               for i in range(M)]
edge_widths = [2.5 if sol(x[i]) == 1 else 1.0 for i in range(M)]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color=edge_colors, width=edge_widths)
plt.title("Minimum Maximal Matching")
plt.savefig("minmax_matching.png", dpi=150, bbox_inches="tight")
plt.show()
```

Matched edges and their endpoints are shown in red.
</div>

<div class="lang-ja" markdown="1">
# 最小極大マッチング問題

無向グラフにおける**マッチング**とは、共通の端点を持たない辺の集合のことです。
**極大マッチング**とは、マッチング条件を違反せずにはこれ以上辺を追加できないマッチングのことです。
無向グラフ $G=(V,E)$ が与えられたとき、最小極大マッチング問題は、辺の数が最小となる極大マッチング $S \subseteq E$ を求めます。

マッチングの極大性条件はコンパクトに記述できます。
ノード $u\in V$ に対して、$N(u)$ を $u$ に接続する $S$ 内の辺の集合とします。
すると、$S$ が極大マッチングであるための必要十分条件は、すべての辺 $(u,v)\in E$ に対して以下の条件が成り立つことです：

$$
 1 \leq |N(u)|+|N(v)| \leq 2
$$

> **参考文献**:
> Nakahara, Y., Tsukiyama, S., Nakano, K., Parque, V., & Ito, Y. (2025). **A penalty-free QUBO formulation for the minimum maximal matching problem**. International Journal of Parallel, Emergent and Distributed Systems, 1--19. https://doi.org/10.1080/17445760.2025.2579546


## 最小極大マッチングの QUBO 定式化
$m$ 個のバイナリ変数 $x_0, x_1, \ldots, x_{m-1}$ を導入し、$x_i=1$ は辺 $i$ が選択されている場合にのみ成り立ちます。
目的は選択された辺の数を最小化することです：

$$
\begin{aligned}
\text{objective} &= \sum_{i=0}^{m-1} x_i .
\end{aligned}
$$

極大性条件を課すために：

$$
\begin{aligned}
\text{constraint} &= \sum_{(u,v)\in E} (1 \leq |N(u)|+|N(v)| \leq 2)
\end{aligned}
$$

## PyQBPP プログラム
```python
import pyqbpp as qbpp

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),  (3, 7),
    (3, 13), (4, 6),  (4, 7),  (4, 14), (5, 8),  (6, 8),  (6, 12),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),(10, 12),
    (11, 13),(11, 15),(12, 14),(12, 15),(13, 15),(14, 15)]
M = len(edges)

adj = [[] for _ in range(N)]
for i in range(M):
    adj[edges[i][0]].append(i)
    adj[edges[i][1]].append(i)

x = qbpp.var("x", M)

objective = qbpp.sum(x)

constraint = 0
for u, v in edges:
    t = 0
    for idx in adj[u]:
        t += x[idx]
    for idx in adj[v]:
        t += x[idx]
    constraint += qbpp.between(t, 1, 2)

f = objective + constraint

f.simplify_as_binary()
solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol(objective)}")
print(f"constraint = {sol(constraint)}")
print("Selected edges:", end="")
for i in range(M):
    if sol(x[i]) == 1:
        print(f" {edges[i]}", end="")
print()
```
まず `M` 個のバイナリ変数のベクトル `x` を定義し、次に上記の定式化に基づいて `objective`、`constraint`、`f` を定義します。
Exhaustive Solver を用いて `f` の最適解を求めます。

このプログラムは以下の出力を生成します：
```
objective = 6
constraint = 0
```
したがって、$S$ は6本の辺を含み、最小極大マッチングを形成しています。

### C++ QUBO++ との比較

| C++ QUBO++                   | PyQBPP                              |
|------------------------------|---------------------------------------|
| `1 <= t <= 2`                | `between(t, 1, 2)`                   |

## matplotlib による可視化
以下のコードは、最小極大マッチングの解を可視化します：
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

matched_nodes = set()
for i in range(M):
    if sol(x[i]) == 1:
        matched_nodes.add(edges[i][0])
        matched_nodes.add(edges[i][1])
colors = ["#e74c3c" if i in matched_nodes else "#d5dbdb" for i in range(N)]
edge_colors = ["#e74c3c" if sol(x[i]) == 1 else "#cccccc"
               for i in range(M)]
edge_widths = [2.5 if sol(x[i]) == 1 else 1.0 for i in range(M)]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color=edge_colors, width=edge_widths)
plt.title("Minimum Maximal Matching")
plt.savefig("minmax_matching.png", dpi=150, bbox_inches="tight")
plt.show()
```

マッチングに選択された辺とその端点は赤で表示されます。
</div>
