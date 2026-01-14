---
layout: default
title: "3-Digit Math Problem"
---

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

#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

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