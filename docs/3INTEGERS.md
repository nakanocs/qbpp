---
layout: default
title: "Math Problem: Find Three Integers"
---

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
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

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
