---
layout: default
title: "Multiplier Simulation and Factorization"
---

# Multiplier Simulation and Factorization
Multiplication of two integers can be performed using additions.
In this section, we design a multiplier for two 4-bit integers using full adders.
The figure below shows how two 4-bit integers $x_3x_2x_1x_0$ and $y_3y_2y_1y_0$ are multiplied to obtain an 8-bit integer $z_7z_6z_5z_4z_3z_2z_1z_0$.
In this figure, $p_{i,j}=x_iy_j$ ($0\leq i,j\leq 3$) and these partial products are summed to compute the final 8-bit result.

<p align="center">
 <img src="images/multiplication.svg" alt="4-bit multiplication" width="50%">
</p>

We use a 4-bit ripple-carry adder that computes the sum of two 4-bit integers
$a_3a_2a_1a_0$ and $b_3b_2b_1b_0$ producing the 5-bit sum $z_4z_3z_2z_1z_0$.
It consists of four full adders connected by a 5-bit carry wire $c_4c_3c_2c_1c_0$
that propagates carries.

<p align="center">
 <img src="images/adder4.svg" alt="The 4-bit ripple carry adder" width="50%">
</p>

A 4-bit multiplier can be constructed using three 4-bit adders.
They are connected by wires $c_{i,j}$ ($0\leq i\leq 2, 0\leq j\leq 3$) to propagate intermediate sum bits, as shown below:
<p align="center">
 <img src="images/multiplier.svg" alt="The 4-bit multiplier using three 4-bit adders" width="50%">
</p>

## QUBO formulation for multiplier
We will show QUBO formulation for simulating the `N`-bit multiplier.
To do this, we implement functions that construct a full adder, an adder, and a multiplier.

### Full adder
The following QUBO expression simulates a full adder with three input bits `a`, `b`, and `i`, and two output bits: carry-out `o` and sum `s`:
```cpp
qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}
```
The function `fa` returns an expression that enforces consistency between the input and output bits of a full adder.

### Adder
Assume that vectors `a`, `b`, and `s` of `qbpp::Expr` objects represent integers.
We assume that `a` and `b` each have `N` elements representing `N`-bit integers, while `s` has `N + 1` elements representing an `(N + 1)`-bit integer.
The following function adder returns a QUBO expression whose minimum value is 0 if and only if `a + b == s` holds:
{% raw %}
```cpp
qbpp::Expr adder(const qbpp::Vector<qbpp::Expr>& a,
                 const qbpp::Vector<qbpp::Expr>& b,
                 const qbpp::Vector<qbpp::Expr>& s) {
  auto N = a.size();
  auto c = qbpp::var(N + 1);
  auto f = qbpp::toExpr(0);
  for (size_t j = 0; j < N; ++j) {
    f += fa(a[j], b[j], c[j], c[j + 1], s[j]);
  }
  f.replace({{c[0], 0}, {c[N], s[N]}});
  return f;
}
```
{% endraw %}
In this function, `c` is a vector of `N + 1` variables used to connect the carry-out and carry-in signals of the `fa` blocks, forming an `N`-bit ripple-carry adder.

### Multiplier
Assume that vectors `x`, `y`, and `z` of `qbpp::Expr` represent integers.
We assume that `x` and `y` each have `N` elements and that `z` has `2 * N` elements.
The following function multiplier returns a QUBO expression whose minimum value is 0 if and only if `x * y == z` holds.
```cpp
qbpp::Expr multiplier(const qbpp::Vector<qbpp::Expr>& x,
                      const qbpp::Vector<qbpp::Expr>& y,
                      const qbpp::Vector<qbpp::Expr>& z) {
  auto N = x.size();
  auto c = qbpp::var("c", N - 1, N + 1);

  auto f = qbpp::toExpr(0);

  for (size_t i = 0; i < N - 1; ++i) {
    qbpp::Vector<qbpp::Expr> a, b, s;
    for (size_t j = 0; j < N; ++j) {
      b.push_back(x[i + 1] * y[j]);
    }

    if (i == 0) {
      for (size_t j = 0; j < N - 1; ++j) {
        a.push_back(x[0] * y[j + 1]);
      }
      a.push_back(0);
    } else {
      for (size_t j = 0; j < N; ++j) {
        a.push_back(c[i - 1][j + 1]);
      }
    }

    for (size_t j = 0; j < N + 1; ++j) {
      s.push_back(c[i][j]);
    }
    f += adder(a, b, s);
  }
  f += z[0] - x[0] * y[0] == 0;

  qbpp::MapList ml;
  for (size_t i = 0; i < N - 2; ++i) {
    ml.push_back({c[i][0], z[i + 1]});
  }
  for (size_t i = 0; i < N + 1; ++i) {
    ml.push_back({c[N - 2][i], z[N + i - 1]});
  }
  return f.replace(ml).simplify_as_binary();
}
```
This function uses an `(N−1)×(N+1)` matrix `c` of `qbpp::Var` objects to connect the `N−1` adders of `N` bits.
Since each bit of `z` corresponds to one element of `c`, their correspondence is defined in `ml`, and the replacements are performed using `replace()`.

## QUBO++ program for factorization
Using the function `multiplier`, we can factor a composite integer into two factors.
The following program constructs a 4-bit multiplier with
- `x`: 4 binary variables,
- `y`: 4 binary variables,
- `z`: a vector of constants `{1, 1, 1, 1, 0, 0, 0, 1}`, representing the 8-bit integer `10001111` `(143)`, and stores the resulting expression in `f`:
{% raw %}
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}

qbpp::Expr adder(const qbpp::Vector<qbpp::Expr>& a,
                 const qbpp::Vector<qbpp::Expr>& b,
                 const qbpp::Vector<qbpp::Expr>& s) {
  auto N = a.size();
  auto c = qbpp::var(N + 1);
  auto f = qbpp::toExpr(0);
  for (size_t j = 0; j < N; ++j) {
    f += fa(a[j], b[j], c[j], c[j + 1], s[j]);
  }
  return f.replace({{c[0], 0}, {c[N], s[N]}});
}

qbpp::Expr multiplier(const qbpp::Vector<qbpp::Expr>& x,
                      const qbpp::Vector<qbpp::Expr>& y,
                      const qbpp::Vector<qbpp::Expr>& z) {
  auto N = x.size();
  auto c = qbpp::var("c", N - 1, N + 1);

  auto f = qbpp::toExpr(0);

  for (size_t i = 0; i < N - 1; ++i) {
    qbpp::Vector<qbpp::Expr> a, b, s;
    for (size_t j = 0; j < N; ++j) {
      b.push_back(x[i + 1] * y[j]);
    }

    if (i == 0) {
      for (size_t j = 0; j < N - 1; ++j) {
        a.push_back(x[0] * y[j + 1]);
      }
      a.push_back(0);
    } else {
      for (size_t j = 0; j < N; ++j) {
        a.push_back(c[i - 1][j + 1]);
      }
    }

    for (size_t j = 0; j < N + 1; ++j) {
      s.push_back(c[i][j]);
    }
    f += adder(a, b, s);
  }
  f += z[0] - x[0] * y[0] == 0;

  qbpp::MapList ml;
  for (size_t i = 0; i < N - 2; ++i) {
    ml.push_back({c[i][0], z[i + 1]});
  }
  for (size_t i = 0; i < N + 1; ++i) {
    ml.push_back({c[N - 2][i], z[N + i - 1]});
  }
  return f.replace(ml).simplify_as_binary();
}

int main() {
  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  qbpp::Vector<int> z = {1, 1, 1, 1, 0, 0, 0, 1};
  auto f = multiplier(x, y, z).simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  for (auto it = x.rbegin(); it != x.rend(); ++it) {
    std::cout << sol(*it);
  }
  std::cout << " * ";
  for (auto it = y.rbegin(); it != y.rend(); ++it) {
    std::cout << sol(*it);
  }
  std::cout << " = ";
  for (auto it = z.rbegin(); it != z.rend(); ++it) {
    std::cout << *it;
  }
  std::cout << std::endl;
}
```
{% endraw %}
The Easy Solver is executed on `f`, and the obtained solution is stored in `sol`.
The resulting values of `x` and `y` are printed as:
```
1011 * 1101 = 10001111
```
This output indicates $11\times 13 = 143$, demonstrating the factorization result.
