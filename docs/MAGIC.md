---
layout: default
title: "Magic Square"
---

# Magic Square
A 3-by-3 magic square is a 3-by-3 matrix that contains each integer from 1 to 9 exactly once, such that the sum of every row, every column, and the two diagonals is 15.
An example is shown below:
```
8 1 6 
3 5 7 
4 9 2 
```

## A formulation for finding magic square
We formulate the problem of finding a 3-by-3 magic square 
$S=(s_{i,j})$ ($0\leq i,j\leq 2$) using one-hot encoding.
We introduce binary variables $x_{i,j,k}$ ($0\leq i,j\leq 2, 0\leq k\leq 8$), where:

$$
\begin{aligned}
x_{i,j,k}=1 &\Longleftrightarrow & s_{i,j}=k+1
\end{aligned}
$$

Thus, $X=(x_{i,j,k})$ is a $3\times 3\times 9$ binary array.
We impose the following four constraints.

1. One-hot constraint (one value per cell):
For each cell $(i,j)$, exactly one of $x_{i,j,0}, x_{i,j,1}, \ldots,x _{i,j,8}$ must be 1:

$$
\begin{aligned}
c_1(i,j): & \sum_{k=0}^8 x _{i,j,k}=1 & (0\leq i,j\leq 2)
\end{aligned}
$$

2. Each value $k+1$ must appear in exactly one cell:

$$
\begin{aligned}
c_2(k): & \sum_{i=0}^2\sum_{j=0}^2x _{i,j,k}=1 & (0\leq k\leq 8)
\end{aligned}
$$

3. The sum of each row and each column must be 15:
$$
\begin{aligned}
c_3(i): & \sum_{j=0}^2\sum_{k=0}^8  (k+1)x _{i,j,k} = 15  &(0\leq i\leq 2)\\
c_3(j): & \sum_{i=0}^2\sum_{k=0}^8 (k+1)x _{i,j,k} = 15 &(0\leq j\leq 2)
\end{aligned}
$$

4. The sums of diagonal and anti-diagonal
The two diagonal sums must also be 15:
$$
\begin{aligned}
c_4: &  \sum_{k=0}^8 (k+1) (x_{0,0,k}+x_{1,1,k}+x_{2,2,k}) = 15 \\
c_4:  & \sum_{k=0}^8 (k+1) (x_{0,2,k}+x_{1,1,k}+x_{2,0,k}) = 15 
\end{aligned}
$$

When all constraints are satisfied, the assignment $X=(x_{i,j,k})$ represents a valid 3-by-3 magic square.

## QUBO++ prgram for the magic square
The following QUBO++ program implements these constraints and finds a magic square:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = qbpp::var("x", 3, 3, 9);

  auto c1 = qbpp::sum(qbpp::vector_sum(x) == 1);

  auto temp = qbpp::expr(9);
  for (size_t i = 0; i < 3; ++i)
    for (size_t j = 0; j < 3; ++j)
      for (size_t k = 0; k < 9; ++k) {
        temp[k] += x[i][j][k];
      }
  auto c2 = qbpp::sum(temp == 1);

  auto row = qbpp::expr(3);
  auto column = qbpp::expr(3);
  for (size_t i = 0; i < 3; ++i)
    for (size_t j = 0; j < 3; ++j)
      for (size_t k = 0; k < 9; ++k) {
        row[i] += (k + 1) * x[i][j][k];
        column[j] += (k + 1) * x[i][j][k];
      }
  auto c3 = qbpp::sum(row == 15) + qbpp::sum(column == 15);

  auto diag = qbpp::Expr(0);
  for (size_t k = 0; k < 9; ++k)
    diag += (k + 1) * (x[0][0][k] + x[1][1][k] + x[2][2][k]);
  auto anti_diag = qbpp::Expr(0);
  for (size_t k = 0; k < 9; ++k)
    anti_diag += (k + 1) * (x[0][2][k] + x[1][1][k] + x[2][0][k]);
  auto c4 = (diag == 15) + (anti_diag == 15);

  auto f = c1 + c2 + c3 + c4;
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  auto result = qbpp::onehot_to_int(sol(x));
  for (size_t i = 0; i < 3; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << result[i][j] + 1 << " ";
    }
    std::cout << std::endl;
  }
}
```
In this program, we define a $3\times 3\times9$ array of binary variables `x`.
We then build four constraint expressions `c1`, `c2`, `c3`, and `c4`, and combine them into `f`.
The expression `f` achieves the minimum energy 0 when all constraints are satisfied.

We create an Easy Solver object solver for `f` and set the target energy to 0, so the search terminates as soon as a feasible (optimal) solution is found.
The returned solution is stored in `sol`.
Finally, we convert the one-hot representation into integers using `qbpp::onehot_to_int()`, which returns a $3\times 3$ array of integers in 
$\{0,1, \ldots, 8\}$. We print the resulting square by adding $1$ to each entry.

This program produces the following output:
```
8 1 6 
3 5 7 
4 9 2 
```

## Fixing variable partially
Suppose we want to find a solution in which the top-left cell is assigned the value 2.
In the one-hot encoding, the value 2 corresponds to $k=1$, so we fix

$$
\begin{aligned}
 x_{0,0,k} &=1 & {\rm if\,\,} k=1\\
 x_{0,0,k} &=0 & {\rm if\,\,} k\neq 1
\end{aligned}
$$

Moreover, since constraint $c_2$ enforces that each number $k+1$ appears exactly once, fixing 
immediately implies that no other cell can take the value 2.
Therefore, we can also fix:

$$
\begin{aligned}
 x_{i,j,1} &=0 & {\rm if\,\,} (i,j)\neq (0,0)\\
\end{aligned}
$$

These fixed assignments reduce the number of remaining binary variables, which is often beneficial for local-search-based solvers.

## QUBO++ program for the magic square with fixing variable partially
We modify the program above as follows:
```cpp
  qbpp::MapList ml;
  for (size_t k = 0; k < 9; ++k) {
    if (k == 1)
      ml.push_back({x[0][0][k], 1});
    else
      ml.push_back({x[0][0][k], 0});
  }

  for (size_t i = 0; i < 3; ++i)
    for (size_t j = 0; j < 3; ++j) {
      if (!(i == 0 && j == 0)) {
        ml.push_back({x[i][j][1], 0});
      }
    }

  qbpp::Sol full_sol(f);
  f.replace(ml);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  full_sol.set(sol).set(ml);
  auto result = qbpp::onehot_to_int(full_sol(x));
  for (size_t i = 0; i < 3; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << result[i][j] + 1 << " ";
    }
    std::cout << std::endl;
  }
```

In this code, we create a `qbpp::MapList` object `ml` and add fixed assignments using `push_back()`.
We then create `full_sol`, a solution object for the original expression `f`.
Calling `f.replace(ml)` substitutes the fixed values into `f` in place, so the variables listed in `ml` disappear from `f`.
As a result, the solution `sol` returned by the solver does not include those fixed variables.
Finally, we reconstruct a complete assignment by merging `sol` and `ml` into `full_sol` via `set()`.
The reconstructed solution `full_sol` represents the full magic square.

This program produces the following output:
```
2 7 6 
9 5 1 
4 3 8 
```
We can confirm that the top-left cell is 2, as intended.
