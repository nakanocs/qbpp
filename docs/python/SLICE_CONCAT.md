---
layout: default
nav_exclude: true
title: "Slice and Concat"
nav_order: 19
---
<div class="lang-en" markdown="1">
# Slice and Concat

PyQBPP supports Python-style slicing and a `concat()` function for manipulating vectors.
This page demonstrates these operations through **domain wall encoding** and the **Dual-Matrix Domain Wall** method.

## Slicing

PyQBPP vectors support standard Python slice notation. Slicing returns a new `Vector`:

```python
import pyqbpp as qbpp

x = qbpp.var("x", 8)
print(x[:3])     # first 3:  [x[0], x[1], x[2]]
print(x[-3:])    # last 3:   [x[5], x[6], x[7]]
print(x[2:5])    # range:    [x[2], x[3], x[4]]
```

For multi-dimensional vectors, use tuple indexing (similar to NumPy):

```python
x = qbpp.var("x", 3, 5)
print(x[:, :3])    # first 3 columns of each row
print(x[:, -2:])   # last 2 columns of each row
print(x[1:3, 1:4]) # rows 1-2, columns 1-3

x = qbpp.var("x", 2, 3, 4)
print(x[:, :, :2]) # first 2 elements along the 3rd dimension
```

## Concat

The `concat()` function joins vectors or prepends/appends scalars:

```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)

# 1D: scalar + vector, vector + scalar
y = qbpp.concat(1, qbpp.concat(x, 0))
# y = [1, x[0], x[1], x[2], x[3], 0]

# 2D with dim parameter
z = qbpp.var("z", 3, 4)

# dim=0: prepend/append a row filled with scalar
zg0 = qbpp.concat(1, qbpp.concat(z, 0, 0), 0)
# adds row of 1s at top, row of 0s at bottom -> 5 x 4

# dim=1: prepend/append scalar to each row
zg1 = qbpp.concat(1, qbpp.concat(z, 0, 1), 1)
# adds 1 at start and 0 at end of each row -> 3 x 6
```

## Domain Wall Encoding

A **domain wall** is a binary pattern $1\cdots 1\, 0\cdots 0$.
For $n$ variables, there are $n+1$ such patterns, representing integers $0$ through $n$.

```python
import pyqbpp as qbpp

n = 8
x = qbpp.var("x", n)

# y = (1, x[0], ..., x[n-1], 0)
y = qbpp.concat(1, qbpp.concat(x, 0))

# Adjacent difference
diff = y[:n+1] - y[-(n+1):]

# Penalty: minimum 1 iff domain wall
f = qbpp.sum(qbpp.sqr(diff))
f.simplify_as_binary()

print("f =", f)

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search_optimal_solutions()

print("energy =", sol.energy())
print("solutions =", len(sol.all_solutions()))
for s in sol.all_solutions():
    bits = "".join(str(s(x[i])) for i in range(n))
    print(f"  {bits}  (sum = {s(qbpp.sum(x))})")
```

### Output

```
f = 1 +2*x[1] +2*x[2] +2*x[3] +2*x[4] +2*x[5] +2*x[6] +2*x[7] -2*x[0]*x[1] -2*x[1]*x[2] -2*x[2]*x[3] -2*x[3]*x[4] -2*x[4]*x[5] -2*x[5]*x[6] -2*x[6]*x[7]
energy = 1
solutions = 9
  00000000  (sum = 0)
  10000000  (sum = 1)
  11000000  (sum = 2)
  11100000  (sum = 3)
  11110000  (sum = 4)
  11111000  (sum = 5)
  11111100  (sum = 6)
  11111110  (sum = 7)
  11111111  (sum = 8)
```

## Dual-Matrix Domain Wall

The **Dual-Matrix Domain Wall** method constructs an $n \times n$ permutation matrix
using two binary matrices: `x` of size $(n{-}1) \times n$ and `y` of size $n \times (n{-}1)$.
Adding guard bits and taking adjacent differences produces two $n \times n$ one-hot matrices.
Matching them yields a permutation matrix — without explicit loops.
For details, see: [https://arxiv.org/abs/2308.01024](https://arxiv.org/abs/2308.01024)

```python
import pyqbpp as qbpp

n = 6
x = qbpp.var("x", n - 1, n)  # (n-1) x n
y = qbpp.var("y", n, n - 1)  # n x (n-1)

# x: guard rows (dim=0), diff -> n x n (column one-hot)
xg = qbpp.concat(1, qbpp.concat(x, 0, 0), 0)
x_oh = xg[:n] - xg[-n:]
x_dw = qbpp.sum(qbpp.sqr(x_oh))

# y: guard cols (dim=1), diff -> n x n (row one-hot)
yg = qbpp.concat(1, qbpp.concat(y, 0, 1), 1)
y_oh = yg[:, :n] - yg[:, -n:]
y_dw = qbpp.sum(qbpp.sqr(y_oh))

# Match: x_oh == y_oh
match = qbpp.sum(x_oh - y_oh == 0)

f = x_dw + y_dw + match
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(2 * n)
sol = solver.search()

print("energy =", sol.energy())
print("permutation:")
for i in range(n):
    print(" ", "".join(str(sol(x_oh[i][j])) for j in range(n)))
```

### Key operations

- **`x[:n]` / `x[-n:]`**: Python slice notation replaces C++ `head()` / `tail()`.
- **`x[:, :n]` / `x[:, -n:]`**: Tuple indexing for slicing along inner dimensions.
- **`concat(1, x, 0)`** (`dim=0`): Adds a guard row of 1s at the top.
- **`concat(1, x, 1)`** (`dim=1`): Prepends 1 to each row.

### Comparison with C++ QUBO++

<table>
<thead>
<tr><th>C++ QUBO++</th><th>PyQBPP</th></tr>
</thead>
<tbody>
<tr><td><code>qbpp::head(v, n)</code></td><td><code>v[:n]</code></td></tr>
<tr><td><code>qbpp::tail(v, n)</code></td><td><code>v[-n:]</code></td></tr>
<tr><td><code>qbpp::slice(v, from, to)</code></td><td><code>v[from:to]</code></td></tr>
<tr><td><code>qbpp::head(v, n, 1)</code></td><td><code>v[:, :n]</code></td></tr>
<tr><td><code>qbpp::tail(v, n, 1)</code></td><td><code>v[:, -n:]</code></td></tr>
<tr><td><code>qbpp::concat(1, v)</code></td><td><code>qbpp.concat(1, v)</code></td></tr>
<tr><td><code>qbpp::concat(1, v, 0)</code></td><td><code>qbpp.concat(1, v, 0)</code></td></tr>
<tr><td><code>qbpp::concat(1, v, 1)</code></td><td><code>qbpp.concat(1, v, 1)</code></td></tr>
</tbody>
</table>

### Output

```
energy = 12
permutation:
  000001
  100000
  000100
  010000
  000010
  001000
```

</div>

<div class="lang-ja" markdown="1">
# スライスと連結

PyQBPPはPythonスタイルのスライスと`concat()`関数によるベクトル操作をサポートしています。
このページでは、**ドメインウォール符号化**と**Dual-Matrix Domain Wall**法を通じてこれらの操作を紹介します。

## スライス

PyQBPPのベクトルはPython標準のスライス記法をサポートしています。スライスは新しい`Vector`を返します:

```python
import pyqbpp as qbpp

x = qbpp.var("x", 8)
print(x[:3])     # 先頭3つ:  [x[0], x[1], x[2]]
print(x[-3:])    # 末尾3つ:  [x[5], x[6], x[7]]
print(x[2:5])    # 範囲:     [x[2], x[3], x[4]]
```

多次元ベクトルにはタプルインデックス（NumPyスタイル）を使います:

```python
x = qbpp.var("x", 3, 5)
print(x[:, :3])    # 各行の先頭3列
print(x[:, -2:])   # 各行の末尾2列
print(x[1:3, 1:4]) # 1-2行, 1-3列

x = qbpp.var("x", 2, 3, 4)
print(x[:, :, :2]) # 3次元目の先頭2要素
```

## 連結 (concat)

`concat()`関数はベクトルの連結やスカラーの追加を行います:

```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)

# 1D: スカラー + ベクトル、ベクトル + スカラー
y = qbpp.concat(1, qbpp.concat(x, 0))
# y = [1, x[0], x[1], x[2], x[3], 0]

# 2D: dimパラメータ付き
z = qbpp.var("z", 3, 4)

# dim=0: スカラーで埋めた行を追加
zg0 = qbpp.concat(1, qbpp.concat(z, 0, 0), 0)
# 上に全1の行、下に全0の行を追加 -> 5 x 4

# dim=1: 各行にスカラーを追加
zg1 = qbpp.concat(1, qbpp.concat(z, 0, 1), 1)
# 各行の先頭に1、末尾に0を追加 -> 3 x 6
```

## ドメインウォール符号化

**ドメインウォール**とは $1\cdots 1\, 0\cdots 0$ のバイナリパターンです。
$n$ 個の変数に対して $n+1$ 個のパターンがあり、整数 $0$ から $n$ を表現できます。

```python
import pyqbpp as qbpp

n = 8
x = qbpp.var("x", n)

# y = (1, x[0], ..., x[n-1], 0)
y = qbpp.concat(1, qbpp.concat(x, 0))

# 隣接差分
diff = y[:n+1] - y[-(n+1):]

# ペナルティ: ドメインウォールのとき最小値 1
f = qbpp.sum(qbpp.sqr(diff))
f.simplify_as_binary()

print("f =", f)

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search_optimal_solutions()

print("energy =", sol.energy())
print("solutions =", len(sol.all_solutions()))
for s in sol.all_solutions():
    bits = "".join(str(s(x[i])) for i in range(n))
    print(f"  {bits}  (sum = {s(qbpp.sum(x))})")
```

### 出力

```
f = 1 +2*x[1] +2*x[2] +2*x[3] +2*x[4] +2*x[5] +2*x[6] +2*x[7] -2*x[0]*x[1] -2*x[1]*x[2] -2*x[2]*x[3] -2*x[3]*x[4] -2*x[4]*x[5] -2*x[5]*x[6] -2*x[6]*x[7]
energy = 1
solutions = 9
  00000000  (sum = 0)
  10000000  (sum = 1)
  11000000  (sum = 2)
  11100000  (sum = 3)
  11110000  (sum = 4)
  11111000  (sum = 5)
  11111100  (sum = 6)
  11111110  (sum = 7)
  11111111  (sum = 8)
```

## Dual-Matrix Domain Wall

**Dual-Matrix Domain Wall** 法は、2つのバイナリ行列 `x`（$(n{-}1) \times n$）と `y`（$n \times (n{-}1)$）を使って $n \times n$ の置換行列を構築します。
ガードビットを追加して隣接差分を取ると、それぞれ $n \times n$ のone-hot行列が得られます。
これらを一致させることで、ループなしで置換行列を表現できます。
詳細は [https://arxiv.org/abs/2308.01024](https://arxiv.org/abs/2308.01024) を参照してください。

```python
import pyqbpp as qbpp

n = 6
x = qbpp.var("x", n - 1, n)  # (n-1) x n
y = qbpp.var("y", n, n - 1)  # n x (n-1)

# x: ガード行追加 (dim=0)、差分 -> n x n（各列one-hot）
xg = qbpp.concat(1, qbpp.concat(x, 0, 0), 0)
x_oh = xg[:n] - xg[-n:]
x_dw = qbpp.sum(qbpp.sqr(x_oh))

# y: ガードビット追加 (dim=1)、差分 -> n x n（各行one-hot）
yg = qbpp.concat(1, qbpp.concat(y, 0, 1), 1)
y_oh = yg[:, :n] - yg[:, -n:]
y_dw = qbpp.sum(qbpp.sqr(y_oh))

# 一致制約: x_oh == y_oh
match = qbpp.sum(x_oh - y_oh == 0)

f = x_dw + y_dw + match
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(2 * n)
sol = solver.search()

print("energy =", sol.energy())
print("permutation:")
for i in range(n):
    print(" ", "".join(str(sol(x_oh[i][j])) for j in range(n)))
```

### 主要な操作

- **`x[:n]` / `x[-n:]`**: Pythonスライスで C++ の `head()` / `tail()` に相当。
- **`x[:, :n]` / `x[:, -n:]`**: タプルインデックスで内側の次元をスライス。
- **`concat(1, x, 0)`**（`dim=0`）: 全1のガード行を上に追加。
- **`concat(1, x, 1)`**（`dim=1`）: 各行の先頭に1を追加。

### C++ QUBO++ との比較

<table>
<thead>
<tr><th>C++ QUBO++</th><th>PyQBPP</th></tr>
</thead>
<tbody>
<tr><td><code>qbpp::head(v, n)</code></td><td><code>v[:n]</code></td></tr>
<tr><td><code>qbpp::tail(v, n)</code></td><td><code>v[-n:]</code></td></tr>
<tr><td><code>qbpp::slice(v, from, to)</code></td><td><code>v[from:to]</code></td></tr>
<tr><td><code>qbpp::head(v, n, 1)</code></td><td><code>v[:, :n]</code></td></tr>
<tr><td><code>qbpp::tail(v, n, 1)</code></td><td><code>v[:, -n:]</code></td></tr>
<tr><td><code>qbpp::concat(1, v)</code></td><td><code>qbpp.concat(1, v)</code></td></tr>
<tr><td><code>qbpp::concat(1, v, 0)</code></td><td><code>qbpp.concat(1, v, 0)</code></td></tr>
<tr><td><code>qbpp::concat(1, v, 1)</code></td><td><code>qbpp.concat(1, v, 1)</code></td></tr>
</tbody>
</table>

### 出力

```
energy = 12
permutation:
  000001
  100000
  000100
  010000
  000010
  001000
```

</div>
