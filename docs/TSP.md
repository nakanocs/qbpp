---
layout: default
title: "TSP"
---

# Traveling Salesman Problem
Traveling Salesman Problem (TSP) for nodes is a problem to find a shortest path vising all nodes.
We assume that nodes are arranged in a plane, and the Euclidean distance is used to compute a path length.
In the figure below, a shortest path to visiting all 9 nodes is illustrated:

<p align="center">
  <img src="images/tsp.png" alt="An example of nodes and the TSP solution" width="50%">
</p>


## QUBO formulation of the TSP
A visiting order of nodes can be represented as a permutation.
Hence, we use [permutation matrix](PERMUTATION) to represent a TSP solution.

Let $X=(x_{i,j})$ ($0\leq i,j\leq n-1$) is a matrix of $n\times n$ binary values.
The matrix $X$ is called a **permutation matrix** if and only if every row and every column has exactly one entry equal to 1, as shown below.

<p align="center">
  <img src="images/matrix.png" alt="A permutation matrix of size 3x3" width="50%">
</p>

To represent a permutation by $X$, each row and column of $X$ must be onehot.
Thus, the following constraints must be satisfied.

$$
\begin{aligned}
{\rm row}:& \sum_{j=0}^{n-1}x_{i,j}=1 & (0\leq i\leq n-1)\\
{\rm column}:& \sum_{i=0}^{n-1}x_{i,j}=1 & (0\leq j\leq n-1)
\end{aligned}
$$

For a permutation matrix $X$, the path length can be computed by the following expression of $X$:

$$
\begin{aligned}
{\rm objective}: &\sum_{k=0}^{k-1} d_{i,j}x_{k,i}x_{(k+1)\bmod n,j}
\end{aligned}
$$
Since $d_{i,j}$ is accumulated if $x_{k,i}=x_{(k+1)\bmod n,j}$
this objective computes the path length of $X$.

## QUBO++ program for TSP
Using a permutation matrix $X$, we can design a QUBO++ program for the TSP as follows:
{% raw %}
```cpp

#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

class Nodes {
  std::vector<std::pair<int, int>> nodes{{0, 0},  {0, 10},  {0, 20},
                                         {10, 0}, {10, 10}, {10, 20},
                                         {20, 0}, {20, 10}, {20, 20}};

 public:
  const std::pair<int, int>& operator[](std::size_t index) const {
    return nodes[index];
  }

  std::size_t size() const { return nodes.size(); }

  int dist(std::size_t i, std::size_t j) const {
    auto [x1, y1] = nodes[i];
    auto [x2, y2] = nodes[j];
    const int dx = x1 - x2;
    const int dy = y1 - y2;
    return static_cast<int>(
        std::llround(std::sqrt(static_cast<double>(dx * dx + dy * dy))));
  }
};

int main() {
  auto nodes = Nodes{};
  auto x = qbpp::var("x", nodes.size(), nodes.size());

  auto constraints = qbpp::sum(qbpp::vector_sum(x) == 1) +
                     qbpp::sum(qbpp::vector_sum(qbpp::transpose(x)) == 1);

  auto objective = qbpp::expr();
  for (size_t i = 0; i < nodes.size(); ++i) {
    auto next_i = (i + 1) % nodes.size();
    for (size_t j = 0; j < nodes.size(); ++j) {
      for (size_t k = 0; k < nodes.size(); ++k) {
        if (k != j) {
          objective += nodes.dist(j, k) * x[i][j] * x[next_i][k];
        }
      }
    }
  }

  auto f = objective + constraint * 1000;
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  solver.enable_default_callback();
  auto sol = solver.search();
  auto result = qbpp::onehot_to_int(sol(x));
  std::cout << "Result: " << result << std::endl;
}
```
{% endraw %}
In this program, positions of nodes from 0 to 8 are given as a `Nodes` object `nodes`. 
For an array `x` of binary variables, the constraints and objective for `nodes` are created, and combined as `f`.
To prioritize constraints, a factor of 1000 is multiplied.
A Easy Solver object is created for `f` and 1.0 seconds time limit is
set.
Also the default callback is enabled.
For the obtained solution `sol` object,
the resulting permutation matrix `sol(x)` is converted
to a list of integers representing the permutation by `qbpp::onehot_to_int()`, which is printed using `qbpp::cout`.

