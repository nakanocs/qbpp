---
layout: default
nav_exclude: true
title: "Cubic Equation (PyQBPP)"
nav_order: 46
---
<div class="lang-en" markdown="1">
# Cubic Equation
Cubic equations over the integers can be solved using PyQBPP. For example, consider

$$
\begin{aligned}
x^3 -147x +286 &=0.
\end{aligned}
$$

This equation has three integer solutions: $x = -13, 2, 11$.

## PyQBPP program for solving the cubic equation
In the following PyQBPP program, we define an integer variable x that takes values in $[-100, 100]$, and we enumerate all optimal solutions using the Exhaustive Solver:
```python
import pyqbpp as qbpp

x = qbpp.between(qbpp.var_int("x"), -100, 100)
f = x * x * x - 147 * x + 286 == 0
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    xv = sol(x)
    if xv not in seen:
        seen.add(xv)
        print(f"x = {xv}")
```
The expression `f` corresponds to the following objective function:

$$
\begin{aligned}
f & = (x^3 -147x +286)^2
\end{aligned}
$$

Since the integer variable `x` is implemented as a linear expression of binary variables, `f` becomes a polynomial of degree 6.

Since Python integers have unlimited precision, there is no need to specify special integer types (unlike the C++ version which requires `COEFF_TYPE=cpp_int`).

Because the coefficient of the highest-order binary variable is not a power of two,
the same integer value can be represented by multiple different assignments of the binary variables.
Therefore, we use a `set` to remove duplicate values of `x`.

This program produces the following output:
```
x = 11
x = 2
x = -13
```
</div>

<div class="lang-ja" markdown="1">
# 3次方程式
整数上の3次方程式は PyQBPP を使って解くことができます。例えば、次の方程式を考えます:

$$
\begin{aligned}
x^3 -147x +286 &=0.
\end{aligned}
$$

この方程式には3つの整数解があります: $x = -13, 2, 11$。

## 3次方程式を解く PyQBPP プログラム
以下の PyQBPP プログラムでは、$[-100, 100]$ の値を取る整数変数 x を定義し、全探索ソルバーを使ってすべての最適解を列挙します:
```python
import pyqbpp as qbpp

x = qbpp.between(qbpp.var_int("x"), -100, 100)
f = x * x * x - 147 * x + 286 == 0
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    xv = sol(x)
    if xv not in seen:
        seen.add(xv)
        print(f"x = {xv}")
```
式 `f` は以下の目的関数に対応します:

$$
\begin{aligned}
f & = (x^3 -147x +286)^2
\end{aligned}
$$

整数変数 `x` はバイナリ変数の線形式として実装されるため、`f` は6次の多項式になります。

Python の整数は任意精度であるため、特別な整数型を指定する必要はありません（C++版では `COEFF_TYPE=cpp_int` が必要です）。

最上位のバイナリ変数の係数が2の冪ではないため、同じ整数値が複数の異なるバイナリ変数の割り当てで表現される可能性があります。
そのため、`set` を使って `x` の重複する値を除去しています。

このプログラムは以下の出力を生成します:
```
x = 11
x = 2
x = -13
```
</div>
