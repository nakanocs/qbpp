---
layout: default
title: "Pythagorean Triples"
---


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
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

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
