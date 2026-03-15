---
layout: default
nav_exclude: true
title: "Greatest Common Divisor"
nav_order: 4
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

## QUBO++ program
Based on the idea above, the following QUBO++ program computes the GCD of two integers,
`P = 858` and `Q = 693`:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int P = 858;
  const int Q = 693;
  auto p = 1 <= qbpp::var_int("p") <= 1000;
  auto q = 1 <= qbpp::var_int("q") <= 1000;
  auto r = 1 <= qbpp::var_int("r") <= 1000;

  auto constraint = (p * r == Q) + (q * r == P);
  auto f = -r + constraint * 1000;

  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();

  std::cout << "GCD = " << sol(r) << std::endl;
  std::cout << sol(p) << " * " << sol(r) << " = " << P << std::endl;
  std::cout << sol(q) << " * " << sol(r) << " = " << Q << std::endl;
}
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
**最大公約数 (GCD)** の計算はHUBO問題として定式化できます。

$p$、$q$、$r$ を以下の制約を満たす正の整数とします:

$$
\begin{aligned}
  p\cdot r &= P \\
  q\cdot r &=Q
\end{aligned}
$$

明らかに、$r$ は $P$ と $Q$ の公約数です。
したがって、これらの制約を満たす $r$ の最大値が $P$ と $Q$ の最大公約数となります。
そのような $r$ を求めるために、HUBO定式化において $-r$ を目的関数として使用します。

## QUBO++ プログラム
上記のアイデアに基づき、以下のQUBO++プログラムは2つの整数 `P = 858` と `Q = 693` の最大公約数を計算します:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int P = 858;
  const int Q = 693;
  auto p = 1 <= qbpp::var_int("p") <= 1000;
  auto q = 1 <= qbpp::var_int("q") <= 1000;
  auto r = 1 <= qbpp::var_int("r") <= 1000;

  auto constraint = (p * r == Q) + (q * r == P);
  auto f = -r + constraint * 1000;

  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();

  std::cout << "GCD = " << sol(r) << std::endl;
  std::cout << sol(p) << " * " << sol(r) << " = " << P << std::endl;
  std::cout << sol(q) << " * " << sol(r) << " = " << Q << std::endl;
}
```
このプログラムでは、`p`、`q`、`r` は $[1,1000]$ の範囲の整数変数として定義されています。
式 constraint は、両方の制約が満たされたときにゼロに評価されるように構成されています。

目的関数 `-r` はペナルティ係数 `1000` を掛けた制約項と組み合わされ、結果の式が `f` に格納されます。

EasySolverは `f` を最小化する解を探索します。
得られた `p`、`q`、`r` の値は以下のように出力されます:
```
GCD = 33
21 * 33 = 858
26 * 33 = 693
```
この出力から、858と693の最大公約数が正しく33として得られたことが確認できます。
</div>
