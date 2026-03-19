---
layout: default
nav_exclude: true
title: "Square Root (PyQBPP)"
nav_order: 42
---
<div class="lang-en" markdown="1">
# Square Root

This example demonstrates how to compute the square root of
$c=2$ using large integers.
Let $s = 10 ^{10}$ be a fixed integer.
Since PyQBPP cannot handle real numbers directly, we compute  $\sqrt{cs^2}$ instead of $\sqrt{c}$.
From the following relation,

$$
\begin{aligned}
\sqrt{c} &= \sqrt{cs^2}/s
\end{aligned}
$$

we can obtain an approximation of $\sqrt{c}$ with 10 decimal-digit precision.

## HUBO formulation of the square root computation
We define an integer variable $x$ that takes values in the range $[s, 2s]$.
We then formulate the problem using the following equation:

$$
\begin{aligned}
x ^ 2 &= cs ^ 2
\end{aligned}
$$

In PyQBPP, this equality constraint is converted into the following HUBO expression:

$$
(x ^ 2 -cs^2)^2
$$

By finding the value of $x$ that minimizes this expression,
we obtain an approximation of the square root of $c$ with
10 decimal-digit precision.
Since $x$ is internally represented as a linear expression of binary variables, this objective function becomes quartic in those binary variables.

## PyQBPP program
The following PyQBPP program constructs a HUBO expression based on the above idea and solves it using the Easy Solver:
```python
from pyqbpp import var_int, between, EasySolver

c = 2
s = 10**10
x = between(var_int("x"), s, c * s)
f = x * x == c * s * s
f.simplify_as_binary()
solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()
print(f"Energy = {sol.energy()}")
print(f"x = {sol(x)}")
```

Since Python integers have unlimited precision, there is no need to specify special integer types (unlike the C++ version which requires `COEFF_TYPE=cpp_int`).
The constant `s`, the integer variable `x`, and the HUBO expression `f` are defined according to the formulation described above.
The Easy Solver is executed with a time limit of 1.0 second.

This program produces the following output:
```
Energy = 57910111919782629376
x = 14142135624
```
We can confirm that the Easy Solver outputs the correct approximation:

$$
 \sqrt{2\times 10^{20}}\approx 14142135624
$$

Note that the reported energy value is not zero, and the equality constraint is not satisfied exactly.
This is simply because there is no exact integer solution to the equality.
Instead, the solver finds a solution that minimizes the error of the equality constraint.
The energy value shown in the output corresponds to the square of this error.
Since the error is minimized, the resulting value of $x$ represents an approximation of the square root.
</div>

<div class="lang-ja" markdown="1">
# 平方根

この例では、大きな整数を用いて $c=2$ の平方根を計算する方法を示します。
$s = 10^{10}$ を固定整数とします。
PyQBPP は実数を直接扱えないため、$\sqrt{c}$ の代わりに $\sqrt{cs^2}$ を計算します。
以下の関係式から、

$$
\begin{aligned}
\sqrt{c} &= \sqrt{cs^2}/s
\end{aligned}
$$

10桁の10進精度で $\sqrt{c}$ の近似値を得ることができます。

## 平方根計算の HUBO 定式化
範囲 $[s, 2s]$ の値を取る整数変数 $x$ を定義します。
次に、以下の等式を用いて問題を定式化します:

$$
\begin{aligned}
x ^ 2 &= cs ^ 2
\end{aligned}
$$

PyQBPP では、この等式制約は以下の HUBO 式に変換されます:

$$
(x ^ 2 -cs^2)^2
$$

この式を最小化する $x$ の値を見つけることで、10桁の10進精度で $c$ の平方根の近似値を得ます。
$x$ は内部的にバイナリ変数の線形式として表現されるため、この目的関数はバイナリ変数に関して4次式になります。

## PyQBPP プログラム
以下の PyQBPP プログラムは、上記の考え方に基づいて HUBO 式を構築し、Easy Solver を用いて解きます:
```python
from pyqbpp import var_int, between, EasySolver

c = 2
s = 10**10
x = between(var_int("x"), s, c * s)
f = x * x == c * s * s
f.simplify_as_binary()
solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()
print(f"Energy = {sol.energy()}")
print(f"x = {sol(x)}")
```

Python の整数は任意精度であるため、特別な整数型を指定する必要はありません（C++版では `COEFF_TYPE=cpp_int` が必要です）。
定数 `s`、整数変数 `x`、HUBO 式 `f` は上述の定式化に従って定義されています。
Easy Solver は制限時間1.0秒で実行されます。

このプログラムは以下の出力を生成します:
```
Energy = 57910111919782629376
x = 14142135624
```
Easy Solver が正しい近似値を出力していることが確認できます:

$$
 \sqrt{2\times 10^{20}}\approx 14142135624
$$

報告されたエネルギー値はゼロではなく、等式制約は厳密には満たされていないことに注意してください。
これは単に、この等式に対する厳密な整数解が存在しないためです。
代わりに、ソルバーは等式制約の誤差を最小化する解を見つけます。
出力に示されているエネルギー値は、この誤差の2乗に対応しています。
誤差が最小化されるため、得られた $x$ の値は平方根の近似値を表しています。
</div>
