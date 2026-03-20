---
layout: default
nav_exclude: true
title: "Greatest Common Divisor"
nav_order: 43
---
<div class="lang-en" markdown="1">
# Greatest Common Divisor (GCD)
Let $P$ and $Q$ be two positive integers.
The computation of the **greatest common divisor (GCD)** can be formulated as a HUBO problem.

Let $p$, $q$, and $r$ be positive integers satisfying the following constraints:

$$
\begin{aligned}
  p\cdot r &= P \\
  q\cdot r &=Q
\end{aligned}
$$

Clearly, $r$ is a common divisor of $P$ and $Q$.
Therefore, the maximum value of $r$ satisfying these constraints is the GCD of $P$ and $Q$.
To find such an $r$, we use $-r$ as the objective function in the HUBO formulation.

## PyQBPP program
Based on the idea above, the following PyQBPP program computes the GCD of two integers,
`P = 858` and `Q = 693`:
```python
import pyqbpp as qbpp

P = 858
Q = 693
p = qbpp.between(qbpp.var_int("p"), 1, 1000)
q = qbpp.between(qbpp.var_int("q"), 1, 1000)
r = qbpp.between(qbpp.var_int("r"), 1, 1000)

constraint = (p * r == Q) + (q * r == P)
f = -r + constraint * 1000

f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

print(f"GCD = {sol(r)}")
print(f"{sol(p)} * {sol(r)} = {P}")
print(f"{sol(q)} * {sol(r)} = {Q}")
```
In this program, `p`, `q`, and `r` are defined as integer variables in the range $[1,1000]$.
The expression constraint is constructed so that it evaluates to zero when both constraints are satisfied.

The objective function `-r` is combined with the constraint term multiplied by a penalty factor of `1000`, and the resulting expression is stored in `f`.

The EasySolver searches for a solution that minimizes `f`.
The resulting values of `p`, `q`, and `r` are printed as follows:
```
GCD = 33
21 * 33 = 858
26 * 33 = 693
```
This output confirms that the GCD of 858 and 693 is correctly obtained as 33.
</div>

<div class="lang-ja" markdown="1">
# 最大公約数 (GCD)
$P$ と $Q$ を2つの正の整数とします。
**最大公約数 (GCD)** の計算は HUBO 問題として定式化できます。

$p$、$q$、$r$ を以下の制約を満たす正の整数とします:

$$
\begin{aligned}
  p\cdot r &= P \\
  q\cdot r &=Q
\end{aligned}
$$

明らかに、$r$ は $P$ と $Q$ の公約数です。
したがって、これらの制約を満たす $r$ の最大値が $P$ と $Q$ の GCD です。
そのような $r$ を求めるために、HUBO 定式化において $-r$ を目的関数として使用します。

## PyQBPP プログラム
上記の考え方に基づき、以下の PyQBPP プログラムは2つの整数 `P = 858` と `Q = 693` の GCD を計算します:
```python
import pyqbpp as qbpp

P = 858
Q = 693
p = qbpp.between(qbpp.var_int("p"), 1, 1000)
q = qbpp.between(qbpp.var_int("q"), 1, 1000)
r = qbpp.between(qbpp.var_int("r"), 1, 1000)

constraint = (p * r == Q) + (q * r == P)
f = -r + constraint * 1000

f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

print(f"GCD = {sol(r)}")
print(f"{sol(p)} * {sol(r)} = {P}")
print(f"{sol(q)} * {sol(r)} = {Q}")
```
このプログラムでは、`p`、`q`、`r` は範囲 $[1,1000]$ の整数変数として定義されています。
式 constraint は、両方の制約が満たされたときにゼロと評価されるように構築されています。

目的関数 `-r` はペナルティ係数 `1000` を掛けた制約項と組み合わされ、結果の式は `f` に格納されます。

EasySolver は `f` を最小化する解を探索します。
得られた `p`、`q`、`r` の値は以下のように出力されます:
```
GCD = 33
21 * 33 = 858
26 * 33 = 693
```
この出力から、858 と 693 の GCD が 33 として正しく求められたことが確認できます。
</div>
