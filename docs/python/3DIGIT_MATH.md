---
layout: default
nav_exclude: true
title: "3-Digit Math (PyQBPP)"
nav_order: 45
---
<div class="lang-en" markdown="1">
# 3-Digit Math Problem

Let us solve the following math problem using PyQBPP.

> **Math Problem**:
> Find all three-digit odd integers whose **product of digits** is **252**.

Let $x$, $y$, and $z$ be the hundreds, tens, and ones digits of the integer, respectively.
More specifically:
- $x$ is an integer in $[1, 9]$,
- $y$ is an integer in $[0, 9]$,
- $t$ is an integer in $[0, 4]$,
- $z = 2t + 1$ (so $z$ is odd).

Then the value $v$ of the three-digit integer $xyz$ is

$$
\begin{aligned}
v&=100x+10y+z
\end{aligned}
$$


We find all solutions satisfying:

$$
\begin{aligned}
xyz &= 252
\end{aligned}
$$

## PyQBPP program
The following PyQBPP program finds all solutions:
```python
from pyqbpp import var_int, between, ExhaustiveSolver

x = between(var_int("x"), 1, 9)
y = between(var_int("y"), 0, 9)
t = between(var_int("t"), 0, 4)
z = 2 * t + 1
v = x * 100 + y * 10 + z

f = x * y * z == 252

f.simplify_as_binary()
solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
results = set()
for sol in sols:
    results.add(sol(v))
for val in sorted(results):
    print(val, end=" ")
print()
```
In this program, **`x`**, **`y`**, and **`t`** are defined as integer variables with the ranges above.
Then **`z`**, **`v`**, and **`f`** are defined as expressions.
We create an Exhaustive Solver instance for `f` and store all optimal solutions in `sols`.

Because `x`, `y`, and `t` are encoded by multiple binary variables, different binary assignments can represent the same integer values.
As a result, the same digit triple (`x`,`y`,`z`) may appear multiple times in `sols`.
Therefore, we use a `set` to remove duplicates by collecting only the resulting integer values `v`.

The integers are printed as follows:
```
479 497 667 749 947
```
</div>

<div class="lang-ja" markdown="1">
# 3桁の数学問題

以下の数学問題をPyQBPPを用いて解きます。

> **数学問題**：
> **各桁の積**が**252**である3桁の奇数をすべて求めてください。

$x$、$y$、$z$ をそれぞれ百の位、十の位、一の位の数字とします。
より具体的には：
- $x$ は $[1, 9]$ の整数、
- $y$ は $[0, 9]$ の整数、
- $t$ は $[0, 4]$ の整数、
- $z = 2t + 1$（$z$ は奇数）。

3桁の整数 $xyz$ の値 $v$ は

$$
\begin{aligned}
v&=100x+10y+z
\end{aligned}
$$


以下を満たすすべての解を求めます：

$$
\begin{aligned}
xyz &= 252
\end{aligned}
$$

## PyQBPPプログラム
以下のPyQBPPプログラムですべての解を求めます：
```python
from pyqbpp import var_int, between, ExhaustiveSolver

x = between(var_int("x"), 1, 9)
y = between(var_int("y"), 0, 9)
t = between(var_int("t"), 0, 4)
z = 2 * t + 1
v = x * 100 + y * 10 + z

f = x * y * z == 252

f.simplify_as_binary()
solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
results = set()
for sol in sols:
    results.add(sol(v))
for val in sorted(results):
    print(val, end=" ")
print()
```
このプログラムでは、**`x`**、**`y`**、**`t`** を上記の範囲の整数変数として定義します。
次に **`z`**、**`v`**、**`f`** を式として定義します。
`f` に対するExhaustive Solverインスタンスを作成し、すべての最適解を `sols` に格納します。

`x`、`y`、`t` は複数のバイナリ変数でエンコードされるため、異なるバイナリ割り当てが同じ整数値を表す場合があります。
その結果、同じ数字の組 (`x`,`y`,`z`) が `sols` に複数回現れる可能性があります。
そのため、結果の整数値 `v` のみを集めることで `set` を使って重複を除去しています。

出力される整数は以下の通りです：
```
479 497 667 749 947
```
</div>
