---
layout: default
nav_exclude: true
title: "Remainder Problem"
nav_order: 41
---
<div class="lang-en" markdown="1">
# Remainder Problem
The following problem can be solved using PyQBPP.
Find the minimum non-negative integer $x$ such that

- the remainder when $x$ is divided by 3 is 2,
- the remainder when $x$ is divided by 5 is 3, and
- the remainder when $x$ is divided by 7 is 5.

Since 3, 5, and 7 are pairwise coprime, it is enough to search $x$ within one period:

$$
 0\leq x \leq 3\times 5\times 7 -1
$$

Introduce non-negative integers $d_3$, $d_5$, and $d_7$ (quotients) and rewrite the remainder conditions as linear equalities:

$$
\begin{aligned}
 x - 3d_3 &= 2 \\
 x - 5d_5 &=3 \\
 x - 7d_7 &= 5
\end{aligned}
$$

We want to minimize $x$ subject to these constraints.
From the range $x$ above, we can bound the quotient variables as

$$
\begin{aligned}
 0&\leq d_3 \leq 5\times 7-1 \\
 0&\leq d_5 \leq 3\times 7-1 \\
 0&\leq d_7 \leq 3\times 5-1
\end{aligned}
$$

## PyQBPP program
The following program finds a solution $x$ for this remainder problem:
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x"), 0, 3 * 5 * 7 - 1)
d3 = between(var_int("d3"), 0, 5 * 7 - 1)
d5 = between(var_int("d5"), 0, 3 * 7 - 1)
d7 = between(var_int("d7"), 0, 3 * 5 - 1)
c3 = x - 3 * d3 == 2
c5 = x - 5 * d5 == 3
c7 = x - 7 * d7 == 5
f = x + 1000 * (c3 + c5 + c7)
f.simplify_as_binary()

solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

print(f"x = {sol.eval(x)}")
print(f"{sol.eval(x)} - 3 * {sol.eval(d3)} = {sol.eval(c3.body)}")
print(f"{sol.eval(x)} - 5 * {sol.eval(d5)} = {sol.eval(c5.body)}")
print(f"{sol.eval(x)} - 7 * {sol.eval(d7)} = {sol.eval(c7.body)}")
```

The three constraints are represented as `c3`, `c5`, and `c7`.
Each of them is converted into a QUBO penalty term that becomes 0 when the corresponding equality holds.

We then minimize `x` with a large penalty weight (1000) so that satisfying the constraints is prioritized over reducing `x`.

Finally, the Easy Solver searches for a low-energy solution of f within the time limit (1.0 second), and the obtained values are printed as follows:
```
x = 68
68 - 3 * 22 = 2
68 - 5 * 13 = 3
68 - 7 * 9 = 5
```
Therefore,

$$
\begin{aligned}
x &\equiv 68 & (\bmod 105)
\end{aligned}
$$

and the minimum solution is $x=68$.
</div>

<div class="lang-ja" markdown="1">
# 余り問題
以下の問題は PyQBPP を使って解くことができます。
次の条件を満たす最小の非負整数 $x$ を求めます:

- $x$ を 3 で割った余りが 2、
- $x$ を 5 で割った余りが 3、
- $x$ を 7 で割った余りが 5。

3、5、7 は互いに素であるため、1周期内で $x$ を探索すれば十分です:

$$
 0\leq x \leq 3\times 5\times 7 -1
$$

非負整数 $d_3$、$d_5$、$d_7$（商）を導入し、余りの条件を線形等式として書き直します:

$$
\begin{aligned}
 x - 3d_3 &= 2 \\
 x - 5d_5 &=3 \\
 x - 7d_7 &= 5
\end{aligned}
$$

これらの制約の下で $x$ を最小化したいです。
上記の $x$ の範囲から、商の変数を以下のように制限できます:

$$
\begin{aligned}
 0&\leq d_3 \leq 5\times 7-1 \\
 0&\leq d_5 \leq 3\times 7-1 \\
 0&\leq d_7 \leq 3\times 5-1
\end{aligned}
$$

## PyQBPP プログラム
以下のプログラムは、この余り問題の解 $x$ を求めます:
```python
from pyqbpp import var_int, between, EasySolver

x = between(var_int("x"), 0, 3 * 5 * 7 - 1)
d3 = between(var_int("d3"), 0, 5 * 7 - 1)
d5 = between(var_int("d5"), 0, 3 * 7 - 1)
d7 = between(var_int("d7"), 0, 3 * 5 - 1)
c3 = x - 3 * d3 == 2
c5 = x - 5 * d5 == 3
c7 = x - 7 * d7 == 5
f = x + 1000 * (c3 + c5 + c7)
f.simplify_as_binary()

solver = EasySolver(f)
solver.time_limit(1.0)
sol = solver.search()

print(f"x = {sol.eval(x)}")
print(f"{sol.eval(x)} - 3 * {sol.eval(d3)} = {sol.eval(c3.body)}")
print(f"{sol.eval(x)} - 5 * {sol.eval(d5)} = {sol.eval(c5.body)}")
print(f"{sol.eval(x)} - 7 * {sol.eval(d7)} = {sol.eval(c7.body)}")
```

3つの制約は `c3`、`c5`、`c7` として表現されています。
それぞれは、対応する等式が成り立つときに 0 となる QUBO ペナルティ項に変換されます。

大きなペナルティ重み（1000）を使って `x` を最小化することで、`x` の削減よりも制約の充足が優先されます。

最後に、Easy Solver が制限時間（1.0秒）内で f の低エネルギー解を探索し、得られた値は以下のように出力されます:
```
x = 68
68 - 3 * 22 = 2
68 - 5 * 13 = 3
68 - 7 * 9 = 5
```
したがって、

$$
\begin{aligned}
x &\equiv 68 & (\bmod 105)
\end{aligned}
$$

最小の解は $x=68$ です。
</div>
