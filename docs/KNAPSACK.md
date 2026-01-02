---
layout: default
title: "Knapsack Problem"
---

# Knapsack Problem
Given a set of items, each with a weight and a value, and a knapsack with a limited weight capacity, **the knapsack problem** aims to select a subset of items that maximizes the total value while keeping the total weight within the capacity.

Let $w_i$ and $v_i$ ($0\leq i\leq n-1$) denote the weight and value of item 
$i$, respectively.
Let $S\in \lbrace 0, 1, \ldots n-1\rbrace$ be the set of selected items.

$$
\begin{aligned}
\text{Maximize:} & \sum_{i\in S} v_i \\
\text{Subject to:} & \sum_{i\in S} w_i \leq W
\end{aligned}
$$

where $W$ is the weight capacity of the knapsack.

## QUBO formulation
To formulate this problem as a QUBO, we introduce a set 
$X$ of $n$ binary variables $x_i\in\lbrace 0,1\rbrace$ ($0\leq i\leq n-1$),
where item $i$ is selected if and only if $x_i=1$.

The above formulation can be rewritten as:

$$
\begin{aligned}
\text{Maximize:} & \sum_{i=0}^{n-1} v_ix_i \\
\text{Subject to:} & \sum_{i=0}^{n-1} w_ix_i \leq W
\end{aligned}
$$

## QUBO++ program
The constraint can be expressed using **the range operator** provided by QUBO++.
The resulting QUBO objective function is defined as:

$$
\begin{aligned}
f(X) &= -\sum_{i=0}^{n-1} v_ix_i + P\times (0\leq \sum_{i=0}^{n-1} w_ix_i \leq W)
\end{aligned}
$$

Since QUBO solvers minimize the objective function, the original maximization objective is negated.
The constant $P$ is a sufficiently large penalty parameter to enforce the constraint.

The following QUBO++ program solves a knapsack problem with 10 items using the Exhaustive Solver:
```cpp

#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Vector<int> w = {10, 20, 30, 5, 8, 15, 12, 7, 17, 18};
  qbpp::Vector<int> v = {60, 100, 120, 60, 80, 150, 110, 70, 150, 160};
  int capacity = 50;

  auto x = qbpp::var("x", w.size());

  auto constraint = 0 <= qbpp::sum(w * x) <= capacity;
  auto objective = qbpp::sum(v * x);

  auto f = -objective + 1000 * constraint;
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t i = 0; i < sols.size(); ++i) {
    const auto sol = sols[i];
    std::cout << "[Solution " << i << "]" << std::endl;
    std::cout << "Energy = " << sol.energy() << std::endl;
    std::cout << "Constraint  = " << sol(*constraint) << std::endl;
    std::cout << "Objective  = " << sol(objective) << std::endl;
    for (size_t j = 0; j < w.size(); ++j) {
      if (sol(x[j]) == 1) {
        std::cout << "Item " << j << ": weight = " << w[j]
                  << ", value =  " << v[j] << std::endl;
      }
    }
  }
}
```

In this program, the expressions constraint and objective are constructed separately and combined into the final QUBO expression f using a penalty coefficient of `1000`.
The Exhaustive Solver is then applied to f to enumerate all optimal solutions.

The following output shows the optimal solutions, including the energy, constraint value, and objective value:
```
[Solution 0]
Energy = -480
Constraint  = 50
Objective  = 480
Item 3: weight = 5, value =  60
Item 5: weight = 15, value =  150
Item 6: weight = 12, value =  110
Item 9: weight = 18, value =  160
[Solution 1]
Energy = -480
Constraint  = 50
Objective  = 480
Item 3: weight = 5, value =  60
Item 4: weight = 8, value =  80
Item 6: weight = 12, value =  110
Item 7: weight = 7, value =  70
Item 9: weight = 18, value =  160
```
We can observe that this instance has two optimal solutions, both achieving a total value of `480` while exactly satisfying the capacity constraint.