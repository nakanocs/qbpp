---
layout: default
nav_exclude: true
title: "Range Constraints"
nav_order: 9
---
<div class="lang-en" markdown="1">
# Range Constraints and Solving Integer Linear Programming

## Polynomial formulation for range constraints
Let $f$ be a polynomial expression of binary variables.
A range constraint has the form **$l\leq f\leq u$** with $l<u$.
Our goal is to design a polynomial expression that takes the minimum value 0 if and only if the range constraint is satisfied.

The key idea is to introduce an **auxiliary integer variable** $a$ that takes values in the range $[l,u]$.
Consider the following expression:

$$
\begin{aligned}
g &= (f-a)^2
\end{aligned}
$$

This expression $g$ takes the minimum value 0 exactly when $f=a$.
Since $a$ can take any integer value in $[l,u]$, the expression
$g$ achieves 0 if and only if $f$ itself takes an integer value within the same range.

Using this auxiliary-variable technique, PyQBPP implements range constraints via the **`between()`** function.

## Solving Integer Linear Programming
An instance of **integer linear programming** consists of an **objective function** and multiple **linear constraints**.
For example, the following integer linear program has two variables, one objective, and two constraints:

$$
\begin{aligned}
\text{Maximize: } & & & 5x + 4y \\
\text{Subject to: } & && 2x + 3y \le 24 \\
                   & & & 7x + 5y \le 54
\end{aligned}
$$

The optimal solution of this problem is $x=4$, $y=5$, with the objective value $40$.

The following PyQBPP program finds this optimal solution using the Easy Solver:
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x"), 0, 10)
y = between(var_int("y"), 0, 10)
f = 5 * x + 4 * y
c1 = between(2 * x + 3 * y, 0, 24)
c2 = between(7 * x + 5 * y, 0, 54)
g = -f + 100 * (c1 + c2)
g.simplify_as_binary()

solver = EasySolver(g)
solver.time_limit(1.0)
sol = solver.search()

print(f"x = {sol.eval(x)}, y = {sol.eval(y)}")
print(f"f = {sol.eval(f)}")
print(f"c1 = {sol.eval(c1)}, c2 = {sol.eval(c2)}")
print(f"2x+3y = {sol.eval(c1.body)}, 7x+5y = {sol.eval(c2.body)}")
```

In this program,
- **`f`** represents the objective function,
- **`c1`** and **`c2`** represent the range constraints created using **`between()`**, and
- **`g`** combines them into a single optimization expression.

Since the goal is maximization, the objective is negated as `-f`.
The constraints `c1` and `c2` are penalized with a weight of 100 to ensure they are satisfied with high priority.

An Easy Solver instance is created for `g`, and a search is performed with a time limit of 1.0 seconds.
After obtaining the optimal solution `sol`, the program prints the values of `x`, `y`, `f`, `c1`, `c2`, and the constraint body expressions.

The program outputs:
```
x = 4, y = 5
f = 40
c1 = 0, c2 = 0
2x+3y = 23, 7x+5y = 53
```
Here,
- **`c1`** is the penalty for the constraint `0 <= 2x + 3y <= 24`, and
- **`c1.body`** represents the linear expression `2x + 3y`.

We can confirm that the solver correctly finds the optimal solution.
</div>

<div class="lang-ja" markdown="1">
# 範囲制約と整数線形計画法の求解

## 範囲制約の多項式定式化
$f$ をバイナリ変数の多項式とします。
範囲制約は $l<u$ のもとで **$l\leq f\leq u$** の形式を持ちます。
目標は、範囲制約が満たされるときかつそのときに限り最小値0を取る多項式を設計することです。

鍵となるアイデアは、範囲 $[l,u]$ の値を取る**補助整数変数** $a$ を導入することです。
以下の式を考えます：

$$
\begin{aligned}
g &= (f-a)^2
\end{aligned}
$$

この式 $g$ は $f=a$ のときちょうど最小値0を取ります。
$a$ は $[l,u]$ の任意の整数値を取れるため、
式 $g$ が0になるのは $f$ 自体が同じ範囲内の整数値を取るときかつそのときに限ります。

この補助変数の手法を用いて、PyQBPPは **`between()`** 関数により範囲制約を実装しています。

## 整数線形計画法の求解
**整数線形計画法**のインスタンスは、**目的関数**と複数の**線形制約**から構成されます。
例えば、以下の整数線形計画は2つの変数、1つの目的関数、2つの制約を持ちます：

$$
\begin{aligned}
\text{Maximize: } & & & 5x + 4y \\
\text{Subject to: } & && 2x + 3y \le 24 \\
                   & & & 7x + 5y \le 54
\end{aligned}
$$

この問題の最適解は $x=4$, $y=5$ で、目的関数の値は $40$ です。

以下のPyQBPPプログラムは、Easy Solverを使ってこの最適解を求めます：
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x"), 0, 10)
y = between(var_int("y"), 0, 10)
f = 5 * x + 4 * y
c1 = between(2 * x + 3 * y, 0, 24)
c2 = between(7 * x + 5 * y, 0, 54)
g = -f + 100 * (c1 + c2)
g.simplify_as_binary()

solver = EasySolver(g)
solver.time_limit(1.0)
sol = solver.search()

print(f"x = {sol.eval(x)}, y = {sol.eval(y)}")
print(f"f = {sol.eval(f)}")
print(f"c1 = {sol.eval(c1)}, c2 = {sol.eval(c2)}")
print(f"2x+3y = {sol.eval(c1.body)}, 7x+5y = {sol.eval(c2.body)}")
```

このプログラムでは、
- **`f`** は目的関数を表し、
- **`c1`** と **`c2`** は **`between()`** を使って作成された範囲制約を表し、
- **`g`** はそれらを1つの最適化式にまとめたものです。

目標が最大化であるため、目的関数は `-f` として符号を反転しています。
制約 `c1` と `c2` は重み100のペナルティを付けて、高い優先度で満たされるようにしています。

`g` に対してEasy Solverのインスタンスを作成し、制限時間1.0秒で探索を実行します。
最適解 `sol` を得た後、プログラムは `x`、`y`、`f`、`c1`、`c2`、および制約本体の式の値を出力します。

プログラムの出力は以下の通りです：
```
x = 4, y = 5
f = 40
c1 = 0, c2 = 0
2x+3y = 23, 7x+5y = 53
```
ここで、
- **`c1`** は制約 `0 <= 2x + 3y <= 24` のペナルティであり、
- **`c1.body`** は線形式 `2x + 3y` を表します。

ソルバーが正しく最適解を見つけていることが確認できます。
</div>
