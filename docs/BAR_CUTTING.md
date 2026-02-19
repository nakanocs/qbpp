---
layout: default
title: "Bar Cutting Problem"
---

# Bar Cutting Problem
Suppose that we are given $M$ identical bars of fixed length $L$, and a set of $N$ orders specified by pairs $(l_j,c_j)$
($0\leq j\leq N−1$), where $l_j$ is the required length and $c_j$ is the required quantity of order $j$.
The **bar cutting problem** aims to determine how the $M$ bars can be cut to satisfy all orders.

In general, the bar cutting problem is formulated as a minimization problem that seeks to minimize the number of used bars.
For simplicity, in this example we consider the feasibility problem of determining whether the $M$ bars can fulfill all $N$ orders.


Let $x_{i,j}$ ($0\leq i\leq M-1, 0\leq j\leq N-1$) denote the number of pieces of order $j$ cut from bar $i$.
The following constraints must be satisfied.

### Order Constraint:
For each order $j$, the total number of pieces assigned across all bars must equal $c_j$:

$$
\begin{aligned}
 \sum_{i=0}^{M-1}x_{i,j} &= c_j & &(0\leq j\leq N-1)
\end{aligned}
$$

### Bar Constraint
For each bar $i$, the total length of the assigned pieces must not exceed $L$:

$$
\begin{aligned}
 \sum_{j=0}^{N-1}l_jx_{i,j} &\leq  L & &(0\leq i\leq M-1)
\end{aligned}
$$

## QUBO++ program
The following QUBO++ program finds a feasible cutting plan using
$M=6$ bars of length $L=60$ and the following $N=4$ orders:

| Order $j$ | 0 | 1 | 2 | 3|
|---|---|---|---|---|
| Length $l_j$ | 13 | 23 | 8 | 11 |
| Quantity $c_j$ | 10 | 4 | 8 | 6 |

The QUBO++ program for this bar cutting problem is as follows:
```cpp

#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  const int L = 60;
  const qbpp::Vector<int> l = {13, 23, 8, 11};
  const qbpp::Vector<int> c = {10, 4, 8, 6};
  const size_t N = l.size();
  const size_t M = 6;

  auto x = qbpp::var_int("x", M, N) == 0;
  for (size_t i = 0; i < M; i++) {
    for (size_t j = 0; j < N; j++) {
      x[i][j] = 0 <= qbpp::var_int() <= c[j];
    }
  }

  auto order_fulfilled_count = qbpp::vector_sum(x, 0);
  auto order_constraint = order_fulfilled_count == c;

  auto bar_length_used = qbpp::expr(M);
  for (size_t i = 0; i < M; i++) {
    bar_length_used[i] = qbpp::sum(x[i] * l);
  }
  auto bar_constraint = 0 <= bar_length_used <= L;

  auto f = qbpp::sum(order_constraint) + qbpp::sum(bar_constraint);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(10.0);
  solver.target_energy(0);
  auto sol = solver.search();
  for (size_t i = 0; i < M; i++) {
    std::cout << "Bar " << i << ":  ";
    for (size_t j = 0; j < N; j++) {
      std::cout << sol(x[i][j]) << "  ";
    }
    std::cout << " used = " << sol(bar_length_used[i])
              << ", waste = " << L - sol(bar_length_used[i]) << std::endl;
  }
  for (size_t j = 0; j < N; j++) {
    std::cout << "Order " << j
              << " fulfilled = " << sol(order_fulfilled_count[j])
              << ", required = " << c[j] << std::endl;
  }
}
```
The program creates an `M`$\times$`N` matrix `x` of integer variables, initialized to the constant value 0.
The nested for loops assign to each entry `x[i][j]` a bounded integer variable, `0 <= qbpp::var_int(...) <= c[j]`, so that `x[i][j]` takes a non-negative integer value no greater than `c[j]`.

The constraints are defined as follows:
- `order_fulfilled_count`: a vector of $N$ expressions where `order_fulfilled_count[j]` represents the total number of pieces produced for order $j$.
- `order_constraint`: a vector of $N$ constraint expressions enforcing `order_fulfilled_count[j] == c[j]` for all $j$.
- `bar_length_used`: a vector of $M$ expressions where `bar_length_used[i]` represents the total length used in bar $i$.
- `bar_constraint`: a vector of $M$ constraint expressions enforcing `0 <= bar_length_used[i] <= L` for all $i$.
- `f`: the sum of all constraint expressions. After calling `f.simplify_as_binary()`, the Easy Solver searches for a solution with target energy 0 (i.e., all constraints satisfied).

The following output is an example feasible solution:
```
Bar 0:  2  0  0  3   used = 59, waste = 1
Bar 1:  4  0  1  0   used = 60, waste = 0
Bar 2:1  1  3  0   used = 60, waste = 0
Bar 3:  0  0  4  2   used = 54, waste = 6
Bar 4:  2  1  0  1   used = 60, waste = 0
Bar 5:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 10, required = 10
Order 1 fulfilled = 4, required = 4
Order 2 fulfilled = 8, required = 8
Order 3 fulfilled = 6, required = 6
```
We observe that all $N=4$ orders are fulfilled using $M=6$ bars.

If we set $M=5$, the solver returns the following infeasible solution, in which not all orders are satisfied:
```
Bar 0:  4  0  1  0   used = 60, waste = 0
Bar 1:  0  0  6  1   used = 59, waste = 1
Bar 2:  2  1  0  1   used = 60, waste = 0
Bar 3:  2  0  0  3   used = 59, waste = 1
Bar 4:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 9, required = 10
Order 1 fulfilled = 3, required = 4
Order 2 fulfilled = 7, required = 8
Order 3 fulfilled = 5, required = 6
```
