---
layout: default
nav_exclude: true
title: "Pythagorean Triples"
nav_order: 1
---
<div class="lang-en" markdown="1">
# Pythagorean Triples

Three integers $x$, $y$, and $z$ are **Pythagorean triples** if they satisfy

$$
\begin{aligned}
x^2+y^2&=z^2
\end{aligned}
$$

To avoid duplicates, we assume $x<y$.

## QUBO++ program for listing Pythagorean Triples
The following program lists Pythagorean triples with $x\leq 16$, $y\leq 16$, and $z\leq 16$:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 16;
  auto y = 1 <= qbpp::var_int("y") <= 16;
  auto z = 1 <= qbpp::var_int("z") <= 16;
  auto f = x * x + y * y - z * z == 0;
  auto c = 1 <= y - x <= +qbpp::inf;
  auto g = f + c;
  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.time_limit(10.0);
  solver.enable_best_energy_sols(10);
  auto sols = solver.search();
  for (const auto& sol : sols) {
    std::cout << "x=" << sol(x) << ", y=" << sol(y) << ", z=" << sol(z)
              << ", *f=" << sol(*f) << ", *c=" << sol(*c) << std::endl;
  }
}
```
In this program, we define integer variables `x`, `y`, and `z` with ranges from 1 to 16.
We then create two constraint expressions:
- `f` for $x^2+y^2-z^2=0$, and
- `c` for $x+1\leq y$.

They are combined into `g`.
The expression `g` attains its minimum value 0 when all constraints are satisfied.

An Easy Solver object `solver` is created for `g` and configured with the following options:
- `time_limit(10.0)`: Terminates the search after 10 seconds.
- `enable_best_energy_sols(10)`: Keeps up to 10 solutions with the best (lowest) energy.

The call to `search()` returns a `qbpp::easy_solver::Sols` object named `sols`, which stores the best solutions.
Since `qbpp::easy_solver::Sols` provides iterator access to the stored best-energy solutions (`begin()`, `end()`, `cbegin()`, and `cend()`), they can be printed using a range-based for loop.

This program produces output like the following:
```
x=3, y=4, z=5, *f=0, *c=1
x=6, y=8, z=10, *f=0, *c=2
x=9, y=12, z=15, *f=0, *c=3
x=5, y=12, z=13, *f=0, *c=7
```
</div>

<div class="lang-ja" markdown="1">
# ピタゴラスの三つ組

3つの整数 $x$、$y$、$z$ が以下を満たすとき、**ピタゴラスの三つ組**と呼ばれます:

$$
\begin{aligned}
x^2+y^2&=z^2
\end{aligned}
$$

重複を避けるため、$x<y$ と仮定します。

## ピタゴラスの三つ組を列挙するQUBO++プログラム
以下のプログラムは、$x\leq 16$、$y\leq 16$、$z\leq 16$ の範囲でピタゴラスの三つ組を列挙します:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 16;
  auto y = 1 <= qbpp::var_int("y") <= 16;
  auto z = 1 <= qbpp::var_int("z") <= 16;
  auto f = x * x + y * y - z * z == 0;
  auto c = 1 <= y - x <= +qbpp::inf;
  auto g = f + c;
  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.time_limit(10.0);
  solver.enable_best_energy_sols(10);
  auto sols = solver.search();
  for (const auto& sol : sols) {
    std::cout << "x=" << sol(x) << ", y=" << sol(y) << ", z=" << sol(z)
              << ", *f=" << sol(*f) << ", *c=" << sol(*c) << std::endl;
  }
}
```
このプログラムでは、1から16の範囲の整数変数 `x`、`y`、`z` を定義しています。
次に、2つの制約式を作成します:
- `f`: $x^2+y^2-z^2=0$
- `c`: $x+1\leq y$

これらを `g` に結合します。
すべての制約が満たされたとき、式 `g` は最小値0を取ります。

`g` に対してEasy Solverオブジェクト `solver` を作成し、以下のオプションを設定します:
- `time_limit(10.0)`: 10秒後に探索を終了します。
- `enable_best_energy_sols(10)`: 最良（最低）エネルギーの解を最大10個保持します。

`search()` の呼び出しは、最良の解を格納する `qbpp::easy_solver::Sols` オブジェクト `sols` を返します。
`qbpp::easy_solver::Sols` は格納された最良エネルギー解へのイテレータアクセス（`begin()`、`end()`、`cbegin()`、`cend()`）を提供するため、範囲ベースのforループで出力できます。

このプログラムは以下のような出力を生成します:
```
x=3, y=4, z=5, *f=0, *c=1
x=6, y=8, z=10, *f=0, *c=2
x=9, y=12, z=15, *f=0, *c=3
x=5, y=12, z=13, *f=0, *c=7
```
</div>
