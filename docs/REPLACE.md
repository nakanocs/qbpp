---
layout: default
title: "REPLACE"
---

# Replace functions

QUBO++ provides the following replace function, which can be used to fix variable values in an expression.
- **`qbpp::replace(const qbpp::Expr& f, const qbpp::MapList& ml)`**

Replaces (fixes) variable values in the expression `f` according to the mapping specified by `ml`.

## Using the replace function to fix variable values
We explain the **`qbpp::replace()`** function using the
[QUBO++ program for partitioning problem](PARTITION).
This program finds a partition of the numbers in the following vector **`w`** into two subsets $P$ and $Q$ ($=\overline{P}$) such that the difference between their sums is minimized:
```cpp
  std::vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
```
We modify this partitioning problem so that 64 must belong to $P$ and 27 must belong to $Q$, ensuring that they are placed in distinct subsets.

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
  f.simplify_as_binary();

  qbpp::MapList ml({{x[0], 1}, {x[1], 0}});
  auto g = qbpp::replace(f, ml);
  g.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(g);
  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f).set(sol).set(ml);

  std::cout << "sol = " << sol << std::endl;
  std::cout << "ml = " << ml << std::endl;
  std::cout << "full_sol = " << full_sol << std::endl;
  std::cout << "f(full_sol) = " << f(full_sol) << std::endl;
  std::cout << "p(full_sol) = " << p(full_sol) << std::endl;
  std::cout << "q(full_sol) = " << q(full_sol) << std::endl;
  std::cout << "P :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](full_sol) == 1) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
  std::cout << "Q :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](full_sol) == 0) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
}
```
{% endraw %}

First, a `qbpp::MapList` object **`ml`** is defined, which specifies fixed values for the variables `x[0]` and `x[1]`.
Given the original expression `f` for the partitioning problem and the qbpp::MapList object `ml`, the **`qbpp::replace()`** function is used to replace `x[0]` and `x[1]` in `f` with the constants 1 and 0, respectively.
The resulting expression is stored in **`g`**.

The Exhaustive Solver is then applied to `g` to find an optimal solution, which is stored in `sol`.
Note that the expression `g` no longer contains the variables `x[0]` and `x[1]`, and consequently, `sol` also does not include assignments for these variables.

To construct a complete solution `full_sol` for the original expression `f`,
a `qbpp::Sol` object is created for `f`, and both `sol` and `ml` are applied to it.

From the output below, we can confirm that 64 is placed in $P$ and 27 is placed in $Q$, as intended:
{% raw %}
```
sol = 4:{{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],0},{x[7],0}}
ml = {{x[0],1},{x[1],0}}
full_sol = 4:{{x[0],1},{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],0},{x[7],0}}
f(full_sol) = 4
p(full_sol) = 206
q(full_sol) = 204
P : 64 47 12 83
Q : 27 74 63 40
```
{% endraw %}
## Using the replace function to replace variables with expressions
The `replace()` function can also replace a variable with an expression, not only with a constant value.

Here, we present a more sophisticated way to ensure that 64 and 27 are placed in distinct subsets in the partitioning problem introduced above.
The key idea is to replace the variable `x[0]` in the expression `f` with the expression `1 - x[1]`.
This enforces the constraint that `x[0]` and `x[1]` always take opposite values, guaranteeing that the corresponding elements (64 and 27) belong to different subsets.

The following C++ program implements this idea:
{% raw %}
```cpp
  qbpp::MapList ml({{x[0], 1 - x[1]}});
  auto g = qbpp::replace(f, ml);
  g.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(g);
  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f).set(sol, ml);
```
{% endraw %}
In this program, a qbpp::MapList object ml is defined so that the variable `x[0]` is replaced by the expression `1 - x[1]`.

The `qbpp::replace()` function applies this substitution to the original expression `f`, and the resulting expression is stored in `g`.
As a result, `g` no longer contains the variable `x[0]`; instead, all occurrences of `x[0]` are replaced by the expression `1 - x[1]`.

The Exhaustive Solver is then used to find an optimal solution for `g`, which is stored in `sol`.
Since `x[0]` does not appear in `g`, the solution sol also does not include an assignment for `x[0]`.

To construct a complete solution `full_sol` for the original expression `f`,
a `qbpp::Sol` object is created for `f`, and both `sol` and `ml` are applied to it.
Note that `set(ml)` alone cannot be used when the values in `ml` involve variables (e.g., `1 - x[1]`).
In such cases, `sol` and `ml` must be set using a single `set(sol, ml)` call,
so that the expressions in `ml` are evaluated using `sol`.

This program produces the following output:
{% raw %}
```
sol = 4:{{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],0},{x[7],0}}
ml = {{x[0],1 -x[1]}}
full_sol = 4:{{x[0],1},{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],0},{x[7],0}}
f(full_sol) = 4
p(full_sol) = 206
q(full_sol) = 204
P : 64 47 12 83
Q : 27 74 63 40
```
{% endraw %}
We can confirm that:
- the solution `sol` does not include x[0],
- `x[0]` and `x[1]` take opposite values, and
- 64 and 27 are placed in distinct subsets, as intended.

## Replace Functions for Integer Variables
Integer variables can be replaced with fixed integer values using the `replace()` functions.

Here, we demonstrate this feature using a simple **multiplication expression**.
Let $p$, $q$, and $r$ be integer variables, and consider the following constraint:

$$
\begin{aligned}
p\times q - r &=0
\end{aligned}
$$

This expression can be interpreted in several ways, leading to different types of problems:
- **Multiplication**: For fixed values of $p$ and $q$, find $r$ that satisfies the expression.
- **Factorization**: For a fixed value of $r$, find $p$ and $q$ that satisfy the expression.
- **Division**: For fixed values of $p$ and $r$, find $q$ that satisfies the expression.

Using the **`qbpp::replace()`** function, integer variables can be fixed to constant values.
We demonstrate QUBO++ programs that solve these problems using `qbpp::replace()`.

### Multiplication
The following program fixes $p=5$ and $q=7$ and finds the product $r=35$:
{% raw %}
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 8;
  auto q = 2 <= qbpp::var_int("q") <= 8;
  auto r = 2 <= qbpp::var_int("r") <= 40;
  auto f = p * q - r == 0;
  f.simplify_as_binary();

  qbpp::MapList ml({{p, 5}, {q, 7}});
  auto g = qbpp::replace(f, ml);
  g.simplify_as_binary();
  std::cout << "g = " << g << std::endl;

  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.target_energy(0);
  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f).set(sol).set(ml);

  std::cout << "p= " << full_sol(p) << ", q= " << full_sol(q)
            << ", r= " << full_sol(r) << std::endl;
}
```
{% endraw %}
In this program, a `qbpp::MapList` object `ml` is used to fix the values of the integer variables 
`p` and `q` in the original expression `f`.
By applying `qbpp::replace(f, ml)`, the variables `p` and `q` in `f` are replaced with the constants 5 and 7, respectively.
The resulting expression is stored in `g`, which now contains only the variable `r`.
The Easy Solver is then applied to `g`, and the resulting solution is stored in `sol`.
To construct a complete solution `full_sol` for the original expression `f`,
a `qbpp::Sol` object is created for `f`, and both `sol` and `ml` are applied to it.
Finally, the values of `p`, `q`, and `r` are printed.

This program produces the following output, confirming that the multiplication result is obtained correctly:
```
g = 1089 -65*r[0] -128*r[1] -248*r[2] -464*r[3] -800*r[4] -413*r[5] +4*r[0]*r[1] +8*r[0]*r[2] +16*r[0]*r[3] +32*r[0]*r[4] +14*r[0]*r[5] +16*r[1]*r[2] +32*r[1]*r[3] +64*r[1]*r[4] +28*r[1]*r[5] +64*r[2]*r[3] +128*r[2]*r[4] +56*r[2]*r[5] +256*r[3]*r[4] +112*r[3]*r[5] +224*r[4]*r[5]
p= 5, q= 7, r= 35
```

### Factorization
For the factorization of $r=35$, the `qbpp::MapList` object `ml` in the QUBO++ program is modified as follows:
{% raw %}
```cpp
  qbpp::MapList ml({{r, 35}});
```
{% endraw %}
By fixing the value of $r$, the solver searches for integer values of $p$ and $q$ that satisfy the constraint

$$
\begin{aligned}
p\times q&=35
\end{aligned}
$$


This program produces the following output:
```
g = 961 -120*p[0] -232*p[1] -336*p[2] -120*q[0] -232*q[1] -336*q[2] +16*p[0]*p[1] +24*p[0]*p[2] -45*p[0]*q[0] -80*p[0]*q[1] -105*p[0]*q[2] +48*p[1]*p[2] -80*p[1]*q[0] -136*p[1]*q[1] -168*p[1]*q[2] -105*p[2]*q[0] -168*p[2]*q[1] -189*p[2]*q[2] +16*q[0]*q[1] +24*q[0]*q[2] +48*q[1]*q[2] +20*p[0]*p[1]*q[0] +48*p[0]*p[1]*q[1] +84*p[0]*p[1]*q[2] +30*p[0]*p[2]*q[0] +72*p[0]*p[2]*q[1] +126*p[0]*p[2]*q[2] +20*p[0]*q[0]*q[1] +30*p[0]*q[0]*q[2] +60*p[0]*q[1]*q[2] +60*p[1]*p[2]*q[0] +144*p[1]*p[2]*q[1] +252*p[1]*p[2]*q[2] +48*p[1]*q[0]*q[1] +72*p[1]*q[0]*q[2] +144*p[1]*q[1]*q[2] +84*p[2]*q[0]*q[1] +126*p[2]*q[0]*q[2] +252*p[2]*q[1]*q[2] +16*p[0]*p[1]*q[0]*q[1] +24*p[0]*p[1]*q[0]*q[2] +48*p[0]*p[1]*q[1]*q[2] +24*p[0]*p[2]*q[0]*q[1] +36*p[0]*p[2]*q[0]*q[2] +72*p[0]*p[2]*q[1]*q[2] +48*p[1]*p[2]*q[0]*q[1] +72*p[1]*p[2]*q[0]*q[2] +144*p[1]*p[2]*q[1]*q[2]
p= 5, q= 7, r= 35
```

### Division
To compute the division $r/p$ with $r=35$ and $p=5$, the qbpp::MapList object ml in the QUBO++ program is modified as follows:
{% raw %}
```cpp
  qbpp::MapList ml({{p, 5}, {r, 35}});
```
{% endraw %}
This program produces the following output:
```cpp
g = 625 -225*q[0] -400*q[1] -525*q[2] +100*q[0]*q[1] +150*q[0]*q[2] +300*q[1]*q[2]
p= 5, q= 7, r= 35
```
This confirms that the division result $q=r/p=7$ is correctly obtained.

> **NOTE**
> QUBO++ also provides a member function version of `replace()` for expressions.
> In other words:
> - **`f.replace(ml)`** updates the expression `f` in place by applying the replacements specified in `ml`.
> - **`qbpp::replace(f, ml)`** returns a new expression in which the replacements have been applied, without modifying the original expression `f`.
>  Use `f.replace(ml)` when you want to permanently modify an existing expression, and use `qbpp::replace(f, ml)` when you want to keep the original expression unchanged.
