---
layout: default
title: "Shift Scheduling"
---

# Shift Scheduling Problem
Consider the following **shift scheduling problem**, which aims to find a schedule that minimizes the total worker cost.

- There are 6 workers and a planning horizon of 31 days, from day 1 to day 31.
For simplicity, we assume that all workers are off on day 0 and day 32.
- Exactly 4 workers must be scheduled on each day from day 1 to day 31.
- The following constraints must be satisfied for each worker:
  - works for either 20 or 21 days,
  - works no more than 6 consecutive days,
  - works no fewer than 3 consecutive days,
  - has no isolated day off; days off must be consecutive.


## QUBO formulation for the shift scheduling problem
The QUBO formulation uses a $6\times 33$ matrix of binary variables $X=(x_{i,j})$ ($0\leq i\leq 5, 0\leq j\leq 32$) where worker $i$ works on day $j$ if and only if $x_{i,j}=1$.

Since all workers are off on day 0 and day 32, we fix 

$$
\begin{aligned}
x_{i,0}=x_{i,32}=0 & &(0\leq i\leq 5).
\end{aligned}
$$

The constraints are formulated as follows.

### Daily staffing constraint
Exactly 4 workers must be scheduled on each day:

$$
\begin{aligned}
\sum_{i=0}^{5} x_{i,j} = 4& &(1\leq j\leq 31)
\end{aligned}
$$

### Total working days constraint
Each worker must work for either 20 or 21 days:

$$
\begin{aligned}
20\leq \sum_{j=0}^{32} x_{i,j} \leq 21& &(0\leq i\leq 5)
\end{aligned}
$$

### Maximum consecutive working days constraint
No worker may work for more than 6 consecutive days:

$$
\begin{aligned}
 x_{i,j}x_{i,j+1}x_{i,j+2}x_{i,j+3}x_{i,j+4}x_{i,j+5}x_{i,j+6} = 0 & &(0\leq i\leq 5, 0\leq j\leq 26)\\
\end{aligned}
$$

### Minimum consecutive working days constraint
Each working period must consist of at least 3 consecutive working days:

$$
\begin{aligned}
 (1-x_{i,j})x_{i,j+1}x_{i,j+2}(1-x_{i,j+3}) = 0 & &(0\leq i\leq 5, 0\leq j\leq 29)\\
(1-x_{i,j})x_{i,j+1}(1-x_{i,j+2}) = 0 & & (0\leq i\leq 5, 0\leq j \leq 30)
\end{aligned}
$$

### No isolated day off constraint
No worker may have a single day off between two working days:

$$
\begin{aligned}
 x_{i,j}(1-x_{i,j+1})x_{i,j+2} = 0 & &(0\leq i\leq 5, 0\leq j\leq 30)\\
\end{aligned}
$$

### Total worker cost
Let $C=(c_i)$ be a cost vector, where $c_i$ denotes the daily cost of assigning worker $i$.
The total worker cost is formulated as:

$$
\begin{aligned}
\sum_{i=0}^5\sum_{j=0}^{32} c_i x_{i,j}
\end{aligned}
$$

This objective function is minimized subject to the constraints described above.

## QUBO++ program for the shift scheduling
The shift scheduling problem defined above can be formulated and solved using QUBO++ as follows:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  const size_t days = 31;
  const qbpp::Vector<int> worker_cost = {13, 13, 12, 12, 11, 10};
  const size_t workers = worker_cost.size();

  auto x = qbpp::var("x", workers, days + 2);

  auto workers_each_day = qbpp::vector_sum(x, 0);
  auto each_day_4_workers = qbpp::toExpr(0);
  for (size_t j = 1; j <= days; ++j) {
    each_day_4_workers += workers_each_day[j] == 4;
  }

  auto workers_working_days = qbpp::vector_sum(x);
  auto work_20_21_days = qbpp::sum(20 <= workers_working_days <= 21);

  auto no_more_than_6_consecutive_working_days = qbpp::toExpr(0);
  for (size_t w = 0; w < workers; ++w) {
    for (size_t j = 0; j <= days - 5; ++j) {
      no_more_than_6_consecutive_working_days +=
          x[w][j] * x[w][j + 1] * x[w][j + 2] * x[w][j + 3] * x[w][j + 4] *
          x[w][j + 5] * x[w][j + 6];
    }
  }
  auto no_less_than_3_consecutive_working_days = qbpp::toExpr(0);
  for (size_t w = 0; w < workers; ++w) {
    for (size_t j = 0; j < days - 1; ++j) {
      no_less_than_3_consecutive_working_days +=
          (1 - x[w][j]) * x[w][j + 1] * x[w][j + 2] * (1 - x[w][j + 3]);
    }
    for (size_t j = 0; j < days; ++j) {
      no_less_than_3_consecutive_working_days +=
          (1 - x[w][j]) * x[w][j + 1] * (1 - x[w][j + 2]);
    }
  }

  auto no_single_day_off = qbpp::toExpr(0);
  for (size_t w = 0; w < workers; ++w) {
    for (size_t j = 0; j <= days - 1; ++j) {
      no_single_day_off += x[w][j] * (1 - x[w][j + 1]) * x[w][j + 2];
    }
  }

  auto total_worker_cost = qbpp::sum(worker_cost * workers_working_days);

  auto constraints = work_20_21_days + no_less_than_3_consecutive_working_days +
                     no_more_than_6_consecutive_working_days +
                     no_single_day_off + each_day_4_workers;
  auto f = total_worker_cost + 10000 * constraints;

  qbpp::MapList ml;
  for (size_t i = 0; i < workers; ++i) {
    ml.push_back({x[i][0], 0});
    ml.push_back({x[i][days + 1], 0});
  }
  f.simplify_as_binary();

  auto g = qbpp::replace(f, ml);
  g.simplify_as_binary();
  workers_working_days.replace(ml);

  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.time_limit(5.0);
  solver.target_energy(0);
  auto sol = solver.search();
  for (size_t i = 0; i < workers; ++i) {
    std::cout << "Worker " << i << ": " << sol(workers_working_days[i])
              << " days worked: ";
    for (size_t j = 1; j <= days; ++j) {
      std::cout << sol(x[i][j]);
    }
    std::cout << std::endl;
  }
  std::cout << "Workers each day        : ";
  for (size_t d = 1; d <= days; ++d) {
    std::cout << sol(workers_each_day[d]);
  }
  std::cout << std::endl;

  auto sol_f = qbpp::Sol(f).set(ml).set(sol);

  std::cout << "Total worker cost: " << sol_f(total_worker_cost) << std::endl;
  std::cout << "Constraints violations: " << sol_f(constraints) << std::endl;
}
```
In this program, the variables and expressions are defined as follows:
- `x`: A $6\times 33$ matrix of binary variables,
- `workers_each_day`: A vector containing the column-wise sums of `x`, representing the number of workers assigned to each day.
- `each_day_4_workers`: A constraint expression that attains a minimum value of 0 if and only if exactly four workers are assigned to each day.
- `workers_working_days`: A vector of row-wise sums of `x`, representing the total number of working days for each worker.
- `work_20_21_days`: A constraint expression that attains a minimum value of 0 if and only if each worker works for either 20 or 21 days.
- `no_more_than_6_consecutive_working_days`:  A constraint expression that attains a minimum value of 0 if and only if no worker works for 7 or more consecutive days.
- `no_less_than_3_consecutive_working_days`: A constraint expression that attains a minimum value of 0 if and only if every working period consists of at least 3 consecutive working days.
- `no_single_day_off`: A constraint expression that attains a minimum value of 0 if and only if no worker has a single day off between two working days.
- `constraints`: The sum of all constraint expressions.
- `total_worker_cost`: An expression representing the total worker cost.

### QUBO construction and solution
By summing `total_worker_cost` and `constraints` with a penalty factor of 10000,
we obtain an expression `f`, which represents a QUBO formulation of the shift scheduling problem.

A `qbpp::MapList` object `ml` is used to fix the values of the variables corresponding to day 0 and day 32.
Applying the `qbpp::replace()` function to `f` with `ml` yields a new expression `g`.

The Easy Solver is then applied to `g`, and the resulting solution is stored in `sol`.
The obtained solution is as follows:
```
Worker 0: 20 days worked: 0001111001110011111001111001111
Worker 1: 20 days worked: 1111001111110001111110011110000
Worker 2: 21 days worked: 0000111100111110011111100111111
Worker 3: 21 days worked: 1111110011111100111000111111000
Worker 4: 21 days worked: 1111100111001111000111000111111
Worker 5: 21 days worked: 1110011110001111100111111000111
Workers each day        : 4444444444444444444444444444444
Total worker cost: 1465
Constraints violations: 0
```
We observe that a feasible shift schedule with a total worker cost of `1465` is obtained, and all constraints are satisfied.