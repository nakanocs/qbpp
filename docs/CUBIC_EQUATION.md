---
layout: default
title: "Cubic Equation"
---

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
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

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
```cpp
x = -100 +x[0] +2*x[1] +4*x[2] +8*x[3] +16*x[4] +32*x[5] +64*x[6] +73*x[7]
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

