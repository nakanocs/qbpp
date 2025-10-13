# QUBO++ Library Documantaion Version 2025.10.13

QUBO (Quadratic Unconstrained Binary Optimization) models use quadratic functions over binary variables {0,1}.
HUBO (High-order Unconstrained Binary Optimization) generalizes QUBO to polynomial functions of arbitrary order.
The goal in QUBO/HUBO is to find a binary assignment that minimizes the objective.
QUBO++ is a C++ library for constructing and transforming these objectives and efficiently searching for optimal or high-quality solutions.
This document explains how to use the QUBO++ library to design QUBO and HUBO expressions.


# Getting Started: HUBO Expressions and the QUBO++ Library

**A HUBO (High-order Unconstrained Binary Optimization) expression** is a polynomial in binary variables.
For example, consider three binary variables $a$, $b$ and $c$, and define


$$
\begin{aligned}
f(a,b,c)
&=(a+b+c-2)(a+2b+3c-3)^2\\
&= f 9a +9b +9c -6a^2  -12b^2 -18c^2 -18ab -24ac -30bc  +a^3 +4b^3 +9c^3 +5a^2b +7a^2c +8ab^2 +15ac^2  +16b^2c +21bc^2 +22abc  \\
&= 4a +b -5ab -2ac +7bc +22abc
\end{aligned}
$$


Expanding $(a+b+c-2)(a+b+c-3)^2$, we obtain an equivalent polynomial expression in the form of a sum of product terms.
Furthermore, using the fact that $x^2=x$ holds for all binary variables $x\in{0,1}$, we can merge equivalent terms to derive a simplified expression.
The resulting HUBO expression has:
* constant term: none 
* linear terms:  $4a$, $b$ 
* quadratic terms: $-5ab$, $-2ac$, $7bc$ 
* cubic term : $22acbc$

In this document, we refer to $(a+b+c)(a+2b+3c-3)$ as **a (HUBO) formula** and
to the expanded, simplified polynomial $4a +b -5ab -2ac +7bc +22abc$ as **a (HUBO) expression**.
The resulting value of a HUBO polynomial, borrowing the terminology of quantum mechanics, is called **the energy**.
A HUBO problem aims to find a binary assignment of variables that minimizes this energy.
It should be clear that $f(a,b,c)$ achieves optimal energy 0 when $a+b+c=0$ or $a+2b+3c=3$.
Hence, it has three optimal solutions: $(a,b,c)=(0,0,0),(1,1,0),(0,0,1)$.

**QUBO++** is a C++ library for constructing HUBO formulas and deriving the corresponding HUBO expressions.
It also includes solvers to search for optimal or near-optimal solutions to a given HUBO expression.
The following QUBO++ program generates the polynomial expression for $f$ and lists all optimal solutions:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (a + b + c) * qbpp::sqr(a + 2* b + 3* c - 3);
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  qbpp::exhaustive_solver::ExhaustiveSolver solver(f);
  auto sols = solver.search_optimal_solutions();
  std::cout << sols << std::endl;
}
```

The header `qbpp.hpp` enables use of the QUBO++ library, and `qbpp_exhaustive_solver.hpp` provides access to the Exhaustive Solver.
In this program, three variables, $a$, $b$, and $c$, are defined, along with the formula $f$.
The formula $f$ is automatically expanded, and the member function `simplify_as_binary()` 
applies the binary identity $x^2=x$ and merges equivalent terms to yield the simplified HUBO expression.
The Ehaustive Solver then evaluates the energy of $f$ for all $2^3$ possible assignments, and outputs the optimal ones.
The output below confirms the HUBO expression and the four optimal solutions:


```text
f = 4*a +b -5*a*b -2*a*c +7*b*c +22*a*b*c
(0) 0:{{a,0},{b,0},{c,0}}
(1) 0:{{a,0},{b,0},{c,1}}
(2) 0:{{a,1},{b,1},{c,0}}
```

As shown in this QUBO++ program, most QUBO++ objects can be displayed using std::cout, which makes debugging easier.


# QUBO++ Core Classes
The QUBO++ library provides the following core classes for designing polynomial expressions involving variables:

* `qbpp::Var`: Stores a symbolic variable in polynomial expressions.
* `qbpp::Expr`: Stores a polynomial expression of qbpp::Var objects.
* `qbpp::Sol`: Stores a solution for a qbpp::Expr object, mapping qbpp::Var objects to binary values.

## qbpp::Var class

`qbpp::Var` objects can be created using `auto` type deduction and the qbpp::var() function.
A `qbpp::Var` object represents a single variable.
Unlike conventional programming languages, it does not store a variable's value;
instead, it symbolically represents a variable within qbpp::Expr objects.

We can use the `qbpp::var()` function to create a qbpp::Var object `a` that stores a variable with the label string  `a` as follows:

```cpp
auto a = qbpp::var("a");
```

The argument `"a"` specifies the label string for the qbpp::Var object, which is used to represent the object as a string.
This example uses qbpp::Var object `a` with the label same string `a`.
However, it is not necessary to be the same.

A vector of qbpp::Var objects can be defined with a specified size as follows:

```cpp
auto x = qbpp::var("x", 3);
```

This creates a vector `x` with three qbpp::Var objects labeled `X`.
Each object can be accessed using `x[0]`, `x[1]`, and `x[2]`, and their label strings, `x[0]`, `x[1]`, and `x[2]`, combine the base label with the corresponding indices for display purposes.

Multi-dimensional `qbpp::Var` objects can be defined similarly.
For example, the following code creates a $3\times 2$ matrix `x` of `qbpp::Var` objects.

```cpp
auto y = qbpp::var("y", 3, 2);
```

Each object can be accessed as `x[0][0]`, `x[0][1]`, `x[1][0]`, `x[1][1]`, `x[2][0]`, and `x[2][1]` and
they are displayed as `x[0][0]`, `x[0][1]`, `x[1][0]`, `x[1][1]`, `x[2][0]`, and `x[2][1]`.

Higher-dimensional `qbpp::Var` objects can be defined similarly, with no limitation on the number of dimensions.
Additionally, a variable can be created without specifying a label string:

```cpp
auto s = qbpp:var();
```

For such variables, default numbered label string such as `"{0}"`, `"{1}"`, and so on are assigned.

The `qbpp::var()` functions shown above call the constructor of the qbpp::Var class.
However, the constructor should not be called directly, as a set of all qbpp::Var objects is maintained internally.

## qbpp::Expr class

A qbpp::Expr object stores a polynomial over qbpp::Var objects.
Expressions are built via overloaded arithmetic and compound-assignment operators with integers, including
`*` (multiplication), `+` (addition), `-` (subtraction / unary negation), `*=` (compound multiplication), `+=` (compound addition), and `-=` (compound subtraction).
You can also rely on type deduction with auto when assigning expressions:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto f = (x[0] - 1) * (x[1] + 1);
  auto g = f * x[2];
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  std::cout << "g = " << g.simplify_as_binary() << std::endl;
  f -= x[0] - x[1];
  g *= f + 1;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  std::cout << "g = " << g.simplify_as_binary() << std::endl;
}
```
This program prints:
```text
f = -1 +x[0] -x[1] +x[0]*x[1]
g = -x[2] +x[0]*x[2] -x[1]*x[2] +x[0]*x[1]*x[2]
f = -1 +x[0]*x[1]
g = 0
```
It is straightforward to verify that the `qbpp::Expr` objects `f` and `g` correctly represent the intended HUBO expressions.

Please note that type deduction with `auto` does not work as intended when the right-hand side is an integer, a variable, or a product term.
For example, the following lines do not create `qbpp::Expr` objects.
They create an `int`, a `qbpp::Var`, and a `qbpp::Term` respectively—none of which store a HUBO expression.
```cpp
  auto f = 0;
  auto g = x[0];
  auto h = 2 * x[0] * x[1] * x[2];
```
Thus, the assignment below causes a compilation error because `g` is **not** a `qbpp::Expr`:
```
  g += 1;
```

To construct a `qbpp::Expr` from non-Expr values, use `qbpp::toExpr()`, which converts supported types to `qbpp::Expr`:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto f = qbpp::toExpr(0);
  auto g = qbpp::toExpr(x[0]);
  auto h = qbpp::toExpr(2 * x[0] * x[1] * x[2]);
  f = g - h;
  g += h;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  std::cout << "g = " << g.simplify_as_binary() << std::endl;
}
```
This QUBO++ program prints:
```text
f = x[0] -2*x[0]*x[1]*x[2]
g = x[0] +2*x[0]*x[1]*x[2]
```
Alternatively, declare the type explicitly:
```cpp
  qbpp::Expr f = 0;
  qbpp::Expr g = x[0];
  qbpp::Expr h = 2 * x[0] * x[1] * x[2];
```

Arrays of `qbpp::Expr` can be defined in the same way as arrays of `qbpp::Var`.
The following QUBO++ program creates arrays `f` and `g`, each holding `qbpp::Expr` objects:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto f = x + 1;
  std::cout << f.simplify_as_binary() << std::endl;
  auto g = qbpp::expr(2);
  g[0] = f[0] + f[1] + x[2];
  g[1] = 2 * f[1] * x[2];
  std::cout << g << std::endl;
}
```
Since `x` is an array of three `qbpp::Var` objects, `x + 1` performs an element-wise addition and returns an array of three `qbpp::Expr` objects assigned to `f`.
`qbpp::expr(2)` creates an array `g` with two zero-initialized `qbpp::Expr` objects, each capable of storing a HUBO expression.
This program assigns expressions to `g[0]` and `g[1]` and then prints:
```text
{[0],1 +x[0]},{[1],1 +x[1]},{[2],1 +x[2]}
{[0],2 +x[0] +x[1] +x[2]},{[1],2*x[2] +2*x[1]*x[2]}
```

## qbpp::Sol class
A qbpp::Sol object stores a solution to a HUBO problem—that is, an assignment of binary values to variables.
Below we illustrate it with the partition problem.

### Partitioning problem and the QUBO formutation.
Let $N_0, N_1, \ldots, N_{n-1}$ be $n$ integers.
The goal is to split them into two subsets $L$ and $\overline{L}$ so that the two subset sums are as equal as possible.
Equivalently, we minimize
$$
\begin{aligned}
   f(L) & = (\sum_{i\in L} N_i - \sum_{i\in\overline{L}}N_i)^2
\end{aligned}
$$
Introduce binary variable $x_0, x_1,\ldots, x_{n-1}$ where $x_i=1$ iff $i\in L$.
Then,
$$
\begin{aligned}
   \sum_{i\in L} N_i & = \sum_{i=0}^{n-1} N_ix_i\\
   \sum_{i\in\overline{L}}N_i     &= \sum_{i=0}^{n-1}N_i(1-x_i)
\end{aligned}
$$
so
$$
\begin{aligned}
   f(L) & = (\sum_{i=0}^{n-1} N_ix_i - \sum_{i=0}^{n-1}N_i(1-x_i))^2
        = (\sum_{i=0}^{n-1}N_i(1-2x_i))^2
\end{aligned}
$$

Thus, minimizing this quadratic function over binary variables yields an optimal partition, i.e., a QUBO instance.
The following QUBO++ program builds the QUBO for $f(L)$ and finds a solution using the Easy Solver.
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  std::vector<int> w = {37, 82, 64, 59, 21, 73, 47, 95};
  auto x = qbpp::var("x", w.size());
  auto f = qbpp::toExpr(0);
  for (size_t i = 0; i < w.size(); ++i) {
    f += w[i] * (1 - 2 * x[i]);
  }
  f *= f;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "L :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (sol(x[i]) == 1) {
      std::cout << " " << w[i] << "(" << i << ")";
    }
  }
  std::cout << std::endl;
  std::cout << "~L: ";
  for (size_t i = 0; i < w.size(); ++i) {
    if (sol(x[i]) == 0) {
      std::cout << w[i] << "(" << i << ") ";
    }
  }
  std::cout << std::endl;
}
```
For the array of eight integers, this code constructs the quadratic objective $f$.
An EasySolver is created for `f`, its time limit is set to 1.0 s, and `search()` returns a solution stored in the `qbpp::Sol` object `sol`.
Note that `sol(x[i])` returns the assigned value of `x[i]` in `sol`.

This program prints:
```text
f = 228484 -65268*x[0] -129888*x[1] -105984*x[2] -98884*x[3] -38388*x[4] -118260*x[5] -81028*x[6] -145540*x[7] +24272*x[0]*x[1] +18944*x[0]*x[2] +17464*x[0]*x[3] +6216*x[0]*x[4] +21608*x[0]*x[5] +13912*x[0]*x[6] +28120*x[0]*x[7] +41984*x[1]*x[2] +38704*x[1]*x[3] +13776*x[1]*x[4] +47888*x[1]*x[5] +30832*x[1]*x[6] +62320*x[1]*x[7] +30208*x[2]*x[3] +10752*x[2]*x[4] +37376*x[2]*x[5] +24064*x[2]*x[6] +48640*x[2]*x[7] +9912*x[3]*x[4] +34456*x[3]*x[5] +22184*x[3]*x[6] +44840*x[3]*x[7] +12264*x[4]*x[5] +7896*x[4]*x[6] +15960*x[4]*x[7] +27448*x[5]*x[6] +55480*x[5]*x[7] +35720*x[6]*x[7]
sol = 0:{{x[0],1},{x[1],1},{x[2],0},{x[3],0},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
L : 37(0) 82(1) 73(5) 47(6)
~L: 64(2) 59(3) 21(4) 95(7) 
```
Since the energy of `sol` is 0, the instance is partitioned into two subsets with exactly equal sums.

