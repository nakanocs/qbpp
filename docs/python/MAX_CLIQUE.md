---
layout: default
nav_exclude: true
title: "Maximum Clique"
nav_order: 53
---
<div class="lang-en" markdown="1">
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
</div>

<div class="lang-ja" markdown="1">
# 最大クリーク問題

無向グラフ $G=(V,E)$ が与えられたとき、最大クリーク問題は、$S$ 内のすべての異なる頂点対が $E$ の辺で接続されているような最大の部分集合 $S\subseteq V$ を求めることを目的とします。

頂点は $0,1,\ldots,n−1$ とラベル付けされているとします。
$n$ 個のバイナリ変数 $x_0, x_1, \ldots, x_{n-1}$ を導入し、$x_i=1$ はノード $i$ が $S$ に属する場合にのみ成り立ちます（$0\leq i\leq n−1$）。
このとき、$S$ のサイズは次のように与えられます：

$$
\begin{aligned}
\text{objective} &= \sum_{i=0}^{n-1}x_i.
\end{aligned}
$$

$S$ がクリークであるためには、選択されたすべてのノード対が辺で接続されていなければなりません。
同値的に、$(i,j)\not\in E$ であるすべてのノード対 $i$ と $j$ について、両方を選択することはできません。
これは以下の制約で表現できます：

$$
\begin{aligned}
\text{constraint} &= \sum_{(i,j)\not\in E}x_ix_j
\end{aligned}
$$

実行可能なクリークは $constraint=0$ を満たします。
したがって、以下の QUBO 定式化 $f$（最小化対象）を得ます：

$$
\begin{aligned}
f &= -\text{objective}+2\times \text{constraint}
\end{aligned}
$$

## 最大クリーク問題の PyQBPP プログラム
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
辺リスト `edges` から隣接行列 `adj` を構築し、与えられたノード対がグラフの辺を形成するかどうかを判定できるようにします。
`N = 16` 個のバイナリ変数のベクトル `x` に対して、上記の QUBO 定式化に従って式 `objective`、`constraint`、`f` を構築します。

このプログラムは以下の出力を生成します：
```
objective = 4
constraint = 0
```
この出力から、制約を違反することなく4ノードの最大クリークが得られました。

## matplotlib による可視化
以下のコードは、最大クリークの解を可視化します：
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

クリークのノードは赤で表示され、クリーク内の辺はハイライトされます。
</div>
