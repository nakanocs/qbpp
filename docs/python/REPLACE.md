---
layout: default
nav_exclude: true
title: "Replace Functions (PyQBPP)"
nav_order: 17
---
<div class="lang-en" markdown="1">
# Replace Functions

PyQBPP provides the following replace function, which can be used to fix variable values in an expression:
- **`replace(f, ml)`**: Returns a new expression in which variables are replaced according to the mapping `ml`.
- **`f.replace(ml)`**: Replaces variables in expression `f` in place.

## Using the replace function to fix variable values
We explain the `replace()` function using the
[partitioning problem](PARTITION).
This program finds a partition of the numbers in a list into two subsets $P$ and $Q$ such that the difference between their sums is minimized.

We modify this problem so that 64 must belong to $P$ and 27 must belong to $Q$:

```python
import pyqbpp as qbpp

w = [64, 27, 47, 74, 12, 83, 63, 40]
x = qbpp.var("x", len(w))
p = qbpp.sum([w[i] * x[i] for i in range(len(w))])
q = qbpp.sum([w[i] * ~x[i] for i in range(len(w))])
f = qbpp.sqr(p - q)
f.simplify_as_binary()

ml = qbpp.MapList([(x[0], 1), (x[1], 0)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(g)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)

print("energy =", full_sol.comp_energy())
P = [w[i] for i in range(len(w)) if full_sol(x[i]) == 1]
Q = [w[i] for i in range(len(w)) if full_sol(x[i]) == 0]
print("P:", P)
print("Q:", Q)
```
In this program, a `MapList` object **`ml`** fixes `x[0]=1` (64 in $P$) and `x[1]=0` (27 in $Q$).
The `replace()` function substitutes these values into `f`, and the Exhaustive Solver finds the optimal partition for the remaining variables.

This program produces the following output:
```
energy = 4
P: [64, 47, 12, 83]
Q: [27, 74, 63, 40]
```

## Using the replace function to replace variables with expressions
The `replace()` function can also replace a variable with an expression.

For example, to ensure that 64 and 27 are placed in distinct subsets,
we replace `x[0]` with `~x[1]` so they always take opposite values:

```python
ml = qbpp.MapList([(x[0], ~x[1])])
g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(g)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol, ml)

print("energy =", full_sol.comp_energy())
P = [w[i] for i in range(len(w)) if full_sol(x[i]) == 1]
Q = [w[i] for i in range(len(w)) if full_sol(x[i]) == 0]
print("P:", P)
print("Q:", Q)
```
This program produces the following output:
```
energy = 4
P: [64, 47, 12, 83]
Q: [27, 74, 63, 40]
```

## Replace functions for integer variables
Integer variables can be replaced with fixed integer values using the `replace()` function.

Here, we demonstrate this feature using a simple **multiplication/factorization** example.
Let $p$, $q$, and $r$ be integer variables with the constraint $p\times q - r = 0$.

### Multiplication
Fix $p=5$ and $q=7$ to find $r=35$:
```python
import pyqbpp as qbpp

p = qbpp.between(qbpp.var_int("p"), 2, 8)
q = qbpp.between(qbpp.var_int("q"), 2, 8)
r = qbpp.between(qbpp.var_int("r"), 2, 40)
f = p * q - r == 0
f.simplify_as_binary()

ml = qbpp.MapList([(p, 5), (q, 7)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.EasySolver(g)
solver.target_energy(0)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)
print(f"p={full_sol(p)}, q={full_sol(q)}, r={full_sol(r)}")
```
This program produces the following output:
```
p=5, q=7, r=35
```

### Factorization
Fix $r=35$ to find $p$ and $q$:
```python
ml = qbpp.MapList([(r, 35)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()
# ... same solver setup ...
```

### Division
Fix $p=5$ and $r=35$ to find $q=7$:
```python
ml = qbpp.MapList([(p, 5), (r, 35)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()
# ... same solver setup ...
```

> **NOTE**
> - **`f.replace(ml)`** updates the expression `f` in place.
> - **`replace(f, ml)`** returns a new expression without modifying the original.

> **NOTE: Negated literals and `replace()`**
> The `replace()` function treats `x` and `~x` as independent keys.
> Specifying `(x, 0)` in a `MapList` does **not** automatically replace `~x` with `1`.
> If the expression contains negated literals such as `~x`, you should explicitly include both mappings:
> ```python
> ml = qbpp.MapList([(x, 0), (~x, 1)])
> ```
</div>

<div class="lang-ja" markdown="1">
# 置換関数

PyQBPPは、式中の変数値を固定するために使用できる以下の置換関数を提供しています：
- **`replace(f, ml)`**: マッピング `ml` に従って変数を置換した新しい式を返します。
- **`f.replace(ml)`**: 式 `f` の変数をその場で置換します。

## 置換関数を使った変数値の固定
[分割問題](PARTITION)を用いて `replace()` 関数を説明します。
このプログラムは、リスト内の数を2つの部分集合 $P$ と $Q$ に分割し、それらの和の差を最小化します。

この問題を、64が $P$ に、27が $Q$ に属さなければならないように変更します：

```python
import pyqbpp as qbpp

w = [64, 27, 47, 74, 12, 83, 63, 40]
x = qbpp.var("x", len(w))
p = qbpp.sum([w[i] * x[i] for i in range(len(w))])
q = qbpp.sum([w[i] * ~x[i] for i in range(len(w))])
f = qbpp.sqr(p - q)
f.simplify_as_binary()

ml = qbpp.MapList([(x[0], 1), (x[1], 0)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(g)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)

print("energy =", full_sol.comp_energy())
P = [w[i] for i in range(len(w)) if full_sol(x[i]) == 1]
Q = [w[i] for i in range(len(w)) if full_sol(x[i]) == 0]
print("P:", P)
print("Q:", Q)
```
このプログラムでは、`MapList` オブジェクト **`ml`** が `x[0]=1`（64を $P$ に）と `x[1]=0`（27を $Q$ に）を固定します。
`replace()` 関数はこれらの値を `f` に代入し、Exhaustive Solverが残りの変数に対する最適な分割を求めます。

このプログラムの出力は以下の通りです：
```
energy = 4
P: [64, 47, 12, 83]
Q: [27, 74, 63, 40]
```

## 置換関数を使った変数の式への置換
`replace()` 関数は変数を式で置換することもできます。

例えば、64と27が異なる部分集合に配置されることを保証するために、
`x[0]` を `~x[1]` で置換して常に反対の値を取るようにします：

```python
ml = qbpp.MapList([(x[0], ~x[1])])
g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(g)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol, ml)

print("energy =", full_sol.comp_energy())
P = [w[i] for i in range(len(w)) if full_sol(x[i]) == 1]
Q = [w[i] for i in range(len(w)) if full_sol(x[i]) == 0]
print("P:", P)
print("Q:", Q)
```
このプログラムの出力は以下の通りです：
```
energy = 4
P: [64, 47, 12, 83]
Q: [27, 74, 63, 40]
```

## 整数変数の置換関数
整数変数は `replace()` 関数を使って固定の整数値で置換できます。

ここでは、簡単な**乗算/素因数分解**の例を使ってこの機能を示します。
$p$, $q$, $r$ を整数変数とし、制約 $p\times q - r = 0$ を課します。

### 乗算
$p=5$ と $q=7$ を固定して $r=35$ を求めます：
```python
import pyqbpp as qbpp

p = qbpp.between(qbpp.var_int("p"), 2, 8)
q = qbpp.between(qbpp.var_int("q"), 2, 8)
r = qbpp.between(qbpp.var_int("r"), 2, 40)
f = p * q - r == 0
f.simplify_as_binary()

ml = qbpp.MapList([(p, 5), (q, 7)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.EasySolver(g)
solver.target_energy(0)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)
print(f"p={full_sol(p)}, q={full_sol(q)}, r={full_sol(r)}")
```
このプログラムの出力は以下の通りです：
```
p=5, q=7, r=35
```

### 素因数分解
$r=35$ を固定して $p$ と $q$ を求めます：
```python
ml = qbpp.MapList([(r, 35)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()
# ... 同じソルバー設定 ...
```

### 除算
$p=5$ と $r=35$ を固定して $q=7$ を求めます：
```python
ml = qbpp.MapList([(p, 5), (r, 35)])
g = qbpp.replace(f, ml)
g.simplify_as_binary()
# ... 同じソルバー設定 ...
```

> **注意**
> - **`f.replace(ml)`** は式 `f` をその場で更新します。
> - **`replace(f, ml)`** は元の式を変更せずに新しい式を返します。

> **注意: 否定リテラルと `replace()`**
> `replace()` 関数は `x` と `~x` を独立したキーとして扱います。
> `MapList` に `(x, 0)` を指定しても、`~x` が自動的に `1` に置換されるわけではありません。
> 式に `~x` のような否定リテラルが含まれている場合、両方のマッピングを明示的に指定してください：
> ```python
> ml = qbpp.MapList([(x, 0), (~x, 1)])
> ```
</div>
