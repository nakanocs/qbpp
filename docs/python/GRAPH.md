---
layout: default
nav_exclude: true
title: "MIS Problem"
nav_order: 50
---
<div class="lang-en" markdown="1">
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
import pyqbpp as qbpp

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),
    (3, 7),  (3, 13), (4, 6),  (4, 7),  (5, 8),  (6, 8),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),
    (10, 12),(11, 13),(12, 14),(13, 15),(14, 15)]

x = qbpp.var("x", N)

objective = -qbpp.sum(x)
constraint = qbpp.expr()
for u, v in edges:
    constraint += x[u] * x[v]
f = objective + constraint * 2
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol(objective)}")
print(f"constraint = {sol(constraint)}")

print("Selected nodes:", end="")
for i in range(N):
    if sol(x[i]) == 1:
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

colors = ["#e74c3c" if sol(x[i]) == 1 else "#d5dbdb" for i in range(N)]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color="#888888", width=1.2)
plt.title("Maximum Independent Set")
plt.savefig("mis.png", dpi=150, bbox_inches="tight")
plt.show()
```

Selected nodes are shown in red, and unselected nodes are shown in gray.
</div>

<div class="lang-ja" markdown="1">
# 最大独立集合（MIS）問題

無向グラフ $G=(V,E)$ の独立集合とは、$S$ 内のどの2頂点も $E$ の辺で接続されていないような頂点の部分集合 $S\subseteq V$ のことです。
最大独立集合（MIS）問題は、要素数が最大の独立集合を求める問題です。

MIS問題は以下のようにQUBOとして定式化できます。
$G$ が $0$ から $n-1$ までインデックス付けされた $n$ 個の頂点を持つとします。
$n$ 個のバイナリ変数 $x_i$ $(0\le i\le n-1)$ を導入し、$x_i=1$ であることと頂点 $i$ が $S$ に含まれることを同値とします。
$\|S\|=\sum_{i=0}^{n-1}x_i$ を最大化したいので、以下の目的関数を最小化します:

$$
\begin{aligned}
\text{objective} = -\sum_{i=0}^{n-1} x_i .
\end{aligned}
$$

独立性を保証するために、すべての辺 $(i,j)\in E$ に対して、両端点を同時に選択してはなりません。
これは以下のペナルティで表現できます:

$$
\begin{aligned}
\text{constraint} = \sum_{(i,j)\in E} x_i x_j .
\end{aligned}
$$

目的関数とペナルティを組み合わせると、以下のQUBO関数が得られます:

$$
\begin{aligned}
f = \text{objective} + 2\times\text{constraint}.
\end{aligned}
$$

ペナルティ係数 $2$ は、集合サイズの増加よりも実行可能性を優先するのに十分です。

## MIS問題のPyQBPPプログラム
上記のMIS問題のQUBO定式化に基づき、以下のPyQBPPプログラムは16ノードのインスタンスを解きます:
```python
import pyqbpp as qbpp

N = 16
edges = [
    (0, 1),  (0, 2),  (1, 3),  (1, 4),  (2, 5),  (2, 6),
    (3, 7),  (3, 13), (4, 6),  (4, 7),  (5, 8),  (6, 8),
    (6, 14), (7, 14), (8, 9),  (9, 10), (9, 12), (10, 11),
    (10, 12),(11, 13),(12, 14),(13, 15),(14, 15)]

x = qbpp.var("x", N)

objective = -qbpp.sum(x)
constraint = qbpp.expr()
for u, v in edges:
    constraint += x[u] * x[v]
f = objective + constraint * 2
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print(f"objective = {sol(objective)}")
print(f"constraint = {sol(constraint)}")

print("Selected nodes:", end="")
for i in range(N):
    if sol(x[i]) == 1:
        print(f" {i}", end="")
print()
```
`N = 16` 個のバイナリ変数のベクトル `x` に対して、上記のQUBO定式化に従って式 `objective`、`constraint`、`f` が構築されます。
次にExhaustive Solverを使用して `f` の最適解を求め、`sol` に格納します。`sol` における `objective` と `constraint` の評価値が表示されます。

このプログラムは以下の出力を生成します:
```
objective = -7
constraint = 0
```
これは、得られた解が7個のノードを選択し、すべての制約を満たしていることを意味します。

## matplotlibによる可視化
以下のコードは、`matplotlib` と `networkx` を使用してMISの解を可視化します:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(N))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

colors = ["#e74c3c" if sol(x[i]) == 1 else "#d5dbdb" for i in range(N)]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color="#888888", width=1.2)
plt.title("Maximum Independent Set")
plt.savefig("mis.png", dpi=150, bbox_inches="tight")
plt.show()
```

選択されたノードは赤色で、選択されていないノードは灰色で表示されます。
</div>
