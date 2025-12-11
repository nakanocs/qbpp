---
layout: default
title: "RANGE"
---

# Range Constraints and Solving Integer Linear Programming

## Polynomial formulation for range constraints
Let $f$ be a polynomial expression of binary variables.
A range constraint has the form $l\leq f\leq u$ with $l<u$.
Our goal is to design a polynomial expression that takes the minimum value 0 if and only if the range constraint is satisfied.

The key idea is to introduce an **auxiliary integer variable** $a$ that takes values in the range $[l,u]$.
Consider the following expression:

$$
\begin{aligned}
g &= (f-a)^2
\end{aligned}
$$

This expression $g$ takes the minimum value 0 exactly when $f=a$.
Since $a$ can take any integer value in $[l,u]$, the expression 
$g$ achieves 0 if and only if $f$ itself takes an integer value within the same range.

Using this auxiliary-variable technique, QUBO++ implements range constraints.
If $f$ is a linear expression, then $g$ becomes a QUBO expression.
If $f$ is cubic or of higher degree, then $g$ becomes a HUBO expression.

>**NOTE**
> QUBO++ internally employs a lightweight improvement that enables range constraints to be encoded with a slightly smaller number of binary variables.


## Solving Integer Linear Programming
An instance of integer linear programming consists of an objective function and multiple linear constraints.
For example, the following integer linear program has two variables, one objective, and two constraints:

$$
\begin{aligned}
{\rm Maximize: } & & 5x+4y\\
{\rm Subject\,\, to: }& & 2x + 3y \leq 24 \\
 & & 7 x + 5 y \leq 54
\end{aligned}
$$

The optimal solution of this problem is $x=4$, $y=5$, with the objective value $40$.

The following QUBO++ program finds this optimal solution using the Easy Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = 0 <= qbpp::var_int("x") <= 10;
  auto y = 0 <= qbpp::var_int("y") <= 10;
  auto f = 5 * x + 4 * y;
  auto c1 = 0 <= 2 * x + 3 * y <= 24;
  auto c2 = 0 <= 7 * x + 5 * y <= 54;
  auto g = -f + 100 * (c1 + c2);
  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "x = " << sol(x) << ", y = " << sol(y) << std::endl;
  std::cout << "f = " << sol(f) << std::endl;
  std::cout << "c1 = " << sol(c1) << ", c2 = " << sol(c2) << std::endl;
  std::cout << "*c1 = " << sol(*c1) << ", *c2 = " << sol(*c2) << std::endl;
}
```

In this QUBO++ program,
- `f` represents the objective function,
- `c1` and `c2` represent the range constraints,
- and `g` combines them into a single optimization expression.

Since the goal is maximization, the objective is negated as `-f`.
The constraints `c1` and `c2` are penalized with a weight of 100 to ensure they are satisfied with high priority.

An Easy Solver instance is created for `g`, and a search is performed with a time limit of 1.0 seconds.
After obtaining the optimal solution `sol`, the program prints the values of `x`, `y`, `f`, `c1`, `c2`, `*c1`, and `*c2`.

The program outputs:
```txt
x = 4, y = 5
f = 40
c1 = 0, c2 = 0
*c1 = 23, *c2 = 53
```
Here, 
- `c1` is the expression for the constraint `0 <= 2 x + 3 y <= 24`,
- and `*c1` represents the linear expression  `2 x + 3 y`.

We can confirm that the solver correctly finds the optimal solution.