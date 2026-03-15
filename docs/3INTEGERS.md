---
layout: default
nav_exclude: true
title: "Find Three Integers"
nav_order: 5
---
<div class="lang-en" markdown="1">
# Math Problem: Find Three Intgers

The following math problem can be solved using QUBO++.

### Problem
Find integers $x$, $y$, $z$ that satisfy:

$$
\begin{aligned}
\frac{1}{x}+\frac{1}{y}+\frac{1}{z} = 1\\
1 < x < y < z
\end{aligned}
$$



### QUBO++ program

Since QUBO++ can handle polynomial expressions, we first rewrite the constraints.
Multiplying both sides of the first constraint by $xyz$ yields:

$$
xy+yz+zx - xyz = 0
$$

The strict inequalities $x<y<z$ can be encoded as

$$
\begin{aligned}
1 &\leq y-z \\
1 &\leq z-y
\end{aligned}
$$

The following QUBO++ program formulates these constraints as a HUBO expression and solves it using the Exhaustive Solver:

```cpp
#define MAXDEG 6
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 10;
  auto y = 1 <= qbpp::var_int("y") <= 10;
  auto z = 1 <= qbpp::var_int("z") <= 10;

  auto c1 = x * y + y * z + z * x - x * y * z == 0;
  auto c2 = 1 <= y - x <= +qbpp::inf;
  auto c3 = 1 <= z - y <= +qbpp::inf;

  auto f = c1 + c2 + c3;
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();

  std::set<std::tuple<qbpp::energy_t, qbpp::energy_t, qbpp::energy_t>> seen;
  for (const auto& sol : sols) {
    const auto key = std::make_tuple(x(sol), y(sol), z(sol));
    if (seen.insert(key).second) {
      auto [x_val, y_val, z_val] = key;
      std::cout << "(x,y,z) = (" << x_val << ", " << y_val << ", " << z_val
                << ")\n";
    }
  }
}
```
The three constraints are encoded as `c1`, `c2`, and `c3`, and combined into a single objective `f`.
The Exhaustive Solver searches for optimal solutions of f and prints the resulting
$(x,y,z)$ tuples.

Because `f` introduces auxiliary variables during binary simplification, the same
$(x,y,z)$ assignment may appear multiple times in the returned solution set.
Therefore, we use a `std::set` to remove duplicates before printing.

This program produces the following output:
```
(x,y,z) = (2, 3, 6)
```
This indicates that the problem has exactly one solution in the searched range, namely $(x,y,z)=(2,3,6)$.
</div>

<div class="lang-ja" markdown="1">
# 数学問題: 3つの整数を求める

以下の数学問題をQUBO++を使って解くことができます。

### 問題
次の条件を満たす整数 $x$, $y$, $z$ を求めよ:

$$
\begin{aligned}
\frac{1}{x}+\frac{1}{y}+\frac{1}{z} = 1\\
1 < x < y < z
\end{aligned}
$$



### QUBO++プログラム

QUBO++は多項式を扱えるため、まず制約を書き換えます。
最初の制約の両辺に $xyz$ を掛けると:

$$
xy+yz+zx - xyz = 0
$$

狭義の不等式 $x<y<z$ は次のように符号化できます:

$$
\begin{aligned}
1 &\leq y-z \\
1 &\leq z-y
\end{aligned}
$$

以下のQUBO++プログラムは、これらの制約をHUBO式として定式化し、Exhaustive Solverを使って解きます:

```cpp
#define MAXDEG 6
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 10;
  auto y = 1 <= qbpp::var_int("y") <= 10;
  auto z = 1 <= qbpp::var_int("z") <= 10;

  auto c1 = x * y + y * z + z * x - x * y * z == 0;
  auto c2 = 1 <= y - x <= +qbpp::inf;
  auto c3 = 1 <= z - y <= +qbpp::inf;

  auto f = c1 + c2 + c3;
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();

  std::set<std::tuple<qbpp::energy_t, qbpp::energy_t, qbpp::energy_t>> seen;
  for (const auto& sol : sols) {
    const auto key = std::make_tuple(x(sol), y(sol), z(sol));
    if (seen.insert(key).second) {
      auto [x_val, y_val, z_val] = key;
      std::cout << "(x,y,z) = (" << x_val << ", " << y_val << ", " << z_val
                << ")\n";
    }
  }
}
```
3つの制約は `c1`、`c2`、`c3` として符号化され、単一の目的関数 `f` にまとめられます。
Exhaustive Solverが `f` の最適解を探索し、得られた $(x,y,z)$ の組を出力します。

`f` はバイナリ簡約化の過程で補助変数を導入するため、同じ $(x,y,z)$ の割り当てが返される解集合に複数回現れる場合があります。
そのため、出力前に `std::set` を使って重複を除去しています。

このプログラムは以下の出力を生成します:
```
(x,y,z) = (2, 3, 6)
```
これは、探索範囲内でこの問題がちょうど1つの解 $(x,y,z)=(2,3,6)$ を持つことを示しています。
</div>
