---
layout: default
nav_exclude: true
title: "N-Queens"
nav_order: 81
---
<div class="lang-en" markdown="1">
# N-Queens Problem
The **8-Queens problem** aims to place 8 queens on a chessboard so that no two queens attack each other; that is, no two queens share the same row, the same column, or the same diagonal (in either direction).
The **N-Queens problem** generalizes this: place
$N$ queens on an $N\times N$ chessboard under the same conditions.

To formulate this problem using PyQBPP, we use an $N\times N$ matrix $X=(x_{i,j})$ of binary variables, where
$x_{i,j}=1$ if a queen is placed at row $i$ and column $j$, and $x_{i,j}=0$ otherwise.
We impose the following constraints:
- Exactly one queen in each row:

$$
\begin{aligned}
\sum_{j=0}^{N-1} x_{i,j}&=1 && (0\leq i\leq N-1)
\end{aligned}
$$

- Exactly one queen in each column:

$$
\begin{aligned}
\sum_{i=0}^{N-1} x_{i,j}&=1 && (0\leq j\leq N-1)
\end{aligned}
$$

- At most one queen on each diagonal (from top-left to bottom-right):
A diagonal is characterized by $i+j=k$.
We consider only diagonals of length at least 2, i.e.,
$k=1,2,\ldots,2N−3$, and require:

$$
\begin{aligned}
0\leq \sum_{\substack{0\le i,j \le N-1\\ i+j=k}}x_{i,j}\leq 1 &&(1\leq k\leq 2N-3)
\end{aligned}
$$

- The sum of each anti-diagonal of $X$ is 0 or 1:
An anti-diagonal is characterized by $j−i=d$.
We consider only anti-diagonals of length at least 2, i.e.,
$d=−(N−2),\ldots,(N−2)$, and require:

$$
\begin{aligned}
0\leq \sum_{\substack{0\le i,j \le N-1\\ j-i=d}}x_{i,j}\leq 1 &&(-(N-2)\leq d\leq (N-2))
\end{aligned}
$$

## PyQBPP program
The following PyQBPP program constructs an expression representing the constraints above and then finds a feasible solution using the Easy Solver:
```python
import pyqbpp as qbpp

n = 8
x = qbpp.var("x", n, n)

f = qbpp.sum(qbpp.vector_sum(x, 0) == 1) + qbpp.sum(qbpp.vector_sum(x, 1) == 1)

m = 2 * n - 3
a = qbpp.expr(m)
b = qbpp.expr(m)

for i in range(m):
    k = i + 1
    for r in range(n):
        c = k - r
        if 0 <= c < n:
            a[i] += x[r][c]

    d = i - (n - 2)
    for r in range(n):
        c = r + d
        if 0 <= c < n:
            b[i] += x[r][c]

f += qbpp.sum(qbpp.between(a, 0, 1))
f += qbpp.sum(qbpp.between(b, 0, 1))

f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(0)
sol = solver.search()
for i in range(n):
    for j in range(n):
        print("Q" if sol(x[i][j]) == 1 else ".", end="")
    print()
```
An `n`$\times$`n` matrix `x` of binary variables is introduced, where `x[i][j] = 1` indicates that a queen is placed at row `i` and column `j`.
The column-wise sums are computed using `vector_sum(x, 0)`, which returns a vector of `n` expressions (one per column).
Applying the `==` operator element-wise produces a vector of penalty expressions; each expression evaluates to 0 if and only if the corresponding column sum equals 1.
Similarly, we can enforce the row-wise one-hot constraints using `vector_sum(x, 1)`.

To enforce diagonal constraints, we build two vectors of expressions, `a` and `b`, each of length `m = 2*n - 3`.
For each index `i`, `a[i]` accumulates variables on a diagonal with a fixed value of `r + c` (diagonals from top-left to bottom-right), excluding diagonals of length 1.
Similarly, `b[i]` accumulates variables on an anti-diagonal with a fixed value of `c - r` (diagonals from top-right to bottom-left), again excluding diagonals of length 1.
The range constraint `between(a, 0, 1)` (and similarly for `b`) is applied element-wise and produces penalties that become 0 if and only if each diagonal/anti-diagonal contains at most one queen.
These penalties are added to `f`.

After converting the expression into a binary QUBO form with `f.simplify_as_binary()`, the Easy Solver searches for a solution with target energy 0.
The resulting assignment `sol` is then printed as an 8-by-8 board, where `Q` denotes a queen and `.` denotes an empty square.
For example, the program may produce the following output:
```
..Q.....
.....Q..
.......Q
.Q......
...Q....
Q.......
......Q.
....Q...
```
This output confirms a valid placement of eight queens, since no two queens share the same row, column, diagonal, or anti-diagonal.
</div>

<div class="lang-ja" markdown="1">
# N-Queens 問題
**8-Queens 問題**は、チェス盤上に8つのクイーンを、互いに攻撃し合わないように配置する問題です。すなわち、同じ行、同じ列、同じ対角線（どちらの方向も）を共有するクイーンが存在しないようにします。
**N-Queens 問題**はこれを一般化したもので、$N\times N$ のチェス盤上に同じ条件のもとで $N$ 個のクイーンを配置します。

この問題を PyQBPP で定式化するために、$N\times N$ のバイナリ変数の行列 $X=(x_{i,j})$ を用います。ここで、$x_{i,j}=1$ は行 $i$、列 $j$ にクイーンが配置されていることを表し、$x_{i,j}=0$ はそうでないことを表します。
以下の制約を課します：
- 各行にちょうど1つのクイーン：

$$
\begin{aligned}
\sum_{j=0}^{N-1} x_{i,j}&=1 && (0\leq i\leq N-1)
\end{aligned}
$$

- 各列にちょうど1つのクイーン：

$$
\begin{aligned}
\sum_{i=0}^{N-1} x_{i,j}&=1 && (0\leq j\leq N-1)
\end{aligned}
$$

- 各対角線（左上から右下）に最大1つのクイーン：
対角線は $i+j=k$ で特徴づけられます。
長さ2以上の対角線のみを考え、すなわち $k=1,2,\ldots,2N−3$ に対して以下を要求します：

$$
\begin{aligned}
0\leq \sum_{\substack{0\le i,j \le N-1\\ i+j=k}}x_{i,j}\leq 1 &&(1\leq k\leq 2N-3)
\end{aligned}
$$

- $X$ の各反対角線の和が0または1：
反対角線は $j−i=d$ で特徴づけられます。
長さ2以上の反対角線のみを考え、すなわち $d=−(N−2),\ldots,(N−2)$ に対して以下を要求します：

$$
\begin{aligned}
0\leq \sum_{\substack{0\le i,j \le N-1\\ j-i=d}}x_{i,j}\leq 1 &&(-(N-2)\leq d\leq (N-2))
\end{aligned}
$$

## PyQBPP プログラム
以下の PyQBPP プログラムは、上記の制約を表す式を構築し、Easy Solver を用いて実行可能解を求めます：
```python
import pyqbpp as qbpp

n = 8
x = qbpp.var("x", n, n)

f = qbpp.sum(qbpp.vector_sum(x, 0) == 1) + qbpp.sum(qbpp.vector_sum(x, 1) == 1)

m = 2 * n - 3
a = qbpp.expr(m)
b = qbpp.expr(m)

for i in range(m):
    k = i + 1
    for r in range(n):
        c = k - r
        if 0 <= c < n:
            a[i] += x[r][c]

    d = i - (n - 2)
    for r in range(n):
        c = r + d
        if 0 <= c < n:
            b[i] += x[r][c]

f += qbpp.sum(qbpp.between(a, 0, 1))
f += qbpp.sum(qbpp.between(b, 0, 1))

f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(0)
sol = solver.search()
for i in range(n):
    for j in range(n):
        print("Q" if sol(x[i][j]) == 1 else ".", end="")
    print()
```
`n`$\times$`n` のバイナリ変数の行列 `x` を導入し、`x[i][j] = 1` は行 `i`、列 `j` にクイーンが配置されていることを示します。
列方向の和は `vector_sum(x, 0)` を用いて計算され、`n` 個の式のベクトル（各列に1つ）を返します。
`==` 演算子を要素ごとに適用すると、ペナルティ式のベクトルが生成されます。各式は、対応する列の和が1に等しい場合にのみ0と評価されます。
同様に、`vector_sum(x, 1)` を用いて行方向の one-hot 制約を課すことができます。

対角線制約を課すために、長さ `m = 2*n - 3` の2つの式ベクトル `a` と `b` を構築します。
各インデックス `i` について、`a[i]` は `r + c` の値が固定された対角線（左上から右下）上の変数を累積します（長さ1の対角線は除外）。
同様に、`b[i]` は `c - r` の値が固定された反対角線（右上から左下）上の変数を累積します（長さ1の対角線は除外）。
範囲制約 `between(a, 0, 1)`（`b` も同様）は要素ごとに適用され、各対角線/反対角線に最大1つのクイーンが含まれる場合にのみ0となるペナルティを生成します。
これらのペナルティは `f` に加算されます。

`f.simplify_as_binary()` で式をバイナリ QUBO 形式に変換した後、Easy Solver がターゲットエネルギー0の解を探索します。
得られた割り当て `sol` は 8x8 の盤面として出力され、`Q` はクイーン、`.` は空のマスを表します。
例えば、プログラムは以下のような出力を生成する場合があります：
```
..Q.....
.....Q..
.......Q
.Q......
...Q....
Q.......
......Q.
....Q...
```
この出力は、同じ行、列、対角線、反対角線を共有するクイーンが存在しないため、8つのクイーンの有効な配置であることを確認しています。
</div>
