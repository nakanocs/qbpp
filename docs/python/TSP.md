---
layout: default
nav_exclude: true
title: "TSP (PyQBPP)"
nav_order: 62
---
<div class="lang-en" markdown="1">
# Traveling Salesman Problem
The Traveling Salesman Problem (TSP) asks for the shortest tour that visits every node exactly once and returns to the start.
We assume that the nodes are placed on a plane and that the tour length is measured using the Euclidean distance.

<p align="center">
  <img src="../images/tsp_solution.svg" alt="An example of nodes and the TSP solution" width="50%">
</p>


## QUBO formulation of the TSP
A tour can be represented by a permutation of the nodes.
Accordingly, we use a [permutation matrix](PERMUTATION) to encode a TSP solution.

Let $X=(x_{i,j})$ ($0\leq i,j\leq n-1$) be a matrix of $n\times n$ binary values.
We interpret  $x_{k,i}$ as "the $k$-th position in the tour is node $i$".
Therefore, every row and every column of $X$ must be one-hot:

$$
\begin{aligned}
{\rm row}:& \sum_{j=0}^{n-1}x_{i,j}=1 & (0\leq i\leq n-1)\\
{\rm column}:& \sum_{i=0}^{n-1}x_{i,j}=1 & (0\leq j\leq n-1)
\end{aligned}
$$

Let $d_{i,j}$ denote the distance between nodes $i$ and $j$.
Then the tour length for a permutation matrix $X$ can be written as:

$$
\begin{aligned}
{\rm objective}: &\sum_{k=0}^{k-1} d_{i,j}x_{k,i}x_{(k+1)\bmod n,j}
\end{aligned}
$$

## PyQBPP program for TSP
```python
import math
import pyqbpp as qbpp

nodes = [(10, 12),  (33, 125),  (12, 226),
         (121, 11), (108, 142), (111, 243),
         (220, 4),  (210, 113), (211, 233)]

def dist(i, j):
    dx = nodes[i][0] - nodes[j][0]
    dy = nodes[i][1] - nodes[j][1]
    return round(math.sqrt(dx * dx + dy * dy))

n = len(nodes)
x = qbpp.var("x", n, n)

constraint = qbpp.sum(qbpp.vector_sum(x, 1) == 1) + qbpp.sum(qbpp.vector_sum(x, 0) == 1)

objective = qbpp.expr()
for i in range(n):
    next_i = (i + 1) % n
    for j in range(n):
        for k in range(n):
            if k != j:
                objective += dist(j, k) * x[i][j] * x[next_i][k]

f = objective + constraint * 1000
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

# Extract tour from permutation matrix
tour = []
for i in range(n):
    for j in range(n):
        if sol(x[i][j]) == 1:
            tour.append(j)
            break
print(f"Tour: {tour}")
```
In this program, the coordinates of nodes are stored in a list.
We create a 2D array `x` of binary variables and construct the one-hot constraints together with the tour-length objective.

This program produces the following output:
```
Tour: [7, 8, 5, 2, 4, 1, 0, 3, 6]
```

## Fixing the first node
Without loss of generality, we can assume that node 0 is the starting node of the tour.
By fixing the start node, we can reduce the number of binary variables in the QUBO expression.

```python
import pyqbpp as qbpp

ml = [(x[0][0], 1)]
ml += [(x[i][0], 0) for i in range(1, n)]
ml += [(x[0][i], 0) for i in range(1, n)]

g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.EasySolver(g)
solver.time_limit(1.0)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)

# Extract tour
tour = []
for i in range(n):
    for j in range(n):
        if full_sol(x[i][j]) == 1:
            tour.append(j)
            break
print(f"Tour: {tour}")
```
First, we create a list of pairs `ml`, which stores fixed assignments of variables.
Next, we call `replace(f, ml)`, which returns a new expression obtained by substituting the fixed values.
Since `sol` corresponds to the reduced problem, we create a `Sol` object for `f` and set both the solver output `sol` and the fixed assignments `ml`.

This program produces the following tour starting from node 0:
```
Tour: [0, 3, 6, 7, 8, 5, 2, 1, 4]
```

### Comparison with C++ QUBO++

| C++ QUBO++                        | PyQBPP                            |
|------------------------------------|------------------------------------|
| `qbpp::onehot_to_int(sol(x))`    | Manual loop over `sol(x[i][j])` |
| `qbpp::MapList ml;`              | `ml = [(x[0][0], 1)]`             |
| `ml.push_back({x[0][0], 1})`    | `ml.append((x[0][0], 1))`         |
| `qbpp::replace(f, ml)`          | `replace(f, ml)`                   |
| `qbpp::Sol(f).set(sol).set(ml)` | `Sol(f); full_sol.set(sol); full_sol.set(ml)` |

## Visualization using matplotlib
The following code visualizes the TSP solution:
```python
import matplotlib.pyplot as plt

plt.figure(figsize=(6, 6))
for i, (nx_, ny) in enumerate(nodes):
    plt.plot(nx_, ny, "ko", markersize=8)
    plt.annotate(str(i), (nx_, ny), textcoords="offset points", xytext=(5, 5))

for i in range(n):
    fr = tour[i]
    to = tour[(i + 1) % n]
    plt.annotate("", xy=(nodes[to][0], nodes[to][1]),
                 xytext=(nodes[fr][0], nodes[fr][1]),
                 arrowprops=dict(arrowstyle="->", color="#e74c3c", lw=2))
plt.title("TSP Tour")
plt.savefig("tsp.png", dpi=150, bbox_inches="tight")
plt.show()
```

The tour is shown as directed red arrows connecting the nodes.
</div>

<div class="lang-ja" markdown="1">
# 巡回セールスマン問題
巡回セールスマン問題（TSP）は、すべてのノードをちょうど1回ずつ訪問して出発点に戻る最短巡回路を求める問題です。
ノードは平面上に配置され、巡回路の長さはユークリッド距離で測定するものとします。

<p align="center">
  <img src="../images/tsp_solution.svg" alt="An example of nodes and the TSP solution" width="50%">
</p>


## TSPのQUBO定式化
巡回路はノードの順列で表現できます。
そこで、TSPの解を符号化するために[置換行列](PERMUTATION)を使用します。

$X=(x_{i,j})$（$0\leq i,j\leq n-1$）を $n\times n$ のバイナリ値の行列とします。
$x_{k,i}$ を「巡回路の $k$ 番目の位置がノード $i$ である」と解釈します。
したがって、$X$ の各行と各列はone-hotでなければなりません：

$$
\begin{aligned}
{\rm row}:& \sum_{j=0}^{n-1}x_{i,j}=1 & (0\leq i\leq n-1)\\
{\rm column}:& \sum_{i=0}^{n-1}x_{i,j}=1 & (0\leq j\leq n-1)
\end{aligned}
$$

$d_{i,j}$ をノード $i$ と $j$ の間の距離とします。
置換行列 $X$ に対する巡回路の長さは以下のように記述できます：

$$
\begin{aligned}
{\rm objective}: &\sum_{k=0}^{k-1} d_{i,j}x_{k,i}x_{(k+1)\bmod n,j}
\end{aligned}
$$

## TSPのPyQBPPプログラム
```python
import math
import pyqbpp as qbpp

nodes = [(10, 12),  (33, 125),  (12, 226),
         (121, 11), (108, 142), (111, 243),
         (220, 4),  (210, 113), (211, 233)]

def dist(i, j):
    dx = nodes[i][0] - nodes[j][0]
    dy = nodes[i][1] - nodes[j][1]
    return round(math.sqrt(dx * dx + dy * dy))

n = len(nodes)
x = qbpp.var("x", n, n)

constraint = qbpp.sum(qbpp.vector_sum(x, 1) == 1) + qbpp.sum(qbpp.vector_sum(x, 0) == 1)

objective = qbpp.expr()
for i in range(n):
    next_i = (i + 1) % n
    for j in range(n):
        for k in range(n):
            if k != j:
                objective += dist(j, k) * x[i][j] * x[next_i][k]

f = objective + constraint * 1000
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

# Extract tour from permutation matrix
tour = []
for i in range(n):
    for j in range(n):
        if sol(x[i][j]) == 1:
            tour.append(j)
            break
print(f"Tour: {tour}")
```
このプログラムでは、ノードの座標をリストに格納しています。
バイナリ変数の2次元配列 `x` を作成し、one-hot制約と巡回路長の目的関数を構築します。

このプログラムの出力は以下のとおりです：
```
Tour: [7, 8, 5, 2, 4, 1, 0, 3, 6]
```

## 最初のノードの固定
一般性を失うことなく、ノード0を巡回路の開始ノードとすることができます。
開始ノードを固定することで、QUBO式のバイナリ変数の数を削減できます。

```python
import pyqbpp as qbpp

ml = [(x[0][0], 1)]
ml += [(x[i][0], 0) for i in range(1, n)]
ml += [(x[0][i], 0) for i in range(1, n)]

g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.EasySolver(g)
solver.time_limit(1.0)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)

# Extract tour
tour = []
for i in range(n):
    for j in range(n):
        if full_sol(x[i][j]) == 1:
            tour.append(j)
            break
print(f"Tour: {tour}")
```
まず、変数の固定値を格納するペアのリスト `ml` を作成します。
次に `replace(f, ml)` を呼び出し、固定値を代入した新しい式を取得します。
`sol` は縮小された問題に対応するため、`f` に対する `Sol` オブジェクトを作成し、ソルバーの出力 `sol` と固定値 `ml` の両方を設定します。

このプログラムはノード0から始まる以下の巡回路を出力します：
```
Tour: [0, 3, 6, 7, 8, 5, 2, 1, 4]
```

### C++ QUBO++との比較

| C++ QUBO++                        | PyQBPP                            |
|------------------------------------|------------------------------------|
| `qbpp::onehot_to_int(sol(x))`    | `sol(x[i][j])` による手動ループ |
| `qbpp::MapList ml;`              | `ml = [(x[0][0], 1)]`             |
| `ml.push_back({x[0][0], 1})`    | `ml.append((x[0][0], 1))`         |
| `qbpp::replace(f, ml)`          | `replace(f, ml)`                   |
| `qbpp::Sol(f).set(sol).set(ml)` | `Sol(f); full_sol.set(sol); full_sol.set(ml)` |

## matplotlibによる可視化
以下のコードはTSPの解を可視化します：
```python
import matplotlib.pyplot as plt

plt.figure(figsize=(6, 6))
for i, (nx_, ny) in enumerate(nodes):
    plt.plot(nx_, ny, "ko", markersize=8)
    plt.annotate(str(i), (nx_, ny), textcoords="offset points", xytext=(5, 5))

for i in range(n):
    fr = tour[i]
    to = tour[(i + 1) % n]
    plt.annotate("", xy=(nodes[to][0], nodes[to][1]),
                 xytext=(nodes[fr][0], nodes[fr][1]),
                 arrowprops=dict(arrowstyle="->", color="#e74c3c", lw=2))
plt.title("TSP Tour")
plt.savefig("tsp.png", dpi=150, bbox_inches="tight")
plt.show()
```

巡回路はノードを結ぶ赤い有向矢印で表示されます。
</div>
