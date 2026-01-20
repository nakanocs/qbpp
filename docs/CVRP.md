---
layout: default
title: "CVRP"
---

# Capacitated Vehicle Routing Problem (CVRP)
The **Capacitated Vehicle Routing Problem (CVRP)** aims to find a set of routes for $V$ **vehicles** that start and end at a single **depot** and collectively serve all **customers**.
We index the locations by $i \in \lbrace 0,1,\ldots,N-1\rbrace$, where location 0 denotes the depot and locations $1,\ldots,N-1$ are customers.  
Each customer $i\in \lbrace 1,\ldots,N-1\rbrace$ has a demand $d_i$ to be delivered (and we set $d_0=0$ for the depot).  
Each vehicle $v \in \lbrace 0,\ldots,V-1\rbrace$ departs from the depot, visits a subset of customers, and returns to the depot, subject to the capacity constraint that the total delivered demand on its route does not exceed the vehicle capacity $q_v$.
The objective is to minimize the total travel cost over all vehicles.

We assume that locations are points in the two-dimensional plane and that the travel cost between two locations is the Euclidean distance.
Let $c_{i,j}$ denote the distance (cost) between locations $i$ and $j$.

## QUBO++ formulation: array of binary variables
We use a $V\times N\times N$ array $A=(a_{v,t,i})$ of binary variables to formulate the CVRP as a QUBO expression,
where $a_{v,t,i}$ is 1 if and only if the $t$-th visited location of vehicle $v$ is location $i$.

Below is an example assignment of $A=(a_{v,t,i})$ with $V=3$ and $N=10$ representing
a solution of the CVRP.
Each vehicle starts from the depot (location 0), visits some customers, and then returns to the depot.
After a vehicle returns to the depot, it stays at the depot for the remaining time steps.
Each customer $1, \ldots, 9$ is visited exactly once by exactly one vehicle, so this array represents a feasible CVRP solution.

### Vehicle $v=0$
Representing tour: $0\rightarrow 4\rightarrow 0$

| t i | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | onehot value |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 4 |
| 2 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 3 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 4 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 5 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 6 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 7 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 8 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 9 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |


### Vehicle $v=1$
Representing tour: $0\rightarrow 6\rightarrow 5\rightarrow 8\rightarrow  0$

| $t\backslash i$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | onehot value |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 6 |
| 2 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 5 |
| 3 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 8 |
| 4 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 5 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 6 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 7 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 8 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 9 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### Vehicle $v=2$

Representing tour: $0\rightarrow 7\rightarrow 9\rightarrow 1\rightarrow  3\rightarrow  2\rightarrow  0$
| $t\backslash i$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | onehot value | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 7 |
| 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 9 |
| 3 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| 4 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 3 |
| 5 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 |
| 6 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 7 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 8 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 9 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |


## Constraints for QUB++ formulation

### Row constraint (one-hot at each time)
Each vehicle must be at exactly one location at each time $t$.
We impose the one-hot constraint:

$$
\begin{aligned}
\text{row\_constraint} & = \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\bigr(\sum_{i=0}^{N-1} a_{v,t,i} = 1\bigl)\\
 &= \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\bigr(1-\sum_{i=0}^{N-1} a_{v,t,i}\bigl)^2
\end{aligned}
$$

\text{row\_constraint} attains its minimum value $0$
if and only if every row is one-hot.

We also fix the first position to be the depot:

$$
\begin{aligned}
a_{v,0,0}& = 1 && (0\leq v\leq V-1) \\
a_{v,0,i}& = 0 && (0\leq v\leq V-1, 1\leq i\leq N-1)
\end{aligned}
$$

These can be enforced by fixing variables.

### Consecutive-depot constraint (no “leaving the depot again”)
We disallow patterns where a vehicle returns to the depot and later visits customers again.
Using the depot-indicator variable $a_{v,t,0}$, we require that the sequence

$$
a_{v,1,0}, a_{v,2,0}, \ldots, a_{v,N-1,0}
$$

consists of consecutive 0’s followed by consecutive 1’s (i.e., once it becomes 1, it stays 1).
This is enforced by penalizing the forbidden transition $1\rightarrow 0$:
$$
\begin{aligned}
\text{consecutive\_constraint} &= \sum_{v=0}^{V-1}\sum_{t=1}^{N-2} (1-a_{v,t})a_{v,t+1}
\end{aligned}
$$

$\text{consecutive\_constraint}$ attains 0 if and only if the depot indicators are monotone nondecreasing in 
$t$ ($1\leq t\leq N-1$).

This constraint is redundant if the distance metric satisfies the triangle inequality (such “leave again” solutions cannot be optimal), but it often helps solvers avoid such non-optimal structures.

### Column constraint
Each customer must be visited exactly once by exactly one vehicle at exactly one time:

$$
\begin{aligned}
\text{column\_constraint}
& = \sum_{i=1}^{N-1}\bigr(\sum_{v=0}^{V-1}\sum_{t=0}^{N-1} a_{v,t,i} = 1\bigl)\\
 &= \sum_{i=1}^{N-1}\bigr(1-\sum_{v=0}^{V-1}\sum_{t=0}^{N-1} a_{v,t,i}\bigl)^2
\end{aligned}
$$

$\text{column\_constraint}=0$ holds if and only if every customer $i = 1, \dots ,N−1$ is visited exactly once.

### Capacity constraint 
For each vehicle $v$, the total delivered demand is

$$
\sum_{t=1}^{N-1}\sum_{i=1}^{N-1}d_ix_{t,i},
$$

which must be at most $q_v$.
Then the following constraint must be 0:

$$
\begin{aligned}
\text{capacity\_constraint} &= \sum_{v=0}^{V-1}\Bigr(0\leq \sum_{t=1}^{N-1}\sum_{i=1}^{N-1}d_ix_{t,i}\leq q_v\Bigl)
\end{aligned}
$$

\text{capacity\_constraint} holds if all vehicles do not exceed their capacity.


## Objective for QUBO formulation

The total tour cost is computed from consecutive visited locations:

$$
\begin{aligned}
\text{objective} &= \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\sum_{i=0}^{N-1}\sum_{j=0}^{N-1}d_{i,j}x_{v,t,i}x_{v,(t+1)\bmod N, j}
\end{aligned}
$$

Under the Euclidean metric, we have $c_{i,i}=0$ for all $i$, so staying at the same location does not add extra cost.

## QUBO formulation for the CVRP

Combining the objective and constraints, we obtain the QUBO:

$$
\begin{aligned}
f &= \text{objective} + P\cdot (\text{row\_constraint}+\text{consecutive\_constraint}+\text{column\_constraint}+\text{capacity\_constraint}),
\end{aligned}
$$

where $P$ is a sufficiently large constant so that feasibility (constraints) is prioritized over cost minimization.

## QUBO++ program
The following QUBO++ program finds a solution to the CVRP instance with $N=10$
$V=3$ vehicles.
The vector `locations` stores triples `(x,y,d)`, where  `(x,y)` is the 2D coordinate of the location and  `d` is the customer demand (with demand 0 for the depot).
The vector `vehicle_capacity` stores the capacities of the $V=3$ vehicles.
In this example, it is set to `{100, 200, 300}`, so vehicles 0, 1, and 2 have small, medium, and large capacities, respectively.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"
#include "qbpp_graph.hpp"

int main() {
  std::vector<std::tuple<float, float, int>> locations = {
      {200, 200, 0},  {247, 296, 44}, {31, 393, 57}, {96, 398, 94},
      {391, 230, 91}, {118, 95, 66},  {197, 99, 59}, {224, 8, 10},
      {3, 10, 52},    {281, 379, 83}};
  std::vector<int> vehicle_capacity = {100, 200, 300};

  const size_t N = locations.size();

  const int V = vehicle_capacity.size();

  auto a = qbpp::var("a", V, N, N);

  auto row_constraint = qbpp::sum(qbpp::vector_sum(a) == 1);

  auto column_sum = qbpp::expr(N - 1);
  for (size_t v = 0; v < V; ++v) {
    for (size_t t = 0; t < N; ++t) {
      for (size_t i = 1; i < N; ++i) {
        column_sum[i - 1] += a[v][t][i];
      }
    }
  }
  auto column_constraint = qbpp::sum(column_sum == 1);

  auto consecutive_constraint = qbpp::toExpr(0);
  for (size_t v = 0; v < V; ++v) {
    for (size_t t = 1; t < N - 1; ++t) {
      consecutive_constraint += a[v][t][0] * (1 - a[v][t + 1][0]);
    }
  }

  auto vehicle_load = qbpp::expr(V);
  auto capacity_constraint = qbpp::toExpr(0);
  for (size_t v = 0; v < V; ++v) {
    for (size_t t = 0; t < N; ++t) {
      for (size_t i = 1; i < N; ++i) {
        const auto [x, y, q] = locations[i];
        vehicle_load[v] += a[v][t][i] * q;
      }
    }
    capacity_constraint += 0 <= vehicle_load[v] <= vehicle_capacity[v];
  }

  auto objective = qbpp::toExpr(0);
  for (size_t v = 0; v < V; ++v) {
    for (size_t t = 0; t < N; ++t) {
      auto next_t = (t + 1) % N;
      for (size_t i = 0; i < N; ++i) {
        const auto [x1, y1, q1] = locations[i];
        for (size_t j = 0; j < N; ++j) {
          const auto [x2, y2, q2] = locations[j];
          int dist = static_cast<int>(
              std::sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2)));
          objective += dist * a[v][t][i] * a[v][next_t][j];
        }
      }
    }
  }

  auto f = objective + 100000 * (row_constraint + column_constraint + consecutive_constraint + capacity_constraint);

  qbpp::MapList ml;
  for (size_t v = 0; v < V; ++v) {
    ml.push_back({a[v][0][0], 1});
    for (size_t i = 1; i < N; ++i) {
      ml.push_back({a[v][0][i], 0});
    }
  }

  auto g = qbpp::replace(f, ml);
  f.simplify_as_binary();
  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);

  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f);
  full_sol.set(ml);
  full_sol.set(sol);

  std::cout << "row_constraint = " << row_constraint(full_sol) << std::endl;
  std::cout << "column_constraint = " << column_constraint(full_sol) << std::endl;
  std::cout << "consecutive_constraint = " << consecutive_constraint(full_sol) << std::endl;
  std::cout << "capacity_constraint = " << capacity_constraint(full_sol) << std::endl;
  std::cout << "objective = " << objective(full_sol) << std::endl;

  auto tour = qbpp::onehot_to_int(full_sol(a));

  for (size_t v = 0; v < V; ++v) {
    std::cout << "Vehicle " << v << " : load = " << vehicle_load[v](full_sol)
              << " / " << vehicle_capacity[v];
    std::cout << " : 0 ";
    for (size_t t = 1; t < N; ++t) {
      auto node = tour[v][t];
      if (node > 0) {
        std::cout << "-> " << node << "("
                  << std::get<2>(locations[static_cast<size_t>(node)]) << ") ";
      }
    }
    std::cout << "-> 0" << std::endl;
  }

  qbpp::graph::GraphDrawer graph;
  for (size_t i = 0; i < locations.size(); ++i) {
    const auto [x, y, q] = locations[i];
    graph.add_node(qbpp::graph::Node(i).position(x, y).xlabel(
        q != 0 ? std::to_string(q) : ""));
  }

  for (size_t v = 0; v < V; ++v) {
    for (size_t i = 0; i < N; ++i) {
      int from = tour[v][i];
      int to = tour[v][(i + 1) % N];
      if (from < 0 || to < 0 || from == to) continue;
      graph.add_edge(qbpp::graph::Edge(from, to).color(v + 1).penwidth(2.0f));
    }
  }
  graph.draw();
  graph.write("cvrp.svg");
}
```
This program defines a $V\times N\times N$ array `a` of binary variables.
It then defines the objective term `objective` and the constraint terms
`row_constraint + column_constraint + consecutive_constraint + capacity_constraint`
according to the formulation described above, and combines them into the penalized objective
`f = objective + 100000 * ( ... )`.

To fix the first visited location ($t=0$)
of every vehicle to the depot (location 0), the program constructs a mapping `ml` and applies it to `f` using `qbpp::replace()`.
The resulting expression is stored in `g`.
The Easy Solver is then used to search for an assignment `sol` that minimizes `g`.

Since `g` does not contain the variables fixed by `ml`, the program merges `sol` and `ml` into `full_sol`, which assigns values to all variables in `a`.
Using `full_sol`, it prints the values of each `constraint` term and the `objective` value, and also prints the resulting tours of the $V=3$ vehicles.

For example, the program prints the following results:
```
row_constraint = 0
column_constraint = 0
consecutive_constraint = 0
vehicle_constraint = 0
objective = 2142
Vehicle 0 : load = 91 / 100 : 0 -> 4(91) -> 0
Vehicle 1 : load = 177 / 200 : 0 -> 6(59) -> 5(66) -> 8(52) -> 0
Vehicle 2 : load = 288 / 300 : 0 -> 7(10) -> 9(83) -> 1(44) -> 3(94) -> 2(57) -> 0
```
Finally, the program visualizes the obtained solution as a graph and writes it to `cvrp.svg`:


<p align="center">
  <img src="images/cvrp.svg" alt="A solution of the CVRP instance." width="80%">
</p>








