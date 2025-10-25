# QUBO++ Library Documantaion Version 2025.10.16

**QUBO (Quadratic Unconstrained Binary Optimization) models** use quadratic functions over binary variables {0,1}.
**HUBO** (High-order Unconstrained Binary Optimization) generalizes QUBO to polynomial functions of arbitrary order.
The goal in QUBO/HUBO is to find a binary assignment that minimizes the objective.
**QUBO++** is a C++ library for constructing and transforming these objectives and efficiently searching for optimal or high-quality solutions.
This document explains how to use the QUBO++ library to design QUBO and HUBO expressions.

It should be clear that the class of **HUBO** includes **QUBO**.
Therefore, in this document, we use the term **QUBO** when an expression is guaranteed to be quadratic; otherwise, we refer to it as **HUBO**.

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

As shown in this QUBO++ program, most QUBO++ objects can be displayed using `std::cout`, which makes debugging easier.


# QUBO++ Core Classes
The QUBO++ library provides the following core classes for designing polynomial expressions involving variables:

| Classes         | Role                                                                                 |
|-----------------|--------------------------------------------------------------------------------------|
| qbpp::Var       | Stores a single symbolic variable.                                                   |
| qbpp::Expr      | Stores a polynomial expression of qbpp::Var objects.                                 |
| qbpp::Sol       | Stores a solution for qbpp::Expr objects.                                            |
| qbpp::VarInt    | Stores a set of qbpp::Var objects representing an integer.                           |
| qbpp::Term      | Stores a single term included in qbpp::Expr objects.                                 |



# qbpp::Var class: Variable object

## Creating qbpp::Var objects
`qbpp::Var` objects can be created using `auto` type deduction and the qbpp::var() function.
A `qbpp::Var` object represents a single variable.
Unlike conventional programming languages, it does not store a variable's value;
instead, it symbolically represents a variable within `qbpp::Expr` objects.

We can use the `qbpp::var()` function to create a `qbpp::Var` object `a` that stores a variable with the label string  `a` as follows:

```cpp
auto a = qbpp::var("a");
```

The argument `"a"` specifies the name string for the `qbpp::Var` object, which is used to represent the object as a string.
In this example, the variable is `qbpp::Var` `a` and its name is also `a`.
The name does not have to match the C++ variable identifier, but using the same string is recommended.

A vector of `qbpp::Var` objects can be defined with a specified size as follows:

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

### Summary
* `qbpp::var(const std::string& name)` returns a `qbpp::Var` object representing a symbolic variable with the given `name`, which is used when displaying it with `std::cout`.
* `qbpp::var(const std::string& label, size_t n0, size_t n1, ...)` returns an array of `qbpp::Var` objects of shape `n0 × n1 × ...`, with `name`. Each `qbpp::Var` object can be accessed using the array notation `x[i0][i1]...`, just like a regular C++ array or multi-dimensional variable. QUBO++ has no limitation of the dsimension depth.
* If `name` is omitted, default numbered names such as `{0}`, `{1}`, and so on are assigned.


# qbpp::Expr class: Expression object

## Creating qbpp::Expr objects
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

## Summary
* `qbpp::expr()` returns a single qbpp::Expr object that can store a polynomial over `qbpp::Var` objects.
* Unlike `qbpp::Var` objects, `qbpp::Expr` objects have no name.
* `qbpp::expr(size_t n0, size_t n1, ...)` creates an array of `qbpp::Expr` objects with dimensions `n0 × n1 × ...`, each capable of storing a polynomial over `qbpp::Var` objects.
* Each `qbpp::Expr` in the array can be accessed using the notation `f[i0][i1]...`.
* Arithmetic and compound assignment operators for `qbpp::Expr` are supported. In addition, various arithmetic operators and functions for qbpp::Expr objects are available and will be explained later.

# qbpp::Sol class: Solution object
A qbpp::Sol object stores a solution to a HUBO problem—that is, an assignment of binary values to variables.
Below we illustrate it with the partition problem.

## Partitioning problem and the QUBO formutation.
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
  std::vector<int> N = {37, 82, 64, 59, 21, 73, 47, 95};
  auto x = qbpp::var("x", N.size());
  auto f = qbpp::toExpr(0);
  for (size_t i = 0; i < N.size(); ++i) {
    f += N[i] * (1 - 2 * x[i]);
  }
  f *= f;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "L :";
  for (size_t i = 0; i < N.size(); ++i) {
    if (sol(x[i]) == 1) {
      std::cout << " " << N[i] << "(" << i << ")";
    }
  }
  std::cout << std::endl;
  std::cout << "~L: ";
  for (size_t i = 0; i < N.size(); ++i) {
    if (sol(x[i]) == 0) {
      std::cout << N[i] << "(" << i << ") ";
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

## Creating qbpp::Sol objects and updating assigned binary values
A `qbpp::Sol` object is created associated with a fixed `qbpp::Expr` object, which cannot be changed.
It stores a solution: a mapping from qbpp::Var objects in the `qbpp::Expr` object to binary values (0/1).
The stored solution can be updated.

The following QUBO++ program creates a HUBO expression $f(a,b,c) = (a+b+c)^3$ and an associated solution using the `qbpp::Sol` constructor, which creates a zero-initialized `qbpp::Sol` object.
After that, the binary value assignment to qbpp::Var objects is updated by the `set()` member function.
The `set()` function accepts a `qbpp::Var` object with the binary value to be updated.
It also accepts a list of pairs of a `qbpp::Var` object and a binary value for batch updating.

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = a + 2 * b + 3 * c;
  f *= f * f;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto sol = qbpp::Sol(f);
  std::cout << "0: sol = " << sol << std::endl;
  sol.set(a, 1);
  std::cout << "1: sol = " << sol << std::endl;
  sol.set(c, 1);
  std::cout << "1: sol = " << sol << std::endl;
  sol.set({{a, 0}, {b, 1}});
  std::cout << "2: sol = " << sol << std::endl;
}
```

The following output confirms that the assigned binary values and the resulting energy values are correct.

```text
f = a +8*b +27*c +18*a*b +36*a*c +90*b*c +36*a*b*c
0: sol = 0:{{a,0},{b,0},{c,0}}
1: sol = 1:{{a,1},{b,0},{c,0}}
1: sol = 64:{{a,1},{b,0},{c,1}}
2: sol = 125:{{a,0},{b,1},{c,1}}
```

Solvers bundled with QUBO++ provide member functions that search for a solution and return a `qbpp::Sol` object containing the result. 
n typical usage, a `qbpp::Sol` object is obtained this way.

## Reading the value of binary variables and the energy
The binary value assigned to a `qbpp::Var` in a `qbpp::Sol` can be obtained with the call `operator()`:
`sol(x)` returns 0 or 1 for that variable `x`.
A `qbpp::Sol` also provides `energy()`, which returns the (cached) energy of the stored solution.
The call `operator()` of `qbpp::Sol` also accepts a `qbpp::Expr`: `sol(expr)` evaluates the expression under the current assignment.

The energy value in `qbpp::Sol` is cached lazily. When you change any variable via set(), the cached energy is invalidated.
To (re)compute and store the energy, call `comp_energy()`; subsequent calls to `energy()` will then return the cached value.

The example below defines three binary variables `a`, `b`, `c`, builds two penalty expressions
$f = (a + b + c − 1)^2$ and $g = (a + 2b + 3c − 2)^2$, and combines them as $h = f + g$.
It then runs a simple solver to find a solution (targeting energy 0), prints the variable values and energies, flips a to 1 using `set()`, and demonstrates that the energy cache is invalidated and recomputed via `comp_energy()`.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = qbpp::sqr(a + b + c - 1);
  auto g = qbpp::sqr(a + 2 * b + 3 * c - 2);
  auto h = f + g;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  std::cout << "g = " << g.simplify_as_binary() << std::endl;
  std::cout << "h = " << h.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(h);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << "a = " << sol(a) << std::endl;
  std::cout << "b = " << sol(b) << std::endl;
  std::cout << "c = " << sol(c) << std::endl;
  std::cout << "h(a,b,c) = " << sol.energy() << std::endl;
  std::cout << "f(a,b,c) = " << sol(f) << std::endl;
  std::cout << "g(a,b,c) = " << sol(g) << std::endl;
  sol.set(a, 1);
  std::cout << "a = " << sol(a) << std::endl;
  std::cout << "b = " << sol(b) << std::endl;
  std::cout << "c = " << sol(c) << std::endl;
  std::cout << "h(a,b,c) = " << sol.comp_energy() << std::endl;
  std::cout << "f(a,b,c) = " << sol(f) << std::endl;
  std::cout << "g(a,b,c) = " << sol(g) << std::endl;
}
```
This program prints:
```text
f = 1 -a -b -c +2*a*b +2*a*c +2*b*c
g = 4 -3*a -4*b -3*c +4*a*b +6*a*c +12*b*c
h = 5 -4*a -5*b -4*c +6*a*b +8*a*c +14*b*c
a = 0
b = 1
c = 0
h(a,b,c) = 0
f(a,b,c) = 0
g(a,b,c) = 0
a = 1
b = 1
c = 0
h(a,b,c) = 2
f(a,b,c) = 1
g(a,b,c) = 1
```

## Summary
* Solvers return `qbpp::Sol` objects that store the obtained solution.
* `qbpp::Sol(f)` constructs a zero-initialized `qbpp::Sol` for the expression `f` (qbpp::Expr).
* `set(x, 0)` / `set(x, 1)` set the value of variable `x` to `0` / `1`, respectively.
* `energy()` returns the cached energy. If any variable value has been modified via `set()`, the cache is invalidated and calling `energy()` raises an error.
* `comp_energy()` recomputes the energy, stores it in the cache, and returns the value.

# qbpp::VarInt class: Integer class

A qbpp::VarInt object is used to represent an integer value within a specified range.
It corresponds to a qbpp::Expr object composed of multiple qbpp::Var objects.
qbpp::VarInt objects can be created using the qbpp::var_int() global function.

A single qbpp::VarInt object `x`, representing an integer value in the range $[1,10]$, can be created and displayed with the following code:

```cpp
  auto x = 1 <= qbpp::var_int("x") <= 10;
  std::cout << "x = " << x << std::endl;
```

From the output of the code below, we can see that `x` is a qbpp::Expr object, composed of four qbpp::Var objects: `x[0]`, `x[1]`, `x[2]`, and `x[3]`, representing an integer value in the range @f$[1,10]@f$.

```cpp
x = 1 +x[0] +2*x[1] +4*x[2] +2*x[3]
```

The integers specifying the range can be negative. 

Similarly to the creation of qbpp::Var objects, a multi-dimensional array of qbpp::VarInt objects with the same specified range can be created as follows:

```cpp
  auto x = 1 <= qbpp::var_int("x", 3, 4) <= 10;
  std::cout << str(x, "x") << std::endl;
```

Each qbpp::VarInt object can be accessed by its indices, and the corresponding qbpp::Expr object can be displayed as follows:

```cpp
{x[0][0],1 +x[0][0][0] +2*x[0][0][1] +4*x[0][0][2] +2*x[0][0][3]},{x[0][1],1 +x[0][1][0] +2*x[0][1][1] +4*x[0][1][2] +2*x[0][1][3]},{x[0][2],1 +x[0][2][0] +2*x[0][2][1] +4*x[0][2][2] +2*x[0][2][3]},{x[0][3],1 +x[0][3][0] +2*x[0][3][1] +4*x[0][3][2] +2*x[0][3][3]},{x[1][0],1 +x[1][0][0] +2*x[1][0][1] +4*x[1][0][2] +2*x[1][0][3]},{x[1][1],1 +x[1][1][0] +2*x[1][1][1] +4*x[1][1][2] +2*x[1][1][3]},{x[1][2],1 +x[1][2][0] +2*x[1][2][1] +4*x[1][2][2] +2*x[1][2][3]},{x[1][3],1 +x[1][3][0] +2*x[1][3][1] +4*x[1][3][2] +2*x[1][3][3]},{x[2][0],1 +x[2][0][0] +2*x[2][0][1] +4*x[2][0][2] +2*x[2][0][3]},{x[2][1],1 +x[2][1][0] +2*x[2][1][1] +4*x[2][1][2] +2*x[2][1][3]},{x[2][2],1 +x[2][2][0] +2*x[2][2][1] +4*x[2][2][2] +2*x[2][2][3]},{x[2][3],1 +x[2][3][0] +2*x[2][3][1] +4*x[2][3][2] +2*x[2][3][3]}
```

If a string is not provided, default names such as `{0}`, `{1}`, and so on are assigned, similarly to `qbpp::Var` objects.

In most cases, `qbpp::VarInt` objects are implicitly converted to their corresponding `qbpp::Expr` objects.

# Operators and Functions for qbpp::Expr

Operators and functions for `qbpp::Expr` are divided into two categories: **global functions** and **member functions** as follows:

* **Global functions**: Take at least one `ExprType` object as an argument. In many cases, they return a `qbpp::Expr` object.
* **Member functions**: Defined as member functions of the `qbpp::Expr` class. In many cases, they update the calling object based on the result of the function.

For example, the `sqr()` function, which computes the square of a qbpp::Expr object, is available as both a global and a member function.
The global version, `sqr(f)`, returns the square of `f` without updating `f`.
In contrast, the member function `f.sqr()` returns the square of `f` and updates `f` with this value.

The following table summarizes the available operators and functions:

| Operators/Functions           | Operator Symbols/Function Names                      | Function Type | Return Type    | Argument Type          |
|-------------------------------|------------------------------------------------------|---------------|----------------|------------------------|
| Type Conversion               | toExpr()                                             | Global        | qbpp::Expr     | ExprType               |
| Type Conversion               | toInt()                                              | Global        | Int            | Expr                   |
| Assignment                    | `=`                                                  | Member        | qbpp::Expr     | ExprType               |
| Binary Operators              | `+`, `-`, `*`                                        | Global        | qbpp::Expr     | ExprType-ExprType      |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member        | qbpp::Expr     | ExprType               |
| Division                      | `/`                                                  | Global        | qbpp::Expr     | ExprType-Int           |
| Compound Division             | `/=`                                                 | Member        | qbpp::Expr     | Int                    |
| Unary Operators               | `+`, `-`                                             | Global        | qbpp::Expr     | ExprType               |
| Comparison (Equality)         | `==`                                                 | Global        | qbpp::ExprExpr | ExprType-Int           |
| Comparison (Range Comparison) | `<= <=`                                              | Global        | qbpp::ExprExpr | IntInf-ExprType-IntInf |
| Square                        | sqr()                                                | Global        | qbpp::Expr     | ExprType               |
| Square                        | sqr()                                                | Member        | qbpp::Expr     | -                      |
| GCD                           | gcd()                                                | Global        | Int            | ExprType               |
| Simplify                      | simplify(), simplify_as_binary(), simplify_as_spin() | Global        | qbpp::Expr     | ExprType               |
| Simplify                      | simplify(), simplify_as_binary(), simplify_as_spin() | Member        | qbpp::Expr     | -                      |
| Eval                          | eval()                                               | Global        | Int            | ExprType-MapList       |
| Eval                          | operator()                                           | Member        | Int            | ExprType-MapList       |
| Replace                       | replace()                                            | Global        | qbpp::Expr     | ExprType-MapList       |
| Replace                       | replace()                                            | Member        | qbpp::Expr     | MapList                |
| Reduce                        | reduce()                                             | Global        | qbpp::Expr     | ExprType               |
| Reduce                        | reduce()                                             | Member        | qbpp::Expr     | MapList                |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Global        | qbpp::Expr     | ExprType               |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Member        | qbpp::Expr     | -                      |

In this table, **Int** denotes an integer, while **IntInf** represents either an integer, -qbpp::inf, or +qbpp::inf, which signify infinite values.
Additionally, `qbpp::ExprExpr` is a derived class of `qbpp::Expr` that encapsulates another `qbpp::Expr` object.

## Simplification Functions for qbpp::Expr

Simplification functions reduce expressions by sorting variables within terms, merging redundant terms, and ordering all terms. The following simplification functions are available:

* `simplify()`: Sorts variables within terms, merges redundant terms, and orders all terms.
* `simplify_as_binary()`: Simplifies expressions under the assumption that all variables take binary values ($0/1$), such that $x^2 = x$ for all variables $x$.
* `simplify_as_spin()`: Simplifies expressions assuming all variables are spin variables ($-1/+1$), such that $x^2 = 1$ for all variables $x$.

Therefore, after applying either `simplify_as_binary()` or `simplify_as_spin()`, no terms will contain duplicated variables.

The following example demonstrates how these functions are used and the differences in their outputs:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto f = qbpp::sqr(a + b - 1);
  std::cout << "f = " << f << std::endl;
  std::cout << "qbpp::simplify(f) = " << qbpp::simplify(f) << std::endl;
  std::cout << "qbpp::simplify_as_binary(f) = " << qbpp::simplify_as_binary(f) << std::endl;
  std::cout << "qbpp::simplify_as_spin(f) = " << qbpp::simplify_as_spin(f) << std::endl;
}
```

The output of this code will display the simplified expressions as follows:

```cpp
f = 1 -a -b -a -b +a*a +a*b +b*a +b*b
qbpp::simplify(f) = 1 -2*a -2*b +a*a +2*a*b +b*b
qbpp::simplify_as_binary(f) = 1 -a -b +2*a*b
qbpp::simplify_as_spin(f) = 3 -2*a -2*b +2*a*b
```

Variables within terms are ordered so that earlier-defined variables appear first.
Additionally, lower-degree terms appear before higher-degree terms.
For terms of the same degree, lexicographically earlier terms are prioritized.

## Comparison Operators for qbpp::Expr

Comparison operators, equality (`==`), and range comparison (`<= <=`) are supported between `qbpp::Expr` objects and integers.
In addition to integers, infinite values represented as `+qbpp::inf` and `-qbpp::inf` can also be specified.
* Expr `==` Int: Returns a qbpp::Expr object that evaluates to a minimum value of 0 if the equality is satisfied.
* IntInf `<=` Expr `<=` IntInf: Returns a qbpp::Expr object that evaluates to a minimum value of 0 if the range comparison is satisfied.

Single inequalities are intentionally not supported to prevent potential misuse caused by misunderstandings.
Instead, `qbpp::inf` can be used to represent single inequalities. Specifically:
* Int `<=` Expr can be expressed as Int `<=` Expr `<=` `+qbpp::inf`, and
* Expr `<=` Int can be expressed as `-qbpp::inf` `<=` Expr `<=` Int.

Other comparison operators such as !=, <, >=, and > are not supported.

### Example of Equality
As an example of using equality operators, we present a QUBO++ program that solves the following linear equations:

$$
\begin{aligned}
x + y &= 10 \\
2x+4y &= 24 
\end{aligned}
$$

The following code demonstrates how to solve these linear equations using the Exhaustive Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto x = 0 <= qbpp::var_int("x") <= 10;
  auto y = 0 <= qbpp::var_int("y") <= 10;
  auto f = x + y == 10;
  auto g = 2 * x + 4 * y == 24;
  auto h = f + g;
  h.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(h);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "x = " << sol(x) << std::endl;
  std::cout << "y = " << sol(y) << std::endl;
  std::cout << "*f = " << *f << " = " << sol(*f) << std::endl;
  std::cout << "*g = " << *g << " = " << sol(*g) << std::endl;
  std::cout << "f = " << f << " = " << sol(f) << std::endl;
  std::cout << "g = " << g << " = " << sol(g) << std::endl;
}
```
In this code, `f` and `g` are instances of the `qbpp::ExprExpr` class, which is derived from `qbpp::Expr`.
The base class stores a QUBO expression that attains a minimum value of 0 when the equalities hold.
Additionally, the derived class contains another `qbpp::Expr` object representing the QUBO expression on the left-hand side of the equality.
This additional `qbpp::Expr` object can be accessed using the `operator*`, meaning that `*f` and `*g` return the QUBO expression on the left-hand side.

The output below confirms that the correct solution to the linear equations is obtained:
```cpp
sol = 0:{{x[0],1},{x[1],0},{x[2],1},{x[3],1},{y[0],0},{y[1],1},{y[2],0},{y[3],0}}
x = 8
y = 2
*f = x[0] +2*x[1] +4*x[2] +3*x[3] +y[0] +2*y[1] +4*y[2] +3*y[3] = 10
*g = 2*x[0] +4*x[1] +8*x[2] +6*x[3] +4*y[0] +8*y[1] +16*y[2] +12*y[3] = 24
f = 100 +x[0]*x[0] +2*x[1]*x[0] +4*x[2]*x[0] +3*x[3]*x[0] +y[0]*x[0] +2*y[1]*x[0] +4*y[2]*x[0] +3*y[3]*x[0] +2*x[0]*x[1] +4*x[1]*x[1] +8*x[2]*x[1] +6*x[3]*x[1] +2*y[0]*x[1] +4*y[1]*x[1] +8*y[2]*x[1] +6*y[3]*x[1] +4*x[0]*x[2] +8*x[1]*x[2] +16*x[2]*x[2] +12*x[3]*x[2] +4*y[0]*x[2] +8*y[1]*x[2] +16*y[2]*x[2] +12*y[3]*x[2] +3*x[0]*x[3] +6*x[1]*x[3] +12*x[2]*x[3] +9*x[3]*x[3] +3*y[0]*x[3] +6*y[1]*x[3] +12*y[2]*x[3] +9*y[3]*x[3] +x[0]*y[0] +2*x[1]*y[0] +4*x[2]*y[0] +3*x[3]*y[0] +y[0]*y[0] +2*y[1]*y[0] +4*y[2]*y[0] +3*y[3]*y[0] +2*x[0]*y[1] +4*x[1]*y[1] +8*x[2]*y[1] +6*x[3]*y[1] +2*y[0]*y[1] +4*y[1]*y[1] +8*y[2]*y[1] +6*y[3]*y[1] +4*x[0]*y[2] +8*x[1]*y[2] +16*x[2]*y[2] +12*x[3]*y[2] +4*y[0]*y[2] +8*y[1]*y[2] +16*y[2]*y[2] +12*y[3]*y[2] +3*x[0]*y[3] +6*x[1]*y[3] +12*x[2]*y[3] +9*x[3]*y[3] +3*y[0]*y[3] +6*y[1]*y[3] +12*y[2]*y[3] +9*y[3]*y[3] -10*x[0] -20*x[1] -40*x[2] -30*x[3] -10*y[0] -20*y[1] -40*y[2] -30*y[3] -10*x[0] -20*x[1] -40*x[2] -30*x[3] -10*y[0] -20*y[1] -40*y[2] -30*y[3] = 0
g = 576 +4*x[0]*x[0] +8*x[1]*x[0] +16*x[2]*x[0] +12*x[3]*x[0] +8*y[0]*x[0] +16*y[1]*x[0] +32*y[2]*x[0] +24*y[3]*x[0] +8*x[0]*x[1] +16*x[1]*x[1] +32*x[2]*x[1] +24*x[3]*x[1] +16*y[0]*x[1] +32*y[1]*x[1] +64*y[2]*x[1] +48*y[3]*x[1] +16*x[0]*x[2] +32*x[1]*x[2] +64*x[2]*x[2] +48*x[3]*x[2] +32*y[0]*x[2] +64*y[1]*x[2] +128*y[2]*x[2] +96*y[3]*x[2] +12*x[0]*x[3] +24*x[1]*x[3] +48*x[2]*x[3] +36*x[3]*x[3] +24*y[0]*x[3] +48*y[1]*x[3] +96*y[2]*x[3] +72*y[3]*x[3] +8*x[0]*y[0] +16*x[1]*y[0] +32*x[2]*y[0] +24*x[3]*y[0] +16*y[0]*y[0] +32*y[1]*y[0] +64*y[2]*y[0] +48*y[3]*y[0] +16*x[0]*y[1] +32*x[1]*y[1] +64*x[2]*y[1] +48*x[3]*y[1] +32*y[0]*y[1] +64*y[1]*y[1] +128*y[2]*y[1] +96*y[3]*y[1] +32*x[0]*y[2] +64*x[1]*y[2] +128*x[2]*y[2] +96*x[3]*y[2] +64*y[0]*y[2] +128*y[1]*y[2] +256*y[2]*y[2] +192*y[3]*y[2] +24*x[0]*y[3] +48*x[1]*y[3] +96*x[2]*y[3] +72*x[3]*y[3] +48*y[0]*y[3] +96*y[1]*y[3] +192*y[2]*y[3] +144*y[3]*y[3] -48*x[0] -96*x[1] -192*x[2] -144*x[3] -96*y[0] -192*y[1] -384*y[2] -288*y[3] -48*x[0] -96*x[1] -192*x[2] -144*x[3] -96*y[0] -192*y[1] -384*y[2] -288*y[3] = 0
```

### Example of Inequality

The following code computes qbpp::Expr object `f` that takes minimum value of 0 if
$
2\leq a+2b+3c +4d\leq 4
$
is satisfied and finds all optimal solutions:

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = 2 <= a + 2 * b + 3 * c + 4 * d <= 4;
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  (*f).simplify_as_binary();
  std::cout << "*f = " << *f << std::endl;
  qbpp::exhaustive_solver::ExhaustiveSolver solver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto &sol : sols) {
    std::cout << sol << " *f = " << sol(*f) << std::endl;
  }
}
```

The output of this code is as follows:

```cpp
f = 6 -4*a -6*b -6*c -4*d +6*{0} +4*a*b +6*a*c +8*a*d -2*a*{0} +12*b*c +16*b*d -4*b*{0} +24*c*d -6*c*{0} -8*d*{0}
*f = a +2*b +3*c +4*d
0:{{a,0},{b,0},{c,0},{d,1},{{0},1}} *f = 4
0:{{a,0},{b,0},{c,1},{d,0},{{0},0}} *f = 3
0:{{a,0},{b,0},{c,1},{d,0},{{0},1}} *f = 3
0:{{a,0},{b,1},{c,0},{d,0},{{0},0}} *f = 2
0:{{a,1},{b,0},{c,1},{d,0},{{0},1}} *f = 4
0:{{a,1},{b,1},{c,0},{d,0},{{0},0}} *f = 3
0:{{a,1},{b,1},{c,0},{d,0},{{0},1}} *f = 3
```

Here, `{0}` is an auxiliary variable required to implement the range comparison.
We can confirm that the output includes all solutions that satisfy the range comparison.

The following code computes qbpp::Expr object `f` that takes minimum value of 0 if
$
8\leq a+2b+3c +4d
$
is satisfied and finds all optimal solutions:

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = 8 <= a + 2 * b + 3 * c + 4 * d <= qbpp::inf;
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  (*f).simplify_as_binary();
  std::cout << "*f = " << *f << std::endl;
  qbpp::exhaustive_solver::ExhaustiveSolver solver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto &sol : sols) {
    std::cout << sol << " *f = " << sol(*f) << std::endl;
  }
}
```

In this code, `qbpp::inf` is used to indicate that the range comparison has no upper bound.
The output of this code is as follows:

```cpp
f = 72 -16*a -30*b -42*c -52*d +18*{0} +4*a*b +6*a*c +8*a*d -2*a*{0} +12*b*c +16*b*d -4*b*{0} +24*c*d -6*c*{0} -8*d*{0}
*f = a +2*b +3*c +4*d
0:{{a,0},{b,1},{c,1},{d,1},{{0},0}} *f = 9
0:{{a,0},{b,1},{c,1},{d,1},{{0},1}} *f = 9
0:{{a,1},{b,0},{c,1},{d,1},{{0},0}} *f = 8
0:{{a,1},{b,1},{c,1},{d,1},{{0},1}} *f = 10
```

We can confirm that the output includes all solutions that satisfy the range comparison.


## Evaluation Functions for qbpp::Expr

Given a mapping of variables to integer values, the evaluation function computes the energy of a qbpp::Expr.
This mapping must include all variables present in the qbpp::Expr and should be provided as a qbpp::MapList object, which is a list of pairs consisting of qbpp::Var objects and corresponding integer values.
The following code demonstrates how to use the evaluation functions with a qbpp::MapList object:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = 2 <= a + 2 * b + 3 * c <= 3;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  qbpp::MapList var_map = {{a, 0}, {b, 0}, {c, 0}};
  std::cout << "f(0, 0, 0) = " << f(var_map) << std::endl;
  std::cout << "f(0, 1, 0) = " << f({{a, 0}, {b, 1}, {c, 0}}) << std::endl;
}
```

In this code, the energy value of `f` is evaluated using a qbpp::MapList object, `var_map`.
The energy is also evaluated for a set of values provided using an initializer list.
The expected output of the code is as follows:

```cpp
f = 3 -2*a -3*b -3*c +2*a*b +3*a*c +6*b*c
f(0, 0, 0) = 3
f(0, 1, 0) = 0
```

## Replacement Functions for qbpp::Expr

The replacement functions substitute qbpp::Var and qbpp::VarInt objects in qbpp::Expr objects with other qbpp::Expr objects.
Since a qbpp::Expr object can also store integers, variables can be replaced with integers as well.
The mapping of qbpp::Var objects to qbpp::Expr objects is defined using qbpp::MapList objects.
Unlike evaluation functions, it is not necessary to define mappings for all variables.

The following code demonstrates how to use replacement functions to replace variables:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (2 <= a + 2 * b + 3 * c <= 3).simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  qbpp::MapList var_map = {{a, 0}, {b, c - 1}};
  std::cout << "f(0, c-1, c) = " << replace(f, var_map).simplify_as_binary() << std::endl;
}
```

In this code, the energy value of `f` is evaluated using a qbpp::MapList object, `var_map`.
The energy is also evaluated for a set of values provided using an initializer list.
The expected output of the code is as follows:

```cpp
f = 3 -2*a -3*b -3*c +2*a*b +3*a*c +6*b*c
f(0, c-1, c) = 6 -6*c
```

The eval() and replace() functions can be used with qbpp::MapList objects and qbpp::VarInt objects, as demonstrated in the following example:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var_int("a") <= 5;
  auto b = qbpp::var_int("b") <= 5;
  auto f = (a + b) * (a - b);
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  std::cout << "f(2, b) = " << f.replace({{a, 2}}).simplify_as_binary() << std::endl;
  std::cout << "f(2, 1) = " << qbpp::eval(f, {{b, 1}}) << std::endl;
}
```

Based on the following output, we can confirm that the functions are working correctly:

```cpp
f = a[0] +4*a[1] +4*a[2] -b[0] -4*b[1] -4*b[2] +4*a[0]*a[1] +4*a[0]*a[2] +8*a[1]*a[2] -4*b[0]*b[1] -4*b[0]*b[2] -8*b[1]*b[2]
f(2, b) = 4 -b[0] -4*b[1] -4*b[2] -4*b[0]*b[1] -4*b[0]*b[2] -8*b[1]*b[2]
f(2, 1) = 3
```

Since the replacement function involves intensive computation, repeated applications should be avoided.
When replacing multiple variables, a single MapList object defining all replacements should be created, and the replacement function should be called only once for this MapList object.
Alternatively, individual MapList objects can be created for each replacement, and the replacement function can be called repeatedly for each one to achieve the same result.
However, this repeated execution of the replacement function is computationally expensive and should be avoided.

## Reduction Functions for qbpp::Expr

The reduction function reduces the degree of all terms in qbpp::Expr objects to two, in order to obtain QUBO expressions.
The reduction is performed using auxiliary variables, ensuring that the optimal solutions remain unchanged.

The following code demonstrates the reduction of the degree-3 expression @f$+abc@f$:

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = a * b * c;
  std::cout << "f = " << f << std::endl;
  auto g = qbpp::reduce(f);
  std::cout << "g = " << g << std::endl;
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(g);
  auto sol_all = solver.search_all_solutions();
  std::cout << sol_all << std::endl;
}
```

This code uses the ExhaustiveSolver to evaluate all possible solutions, producing the following output:

```
f = a*b*c
g = {0} +a*b +a*c -a*{0} +b*c -b*{0} -c*{0}
0:0:{{a,0},{b,0},{c,0},{{0},0}}
1:0:{{a,0},{b,0},{c,1},{{0},0}}
2:0:{{a,0},{b,0},{c,1},{{0},1}}
3:0:{{a,0},{b,1},{c,0},{{0},0}}
4:0:{{a,0},{b,1},{c,0},{{0},1}}
5:0:{{a,0},{b,1},{c,1},{{0},1}}
6:0:{{a,1},{b,0},{c,0},{{0},0}}
7:0:{{a,1},{b,0},{c,0},{{0},1}}
8:0:{{a,1},{b,0},{c,1},{{0},1}}
9:0:{{a,1},{b,1},{c,0},{{0},1}}
10:1:{{a,0},{b,0},{c,0},{{0},1}}
11:1:{{a,0},{b,1},{c,1},{{0},0}}
12:1:{{a,1},{b,0},{c,1},{{0},0}}
13:1:{{a,1},{b,1},{c,0},{{0},0}}
14:1:{{a,1},{b,1},{c,1},{{0},1}}
15:3:{{a,1},{b,1},{c,1},{{0},0}}
```

The auxiliary variable, shown as `{0}` is introduced to ensure the equivalent QUBO expression.
From the result of the ExhaustiveSolver, we can confirm that @f$g=1@f$ when
@f$a=b=c=1@f$, and that @f$g@f$ can take minimum value of 0 otherwise.
Thus, @f$g@f$ and @f$f=abc@f$ are equivalent.

## Conversion Functions between QUBO and Ising Expressions

QUBO and Ising models are quadratic expressions involving binary (@f$0/1@f$) and spin (@f$-1/+1@f$) variables, respectively.
The binary variable @f$x@f$ and the spin variable @f$s@f$ are related by the equation @f$2x - 1 = s@f$.
Based on this relationship, the following functions allow for equivalent conversions between QUBO and Ising expressions:

* binary_to_spin(): Converts an qbpp::Expr object representing a QUBO expression into an equivalent Ising expression by replacing all variables @f$x@f$ with @f$(x + 1) / 2@f$.
After that all terms are multiplied by 4 to ensure that all coefficients are integers.
* spin_to_binary(): Converts an qbpp::Expr object representing an Ising expression into an equivalent QUBO expression by replacing all variables @f$x@f$ with @f$2x - 1@f$.

The following example demonstrates how to use these conversion functions:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto f = (a - 1) * (a + b) + 1;
  f.simplify();
  std::cout << "f = " << f << std::endl;
  auto qubo = qbpp::simplify_as_binary(f);
  auto qubo_ising = qbpp::binary_to_spin(qubo).simplify_as_spin();
  auto qubo_ising_qubo = qbpp::spin_to_binary(qubo_ising).simplify_as_binary();
  auto ising = qbpp::simplify_as_spin(f);
  auto ising_qubo = qbpp::spin_to_binary(ising).simplify_as_binary();
  auto ising_qubo_ising = qbpp::binary_to_spin(ising_qubo).simplify_as_spin();
  std::cout << "qubo = " << qubo << " / qubo_ising = " << qubo_ising << " / qubo_ising_qubo = " << qubo_ising_qubo << std::endl;
  std::cout << "ising = " << ising << " / ising_qubo = " << ising_qubo << " / ising_qubo_ising = " << ising_qubo_ising << std::endl;
}
```

The output of this code is:

```cpp
f = 1 -a -b +a*a +a*b
qubo = 1 -b +a*b / qubo_ising = 3 +a -b +a*b / qubo_ising_qubo = 4 -4*b +4*a*b
ising = 2 -a -b +a*b / ising_qubo = 5 -4*a -4*b +4*a*b / ising_qubo_ising = 8 -4*a -4*b +4*a*b
```
