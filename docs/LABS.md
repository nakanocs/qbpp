---
layout: default
title: "LABS Problem"
---

# Low-Autocorrelation Binary Sequence (LABS) Problem

The **Low Autocorrelation Binary Sequence (LABS)** problem aims to find a spin sequence $S=(s_i)$ ($s_i=\pm 1, 0\leq i\leq n-1$) 
that minimizes its autocorrelation.
The autocorrelation of $S$ with alignment $d$ is defined as

$$
\left(\sum_{i=0}^{n-d-1}s_is_{i+d}\right)^2
$$


The LABS objective function is the sum of these autocorrelations over all alignments:

$$
\begin{aligned}
\text{LABS}(S) &= \sum_{d=1}^{n-1}\left(\sum_{i=0}^{n-d-1}s_is_{i+d}\right)^2
\end{aligned}
$$

The LABS problem is to find a sequence $S$ that minimizes $\text{LABS}(S)$.

## Spin-to-binary conversion
Since the solvers bundled with QUBO++ do not support spin variables directly,
we convert the spin variables to binary variables using the following transformation:

$$
\begin{aligned}
 s_i &\leftarrow 2s_i - 1
\end{aligned}
$$

After this conversion, each $s_i$ can be treated as a binary variable,
and HUBO solvers for binary variables can be used to find a solution to $\text{LABS}(S)$.

QUBO++ provides this conversion through the spin_to_binary() function.

## QUBO++ program for the LABS
The following QUBO++ program formulates and solves the LABS problem:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  const int n = 30;

  auto s = qbpp::var("s", n);
  auto labs = qbpp::expr();
  for (size_t d = 1; d < n; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < n - d; ++i) {
      temp += s[i] * s[i + d];
    }
    labs += qbpp::sqr(temp);
  }

  labs.spin_to_binary();
  labs.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(labs);
  solver.time_limit(10.0);
  solver.enable_best_energy_sols();
  auto sols = solver.search();
  size_t i = 0;
  for (const auto& sol : sols.best_sols()) {
    std::cout << i++ << ": LABS = ";
    std::cout << sol.energy() << " : ";
    for (size_t j = 0; j < n; ++j) {
      std::cout << (sol(s[j]) ? "+" : "-");
    }
    std::cout << std::endl;
  }
}
```
In this program, `s` stores a vector of `n` variables.
The `qbpp::Expr` object `labs` is constructed using a nested loop,
directly following the mathematical definition of the LABS objective.

Afterward, `labs` is converted into an expression over binary variables
using the `spin_to_binary()` function and simplified by
`simplify_as_binary()`.

The Easy Solver is then executed with a time limit of 10 seconds.
Since `enable_best_energy_sols()` is enabled, all solutions achieving
the minimum energy are stored in sols.

Using a range-based for loop, all best-energy solutions are printed.
A typical output of this program is:
```
0: LABS = 59 : -----+++++-++-++-+-+-+++--+++-
1: LABS = 59 : -+-++-+-+---+++-------+--++-++
2: LABS = 59 : -+-+--+-+---+++-------+--++-++
3: LABS = 59 : +-+-++-+-+++---+++++++-++--+--
4: LABS = 59 : --+--++-+++++++---+++-+-++-+-+
5: LABS = 59 : ----++++++-++-++-+-+-+++--+++-
6: LABS = 59 : +-+--+-+-+++---+++++++-++--+--
7: LABS = 59 : ++-++--+-------+++---+-+-++-+-
8: LABS = 59 : -+++--+++-+-+-++-++-++++++----
9: LABS = 59 : +---++---+-+-+--+--+-----+++++
```
In this run, multiple solutions achieving the same minimum LABS value are obtained.