# QUBO++ Library Documantaion Version 2025.10.13

QUBO (Quadratic Unconstrained Binary Optimization) models use quadratic functions over binary variables {0,1}.
HUBO (High-order Unconstrained Binary Optimization) generalizes QUBO to polynomial functions of arbitrary order.
The goal in QUBO/HUBO is to find a binary assignment that minimizes the objective.
QUBO++ is a C++ library for constructing and transforming these objectives and efficiently searching for optimal or high-quality solutions.
This document explains how to use the QUBO++ library to design QUBO and HUBO expressions.


## HUBO Expressions and QUBO++ Library

**A HUBO (High-order Unconstrained Binary Optimization) expression** is a polynomial in binary variables.
For example, consider three binary variables $a$, $b$ and $c$, and define


$$
\begin{aligned}
f(a,b,c)
&=(a+b+c-2)(a+b+c-3)^2\\
&= 4a +4b +4c -4a^2 -4b^2 -4c^2 -8ab -8ac -8bc +a^3 +b^3 +c^3 +3a^2b +3a^2c +3ab^2 +3b^2c+3ac^2+3bc^2 +6abc \\
&= a +b +c -2ab -2ac -2bc +6abc
\end{aligned}
$$


Expanding $(a+b+c-2)(a+b+c-3)^2$, we obtain an equivalent polynomial expression in the form of a sum of product terms.
Furthermore, using the fact that $x^2=x$ holds for all binary variables $x\in{0,1}$, we can merge equivalent terms to derive a simplified expression.
The resulting HUBO expression has:
* constant term: none 
* linear terms: $a$, $b$, $c$
* quadratic terms: $-2ab$, $-2ac$, $-2bc$
* cubic term : $6abc$

In this document, we refer to $(a+b+c-2)(a+b+c-3)$ as **a (HUBO) formula** and
to the expanded, simplified polynomial $a +b +c -2ab -2ac -2bc +6abc$ as **a (HUBO) expression**.
The resulting value of a HUBO polynomial, borrowing the terminology of quantum mechanics, is called **the energy**.
A HUBO problem aims to find a binary assignment of variables that minimizes this energy.
It should be clear that $f(a,b,c)$ achieves optimal energy 0 when $a+b+c=0$ and $2$.
Hence, it has four optimal solutions: $(a,b,c)=(0,0,0),(1,1,0),(1,0,1),(0,1,1)$.

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
  auto f = (a + b + c) * qbpp::sqr(a + b + c - 2);
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

{% raw %}
````text
f = a +b +c -2*a*b -2*a*c -2*b*c +6*a*b*c
(0) 0:{{a,0},{b,0},{c,0}}
(1) 0:{{a,0},{b,1},{c,1}}
(2) 0:{{a,1},{b,0},{c,1}}
(3) 0:{{a,1},{b,1},{c,0}}
````
{% endraw %}

