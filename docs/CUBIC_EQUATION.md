---
layout: default
nav_exclude: true
title: "Cubic Equation"
nav_order: 7
---
<div class="lang-en" markdown="1">
# Cubic Equation
Cubic equations over the integers can be solved using QUBO++. For example, consider

$$
\begin{aligned}
x^3 -147x +286 &=0.
\end{aligned}
$$

This equation has three integer solutions: $x = -13, 2, 11$.

## QUBO++ program for solving the cubic equations
In the following QUBO++ program, we define an integer variable x that takes values in $[-100, 100]$, and we enumerate all optimal solutions using the Exhaustive Solver:
```cpp
#define COEFF_TYPE cpp_int
#define ENERGY_TYPE cpp_int

#define MAXDEG 6
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = -100 <= qbpp::var_int("x") <= 100;
  auto f = x * x * x - 147 * x + 286 == 0;
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();

  for (const auto& sol : sols) {
    std::cout << "x= " << x(sol) << " sol = " << sol << std::endl;
  }
}
```
The expression `f` corresponds to the following objective function:

$$
\begin{aligned}
f & = (x^3 -147x +286)^2
\end{aligned}
$$

Since the integer variable `x` is implemented as a linear expression of binary variables, `f` becomes a polynomial of degree 6.
This program produces the following output:
{% raw %}
```
x = 11 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],0},{x[7],1}}
x = 2 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
x = -13 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{x[4],0},{x[5],0},{x[6],0},{x[7],1}}
x = 2 sol = 0:{{x[0],1},{x[1],0},{x[2],1},{x[3],1},{x[4],1},{x[5],0},{x[6],0},{x[7],1}}
x = -13 sol = 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],0},{x[4],1},{x[5],0},{x[6],1},{x[7],0}}
x = 11 sol = 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
```
{% endraw %}
The first line indicates that the integer variable `x` is encoded using 8 binary variables.
Also, the program outputs 6 optimal solutions even though the original cubic equation has only 3 integer solutions.
This happens because the coefficient `73` of `x[7]` is not a power of two, so the same integer value can be represented by multiple different assignments of the binary variables encoding `x`.

To eliminate duplicate values of `x`, you can modify the program to use `std::unordered_set` as follows:
```cpp
#include <unordered_set>

... omitted ...

  std::unordered_set<qbpp::energy_t> seen;
  for (const auto& sol : sols) {
    const auto xv = x(sol);
    if (!seen.insert(xv).second) continue;
    std::cout << "x = " << xv << " sol = " << sol << "\n";
  }
```
This modified program outputs the following unique solutions:
{% raw %}
```
x = 11 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],0},{x[7],1}}
x = 2 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
x = -13 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{x[4],0},{x[5],0},{x[6],0},{x[7],1}}
```
{% endraw %}
</div>

<div class="lang-ja" markdown="1">
# 3次方程式
QUBO++ を用いて整数上の3次方程式を解くことができます。例えば、次の方程式を考えます:

$$
\begin{aligned}
x^3 -147x +286 &=0.
\end{aligned}
$$

この方程式には3つの整数解があります: $x = -13, 2, 11$。

## 3次方程式を解くQUBO++プログラム
以下のQUBO++プログラムでは、$[-100, 100]$ の範囲の整数変数 x を定義し、Exhaustive Solverを用いてすべての最適解を列挙します:
```cpp
#define COEFF_TYPE cpp_int
#define ENERGY_TYPE cpp_int

#define MAXDEG 6
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = -100 <= qbpp::var_int("x") <= 100;
  auto f = x * x * x - 147 * x + 286 == 0;
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();

  for (const auto& sol : sols) {
    std::cout << "x= " << x(sol) << " sol = " << sol << std::endl;
  }
}
```
式 `f` は以下の目的関数に対応します:

$$
\begin{aligned}
f & = (x^3 -147x +286)^2
\end{aligned}
$$

整数変数 `x` はバイナリ変数の線形式として実装されるため、`f` は6次の多項式になります。
このプログラムの出力は以下のとおりです:
{% raw %}
```
x = 11 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],0},{x[7],1}}
x = 2 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
x = -13 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{x[4],0},{x[5],0},{x[6],0},{x[7],1}}
x = 2 sol = 0:{{x[0],1},{x[1],0},{x[2],1},{x[3],1},{x[4],1},{x[5],0},{x[6],0},{x[7],1}}
x = -13 sol = 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],0},{x[4],1},{x[5],0},{x[6],1},{x[7],0}}
x = 11 sol = 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
```
{% endraw %}
最初の行は、整数変数 `x` が8個のバイナリ変数を用いてエンコードされていることを示しています。
また、元の3次方程式には3つの整数解しかないにもかかわらず、プログラムは6つの最適解を出力しています。
これは、`x[7]` の係数 `73` が2のべき乗ではないため、`x` をエンコードするバイナリ変数の異なる割り当てによって同じ整数値を表現できるためです。

`x` の重複した値を除去するには、`std::unordered_set` を使用するようにプログラムを以下のように変更します:
```cpp
#include <unordered_set>

... omitted ...

  std::unordered_set<qbpp::energy_t> seen;
  for (const auto& sol : sols) {
    const auto xv = x(sol);
    if (!seen.insert(xv).second) continue;
    std::cout << "x = " << xv << " sol = " << sol << "\n";
  }
```
この修正されたプログラムは、以下の重複のない解を出力します:
{% raw %}
```
x = 11 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],0},{x[7],1}}
x = 2 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
x = -13 sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{x[4],0},{x[5],0},{x[6],0},{x[7],1}}
```
{% endraw %}
</div>
