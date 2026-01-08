---
layout: default
title: "Integer Linear Programming"
---

# Integer Linear Programming (ILP)
Integer Linear Programming (ILP) can be converted into a QUBO expression using QUBO++.
As an example, consider the following ILP:

$$
\begin{aligned}
\text{Maximize:} && 2x_0 +5x_1+5x_2\\
\text{Subject to:} && x_0 + 3 x_1 + x_2 &\leq 12 \\
                &&  x_0 + 2x_2 &\leq 5\\
                && x_1 + x_2 &\leq 4;
\end{aligned}
$$

## QUBO++ program
The following QUBO++ program formulates this ILP as a QUBO expression and solves it using the Easy Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = 0 <= qbpp::var_int("x", 3) <= 5;
  auto objective = 2 * x[0] + 5 * x[1] + 5 * x[2];
  auto c1 = 0 <= x[0] + 3 * x[1] + x[2] <= 12;
  auto c2 = 0 <= x[0] + 2 * x[2] <= 5;
  auto c3 = 0 <= x[1] + x[2] <= 4;

  auto f = -objective + 100 * (c1 + c2 + c3);
  f.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "x0 = " << sol(x[0]) << ", x1 = " << sol(x[1])
            << ", x2 = " << sol(x[2]) << std::endl;
  std::cout << "objective = " << sol(objective) << std::endl;
  std::cout << "*c1 = " << sol(*c1) << ", *c2 = " << sol(*c2)
            << ", *c3 = " << sol(*c3) << std::endl;
}
```
In this program, `x` is a vector of three `qbpp::VarInt` objects, each taking an integer value in the range $[0, 5]$.
The objective function and the three constraints are represented by `objective`, `c1`, `c2`, and `c3`, respectively.
They are combined into a single QUBO expression `f`, where the penalty constant `100` is used to enforce the constraints.

The Easy Solver searches for a low-energy solution of `f` and returns it as `sol`.
The obtained solution and the values of `objective`, `*c1`, `*c2`, and `*c3` are printed as follows:
```
x0 = 2, x1 = 3, x2 = 1
objective = 24
*c1 = 12, *c2 = 4, *c3 = 4
```
We observe that a obtained solution with the objective 24 satisfies all constraints.