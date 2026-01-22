---
layout: default
title: "Multiplier SIMULATION and Factorization"
---

# 

## Multiplier SIMULATION and Factorization
Multiplication of two integers can be done using additions.
The below figure shows how 4-bit integers $x_3x_2x_1x_0$ and
$y_3y_2y_1y_0$ and multiplied into 8-bit integer $z_7z_6z_5z_4z_3z_2z_1z_0$.
In this figure, $p_{i,j}=x_iy_j$ ($0\leq i,j\leq 3$) and
the sum of them is computed to obtain the 8-bit integer.

<p align="center">
 <img src="images/multiplication.svg" alt="4-bit multiplication" width="50%">
</p>

The sum of them computed by three 4-bit adders.
They are connected by wires $c_{i,j}$ ($0\leq i\leq 2, 0\leq j\leq 3$).
The figure below shows how they are connected:
<p align="center">
 <img src="images/multiplier.svg" alt="The 4-bit multiplier using tree 4-bit adders" width="50%">
</p>

## QUBO formulation for full adder
We will show QUBO formulation for simulating the 4-bit multiplier.
For this purpose, we implement functions to create
a full adder, an adder, and multiplier.

### Full adder
The QUBO exprssion for simuating the full adder 
with 3 input bits `a`, `b`, `i`, and 2 outbit bits carry out `o` and the sum `s` is as follows:
```cpp
qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}
```
The function `fa` returns an expression for simulating a full adder
such that the input and output bits are consistent with the full adder.

### Adder
Consider that the vectors of `qbpp::Expr` objects `a`, `b`, and `s` representing integers
are given.
We assume that `a` and `b` has `N` elements, while `s` has `N + 1`
elements.
The following function `adder` retuns the QUBO expression
that achieves a minimum value of 0 if the sum of `a` and `b`
is equal to `s`:
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
In this function, `c` is a vector of `N` variables used to
connect four `fa` objects.
They are 



A full adder can be formulated using the following expression:

$$
\begin{aligned}
fa(a,b,i,c,s) &=((a+b+i)-(2o+s))^2
\end{aligned}
$$

This expression attains its minimum value of 0 if and only if the five variables take values consistent with a valid full-adder operation.
The following QUBO++ program verifies this formulation using the exhaustive solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto i = qbpp::var("i");
  auto o = qbpp::var("o");
  auto s = qbpp::var("s");
  auto fa = (a + b + i) - (2 * o + s) == 0;
  fa.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(fa);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
In this QUBO program, the constraint $fa(a,b,i,c,s)$ is implemented using the equality operator `==`, which intuitively represents the constraint $a+b+i=2o+s$.
The program produces the following output, confirming that the expression correctly models a full adder:
{% raw %}
```
(0) 0:{{a,0},{b,0},{i,0},{o,0},{s,0}}
(1) 0:{{a,0},{b,0},{i,1},{o,0},{s,1}}
(2) 0:{{a,0},{b,1},{i,0},{o,0},{s,1}}
(3) 0:{{a,0},{b,1},{i,1},{o,1},{s,0}}
(4) 0:{{a,1},{b,0},{i,0},{o,0},{s,1}}
(5) 0:{{a,1},{b,0},{i,1},{o,1},{s,0}}
(6) 0:{{a,1},{b,1},{i,0},{o,1},{s,0}}
(7) 0:{{a,1},{b,1},{i,1},{o,1},{s,1}}
```
{% endraw %}

If some bits are fixed, the valid values of the remaining bits can be derived.
For example, the three input bits can be fixed using the `replace()` function:
{% raw %}
```cpp
  fa.replace({{a, 1}, {b, 1}, {i, 0}});
```
{% endraw %}


The program then produces the following output:

{% raw %}
```
(0) 0:{{o,1},{s,0}}
```
{% endraw %}

Conversely, if the two output bits are fixed:
{% raw %}
```cpp
  fa.replace({{o, 1}, {s, 0}});
```
{% endraw %}
the program produces all valid combinations of the input bits:
{% raw %}
```
(0) 0:{{a,0},{b,1},{i,1}}
(1) 0:{{a,1},{b,0},{i,1}}
(2) 0:{{a,1},{b,1},{i,0}}
```
{% endraw %}


## Simulating a ripple carry adder using multiple full adders
Using the QUBO expression for a full adder, we can construct a QUBO expression that simulates a ripple-carry adder.
The following QUBO++ program creates a QUBO expression for simulating a 4-bit adder by combining four full adders:
{% raw %}
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto i = qbpp::var("i");
  auto o = qbpp::var("o");
  auto s = qbpp::var("s");
  auto fa = (a + b + i) - (2 * o + s) == 0;

  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  auto c = qbpp::var("c", 5);
  auto z = qbpp::var("s", 4);

  auto fa0 = qbpp::replace(fa, {{a, x[0]}, {b, y[0]}, {i, c[0]}, {o, c[1]}, {s, z[0]}});
  auto fa1 = qbpp::replace(fa, {{a, x[1]}, {b, y[1]}, {i, c[1]}, {o, c[2]}, {s, z[1]}});
  auto fa2 = qbpp::replace(fa, {{a, x[2]}, {b, y[2]}, {i, c[2]}, {o, c[3]}, {s, z[2]}});
  auto fa3 = qbpp::replace(fa, {{a, x[3]}, {b, y[3]}, {i, c[3]}, {o, c[4]}, {s, z[3]}});
  auto adder = fa0 + fa1 + fa2 + fa3;
  adder.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(adder);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
{% endraw %}
In this QUBO++ program, four `qbpp::Expr` objects representing full adders are created using the `replace()` function and combined into a single expression, `adder`.
The Exhaustive Solver is then used to enumerate all optimal solutions.

This program produces 512 valid solutions, corresponding to all possible input combinations of a 4-bit adder:
{% raw %}
```
(0) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],0},{c[0],0},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],0},{s[1],0},{s[2],0},{s[3],0}}
(1) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],0},{c[0],1},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],1},{s[1],0},{s[2],0},{s[3],0}}
(2) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],1},{c[0],0},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],0},{s[1],0},{s[2],0},{s[3],1}}
(3) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],1},{c[0],1},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],1},{s[1],0},{s[2],0},{s[3],1}}

... omitted ...

(510) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[0],0},{c[1],1},{c[2],1},{c[3],1},{c[4],1},{s[0],0},{s[1],1},{s[2],1},{s[3],1}}
(511) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[0],1},{c[1],1},{c[2],1},{c[3],1},{c[4],1},{s[0],1},{s[1],1},{s[2],1},{s[3],1}}
```
{% endraw %}
Alternatively, we can define a C++ function `fa` to construct full-adder constraints in a more concise and readable manner:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

qbpp::Expr fa(qbpp::Var a, qbpp::Var b, qbpp::Var i, qbpp::Var o, qbpp::Var s) {
  return (a + b + i) - (2 * o + s) == 0;
}

int main() {
  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  auto c = qbpp::var("c", 5);
  auto z = qbpp::var("s", 4);
  auto fa0 = fa(x[0], y[0], c[0], c[1], z[0]);
  auto fa1 = fa(x[1], y[1], c[1], c[2], z[1]);
  auto fa2 = fa(x[2], y[2], c[2], c[3], z[2]);
  auto fa3 = fa(x[3], y[3], c[3], c[4], z[3]);
  auto adder = fa0 + fa1 + fa2 + fa3;
  adder.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(adder);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
This program produces the same 512 optimal solutions as the previous implementation.

If some of the binary variables are fixed, the valid values of the remaining variables can be derived using the Exhaustive Solver.
For example, the following `qbpp::MapList` object `ml` fixes the carry-in, carry-out, and sum bits:
{% raw %}
```cpp
  qbpp::MapList ml = {{c[4], 1}, {c[0], 0}, {z[3], 1},
                      {z[2], 1}, {z[1], 0}, {z[0], 1}};
  adder.replace(ml);
```
{% endraw %}
The resulting program produces the following output:
{% raw %}
```
(0) 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[1],0},{c[2],1},{c[3],1}}
(1) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],0},{y[1],1},{y[2],1},{y[3],1},{c[1],0},{c[2],1},{c[3],1}}
```
{% endraw %}
