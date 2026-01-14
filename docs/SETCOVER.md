---
layout: default
title: "Minimum Set Cover Problem"
---

# Minimum Set Cover Problem
Let $U$ be a universe set, and let ${\cal 𝐹}=\{S_0, S_1, \ldots S_{m-1}\}$ be a family of subsets of $U$.
A subfamily $\cal S\subseteq \cal F$ is called a **set cover** if it covers all elements of $U$, i.e.,

$$
\begin{aligned}
\bigcup_{S_j\subseteq \cal S}S_j &= U.
\end{aligned}
$$

The **minimum set cover problem** is to find a set cover 
$\cal S$ with the minimum cardinality.
Here, we consider the weighted version, where each subset 
$S_j$ has a weight $w_j$, and the goal is to find a **set cover with the minimum total weight**.

## HUBO formulation of the minimum set cover problem
We formulate this problem as a HUBO problem.
Assume that $U=\lbrace 0,1,\ldots, n-1\rbrace$, and $m$ subsets $S_0, S_1, \ldots, S_{m-1}$ are given.
We introduce $m$ binary variables $x_0, x_1, \ldots, x_{m-1}$,
where $x_j=1$ if and only if $S_j\in\cal S$.  

For each element $i\in U$, define the following expression:

$$
\begin{aligned}
c_i &=\prod_{j: i\in S_j}(1-x_j) && (0\leq i\leq n-1)
\end{aligned}
$$

If none of the selected subsets contains $i$, then $i$ is not covered.
In that case, $x_j=0$ holds for all $j$ such that $i\in S_j$
and hence $c_i=1$.
On the other hand, if at least one selected subset contains $i$, then $x_j$=1 for some $j$ with $i\in S$, and the factor $(1−x_j)$ becomes 0, so $c_i=1$.
Therefore, the following **constraint** becomes 0 if and only if all elements are covered:

$$
\begin{aligned}
\text{constraint} &=\sum_{i=0}^{n-1}c_i
\end{aligned}
$$

The **objective** is to minimize the total weight of the selected subsets:

$$
\begin{aligned}
\text{objective} &=\sum_{j=0}^{m-1}w_jx_j
\end{aligned}
$$

We can now construct a HUBO objective function for the weighted minimum set cover problem as:

$$
\begin{aligned}
f &= \text{objective}+P\times\text{constraint}, 
\end{aligned}
$$

where $P$ is a sufficiently large positive constant to prioritize feasibility over the objective.

## QUBO++ program for the minimum set cover problem
The following QUBO++ program constructs a HUBO expression for a weighted minimum set cover instance with $n=10$ elements and $m=8$ subsets:
```cpp
#include <set>

#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  const size_t n = 10;
  qbpp::Vector<qbpp::Vector<size_t>> cover = {
      {0, 1, 2}, {2, 3, 4},       {4, 5, 6},    {6, 7, 8},
      {9, 0, 1}, {1, 3, 5, 7, 9}, {0, 3, 6, 9}, {1, 4, 7, 8}};
  qbpp::Vector cost = {3, 4, 3, 2, 3, 4, 3, 3};
  auto m = cover.size();

  auto x = qbpp::var("x", m);

  auto c = qbpp::expr(n);
  for (size_t j = 0; j < n; ++j) {
    c[j] = 1;
  }
  for (size_t i = 0; i < m; ++i) {
    for (size_t j : cover[i]) {
      c[j] *= (1 - x[i]);
    }
  }

  auto objective = qbpp::sum(cost * x);

  auto constraint = qbpp::sum(c);

  auto f = objective + 1000 * constraint;

  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);

  auto sol = solver.search();

  std::cout << "objective = " << objective(sol) << std::endl;
  std::cout << "constraint = " << constraint(sol) << std::endl;

  for (size_t i = 0; i < m; ++i) {
    if (sol(x[i]) == 1) {
      std::cout << "Set " << i << ": " << cover[i] << " cost = " << cost[i]
                << std::endl;
    }
  }
}
```
This program defines a vector **`x`** of $m=8$ **binary variables** and constructs a vector **`c`** of $n=10$ **expressions**.
Each expression `c[j]` corresponds to an element $j\in U$ and is initialized to 1.
For every subset $S_i$, and for every element $j\in S_i$, we multiply `c[j]` by 
`(1 − x[i])`.
As a result, `c[j]` becomes 0 if at least one selected subset covers element `j`, and remains 1 otherwise.

The **`constraint`** is defined as the sum of all entries in `c`, and it becomes 0 if and only if all elements are covered.
The weighted **`objective`** is defined as the dot product of `cost` and `x`.
They are combined into the HUBO expression:

$$
\begin{aligned}
f &= \text{objective} + 1000\times\text{constraint},
\end{aligned}
$$

where the penalty constant 1000 is chosen sufficiently large to prioritize feasibility.

The Exhaustive Solver is then used to find an optimal solution `sol`.
The program prints the values of `objective` and `constraint`, and finally lists all selected subsets. For example, the output is:
```
objective = 11
constraint = 0
Set 0: {0,1,2} cost = 3
Set 2: {4,5,6} cost = 3
Set 3: {6,7,8} cost = 2
Set 6: {0,3,6,9} cost = 3
```
This output indicates that a feasible set cover with total cost 11 is obtained.

## QUBO formulation for the minimum set cover problem
The HUBO formulation above may contain terms of degree three or higher, and therefore it is not necessarily a QUBO expression.
To obtain a QUBO formulation, we rewrite the covering constraint.

For each element $i\in U$, define

$$
\begin{aligned}
c_i &=\sum_{j: i\in S_j} x_j && (0\leq i\leq n-1)
\end{aligned}
$$

If $c_i\geq 1$, then at least one selected subset $S_j$ covers $i$.
If $c_i=0$, then no selected subset covers $i$.
Thus, we can express the covering constraint in the QUBO++ style as:

$$
\begin{aligned}
\text{constraint} &= \sum_{i=0}^{n-1} (c_i\geq 1)
\end{aligned}
$$

This constraint takes the minimum value 0 if and only if all elements are covered.
Based on this formulation, the QUBO++ program can be modified as follows:
```cpp
  auto c = qbpp::expr(n);
  for (size_t i = 0; i < m; ++i) {
    for (size_t j : cover[i]) {
      c[j] += x[i];
    }
  }

  auto constraint = qbpp::sum(1 <= c <= +qbpp::inf);
```
In this program, the expressions `1 <= c[j] <= +qbpp::inf` are created for all 
`j`, and their sum is stored in `constraint`.

> **Remark**.
> The term `1 <= c[j] <= +qbpp::inf` may introducing auxiliary binary variables
> internally. As a result, the final expression can be handled by a QUBO solver, 
> while preserving the intended meaning of the coverage constraint.

With this modification, the program produces the same optimal solution as the HUBO version.