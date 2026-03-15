---
layout: default
nav_exclude: true
title: "Graph Coloring"
nav_order: 59
---
<div class="lang-en" markdown="1">
# Graph Coloring Problem
Given an undirected graph $G=(V,E)$, the **graph coloring problem** aims to assign a color to each node so that adjacent nodes receive different colors.
More specifically, for a set $C$ of colors, the goal is to find an assignment $\sigma:V\rightarrow C$ such that for every edge $(u,v)\in E$, we have $\sigma(u)\neq \sigma(v)$.

Let $V=\lbrace 0,1,\ldots ,n−1\rbrace$ and $C=\lbrace 0,1,\ldots ,m−1\rbrace$.
We introduce an $n\times m$ matrix $X=(x_{i,j})$ of binary variables, where $x_{i,j}=1$ if and only if node $i$ is assigned color $j$.

### One-hot constraint
Since exactly one color must be assigned to each node, each row of
$X$ must be one-hot:

$$
\begin{aligned}
  \text{onehot}&= \sum_{i=0}^{n-1}\Bigl(\sum_{j=0}^{m-1}x_{i,j}==1\Bigr)
\end{aligned}
$$

### Adjacent nodes must differ
For each edge, its endpoints must not share the same color:

$$
\begin{aligned}
  \text{different}&= \sum_{(u,v)\in E}\sum_{j=0}^{m-1}x_{u,j}x_{v,j}
\end{aligned}
$$

## QUBO objective

$$
\begin{aligned}
  f &= \text{onehot}+\text{different}
\end{aligned}
$$

## PyQBPP program
Since any planar graph can be colored with at most four colors, we use a planar graph with 16 nodes and $m=4$ colors as an example:
```python
from pyqbpp import var, sum, Expr, vector_sum, EasySolver

n = 16
edges = [
    (0, 1),  (0, 2),  (0, 4),  (1, 3),  (1, 4),  (1, 7),  (2, 5),
    (2, 6),  (3, 7),  (3, 13), (3, 15), (4, 6),  (4, 7),  (4, 14),
    (5, 8),  (6, 8),  (6, 14), (7, 14), (7, 15), (8, 9),  (8, 12),
    (9, 10), (9, 11), (9, 12), (10, 11),(10, 12),(10, 13),(10, 14),
    (10, 15),(11, 13),(12, 14),(13, 15),(14, 15)]
m = 4

x = var("x", n, m)

onehot = sum(vector_sum(x) == 1)
different = Expr(0)
for u, v in edges:
    different += sum(x[u] * x[v])

f = onehot + different

f.simplify_as_binary()
solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print(f"onehot = {sol.eval(onehot)}")
print(f"different = {sol.eval(different)}")

# Extract node colors
for i in range(n):
    for j in range(m):
        if sol.get(x[i][j]) == 1:
            print(f"Node {i}: color {j}")
            break
```
In this program, we first define an $n\times m$ matrix `x` of binary variables, and then construct the expressions `onehot`, `different`, and `f`.
We solve the resulting QUBO using the Easy Solver with target energy 0.

### Result for $m=4$
This program produces the following output:
```
onehot = 0
different = 0
```
Therefore, a valid 4-coloring is found.

### Result for $m=3$
When running with $m=3$, the program produces:
```
onehot = 1
different = 0
```
This output indicates that the solver failed to assign a color to exactly one node (i.e., one row is not one-hot).

## Visualization using matplotlib
The following code visualizes the Graph Coloring solution:
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(n))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

palette = ["#e74c3c", "#3498db", "#2ecc71", "#f39c12", "#9b59b6", "#1abc9c"]
node_color = [next((k for k in range(m) if sol.get(x[i][k]) == 1), 0) for i in range(n)]
colors = [palette[c % len(palette)] for c in node_color]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color="#888888", width=1.2)
plt.title(f"Graph Coloring ({m} colors)")
plt.savefig("graph_color.png", dpi=150, bbox_inches="tight")
plt.show()
```

Each node is colored according to its assigned color.
</div>

<div class="lang-ja" markdown="1">
# グラフ彩色問題
無向グラフ $G=(V,E)$ が与えられたとき、**グラフ彩色問題**は、隣接するノードが異なる色を持つように各ノードに色を割り当てることを目的とします。
より具体的には、色の集合 $C$ に対して、すべての辺 $(u,v)\in E$ について $\sigma(u)\neq \sigma(v)$ を満たす割り当て $\sigma:V\rightarrow C$ を求めます。

$V=\lbrace 0,1,\ldots ,n−1\rbrace$、$C=\lbrace 0,1,\ldots ,m−1\rbrace$ とします。
$n\times m$ のバイナリ変数の行列 $X=(x_{i,j})$ を導入し、$x_{i,j}=1$ はノード $i$ に色 $j$ が割り当てられている場合にのみ成り立ちます。

### One-hot 制約
各ノードにちょうど1つの色が割り当てられなければならないため、$X$ の各行は one-hot でなければなりません：

$$
\begin{aligned}
  \text{onehot}&= \sum_{i=0}^{n-1}\Bigl(\sum_{j=0}^{m-1}x_{i,j}==1\Bigr)
\end{aligned}
$$

### 隣接ノードは異なる色
各辺について、その両端点は同じ色を共有してはなりません：

$$
\begin{aligned}
  \text{different}&= \sum_{(u,v)\in E}\sum_{j=0}^{m-1}x_{u,j}x_{v,j}
\end{aligned}
$$

## QUBO 目的関数

$$
\begin{aligned}
  f &= \text{onehot}+\text{different}
\end{aligned}
$$

## PyQBPP プログラム
任意の平面グラフは最大4色で彩色可能であるため、16ノードの平面グラフと $m=4$ 色を例として使用します：
```python
from pyqbpp import var, sum, Expr, vector_sum, EasySolver

n = 16
edges = [
    (0, 1),  (0, 2),  (0, 4),  (1, 3),  (1, 4),  (1, 7),  (2, 5),
    (2, 6),  (3, 7),  (3, 13), (3, 15), (4, 6),  (4, 7),  (4, 14),
    (5, 8),  (6, 8),  (6, 14), (7, 14), (7, 15), (8, 9),  (8, 12),
    (9, 10), (9, 11), (9, 12), (10, 11),(10, 12),(10, 13),(10, 14),
    (10, 15),(11, 13),(12, 14),(13, 15),(14, 15)]
m = 4

x = var("x", n, m)

onehot = sum(vector_sum(x) == 1)
different = Expr(0)
for u, v in edges:
    different += sum(x[u] * x[v])

f = onehot + different

f.simplify_as_binary()
solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print(f"onehot = {sol.eval(onehot)}")
print(f"different = {sol.eval(different)}")

# Extract node colors
for i in range(n):
    for j in range(m):
        if sol.get(x[i][j]) == 1:
            print(f"Node {i}: color {j}")
            break
```
このプログラムでは、まず $n\times m$ のバイナリ変数の行列 `x` を定義し、次に式 `onehot`、`different`、`f` を構築します。
得られた QUBO を、ターゲットエネルギー0で Easy Solver を用いて解きます。

### $m=4$ の場合の結果
このプログラムは以下の出力を生成します：
```
onehot = 0
different = 0
```
したがって、有効な4彩色が見つかりました。

### $m=3$ の場合の結果
$m=3$ で実行すると、プログラムは以下の出力を生成します：
```
onehot = 1
different = 0
```
この出力は、ソルバーがちょうど1つのノードに色を割り当てることに失敗したことを示しています（すなわち、1つの行が one-hot ではありません）。

## matplotlib による可視化
以下のコードは、グラフ彩色の解を可視化します：
```python
import matplotlib.pyplot as plt
import networkx as nx

G = nx.Graph()
G.add_nodes_from(range(n))
G.add_edges_from(edges)
pos = nx.spring_layout(G, seed=42)

palette = ["#e74c3c", "#3498db", "#2ecc71", "#f39c12", "#9b59b6", "#1abc9c"]
node_color = [next((k for k in range(m) if sol.get(x[i][k]) == 1), 0) for i in range(n)]
colors = [palette[c % len(palette)] for c in node_color]
nx.draw(G, pos, with_labels=True, node_color=colors, node_size=400,
        font_size=9, edge_color="#888888", width=1.2)
plt.title(f"Graph Coloring ({m} colors)")
plt.savefig("graph_color.png", dpi=150, bbox_inches="tight")
plt.show()
```

各ノードは割り当てられた色に従って彩色されます。
</div>
