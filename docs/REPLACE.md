---
layout: default
title: "REPLACE"
---

# Replace functions

QUBO++ provides the following replace function, which can be used to fix variable values in an expression.
- `qbpp::replace(const qbpp::Expr& f, const qbpp::MapList& ml)`

Replaces (fixes) variable values in the expression f according to the mapping specified by ml.

## Using the replace function to fix Variable Values
We explain the `qbpp::replace()` function using the
[QUBO++ program for partitioning problem](PARTITION)> for explining
the `qbpp::replace()` function.
This program finds a partition of the numbers in the following vector `w` into two subsets $L$ and $\overline{L}$ such that the difference between their sums is minimized:
```cpp
  std::vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
```
We modify this partitioning problem so that 64 must belong to $L$ and 27 must belong to $\overline{L}$, ensuring that they are placed in distinct subsets.

To enforce this constraint, the values of `x[0]` and `x[1]` are fixed to 1 and 0, respectively, using the `qbpp::replace()` function.

The complete QUBO++ program is shown below:

{% raw %}
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
  auto x = qbpp::var("x", w.size());
  auto p = qbpp::sum(w * x);
  auto q = qbpp::sum(w * (1 - x));
  auto f = qbpp::sqr(p - q);
  qbpp::MapList ml({{x[0], 1}, {x[1], 0}});
  auto g = qbpp::replace(f, ml);
  g.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(g);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;

  qbpp::Sol full_sol(f);
  full_sol.set(sol);
  full_sol.set(ml);
  std::cout << "f(full_sol) = " << f(full_sol) << std::endl;
  std::cout << "p(full_sol) = " << p(full_sol) << std::endl;
  std::cout << "q(full_sol) = " << q(full_sol) << std::endl;
  std::cout << "L  :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](full_sol) == 1) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
  std::cout << "~L :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](full_sol) == 0) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
}
```
{% endraw %}

First, a `qbpp::MapList` object `ml` is defined, which specifies fixed values for the variables `x[0]` and `x[1]`.
Given the original expression `f` for the partitioning problem and the qbpp::MapList object `ml`, the `qbpp::replace()` function is used to replace `x[0]` and `x[1]` in `f` with the constants 1 and 0, respectively.
The resulting expression is stored in `g`.

The Exhaustive Solver is then applied to `g` to find an optimal solution, which is stored in `sol`.
Note that the expression `g` no longer contains the variables `x[0]` and `x[1]`, and consequently, `sol` also does not include assignments for these variables.

To construct a complete solution that includes all variables, we create a `qbpp::Sol` object `full_sol` from the original expression `f`.
Initially, `full_sol` represents an all-zero solution.
We then use the `set()` member function to incorporate both:
- the solution `sol` obtained from the reduced problem, and
- the fixed assignments specified in `ml`.
These assignments are combined and stored in `full_sol`.

From the output below, we can confirm that 64 is placed in $L$ and 27 is placed in $\overline{L}$, as intended:
```
f(full_sol) = 4
p(full_sol) = 206
q(full_sol) = 204
L  : 64 47 12 83
~L : 27 74 63 40
```

## Using the replace function to replace variables with expressions
The `replace()` function can also replace a variable with an expression, not only with a constant value.

Here, we present a more sophisticated way to ensure that 64 and 27 are placed in distinct subsets in the partitioning problem introduced above.
The key idea is to replace the variable `x[0]` in the expression `f` with the expression `1 - x[1]`.
This enforces the constraint that `x[0]` and `x[1]` always take opposite values, guaranteeing that the corresponding elements (64 and 27) belong to different subsets.

The following C++ program implements this idea:
{% raw %}
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
  auto x = qbpp::var("x", w.size());
  auto p = qbpp::sum(w * x);
  auto q = qbpp::sum(w * (1 - x));
  auto f = qbpp::sqr(p - q);
  qbpp::MapList ml({{x[0], 1 - x[1]}});
  auto g = qbpp::replace(f, ml);
  g.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(g);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;

  qbpp::Sol full_sol(f);
  full_sol.set(sol);
  full_sol.set({{x[0], sol(1 - x[1])}});
  std::cout << "f(full_sol) = " << f(full_sol) << std::endl;
  std::cout << "p(full_sol) = " << p(full_sol) << std::endl;
  std::cout << "q(full_sol) = " << q(full_sol) << std::endl;
  std::cout << "L  :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](full_sol) == 1) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
  std::cout << "~L :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](full_sol) == 0) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
}
```
{% endraw %}
In this program, a qbpp::MapList object ml is defined so that the variable `x[0]` is replaced by the expression `1 - x[1]`.

The `qbpp::replace()` function applies this substitution to the original expression `f`, and the resulting expression is stored in `g`.
As a result, `g` no longer contains the variable `x[0]`; instead, all occurrences of `x[0]` are replaced by the expression `1 - x[1]`.

The Exhaustive Solver is then used to find an optimal solution for `g`, which is stored in `sol`.
Since `x[0]` does not appear in `g`, the solution sol also does not include an assignment for `x[0]`.

To construct a complete solution for the original expression `f`, a `qbpp::Sol` object `full_sol` is created.
First, the assignments in `sol` are copied into `full_sol`.
Then, the value of `x[0]` is explicitly set using the expression `1 - x[1]`, evaluated under `sol`, by calling:
{% raw %}
```cpp
  full_sol.set({{x[0], sol(1 - x[1])}});
```
{% endraw %}
This ensures that the constraint `x[0] = 1 - x[1]` is consistently enforced in the final solution.

This program produces the following output:
{% raw %}
```
sol = 4:{{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],0},{x[7],0}}
f(full_sol) = 4
p(full_sol) = 206
q(full_sol) = 204
L  : 64 47 12 83
~L : 27 74 63 40
```
{% endraw %}
We can confirm that:
- the solution `sol` does not include x[0],
- `x[0]` and `x[1]` take opposite values, and
- 64 and 27 are placed in distinct subsets, as intended.

> **NOTE**
> QUBO++ also provides a member function version of `replace()` for expressions.
> In other words:
> - `f.replace(ml)` updates the expression `f` in place by applying the replacements specified in `ml`.
> - `qbpp::replace(f, ml)` returns a new expression in which the replacements have been applied, without modifying the original expression `f`.
> 
> Use `f.replace(ml)` when you want to permanently modify an existing expression, and
use `qbpp::replace(f, ml)` when you want to keep the original expression unchanged.