---
layout: default
nav_exclude: true
title: "Factorization (PyQBPP)"
nav_order: 8
---
<div class="lang-en" markdown="1">
# Factorization Through HUBO Expression

## HUBO for factorizing the product of two prime numbers
We consider the **factorization of integers** that are products of two prime numbers.
For example, when the product $pq = 35$ is given, the goal is to recover the two prime factors $p=5$ and $q=7$.

Since $\sqrt{35}=5.91$ and $35/2=17.5$, we can restrict the search ranges of $p$ and $q$ as follows:

$$
\begin{aligned}
  2 \leq &p \leq 5 \\
  6 \leq &q \leq 17
\end{aligned}
$$

For such integer variables, the factorization problem for $35$ can be formulated using the penalty expression:

$$
\begin{aligned}
f(p,q) &= (pq-35)^2
\end{aligned}
$$

Because the integer variables $p$ and $q$ are implemented as linear expressions of binary variables, their product
$pq$ becomes a quadratic expression, and therefore
$f(p,q)$ becomes quartic.
Clearly, $f(p,q)$ attains the minimum value 0 exactly when $p$ and $q$ are the correct factors of 35.

## PyQBPP program for factorization
The following program constructs the HUBO expression $f(p,q)$, and solves the optimization problem using the Easy Solver:
```python
from pyqbpp import var_int, between, EasySolver

p = between(var_int("p"), 2, 5)
q = between(var_int("q"), 6, 17)

f = (p * q) == 35
print("f =", f.simplify_as_binary())

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print("sol =", sol)
print("p =", sol(p))
print("q =", sol(q))
```

In this program, the expression `(p * q) == 35` is automatically converted into `sqr(p * q - 35)`, which achieves an energy value of 0 when the equality is satisfied.
The output of this program is as follows:
```
f = 529 -240*p[0] -408*p[1] -88*q[0] -168*q[1] -304*q[2] -304*q[3] +144*p[0]*p[1] ...
sol = Sol(energy=0, p[0]=1, p[1]=1, q[0]=1, q[1]=0, q[2]=0, q[3]=0)
p = 5
q = 7
```
From the output, we can observe that the expression `f` contains quartic terms, confirming that it is a HUBO expression.
The solver correctly finds the prime factors $p=5$ and $q=7$.

> **NOTE**
> PyQBPP uses arbitrarily large integer arithmetic (Python's native `int`) for all coefficient and energy computations.
> Unlike the C++ version, there is no need to specify `COEFF_TYPE` or `ENERGY_TYPE` macros.
> Factorization of large composite numbers works out of the box without any special configuration.
</div>

<div class="lang-ja" markdown="1">
# HUBO式による素因数分解

## 2つの素数の積の素因数分解のための HUBO
2つの素数の積である**整数の素因数分解**を考えます。
例えば、積 $pq = 35$ が与えられたとき、2つの素因数 $p=5$ と $q=7$ を求めることが目標です。

$\sqrt{35}=5.91$ かつ $35/2=17.5$ であるため、$p$ と $q$ の探索範囲を以下のように制限できます:

$$
\begin{aligned}
  2 \leq &p \leq 5 \\
  6 \leq &q \leq 17
\end{aligned}
$$

このような整数変数に対して、$35$ の素因数分解問題はペナルティ式を用いて以下のように定式化できます:

$$
\begin{aligned}
f(p,q) &= (pq-35)^2
\end{aligned}
$$

整数変数 $p$ と $q$ はバイナリ変数の線形式として実装されるため、その積 $pq$ は2次式となり、したがって $f(p,q)$ は4次式になります。
明らかに、$f(p,q)$ は $p$ と $q$ が 35 の正しい因数であるときに限り最小値 0 を達成します。

## 素因数分解の PyQBPP プログラム
以下のプログラムは HUBO 式 $f(p,q)$ を構築し、Easy Solver を用いて最適化問題を解きます:
```python
from pyqbpp import var_int, between, EasySolver

p = between(var_int("p"), 2, 5)
q = between(var_int("q"), 6, 17)

f = (p * q) == 35
print("f =", f.simplify_as_binary())

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print("sol =", sol)
print("p =", sol(p))
print("q =", sol(q))
```

このプログラムでは、式 `(p * q) == 35` は自動的に `sqr(p * q - 35)` に変換され、等式が満たされたときにエネルギー値 0 を達成します。
このプログラムの出力は以下の通りです:
```
f = 529 -240*p[0] -408*p[1] -88*q[0] -168*q[1] -304*q[2] -304*q[3] +144*p[0]*p[1] ...
sol = Sol(energy=0, p[0]=1, p[1]=1, q[0]=1, q[1]=0, q[2]=0, q[3]=0)
p = 5
q = 7
```
出力から、式 `f` が4次の項を含むことが確認でき、これが HUBO 式であることがわかります。
ソルバーは素因数 $p=5$ と $q=7$ を正しく求めています。

> **注意**
> PyQBPP はすべての係数とエネルギーの計算に任意精度整数演算（Pythonネイティブの `int`）を使用します。
> C++版とは異なり、`COEFF_TYPE` や `ENERGY_TYPE` マクロを指定する必要はありません。
> 大きな合成数の素因数分解も、特別な設定なしにそのまま動作します。
</div>
