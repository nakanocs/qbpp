---
layout: default
nav_exclude: true
title: "Square Root"
nav_order: 3
---
<div class="lang-en" markdown="1">
# Square Root

This example demonstrates how to compute the square root of
$c=2$ using large integers represented by `qbpp::cpp_int`.
Let $s = 10 ^{10}$ be a fixed integer.
Since QUBO++ cannot handle real numbers directly, we compute  $\sqrt{cs^2}$ instead of $\sqrt{c}$.
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

In QUBO++, this equality constraint is converted into the following HUBO expression:

$$
(x ^ 2 -cs^2)^2
$$

By finding the value of $x$ that minimizes this expression,
we obtain an approximation of the square root of $c$ with
10 decimal-digit precision.
Since $x$ is internally represented as a linear expression of binary variables, this objective function becomes quartic in those binary variables.

## QUBO++ parogram
The following QUBO++ program constructs a HUBO expression based on the above idea and solves it using the Easy Solver:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int c = 2;
  auto s = qbpp::cpp_int("10000000000");
  auto x = s <= qbpp::var_int("x") <= c * s;
  auto f = x * x == c * s * s;
  f.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "Energy = " << sol.energy() << std::endl;
  std::cout << "x = " << x << "\n = " << sol(x) << std::endl;
}
```

Since very large coefficients are used, we define the two macros `COEFF_TYPE` and `ENERGY_TYPE` as `qbpp::cpp_int`,
which can represent arbitrarily large integers.
The constant `s`, the integer variable `x`, and the HUBO expression `f` are defined according to the formulation described above.
The Easy Solver is executed with a time limit of 1.0 second.

This program produces the following output:
```
Energy = 57910111919782629376
x = 10000000000 +x[0] +2*x[1] +4*x[2] +8*x[3] +16*x[4] +32*x[5] +64*x[6] +128*x[7] +256*x[8] +512*x[9] +1024*x[10] +2048*x[11] +4096*x[12] +8192*x[13] +16384*x[14] +32768*x[15] +65536*x[16] +131072*x[17] +262144*x[18] +524288*x[19] +1048576*x[20] +2097152*x[21] +4194304*x[22] +8388608*x[23] +16777216*x[24] +33554432*x[25] +67108864*x[26] +134217728*x[27] +268435456*x[28] +536870912*x[29] +1073741824*x[30] +2147483648*x[31] +4294967296*x[32] +1410065409*x[33]
 = 14142135624
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

この例では、`qbpp::cpp_int` で表現される大きな整数を用いて、$c=2$ の平方根を計算する方法を示します。
$s = 10^{10}$ を固定の整数とします。
QUBO++ は実数を直接扱えないため、$\sqrt{c}$ の代わりに $\sqrt{cs^2}$ を計算します。
以下の関係式から、

$$
\begin{aligned}
\sqrt{c} &= \sqrt{cs^2}/s
\end{aligned}
$$

$\sqrt{c}$ の10桁精度の近似値を得ることができます。

## 平方根計算のHUBO定式化
整数変数 $x$ を $[s, 2s]$ の範囲で定義します。
次の等式を用いて問題を定式化します:

$$
\begin{aligned}
x ^ 2 &= cs ^ 2
\end{aligned}
$$

QUBO++ では、この等式制約は以下のHUBO式に変換されます:

$$
(x ^ 2 -cs^2)^2
$$

この式を最小化する $x$ の値を求めることで、$c$ の平方根の10桁精度の近似値が得られます。
$x$ は内部的にバイナリ変数の線形式として表現されるため、この目的関数はバイナリ変数に関して4次式になります。

## QUBO++ プログラム
以下のQUBO++プログラムは、上記のアイデアに基づいてHUBO式を構築し、Easy Solverを用いて解きます:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int c = 2;
  auto s = qbpp::cpp_int("10000000000");
  auto x = s <= qbpp::var_int("x") <= c * s;
  auto f = x * x == c * s * s;
  f.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "Energy = " << sol.energy() << std::endl;
  std::cout << "x = " << x << "\n = " << sol(x) << std::endl;
}
```

非常に大きな係数を使用するため、2つのマクロ `COEFF_TYPE` と `ENERGY_TYPE` を `qbpp::cpp_int` として定義しています。これにより任意精度の整数を表現できます。
定数 `s`、整数変数 `x`、HUBO式 `f` は上述の定式化に従って定義されています。
Easy Solverは制限時間1.0秒で実行されます。

このプログラムの出力は以下のとおりです:
```
Energy = 57910111919782629376
x = 10000000000 +x[0] +2*x[1] +4*x[2] +8*x[3] +16*x[4] +32*x[5] +64*x[6] +128*x[7] +256*x[8] +512*x[9] +1024*x[10] +2048*x[11] +4096*x[12] +8192*x[13] +16384*x[14] +32768*x[15] +65536*x[16] +131072*x[17] +262144*x[18] +524288*x[19] +1048576*x[20] +2097152*x[21] +4194304*x[22] +8388608*x[23] +16777216*x[24] +33554432*x[25] +67108864*x[26] +134217728*x[27] +268435456*x[28] +536870912*x[29] +1073741824*x[30] +2147483648*x[31] +4294967296*x[32] +1410065409*x[33]
 = 14142135624
```
Easy Solverが正しい近似値を出力していることが確認できます:

$$
 \sqrt{2\times 10^{20}}\approx 14142135624
$$

報告されたエネルギー値がゼロではなく、等式制約が厳密には満たされていないことに注意してください。
これは、等式を厳密に満たす整数解が存在しないためです。
代わりに、ソルバーは等式制約の誤差を最小化する解を見つけます。
出力に表示されるエネルギー値は、この誤差の二乗に相当します。
誤差が最小化されるため、得られた $x$ の値は平方根の近似値を表しています。
</div>
