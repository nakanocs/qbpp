---
layout: default
nav_exclude: true
title: "Interval Subset Sum (PyQBPP)"
nav_order: 75
---
<div class="lang-en" markdown="1">
# Interval Subset Sum Problem (ISSP)
The **Interval Subset Sum Problem (ISSP)** is a generalization of the **Subset Sum Problem**.
Given $n$ integer **intervals $[l_i, u_i]$** $(0\leq i\leq n-1)$ and an **upper bound $T$**, the goal is to choose an integer value

$$
\begin{aligned}
v_i &\in \lbrace 0\rbrace \cup [l_i, u_i] && (i = 0,1,\dots,n-1),
\end{aligned}
$$

so as to satisfy the constraint $\sum_{i=0}^{n-1} v_i \leq T$ and maximize the objective $\sum_{i=0}^{n-1} v_i$.

## HUBO formulation
We introduce binary variables $s_i$ and integer variables $v_i$. The product $s_i v_i$ gives the selected value.

## PyQBPP program (HUBO formulation)
```python
import pyqbpp as qbpp

lower = [18, 17, 21, 18, 20, 14, 14, 23]
upper = [19, 17, 22, 19, 20, 16, 15, 25]
T = 100
n = len(lower)

v = [qbpp.between(qbpp.var_int(f"v{i}"), lower[i], upper[i]) for i in range(n)]
s = qbpp.var("s", n)

total = qbpp.sum(qbpp.Vector(v) * s)
constraint = qbpp.between(total, 0, T)
f = -total + 1000 * constraint
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(-T)
sol = solver.search()
for i in range(n):
    if sol(s[i]) == 1:
        print(f"Interval {i}: val = {sol(v[i])}")
print(f"sum = {sol(total)}")
```
This program produces the following output:
```
Interval 0: val = 18
Interval 1: val = 17
Interval 2: val = 22
Interval 4: val = 20
Interval 7: val = 23
sum = 100
```

## QUBO formulation
To avoid quartic terms, we introduce auxiliary integer variables $a_i \in [0, u_i - l_i]$ and define $v_i = l_i s_i + a_i$.
The penalty `constraint1 = sum((1 - s) * a)` enforces $a_i = 0$ when $s_i = 0$.

```python
import pyqbpp as qbpp

lower = [18, 17, 21, 18, 20, 14, 14, 23]
upper = [19, 17, 22, 19, 20, 16, 15, 25]
T = 100
n = len(lower)

a = [qbpp.between(qbpp.var_int(f"a{i}"), 0, upper[i] - lower[i]) for i in range(n)]
s = qbpp.var("s", n)
v = [s[i] * lower[i] + a[i] for i in range(n)]

total = toExpr(0)
for i in range(n):
    total += v[i]

constraint1 = toExpr(0)
for i in range(n):
    constraint1 += (1 - s[i]) * a[i]

constraint2 = qbpp.between(total, 0, T)
f = -total + 1000 * (constraint1 + constraint2)
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(-T)
sol = solver.search()
for i in range(n):
    if sol(s[i]) == 1:
        print(f"Interval {i}: val = {sol(v[i])}")
print(f"sum = {sol(total)}")
```
This produces the same result as the HUBO formulation.
</div>

<div class="lang-ja" markdown="1">
# 区間部分和問題 (ISSP)
**区間部分和問題 (ISSP)** は**部分和問題**の一般化です。
$n$ 個の整数**区間 $[l_i, u_i]$** $(0\leq i\leq n-1)$ と**上限 $T$** が与えられたとき、整数値

$$
\begin{aligned}
v_i &\in \lbrace 0\rbrace \cup [l_i, u_i] && (i = 0,1,\dots,n-1),
\end{aligned}
$$

を選び、制約 $\sum_{i=0}^{n-1} v_i \leq T$ を満たしつつ、目的関数 $\sum_{i=0}^{n-1} v_i$ を最大化します。

## HUBO定式化
バイナリ変数 $s_i$ と整数変数 $v_i$ を導入します。積 $s_i v_i$ が選択された値を表します。

## PyQBPPプログラム（HUBO定式化）
```python
import pyqbpp as qbpp

lower = [18, 17, 21, 18, 20, 14, 14, 23]
upper = [19, 17, 22, 19, 20, 16, 15, 25]
T = 100
n = len(lower)

v = [qbpp.between(qbpp.var_int(f"v{i}"), lower[i], upper[i]) for i in range(n)]
s = qbpp.var("s", n)

total = qbpp.sum(qbpp.Vector(v) * s)
constraint = qbpp.between(total, 0, T)
f = -total + 1000 * constraint
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(-T)
sol = solver.search()
for i in range(n):
    if sol(s[i]) == 1:
        print(f"Interval {i}: val = {sol(v[i])}")
print(f"sum = {sol(total)}")
```
このプログラムの出力は以下の通りです：
```
Interval 0: val = 18
Interval 1: val = 17
Interval 2: val = 22
Interval 4: val = 20
Interval 7: val = 23
sum = 100
```

## QUBO定式化
4次の項を避けるため、補助整数変数 $a_i \in [0, u_i - l_i]$ を導入し、$v_i = l_i s_i + a_i$ と定義します。
ペナルティ `constraint1 = sum((1 - s) * a)` は $s_i = 0$ のとき $a_i = 0$ を強制します。

```python
import pyqbpp as qbpp

lower = [18, 17, 21, 18, 20, 14, 14, 23]
upper = [19, 17, 22, 19, 20, 16, 15, 25]
T = 100
n = len(lower)

a = [qbpp.between(qbpp.var_int(f"a{i}"), 0, upper[i] - lower[i]) for i in range(n)]
s = qbpp.var("s", n)
v = [s[i] * lower[i] + a[i] for i in range(n)]

total = toExpr(0)
for i in range(n):
    total += v[i]

constraint1 = toExpr(0)
for i in range(n):
    constraint1 += (1 - s[i]) * a[i]

constraint2 = qbpp.between(total, 0, T)
f = -total + 1000 * (constraint1 + constraint2)
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.target_energy(-T)
sol = solver.search()
for i in range(n):
    if sol(s[i]) == 1:
        print(f"Interval {i}: val = {sol(v[i])}")
print(f"sum = {sol(total)}")
```
HUBO定式化と同じ結果が得られます。
</div>
