---
layout: default
title: "Exhaustive Solver"
---

# Exhaustive Solver Usage
The **Exhaustive Solver** is a complete-search solver for QUBO/HUBO expressions.
Since all possible assignments are examined, the optimality of the solutions is guaranteed.

Solving a problem with the Exhaustive Solver consists of the following three steps:
1. Create an Exhaustive Solver (or `qbpp::exhaustive_solver::ExhaustiveSolver`) object.
2. Set search options by calling member functions of the solver object.
3. Search for solutions by calling one of the search member functions.


## Creating Exhaustive Solver object
To use the Exhaustive Solver, an Exhaustive Solver object (or
`qbpp::exhaustive_solver::ExhaustiveSolver`) is constructed with an expression
(`or qbpp::Expr`) object as follows:
- **`qbpp::exhaustive_solver::ExhaustiveSolver(const qbpp::Expr& f)`**:
Here, `f` is the expression to be solved.
It must be simplified as a binary expression in advance by calling the
`simplify_as_binary()` function.
This function converts the given expression `f` into an internal format that is
used during the solution search.

## Setting Exhaustive Solver Options
- **`enable_default_callback()`**
Enables the default callback function, which prints newly obtained best solutions.

## Searching Solutions
The Exhaustive Solver searches for solutions by calling one of the following
member functions of the solver object:
- **`search()`**: Returns the first optimal solution found.
- **`search_optimal_solutions()`**: Returns a vector containing all optimal solutions.
- **`search_all_solutions()`**: Returns a vector containing all solutions, sorted in increasing order of
energy.

# Program example
The following program searches for a solution to the
**Low Autocorrelation Binary Sequences (LABS)** problem using the Exhaustive
Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  size_t size = 20;
  auto x = qbpp::var("x", size);
  auto f = qbpp::expr();
  for (size_t d = 1; d < size; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < size - d; ++i) {
      temp += (2 * x[i] - 1) * (2 * x[i + d] - 1);
    }
    f += qbpp::sqr(temp);
  }
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  solver.enable_default_callback();
  auto sol = solver.search();
  std::cout << sol.energy() << ": ";
  for (auto val : sol(x)) {
    std::cout << (val == 0 ? "-" : "+");
  }
  std::cout << std::endl;
}
```
The output of this program is as follows:
{% raw %}
```txt
TTS = 0.000s Energy = 1506
TTS = 0.000s Energy = 1030
TTS = 0.000s Energy = 502
TTS = 0.000s Energy = 446
TTS = 0.000s Energy = 234
TTS = 0.000s Energy = 110
TTS = 0.001s Energy = 106
TTS = 0.001s Energy = 74
TTS = 0.001s Energy = 66
TTS = 0.001s Energy = 42
TTS = 0.001s Energy = 34
TTS = 0.004s Energy = 26
26: --++-++----+----+-+-
```
{% endraw %}
All optimal solutions can be obtained by calling the
**`search_optimal_solutions()`** member function as follows:
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << sol.energy() << ": ";
    for (auto val : sol(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
The output is as follows:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
```
{% endraw %}
Furthermore, all solutions, including non-optimal ones, can be obtained by calling
the **`search_all_solutions()`** member function as follows:
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  solver.enable_default_callback();
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << sol.energy() << ": ";
    for (auto val : sol(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
This program prints all $2^{20}$ solutions in increasing order of energy, as
shown below:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
34: -----+-+--+-++--++--
34: ----+----++-++---+-+
34: ----+--+++--+-+++-+-
34: ---+++-+-+----+--+--
34: ---+++++-+++-++-+-++
34: --+--+----+-+-+++---
34: --+-+--+---+-----+++
34: --++--++-+--+-+-----
34: -+--+------+-+++---+
34: -+--+-+---+---+++++-
[omitted]
```
{% endraw %}
