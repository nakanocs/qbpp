---
layout: default
nav_exclude: true
title: "Magic Square (PyQBPP)"
nav_order: 80
---
<div class="lang-en" markdown="1">
# Magic Square
A 3-by-3 magic square is a 3-by-3 matrix that contains each integer from 1 to 9 exactly once, such that the sum of every row, every column, and the two diagonals is 15.
An example is shown below:
```
8 1 6
3 5 7
4 9 2
```

## A formulation for finding magic square
We formulate the problem of finding a 3-by-3 magic square
$S=(s_{i,j})$ ($0\leq i,j\leq 2$) using one-hot encoding.
We introduce binary variables $x_{i,j,k}$ ($0\leq i,j\leq 2, 0\leq k\leq 8$), where:

$$
\begin{aligned}
x_{i,j,k}=1 &\Longleftrightarrow & s_{i,j}=k+1
\end{aligned}
$$

Thus, $X=(x_{i,j,k})$ is a $3\times 3\times 9$ binary array.
We impose the following four constraints.

1. One-hot constraint (one value per cell):
For each cell $(i,j)$, exactly one of $x_{i,j,0}, x_{i,j,1}, \ldots,x _{i,j,8}$ must be 1:

$$
\begin{aligned}
c_1(i,j): & \sum_{k=0}^8 x _{i,j,k}=1 & (0\leq i,j\leq 2)
\end{aligned}
$$

2. Each value $k+1$ must appear in exactly one cell:

$$
\begin{aligned}
c_2(k): & \sum_{i=0}^2\sum_{j=0}^2x _{i,j,k}=1 & (0\leq k\leq 8)
\end{aligned}
$$

3. The sum of each row and each column must be 15:
$$
\begin{aligned}
c_3(i): & \sum_{j=0}^2\sum_{k=0}^8  (k+1)x _{i,j,k} = 15  &(0\leq i\leq 2)\\
c_3(j): & \sum_{i=0}^2\sum_{k=0}^8 (k+1)x _{i,j,k} = 15 &(0\leq j\leq 2)
\end{aligned}
$$

4. The sums of diagonal and anti-diagonal
The two diagonal sums must also be 15:
$$
\begin{aligned}
c_4: &  \sum_{k=0}^8 (k+1) (x_{0,0,k}+x_{1,1,k}+x_{2,2,k}) = 15 \\
c_4:  & \sum_{k=0}^8 (k+1) (x_{0,2,k}+x_{1,1,k}+x_{2,0,k}) = 15
\end{aligned}
$$

When all constraints are satisfied, the assignment $X=(x_{i,j,k})$ represents a valid 3-by-3 magic square.

## PyQBPP program for the magic square
The following PyQBPP program implements these constraints and finds a magic square:
```python
from pyqbpp import var, sum, vector_sum, toExpr, expr, EasySolver

x = var("x", 3, 3, 9)

c1 = sum(vector_sum(x) == 1)

temp = expr(9)
for i in range(3):
    for j in range(3):
        for k in range(9):
            temp[k] += x[i][j][k]
c2 = sum(temp == 1)

row = expr(3)
column = expr(3)
for i in range(3):
    for j in range(3):
        for k in range(9):
            row[i] += (k + 1) * x[i][j][k]
            column[j] += (k + 1) * x[i][j][k]
c3 = sum(row == 15) + sum(column == 15)

diag = toExpr(0)
for k in range(9):
    diag += (k + 1) * (x[0][0][k] + x[1][1][k] + x[2][2][k])
anti_diag = toExpr(0)
for k in range(9):
    anti_diag += (k + 1) * (x[0][2][k] + x[1][1][k] + x[2][0][k])
c4 = (diag == 15) + (anti_diag == 15)

f = c1 + c2 + c3 + c4
f.simplify_as_binary()

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()
for i in range(3):
    for j in range(3):
        val = next(k for k in range(9) if sol(x[i][j][k]) == 1)
        print(val + 1, end=" ")
    print()
```
In this program, we define a $3\times 3\times9$ array of binary variables `x`.
We then build four constraint expressions `c1`, `c2`, `c3`, and `c4`, and combine them into `f`.
The expression `f` achieves the minimum energy 0 when all constraints are satisfied.

We create an Easy Solver object `solver` for `f` and set the target energy to 0, so the search terminates as soon as a feasible (optimal) solution is found.
The resulting one-hot encoding is decoded by finding the index `k` for which `sol(x[i][j][k]) == 1`.

This program produces the following output:
```
8 1 6
3 5 7
4 9 2
```

## Fixing variables partially
Suppose we want to find a solution in which the top-left cell is assigned the value 2.
In the one-hot encoding, the value 2 corresponds to $k=1$, so we fix

$$
\begin{aligned}
 x_{0,0,k} &=1 & {\rm if\,\,} k=1\\
 x_{0,0,k} &=0 & {\rm if\,\,} k\neq 1
\end{aligned}
$$

Moreover, since constraint $c_2$ enforces that each number $k+1$ appears exactly once, fixing
immediately implies that no other cell can take the value 2.
Therefore, we can also fix:

$$
\begin{aligned}
 x_{i,j,1} &=0 & {\rm if\,\,} (i,j)\neq (0,0)\\
\end{aligned}
$$

These fixed assignments reduce the number of remaining binary variables, which is often beneficial for local-search-based solvers.

## PyQBPP program for the magic square with fixing variables partially
We modify the program above as follows:
```python
from pyqbpp import var, sum, vector_sum, toExpr, expr, replace, MapList, Sol, EasySolver

x = var("x", 3, 3, 9)

c1 = sum(vector_sum(x) == 1)

temp = expr(9)
for i in range(3):
    for j in range(3):
        for k in range(9):
            temp[k] += x[i][j][k]
c2 = sum(temp == 1)

row = expr(3)
column = expr(3)
for i in range(3):
    for j in range(3):
        for k in range(9):
            row[i] += (k + 1) * x[i][j][k]
            column[j] += (k + 1) * x[i][j][k]
c3 = sum(row == 15) + sum(column == 15)

diag = toExpr(0)
for k in range(9):
    diag += (k + 1) * (x[0][0][k] + x[1][1][k] + x[2][2][k])
anti_diag = toExpr(0)
for k in range(9):
    anti_diag += (k + 1) * (x[0][2][k] + x[1][1][k] + x[2][0][k])
c4 = (diag == 15) + (anti_diag == 15)

f = c1 + c2 + c3 + c4
f.simplify_as_binary()

ml = MapList()
for k in range(9):
    if k == 1:
        ml.add(x[0][0][k], 1)
    else:
        ml.add(x[0][0][k], 0)

for i in range(3):
    for j in range(3):
        if not (i == 0 and j == 0):
            ml.add(x[i][j][1], 0)

full_sol = Sol(f)
g = replace(f, ml)
g.simplify_as_binary()

solver = EasySolver(g)
solver.target_energy(0)
sol = solver.search()
full_sol.set(sol)
full_sol.set(ml)

for i in range(3):
    for j in range(3):
        val = next(k for k in range(9) if full_sol(x[i][j][k]) == 1)
        print(val + 1, end=" ")
    print()
```

In this code, we create a `MapList` object `ml` and add fixed assignments using `add()`.
We then create `full_sol`, a solution object for the original expression `f`.
Calling `replace(f, ml)` substitutes the fixed values into `f`, so the variables listed in `ml` disappear from `g`.
As a result, the solution `sol` returned by the solver does not include those fixed variables.
Finally, we reconstruct a complete assignment by merging `sol` and `ml` into `full_sol` via `set()`.
The reconstructed solution `full_sol` represents the full magic square.

This program produces the following output:
```
2 7 6
9 5 1
4 3 8
```
We can confirm that the top-left cell is 2, as intended.
</div>

<div class="lang-ja" markdown="1">
# 魔方陣
3x3の魔方陣とは、1から9までの各整数をちょうど1回ずつ含み、すべての行、列、および2つの対角線の和が15となる3x3の行列です。
以下に例を示します：
```
8 1 6
3 5 7
4 9 2
```

## 魔方陣を求めるための定式化
one-hotエンコーディングを用いて、3x3の魔方陣 $S=(s_{i,j})$（$0\leq i,j\leq 2$）を求める問題を定式化します。
バイナリ変数 $x_{i,j,k}$（$0\leq i,j\leq 2, 0\leq k\leq 8$）を導入します：

$$
\begin{aligned}
x_{i,j,k}=1 &\Longleftrightarrow & s_{i,j}=k+1
\end{aligned}
$$

したがって、$X=(x_{i,j,k})$ は $3\times 3\times 9$ のバイナリ配列です。
以下の4つの制約を課します。

1. one-hot制約（各セルに1つの値）：
各セル $(i,j)$ について、$x_{i,j,0}, x_{i,j,1}, \ldots,x _{i,j,8}$ のうちちょうど1つが1でなければなりません：

$$
\begin{aligned}
c_1(i,j): & \sum_{k=0}^8 x _{i,j,k}=1 & (0\leq i,j\leq 2)
\end{aligned}
$$

2. 各値 $k+1$ はちょうど1つのセルに現れなければなりません：

$$
\begin{aligned}
c_2(k): & \sum_{i=0}^2\sum_{j=0}^2x _{i,j,k}=1 & (0\leq k\leq 8)
\end{aligned}
$$

3. 各行と各列の和は15でなければなりません：
$$
\begin{aligned}
c_3(i): & \sum_{j=0}^2\sum_{k=0}^8  (k+1)x _{i,j,k} = 15  &(0\leq i\leq 2)\\
c_3(j): & \sum_{i=0}^2\sum_{k=0}^8 (k+1)x _{i,j,k} = 15 &(0\leq j\leq 2)
\end{aligned}
$$

4. 対角線と反対角線の和
2つの対角線の和も15でなければなりません：
$$
\begin{aligned}
c_4: &  \sum_{k=0}^8 (k+1) (x_{0,0,k}+x_{1,1,k}+x_{2,2,k}) = 15 \\
c_4:  & \sum_{k=0}^8 (k+1) (x_{0,2,k}+x_{1,1,k}+x_{2,0,k}) = 15
\end{aligned}
$$

すべての制約が満たされたとき、割り当て $X=(x_{i,j,k})$ は有効な3x3の魔方陣を表します。

## 魔方陣のPyQBPPプログラム
以下のPyQBPPプログラムはこれらの制約を実装し、魔方陣を求めます：
```python
from pyqbpp import var, sum, vector_sum, toExpr, expr, EasySolver

x = var("x", 3, 3, 9)

c1 = sum(vector_sum(x) == 1)

temp = expr(9)
for i in range(3):
    for j in range(3):
        for k in range(9):
            temp[k] += x[i][j][k]
c2 = sum(temp == 1)

row = expr(3)
column = expr(3)
for i in range(3):
    for j in range(3):
        for k in range(9):
            row[i] += (k + 1) * x[i][j][k]
            column[j] += (k + 1) * x[i][j][k]
c3 = sum(row == 15) + sum(column == 15)

diag = toExpr(0)
for k in range(9):
    diag += (k + 1) * (x[0][0][k] + x[1][1][k] + x[2][2][k])
anti_diag = toExpr(0)
for k in range(9):
    anti_diag += (k + 1) * (x[0][2][k] + x[1][1][k] + x[2][0][k])
c4 = (diag == 15) + (anti_diag == 15)

f = c1 + c2 + c3 + c4
f.simplify_as_binary()

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()
for i in range(3):
    for j in range(3):
        val = next(k for k in range(9) if sol(x[i][j][k]) == 1)
        print(val + 1, end=" ")
    print()
```
このプログラムでは、$3\times 3\times9$ のバイナリ変数配列 `x` を定義します。
次に、4つの制約式 `c1`、`c2`、`c3`、`c4` を構築し、それらを `f` にまとめます。
式 `f` はすべての制約が満たされたとき最小エネルギー0を達成します。

`f` に対するEasy Solverオブジェクト `solver` を作成し、target energyを0に設定します。これにより、実行可能（最適）解が見つかり次第、探索が終了します。
得られたone-hotエンコーディングは、`sol(x[i][j][k]) == 1` となるインデックス `k` を見つけることでデコードされます。

このプログラムの出力は以下の通りです：
```
8 1 6
3 5 7
4 9 2
```

## 変数の部分的固定
左上のセルに値2を割り当てた解を求めたいとします。
one-hotエンコーディングでは、値2は $k=1$ に対応するため、以下を固定します：

$$
\begin{aligned}
 x_{0,0,k} &=1 & {\rm if\,\,} k=1\\
 x_{0,0,k} &=0 & {\rm if\,\,} k\neq 1
\end{aligned}
$$

さらに、制約 $c_2$ が各数 $k+1$ がちょうど1回現れることを強制するため、この固定は他のどのセルも値2を取れないことを直ちに意味します。
したがって、以下も固定できます：

$$
\begin{aligned}
 x_{i,j,1} &=0 & {\rm if\,\,} (i,j)\neq (0,0)\\
\end{aligned}
$$

これらの固定された割り当ては残りのバイナリ変数の数を減らし、局所探索ベースのソルバーにとって有益な場合が多いです。

## 変数を部分的に固定した魔方陣のPyQBPPプログラム
上記のプログラムを以下のように修正します：
```python
from pyqbpp import var, sum, vector_sum, toExpr, expr, replace, MapList, Sol, EasySolver

x = var("x", 3, 3, 9)

c1 = sum(vector_sum(x) == 1)

temp = expr(9)
for i in range(3):
    for j in range(3):
        for k in range(9):
            temp[k] += x[i][j][k]
c2 = sum(temp == 1)

row = expr(3)
column = expr(3)
for i in range(3):
    for j in range(3):
        for k in range(9):
            row[i] += (k + 1) * x[i][j][k]
            column[j] += (k + 1) * x[i][j][k]
c3 = sum(row == 15) + sum(column == 15)

diag = toExpr(0)
for k in range(9):
    diag += (k + 1) * (x[0][0][k] + x[1][1][k] + x[2][2][k])
anti_diag = toExpr(0)
for k in range(9):
    anti_diag += (k + 1) * (x[0][2][k] + x[1][1][k] + x[2][0][k])
c4 = (diag == 15) + (anti_diag == 15)

f = c1 + c2 + c3 + c4
f.simplify_as_binary()

ml = MapList()
for k in range(9):
    if k == 1:
        ml.add(x[0][0][k], 1)
    else:
        ml.add(x[0][0][k], 0)

for i in range(3):
    for j in range(3):
        if not (i == 0 and j == 0):
            ml.add(x[i][j][1], 0)

full_sol = Sol(f)
g = replace(f, ml)
g.simplify_as_binary()

solver = EasySolver(g)
solver.target_energy(0)
sol = solver.search()
full_sol.set(sol)
full_sol.set(ml)

for i in range(3):
    for j in range(3):
        val = next(k for k in range(9) if full_sol(x[i][j][k]) == 1)
        print(val + 1, end=" ")
    print()
```

このコードでは、`MapList` オブジェクト `ml` を作成し、`add()` で固定された割り当てを追加します。
次に、元の式 `f` に対する解オブジェクト `full_sol` を作成します。
`replace(f, ml)` を呼び出すと、固定された値が `f` に代入され、`ml` に含まれる変数は `g` から消えます。
その結果、ソルバーが返す解 `sol` にはそれらの固定された変数が含まれません。
最後に、`set()` を使って `sol` と `ml` を `full_sol` にマージすることで、完全な割り当てを再構築します。
再構築された解 `full_sol` は完全な魔方陣を表します。

このプログラムの出力は以下の通りです：
```
2 7 6
9 5 1
4 3 8
```
意図通り、左上のセルが2であることを確認できます。
</div>
