---
layout: default
nav_exclude: true
title: "Remainder Problem"
nav_order: 2
---
<div class="lang-en" markdown="1">
# Remainder Problem
The following problem can be solved using QUBO++.
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

## QUBO++ praogram
The following program finds a solution $x$ for this remainder problem:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto x = 0 <= qbpp::var_int("x") <= 3 * 5 * 7 - 1;
  auto d3 = 0 <= qbpp::var_int("d3") <= 5 * 7 - 1;
  auto d5 = 0 <= qbpp::var_int("d5") <= 3 * 7 - 1;
  auto d7 = 0 <= qbpp::var_int("d7") <= 3 * 5 - 1;
  auto c3 = x - 3 * d3 == 2;
  auto c5 = x - 5 * d5 == 3;
  auto c7 = x - 7 * d7 == 5;
  auto f = x + 1000 * (c3 + c5 + c7);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();

  std::cout << "x = " << sol(x) << std::endl;
  std::cout << sol(x) << " - 3 * " << sol(d3) << " = " << sol(*c3) << std::endl;
  std::cout << sol(x) << " - 5 * " << sol(d5) << " = " << sol(*c5) << std::endl;
  std::cout << sol(x) << " - 7 * " << sol(d7) << " = " << sol(*c7) << std::endl;
}
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
# 剰余問題
以下の問題はQUBO++を用いて解くことができます。
次の条件を満たす最小の非負整数 $x$ を求めます:

- $x$ を3で割った余りが2
- $x$ を5で割った余りが3
- $x$ を7で割った余りが5

3、5、7は互いに素であるため、1周期内で $x$ を探索すれば十分です:

$$
 0\leq x \leq 3\times 5\times 7 -1
$$

非負整数 $d_3$、$d_5$、$d_7$（商）を導入し、剰余条件を線形等式として書き直します:

$$
\begin{aligned}
 x - 3d_3 &= 2 \\
 x - 5d_5 &=3 \\
 x - 7d_7 &= 5
\end{aligned}
$$

これらの制約の下で $x$ を最小化したいです。
上記の $x$ の範囲から、商の変数は以下のように制限できます:

$$
\begin{aligned}
 0&\leq d_3 \leq 5\times 7-1 \\
 0&\leq d_5 \leq 3\times 7-1 \\
 0&\leq d_7 \leq 3\times 5-1
\end{aligned}
$$

## QUBO++ プログラム
以下のプログラムは、この剰余問題の解 $x$ を求めます:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto x = 0 <= qbpp::var_int("x") <= 3 * 5 * 7 - 1;
  auto d3 = 0 <= qbpp::var_int("d3") <= 5 * 7 - 1;
  auto d5 = 0 <= qbpp::var_int("d5") <= 3 * 7 - 1;
  auto d7 = 0 <= qbpp::var_int("d7") <= 3 * 5 - 1;
  auto c3 = x - 3 * d3 == 2;
  auto c5 = x - 5 * d5 == 3;
  auto c7 = x - 7 * d7 == 5;
  auto f = x + 1000 * (c3 + c5 + c7);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();

  std::cout << "x = " << sol(x) << std::endl;
  std::cout << sol(x) << " - 3 * " << sol(d3) << " = " << sol(*c3) << std::endl;
  std::cout << sol(x) << " - 5 * " << sol(d5) << " = " << sol(*c5) << std::endl;
  std::cout << sol(x) << " - 7 * " << sol(d7) << " = " << sol(*c7) << std::endl;
}
```

3つの制約は `c3`、`c5`、`c7` として表現されています。
それぞれは、対応する等式が成り立つときに0になるQUBOペナルティ項に変換されます。

次に、制約の充足を $x$ の削減よりも優先するために、大きなペナルティ重み（1000）を用いて `x` を最小化します。

最後に、Easy Solverが制限時間（1.0秒）内で f の低エネルギー解を探索し、得られた値は以下のように出力されます:
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
