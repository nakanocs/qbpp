---
layout: default
nav_exclude: true
title: "Max-Cut"
nav_order: 51
---
<div class="lang-en" markdown="1">
# Max-Cut Problem

Given an undirected graph $G=(V,E)$, the **Max-Cut** problem aims to partition the node set $V$ into two disjoint subsets $S$ and $\overline{S}$ so that the number of edges in $E$ that have one endpoint in $S$ and the other in $\overline{S}$ is **maximized**.

Assume that the nodes are labeled $0,1,\ldots,n-1$.
We introduce $n$ binary variables $x_0, x_1, \ldots, x_{n-1}$, where
$x_i=1$ if and only if node $i$ belongs to $S$ ($0\le i\le n-1$).
Then, the number of edges crossing the cut $(S,\overline{S})$ is given by

$$
\begin{aligned}
\text{objective} &= \sum_{(i,j)\in E}\Bigl(x_i(1-x_j) + (1-x_i)x_j\Bigr).
\end{aligned}
$$

Since the QUBO problems aims to **minimize** an objective function, we obtain a QUBO expression $f$ by **negating** the objective:

$$
\begin{aligned}
f &= -\,\text{objective}.
\end{aligned}
$$

An optimal assignment minimizing $f$ corresponds to a maximum cut of $G$.

## PyQBPP program for the Max-Cut problem
```python
import pyqbpp as qbpp

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),  (3, 7),
    (3, 13), (4, 6),  (4, 7),  (4, 14), (5, 8),  (6, 8),  (6, 12),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),(10, 12),
    (11, 13),(11, 15),(12, 14),(12, 15),(13, 15),(14, 15)]

x = qbpp.var("x", N)

objective = toExpr(0)
for u, v in edges:
    objective += x[u] * ~x[v] + ~x[u] * x[v]

f = -objective
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol(objective)}")
print("S:", end="")
for i in range(N):
    if sol(x[i]) == 1:
        print(f" {i}", end="")
print()
```
This program creates the expressions `objective` and `f`, where `f` is the negation of `objective`.
The Exhaustive Solver minimizes `f`, and an optimal assignment is stored in `sol`.

This program prints the following output:
```
objective = 22
```

## Visualization using matplotlib
The following code visualizes the Max-Cut solution using `matplotlib` and `networkx`:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

colors = ["#e74c3c" if sol(x[i]) == 1 else "#3498db" for i in range(N)]
edge_colors = ["#e74c3c" if sol(x[u]) != sol(x[v]) else "#cccccc"
               for u, v in edges]
edge_widths = [2.5 if sol(x[u]) != sol(x[v]) else 1.0
               for u, v in edges]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color=edge_colors, width=edge_widths)
plt.title("Max-Cut")
plt.savefig("maxcut.png", dpi=150, bbox_inches="tight")
plt.show()
```

The two partitions are shown in red and blue. Cut edges (crossing the partition) are highlighted in red.
</div>

<div class="lang-ja" markdown="1">
# 最大カット問題

無向グラフ $G=(V,E)$ が与えられたとき、**最大カット問題**は、ノード集合 $V$ を2つの互いに素な部分集合 $S$ と $\overline{S}$ に分割し、一方の端点が $S$ に、他方が $\overline{S}$ に属する $E$ 中の辺の数を**最大化**することを目的とします。

ノードは $0,1,\ldots,n-1$ とラベル付けされているとします。
$n$ 個のバイナリ変数 $x_0, x_1, \ldots, x_{n-1}$ を導入し、$x_i=1$ はノード $i$ が $S$ に属する場合にのみ成り立ちます（$0\le i\le n-1$）。
このとき、カット $(S,\overline{S})$ を横切る辺の数は次のように与えられます：

$$
\begin{aligned}
\text{objective} &= \sum_{(i,j)\in E}\Bigl(x_i(1-x_j) + (1-x_i)x_j\Bigr).
\end{aligned}
$$

QUBO 問題は目的関数を**最小化**することを目指すため、目的関数を**符号反転**して QUBO 式 $f$ を得ます：

$$
\begin{aligned}
f &= -\,\text{objective}.
\end{aligned}
$$

$f$ を最小化する最適な割り当ては、$G$ の最大カットに対応します。

## Max-Cut 問題の PyQBPP プログラム
```python
import pyqbpp as qbpp

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),  (3, 7),
    (3, 13), (4, 6),  (4, 7),  (4, 14), (5, 8),  (6, 8),  (6, 12),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),(10, 12),
    (11, 13),(11, 15),(12, 14),(12, 15),(13, 15),(14, 15)]

x = qbpp.var("x", N)

objective = toExpr(0)
for u, v in edges:
    objective += x[u] * ~x[v] + ~x[u] * x[v]

f = -objective
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol(objective)}")
print("S:", end="")
for i in range(N):
    if sol(x[i]) == 1:
        print(f" {i}", end="")
print()
```
このプログラムは式 `objective` と `f` を作成します。`f` は `objective` の符号反転です。
Exhaustive Solver が `f` を最小化し、最適な割り当てが `sol` に格納されます。

このプログラムは以下の出力を生成します：
```
objective = 22
```

## matplotlib による可視化
以下のコードは `matplotlib` と `networkx` を用いて Max-Cut の解を可視化します：
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

colors = ["#e74c3c" if sol(x[i]) == 1 else "#3498db" for i in range(N)]
edge_colors = ["#e74c3c" if sol(x[u]) != sol(x[v]) else "#cccccc"
               for u, v in edges]
edge_widths = [2.5 if sol(x[u]) != sol(x[v]) else 1.0
               for u, v in edges]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color=edge_colors, width=edge_widths)
plt.title("Max-Cut")
plt.savefig("maxcut.png", dpi=150, bbox_inches="tight")
plt.show()
```

2つの分割は赤と青で表示されます。カット辺（分割を横切る辺）は赤でハイライトされます。
</div>
