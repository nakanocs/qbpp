---
layout: default
nav_exclude: true
title: "3-Digit Math"
nav_order: 6
---
<div class="lang-en" markdown="1">
# 3-Digit Math Problem

Let us solve the following math problem using QUBO++.

> **Math Problem**:
> Find all three-digit odd integers whose **product of digits** is **252**.

Let $x$, $y$, and $z$ be the hundreds, tens, and ones digits of the integer, respectively.
More specifically:
- $x$ is an integer in $[1, 9]$,
- $y$ is an integer in $[0, 9]$,
- $t$ is an integer in $[0, 4]$,
- $z = 2t + 1$ (so $z$ is odd).

Then the value $v$ of the three-digit integer $xyz$ is

$$
\begin{aligned}
v&=100x+10y+z
\end{aligned}
$$


We find all solutions satisfying:

$$
\begin{aligned}
xyz &= 252
\end{aligned}
$$

## QUBO++ program
The following QUBO++ program finds all solutions:
```cpp
#include <set>

#define MAXDEG 6
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 9;
  auto y = 0 <= qbpp::var_int("y") <= 9;
  auto t = 0 <= qbpp::var_int("t") <= 4;
  auto z = 2 * t + 1;
  auto v = x * 100 + y * 10 + z;

  auto f = x * y * z == 252;

  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  std::set<int> s;
  for (const auto& sol : sols) {
    s.insert(static_cast<int>(sol(v)));
  }
  for (auto v : s) {
    std::cout << v << " ";
  }
  std::cout << std::endl;
}
```
In this program, **`x`**, **`y`**, and **`t`** are defined as integer variables with the ranges above.
Then **`z`**, **`v`**, and **`f`** are defined as expressions.
We create an Exhaustive Solver instance for `f` and store all optimal solutions in `sols`.

Because `x`, `y`, and `t` are encoded by multiple binary variables, different binary assignments can represent the same integer values.
As a result, the same digit triple (`x`,`y`,`z`) may appear multiple times in `sols`.
Therefore, we use a `std::set<int>` named `s` to remove duplicates by collecting only the resulting integer values `v`.

The integers in `s` are printed as follows:
```
479 497 667 749 947
```
</div>

<div class="lang-ja" markdown="1">
# 3桁の数学問題

以下の数学問題をQUBO++を使って解きましょう。

> **数学問題**:
> **各桁の積**が **252** となる3桁の奇数をすべて求めよ。

$x$、$y$、$z$ をそれぞれ百の位、十の位、一の位の数字とします。
より具体的には:
- $x$ は $[1, 9]$ の整数、
- $y$ は $[0, 9]$ の整数、
- $t$ は $[0, 4]$ の整数、
- $z = 2t + 1$（$z$ は奇数）。

3桁の整数 $xyz$ の値 $v$ は

$$
\begin{aligned}
v&=100x+10y+z
\end{aligned}
$$


次の条件を満たすすべての解を求めます:

$$
\begin{aligned}
xyz &= 252
\end{aligned}
$$

## QUBO++プログラム
以下のQUBO++プログラムはすべての解を求めます:
```cpp
#include <set>

#define MAXDEG 6
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 9;
  auto y = 0 <= qbpp::var_int("y") <= 9;
  auto t = 0 <= qbpp::var_int("t") <= 4;
  auto z = 2 * t + 1;
  auto v = x * 100 + y * 10 + z;

  auto f = x * y * z == 252;

  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  std::set<int> s;
  for (const auto& sol : sols) {
    s.insert(static_cast<int>(sol(v)));
  }
  for (auto v : s) {
    std::cout << v << " ";
  }
  std::cout << std::endl;
}
```
このプログラムでは、**`x`**、**`y`**、**`t`** が上記の範囲を持つ整数変数として定義されています。
次に **`z`**、**`v`**、**`f`** が式として定義されます。
`f` に対するExhaustive Solverインスタンスを作成し、すべての最適解を `sols` に格納します。

`x`、`y`、`t` は複数のバイナリ変数で符号化されるため、異なるバイナリ割り当てが同じ整数値を表す場合があります。
その結果、同じ桁の組 (`x`,`y`,`z`) が `sols` に複数回現れることがあります。
そのため、結果の整数値 `v` のみを収集する `std::set<int>` 型の `s` を使用して重複を除去しています。

`s` 内の整数は以下のように出力されます:
```
479 497 667 749 947
```
</div>
