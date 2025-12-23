---
layout: default
title: "PERMUTATION"
---

# Permutation matrix genration

Many combinatorial optimization problems are permutation-based in the sense that the objective is to find an optimal permutation.
As a fundamental technique for formulating such optimization problems, a matrix of binary variables is used in their QUBO formulation.

## Permutation matrix
Let $X=(x_{i,j})$ ($0\leq i,j\leq n-1$) is a matrix of $n\times n$ binary values.
The matrix $X$ is called a **permutation matrix** if and only if every row and every column has exactly one entry equal to 1, as shown below.

<p align="center">
  <img src="images/matrix.png" alt="Permutation matrix" width="50%">
</p>

A permutation matrix represents a permutation of $n$ numbers $(0,1,\ldots,n-1)$, where $x_{i,j} = 1$ if and only if the $i$-th element is $j$.
For example, the above permutation matrix represents the permutation $(1,2,0,3)$.

## QUBO formulation for permutation matrices
A binary variable matrix $X=(x_{i,j})$ ($0\leq i,j\leq n-1$)
stores a permutation matrix if and only if the sum of each row and each column is 1.
Thus, the following QUBO function takes the minimum value 0 if and only if $X$ stores a permutation matrix:

$$
\begin{aligned}
f(X) &= \sum_{i=0}^{n-1}\left(1-\sum_{j=0}^{n-1}x_{i,j}\right)^2+\sum_{j=0}^{n-1}\left(1-\sum_{i=0}^{n-1}x_{i,j}\right)^2
\end{aligned}
$$

## QUBO++ program for generating permutation matrices
We can desgin a QUBO++ program based on the formula $f(X)$ above as follows:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::expr();

  for (size_t i = 0; i < 4; i++) {
    auto s = qbpp::expr();
    for (size_t j = 0; j < 4; j++) {
      s += x[i][j];
    }
    f += qbpp::sqr(1 - s);
  }

  for (size_t j = 0; j < 4; j++) {
    auto s = qbpp::expr();
    for (size_t i = 0; i < 4; i++) {
      s += x[i][j];
    }
    f += qbpp::sqr(1 - s);
  }

  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t k = 0; k < sols.size(); k++) {
    const auto& sol = sols[k];
    std::cout << "Solution " << k << " : " << sol(x) << std::endl;
  }
}
```

In this program, `qbpp::var("x",4,4)` returns a `qbpp::Var` object
of size $4\times 4$ with `x`.
For a `qbpp::Expr` object `f`, two double for-loops adds
formulas for $f(X)$.
Using the Exhaustive Solver, all optimal solutions are computed and stored in `sols`.
All solutions in `sols` are displayed one-by-one.
Heare, `sol(x)` returns a matrix of values of `x` in `sol`.
This program outputs all 24 permutations as follows:
{% raw %}
```cpp
Solution 0 : {{0,0,0,1},{0,0,1,0},{0,1,0,0},{1,0,0,0}}
Solution 1 : {{0,0,0,1},{0,0,1,0},{1,0,0,0},{0,1,0,0}}
Solution 2 : {{0,0,0,1},{0,1,0,0},{0,0,1,0},{1,0,0,0}}
Solution 3 : {{0,0,0,1},{0,1,0,0},{1,0,0,0},{0,0,1,0}}
Solution 4 : {{0,0,0,1},{1,0,0,0},{0,0,1,0},{0,1,0,0}}
Solution 5 : {{0,0,0,1},{1,0,0,0},{0,1,0,0},{0,0,1,0}}
Solution 6 : {{0,0,1,0},{0,0,0,1},{0,1,0,0},{1,0,0,0}}
Solution 7 : {{0,0,1,0},{0,0,0,1},{1,0,0,0},{0,1,0,0}}
Solution 8 : {{0,0,1,0},{0,1,0,0},{0,0,0,1},{1,0,0,0}}
Solution 9 : {{0,0,1,0},{0,1,0,0},{1,0,0,0},{0,0,0,1}}
Solution 10 : {{0,0,1,0},{1,0,0,0},{0,0,0,1},{0,1,0,0}}
Solution 11 : {{0,0,1,0},{1,0,0,0},{0,1,0,0},{0,0,0,1}}
Solution 12 : {{0,1,0,0},{0,0,0,1},{0,0,1,0},{1,0,0,0}}
Solution 13 : {{0,1,0,0},{0,0,0,1},{1,0,0,0},{0,0,1,0}}
Solution 14 : {{0,1,0,0},{0,0,1,0},{0,0,0,1},{1,0,0,0}}
Solution 15 : {{0,1,0,0},{0,0,1,0},{1,0,0,0},{0,0,0,1}}
Solution 16 : {{0,1,0,0},{1,0,0,0},{0,0,0,1},{0,0,1,0}}
Solution 17 : {{0,1,0,0},{1,0,0,0},{0,0,1,0},{0,0,0,1}}
Solution 18 : {{1,0,0,0},{0,0,0,1},{0,0,1,0},{0,1,0,0}}
Solution 19 : {{1,0,0,0},{0,0,0,1},{0,1,0,0},{0,0,1,0}}
Solution 20 : {{1,0,0,0},{0,0,1,0},{0,0,0,1},{0,1,0,0}}
Solution 21 : {{1,0,0,0},{0,0,1,0},{0,1,0,0},{0,0,0,1}}
Solution 22 : {{1,0,0,0},{0,1,0,0},{0,0,0,1},{0,0,1,0}}
Solution 23 : {{1,0,0,0},{0,1,0,0},{0,0,1,0},{0,0,0,1}}
```
{% endraw %}
> **NOTE**
> A matrix of binary variables is implemented as a nested vector using `qbpp::Vector` class.
> For example, `qbpp::var("x",4,4)` returns a `qbpp::Vector<qbpp::Vector<qbpp::Var>>` objects.
> Each `qbpp::Var` object is represented as `x[i][j]` and the value of `x[i][j]` for `sol` can be obtained by either `sol(x[i][j])` or `x[i][j](sol)`.


## QUBO formulation for permutation matrix using vector functions and operations
The following two functions for a matrix `x` of $n\times n$ binary variables are supported:
- `qbpp::vector_sum(x)`: Computes the sum of each row and returns a vector of size $n$ containing these sums.
- `qbpp::transpose(x)`: Returns the transposed matrix of size $n\times n$.
Using these functions, vectors containing row-wise sums and column-wise sums can be computed by the following expressions.
- `qbpp::vector_sum(qbpp::transpose(x))`: Returns a vector of column-wise sums of size $n$.

A scalar–vector operation is used to subtract 1 from each element:
- `qbpp::vector_sum(x) - 1`: subtracts 1 from each row-wise sum.
- `qbpp::vector_sum(qbpp::transpose(x)) - 1`: subtracts 1 from each column-wise sum.
For these two vectors of size $n$, `qbpp::sqr()` squares each element, and `qbpp::sum()` computes the sum of all elements.

The following QUBO++ program implements a QUBO formulation using these vector functions and operations:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::sum(qbpp::sqr(qbpp::vector_sum(x) - 1)) +
           qbpp::sum(qbpp::sqr(qbpp::vector_sum(qbpp::transpose(x)) - 1));
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t k = 0; k < sols.size(); k++) {
    const auto& sol = sols[k];
    const auto& val_x = qbpp::onehot_to_int(x(sol));
    std::cout << "Solution " << k << ": " << val_x << std::endl;
  }
}
```

In this program, `x(sol)` returns a matrix of assigned values to `x` in `sol`,
which is a matrix of integers of size $n\times n$.
Since `qbpp::onehot_to_int()` converts each one-hot vector to the corresponding integer,
`qbpp::onehot_to_int(x(sol))` returns the permutation as a vector of integers.

This program outputs all permutations as integer vectors as follows:
```txt
Solution 0: {3,2,1,0}
Solution 1: {3,2,0,1}
Solution 2: {3,1,2,0}
Solution 3: {3,1,0,2}
Solution 4: {3,0,2,1}
Solution 5: {3,0,1,2}
Solution 6: {2,3,1,0}
Solution 7: {2,3,0,1}
Solution 8: {2,1,3,0}
Solution 9: {2,1,0,3}
Solution 10: {2,0,3,1}
Solution 11: {2,0,1,3}
Solution 12: {1,3,2,0}
Solution 13: {1,3,0,2}
Solution 14: {1,2,3,0}
Solution 15: {1,2,0,3}
Solution 16: {1,0,3,2}
Solution 17: {1,0,2,3}
Solution 18: {0,3,2,1}
Solution 19: {0,3,1,2}
Solution 20: {0,2,3,1}
Solution 21: {0,2,1,3}
Solution 22: {0,1,3,2}
Solution 23: {0,1,2,3}
```

## Assignment problem and its QUBO formulation
Let $C = (c_{i,j})$ be a cost matrix of size $n \times n$.
The assignment problem for $C$ is to find a permutation
$p:\{0,1,\ldots, n-1\} \rightarrow \{0,1,\ldots, n-1\}$
that minimizes the total cost:

$$
\begin{aligned}
 g(p) &= \sum_{i=0}^{n-1}c_{i,p(i)}
\end{aligned}
$$

We can use a permutation matrix $X = (x_{i,j})$ of size $n \times n$ for a QUBO formulation of this problem by defining:

$$
\begin{aligned}
 g(X) &= \sum_{i=0}^{n-1}\sum_{j=0}^{n-1}c_{i,j}x_{i,j}
\end{aligned}
$$

Clearly, $g(p) = g(X)$ holds if and only if $X$ represents the permutation $p$.

We combine the QUBO formulation for the permutation matrix, $f(X)$, and the total cost, $g(X)$, to obtain a QUBO formulation of the assignment problem:

$$
\begin{aligned}
 h(X) &= P\cdot f(x)+g(x) \\
     &=P\left(\sum_{i=0}^{n-1}\left(1-\sum_{j=0}^{n-1}x_{i,j}\right)^2+\sum_{j=0}^{n-1}\left(1-\sum_{i=0}^{n-1}x_{i,j}\right)^2\right)+\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}c_{i,j}x_{i,j}
\end{aligned}
$$

Here, $P$ is a sufficiently large positive constant that prioritizes the permutation constraints encoded in $f(X)$.

## QUBO++ program for the assignment problem
We are now ready to design a QUBO++ program for the assignment problem.
In this program, a fixed matrix $C$ of size $4\times4$ is given as a nested qbpp::Vector.
The formulas for $f(X)$ and $g(X)$ are defined using vector functions and operations.
Here, `qbpp::vector_sum(x) == 1` returns a QUBO expression that takes the minimum value 0 if the equality is satisfied.
In fact, it returns the same QUBO expression as `qbpp::sqr(qbpp::vector_sum(x) - 1)`.
Also, `c * x` returns a matrix obtained by computing the element-wise product of `c` and `x`,
and therefore `qbpp::sum(c * x)` returns `g(X)`.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  qbpp::Vector<qbpp::Vector<uint32_t>> c = {
      {58, 73, 91, 44}, {62, 15, 87, 39}, {78, 56, 23, 94}, {11, 85, 68, 72}};
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::sum(qbpp::vector_sum(x) == 1) +
           qbpp::sum(qbpp::vector_sum(qbpp::transpose(x)) == 1);
  auto g = qbpp::sum(c * x);
  auto h = 1000 * f + g;
  h.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(h);
  solver.time_limit(1.0);

  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  auto result = qbpp::onehot_to_int(x(sol));
  std::cout << "Result : " << result << std::endl;
  for (size_t i = 0; i < result.size(); ++i) {
    std::cout << "c[" << i << "][" << result[i] << "] = " << c[i][result[i]]
              << std::endl;
  }
}
```

We use the Easy Solver to find a solution of `h`.
For an Easy Solver object solver for `h`, the time limit for searching a solution is set to 1.0 seconds by calling the `time_limit()` member function.
The resulting permutation is stored in `result`, and the selected `c[i][j]` values are printed in turn.
The output of this program is as follows:

{% raw %}
```txt
sol = 93:{{x[0][0],0},{x[0][1],0},{x[0][2],0},{x[0][3],1},{x[1][0],0},{x[1][1],1},{x[1][2],0},{x[1][3],0},{x[2][0],0},{x[2][1],0},{x[2][2],1},{x[2][3],0},{x[3][0],1},{x[3][1],0},{x[3][2],0},{x[3][3],0}}
Result : {3,1,2,0}
c[0][3] = 44
c[1][1] = 15
c[2][2] = 23
c[3][0] = 11
```
{% endraw %}
> **NOTE**
> For an expression `f` and an integer `m`, `f == m` returns an expression `qbpp::sqr(f - m)`,
> which takes the minimum value 0 if and only if the equality `f == m` is satisfied.


