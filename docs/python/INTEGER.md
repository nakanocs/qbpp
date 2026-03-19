---
layout: default
nav_exclude: true
title: "Integer Variables (PyQBPP)"
nav_order: 7
---
<div class="lang-en" markdown="1">
# Integer Variables and Solving Simultaneous Equations

## Integer variables
PyQBPP supports **integer variables**, which are internally implemented using multiple binary variables.
A conventional binary encoding is used to represent integer values.

The following program demonstrates how integer variables are defined:
```python
from pyqbpp import var_int, between

x = between(var_int("x"), 1, 8)
y = between(var_int("y"), -10, 10)
print(f"x = {x} uses {x.var_count()} variables.")
print(f"y = {y} uses {y.var_count()} variables.")
```

An integer variable is defined using the **`between()`** function, which specifies the integer range that the variable can take.
The function **`var_int("name")`** creates a **`VarIntCore`** object with the given `name`, and **`between(var_int("name"), min, max)`** creates a **`VarInt`** object representing the linear expression encoded by binary variables.
The program outputs the following expressions:
```
x = 1 +x[0] +2*x[1] +4*x[2] uses 3 variables.
y = -10 +y[0] +2*y[1] +4*y[2] +8*y[3] +5*y[4] uses 5 variables.
```

> **WARNING**
> The number of binary variables required for an integer variable grows logarithmically with its range.
> When `max - min` is large, the QUBO size increases, so wide integer ranges should be avoided whenever possible.

## QUBO formulation for solving simultaneous equations
PyQBPP can solve systems of simultaneous equations by representing the variables as integer variables.
As an example, we construct a QUBO formulation for the following equations, whose solution is $x=4$ and $y=6$:

$$
\begin{aligned}
x + y = 10\\
2x+4y = 28
\end{aligned}
$$

## PyQBPP program
The following program constructs the QUBO expression, solves it, and decodes the resulting values of $x$ and $y$:
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x"), 0, 10)
y = between(var_int("y"), 0, 10)

f = (x + y) == 10
g = (2 * x + 4 * y) == 28
h = f + g
h.simplify_as_binary()

solver = EasySolver(h)
solver.target_energy(0)
sol = solver.search()

print("sol =", sol)
print("x =", x, "=", sol.eval(x))
print("y =", y, "=", sol.eval(y))
print("f =", sol.eval(f))
print("g =", sol.eval(g))
print("x + y =", sol.eval(f.body))
print("2x + 4y =", sol.eval(g.body))
```
First, `VarInt` objects **`x`** and **`y`** are defined with the range $[0,10]$.
An `Expr` object **`f`** is created to represent the constraint **`(x + y) == 10`**.
Internally, this is equivalent to the QUBO expression `sqr(x + y - 10)`.
Similarly, **`g`** represents the constraint **`(2 * x + 4 * y) == 28`**.
The combined expression **`h = f + g`** encodes both equations.
An Easy Solver instance is created with `h`, and the target energy is set to `0`, since the optimal solution satisfies all constraints.
Calling `search()` returns a `Sol` object `sol` that stores the optimal assignment of all binary variables.

Here,
- **`f`**: The penalty expression enforcing `x + y = 10`. Thus `sol.eval(f) = 0` if and only if the equation is satisfied.
- **`f.body`**: The linear expression `x + y`. Thus `sol.eval(f.body)` returns the actual evaluated value of `x + y`.

The same applies to **`g`** and **`g.body`**.

The program outputs the following result:
```
sol = Sol(energy=0, x[0]=0, x[1]=1, x[2]=1, x[3]=0, y[0]=0, y[1]=0, y[2]=1, y[3]=0)
x = x[0] +2*x[1] +4*x[2] +3*x[3] = 6
y = y[0] +2*y[1] +4*y[2] +3*y[3] = 4
f = 0
g = 0
x + y = 10
2x + 4y = 28
```

Thus, we can confirm that the values of `x`, `y`, and the constraint expressions are consistent with the solution.

> **WARNING**
> PyQBPP supports the `==` operator only when the left-hand side is an expression and the right-hand side is an integer.
> Comparisons of the form `integer == expression` or `expression == expression` are not supported.
</div>

<div class="lang-ja" markdown="1">
# 整数変数と連立方程式の求解

## 整数変数
PyQBPPは**整数変数**をサポートしており、内部的には複数のバイナリ変数を使って実装されています。
整数値の表現には従来のバイナリエンコーディングが使用されます。

以下のプログラムは整数変数の定義方法を示しています。
```python
from pyqbpp import var_int, between

x = between(var_int("x"), 1, 8)
y = between(var_int("y"), -10, 10)
print(f"x = {x} uses {x.var_count()} variables.")
print(f"y = {y} uses {y.var_count()} variables.")
```

整数変数は **`between()`** 関数を使って定義し、変数がとりうる整数範囲を指定します。
関数 **`var_int("name")`** は指定された `name` を持つ **`VarIntCore`** オブジェクトを作成し、**`between(var_int("name"), min, max)`** はバイナリ変数でエンコードされた線形式を表す **`VarInt`** オブジェクトを作成します。
プログラムの出力は以下の通りです。
```
x = 1 +x[0] +2*x[1] +4*x[2] uses 3 variables.
y = -10 +y[0] +2*y[1] +4*y[2] +8*y[3] +5*y[4] uses 5 variables.
```

> **WARNING**
> 整数変数に必要なバイナリ変数の数は、その範囲に対して対数的に増加します。
> `max - min` が大きい場合、QUBOのサイズが増大するため、広い整数範囲はできる限り避けるべきです。

## 連立方程式を解くためのQUBO定式化
PyQBPPは変数を整数変数として表現することで、連立方程式を解くことができます。
例として、解が $x=4$、$y=6$ である以下の連立方程式のQUBO定式化を構築します。

$$
\begin{aligned}
x + y = 10\\
2x+4y = 28
\end{aligned}
$$

## PyQBPP プログラム
以下のプログラムはQUBO式を構築し、それを解いて $x$ と $y$ の値を復号します。
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x"), 0, 10)
y = between(var_int("y"), 0, 10)

f = (x + y) == 10
g = (2 * x + 4 * y) == 28
h = f + g
h.simplify_as_binary()

solver = EasySolver(h)
solver.target_energy(0)
sol = solver.search()

print("sol =", sol)
print("x =", x, "=", sol.eval(x))
print("y =", y, "=", sol.eval(y))
print("f =", sol.eval(f))
print("g =", sol.eval(g))
print("x + y =", sol.eval(f.body))
print("2x + 4y =", sol.eval(g.body))
```
まず、`VarInt` オブジェクト **`x`** と **`y`** を範囲 $[0,10]$ で定義します。
`Expr` オブジェクト **`f`** は制約 **`(x + y) == 10`** を表すために作成されます。
内部的には、これはQUBO式 `sqr(x + y - 10)` と等価です。
同様に、**`g`** は制約 **`(2 * x + 4 * y) == 28`** を表します。
結合式 **`h = f + g`** は両方の方程式をエンコードします。
Easy Solver のインスタンスを `h` で作成し、最適解がすべての制約を満たすため、目標エネルギーを `0` に設定します。
`search()` を呼び出すと、すべてのバイナリ変数の最適な割り当てを格納した `Sol` オブジェクト `sol` が返されます。

ここで、
- **`f`**: `x + y = 10` を強制するペナルティ式。方程式が満たされるとき、かつそのときに限り `sol.eval(f) = 0` となります。
- **`f.body`**: 線形式 `x + y`。`sol.eval(f.body)` は `x + y` の実際の評価値を返します。

**`g`** と **`g.body`** についても同様です。

プログラムの出力は以下の通りです。
```
sol = Sol(energy=0, x[0]=0, x[1]=1, x[2]=1, x[3]=0, y[0]=0, y[1]=0, y[2]=1, y[3]=0)
x = x[0] +2*x[1] +4*x[2] +3*x[3] = 6
y = y[0] +2*y[1] +4*y[2] +3*y[3] = 4
f = 0
g = 0
x + y = 10
2x + 4y = 28
```

このように、`x`、`y` の値と制約式が解と一致していることが確認できます。

> **WARNING**
> PyQBPPの `==` 演算子は、左辺が式で右辺が整数の場合のみサポートされています。
> `整数 == 式` や `式 == 式` の形式の比較はサポートされていません。
</div>
