---
layout: default
nav_exclude: true
title: "Find Three Integers (PyQBPP)"
nav_order: 44
---
<div class="lang-en" markdown="1">
# Math Problem: Find Three Integers

The following math problem can be solved using PyQBPP.

### Problem
Find integers $x$, $y$, $z$ that satisfy:

$$
\begin{aligned}
\frac{1}{x}+\frac{1}{y}+\frac{1}{z} = 1\\
1 < x < y < z
\end{aligned}
$$



### PyQBPP program

Since PyQBPP can handle polynomial expressions, we first rewrite the constraints.
Multiplying both sides of the first constraint by $xyz$ yields:

$$
xy+yz+zx - xyz = 0
$$

The strict inequalities $x<y<z$ can be encoded as

$$
\begin{aligned}
1 &\leq y-x \\
1 &\leq z-y
\end{aligned}
$$

The following PyQBPP program formulates these constraints as a HUBO expression and solves it using the Exhaustive Solver:

```python
import pyqbpp as qbpp

x = qbpp.between(qbpp.var_int("x"), 1, 10)
y = qbpp.between(qbpp.var_int("y"), 1, 10)
z = qbpp.between(qbpp.var_int("z"), 1, 10)

c1 = x * y + y * z + z * x - x * y * z == 0
c2 = qbpp.between(y - x, 1, 9)
c3 = qbpp.between(z - y, 1, 9)

f = c1 + c2 + c3
f.simplify_as_binary()
solver = qbpp.ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    key = (sol(x), sol(y), sol(z))
    if key not in seen:
        seen.add(key)
        xv, yv, zv = key
        print(f"(x,y,z) = ({xv}, {yv}, {zv})")
```
The three constraints are encoded as `c1`, `c2`, and `c3`, and combined into a single objective `f`.
The Exhaustive Solver searches for optimal solutions of f and prints the resulting
$(x,y,z)$ tuples.

Because `f` introduces auxiliary variables during binary simplification, the same
$(x,y,z)$ assignment may appear multiple times in the returned solution set.
Therefore, we use a `set` to remove duplicates before printing.

This program produces the following output:
```
(x,y,z) = (2, 3, 6)
```
This indicates that the problem has exactly one solution in the searched range, namely $(x,y,z)=(2,3,6)$.
</div>

<div class="lang-ja" markdown="1">
# 数学問題：3つの整数を求める

以下の数学問題をPyQBPPを用いて解くことができます。

### 問題
以下を満たす整数 $x$、$y$、$z$ を求めてください：

$$
\begin{aligned}
\frac{1}{x}+\frac{1}{y}+\frac{1}{z} = 1\\
1 < x < y < z
\end{aligned}
$$



### PyQBPPプログラム

PyQBPPは多項式を扱えるため、まず制約を書き換えます。
最初の制約の両辺に $xyz$ を掛けると：

$$
xy+yz+zx - xyz = 0
$$

狭義の不等式 $x<y<z$ は以下のようにエンコードできます：

$$
\begin{aligned}
1 &\leq y-x \\
1 &\leq z-y
\end{aligned}
$$

以下のPyQBPPプログラムは、これらの制約をHUBO式として定式化し、Exhaustive Solverを用いて解きます：

```python
import pyqbpp as qbpp

x = qbpp.between(qbpp.var_int("x"), 1, 10)
y = qbpp.between(qbpp.var_int("y"), 1, 10)
z = qbpp.between(qbpp.var_int("z"), 1, 10)

c1 = x * y + y * z + z * x - x * y * z == 0
c2 = qbpp.between(y - x, 1, 9)
c3 = qbpp.between(z - y, 1, 9)

f = c1 + c2 + c3
f.simplify_as_binary()
solver = qbpp.ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    key = (sol(x), sol(y), sol(z))
    if key not in seen:
        seen.add(key)
        xv, yv, zv = key
        print(f"(x,y,z) = ({xv}, {yv}, {zv})")
```
3つの制約は `c1`、`c2`、`c3` としてエンコードされ、単一の目的関数 `f` にまとめられます。
Exhaustive Solverが `f` の最適解を探索し、得られた $(x,y,z)$ のタプルを出力します。

`f` はバイナリ簡約化の際に補助変数を導入するため、同じ $(x,y,z)$ の割り当てが返される解集合に複数回現れる場合があります。
そのため、出力前に `set` を使って重複を除去しています。

このプログラムの出力は以下の通りです：
```
(x,y,z) = (2, 3, 6)
```
これは、探索範囲内でこの問題がちょうど1つの解 $(x,y,z)=(2,3,6)$ を持つことを示しています。
</div>
