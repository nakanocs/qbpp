{% raw %}
# QUBO++ Library Documantaion Version 2025.10.13

Quadratic Unconstrained Binary Optimization (QUBO) models use quadratic functions over binary variables {0,1}.
High-order Unconstrained Binary Optimization (HUBO) generalizes QUBO to polynomial functions of arbitrary order.
The goal in QUBO/HUBO is to find a binary assignment that minimizes the objective.
QUBO++ is a C++ library for constructing and transforming these objectives and efficiently searching for optimal or high-quality solutions.



## Key features

* **Simplified C++17-based implementation**: The library requires understanding only three essential C++ classes—variables (qbpp::Var), expressions (qbpp::Expr), and solutions (qbpp::Sol).
* **Unlimited integer coefficient support**: Expressions can involve terms with integer coefficients of any magnitude.
* **Support for integer variables**: Integer variables are represented using multiple binary variables.
* **Rich operator and function support**: Provides a wide range of operators and functions for designing and manipulating expressions, including robust vector manipulation capabilities.
* **Built-in solvers**: Includes QUBO/HUBO solvers, **Easy Solver**, **Exhaustive Solver**, and **ABS3 GPU Solver**.
* **Performance acceleration**: Multithreading is leveraged wherever possible. The library incorporates components from Intel® Threading Building Blocks (TBB), licensed under the Apache License 2.0.
* **Advanced solver integration**: Offers API integrations for powerful solvers like the ABS2 GPU Solver and the Gurobi Optimizer for solving QUBO problems.


This document primarily explains how to use the QUBO++ library to design QUBO expressions.
Since all source code files of the QUBO++ library and related programs are documented using Doxygen, users can refer to the documentation for detailed usage information.

## Terms and Conditions
For detailed license information, please refer to the [Terms and Conditions](LICENSE.md).

## QUBO Expressions and QUBO++ Library

**A QUBO (Quadratic Unconstrained Optimization Problem) expression** is a quadratic formula consisting of binary variables.
It includes a constant term, linear terms, and quadratic terms of variables.
For example, we can derive a QUBO model $f$ of three binary variables $a$, $b$, and $c$ as follows:
$$
\begin{aligned}
f(a,b,c)
&=(a+b+c-2)(a+b+c-3)\\
&= a^2+b^2+c^2 + 2ab + 2ac + 2bc -5(a+b+c) + 6\\
&= 6 -4a -4b -4c + 2ab + 2ac + 2bc .
\end{aligned}
$$
By expanding $(a+b+c-2)(a+b+c-3)$, we obtain an equivalent quadratic formula.
We can simplify it using the fact that $x^2=x$ holds for all binary variables $x$.
The resulting quadratic formula has:

* constant term: $6$
* linear terms: $-4a  -4b  -4c$
* quadratic terms: $2ab + 2ac + 2bc$

In this document, we term $(a+b+c-2)(a+b+c-3)$ as **a formula** and
$6 -4a -4b -4c +2ab +2ac +2bc$ as **a QUBO expression**.
**QUBO++** is a C++ library designed to create QUBO formulas, derive the corresponding QUBO expressions, and includes APIs to call QUBO solvers for finding optimal solutions of QUBO models.

A QUBO++ program for generating a QUBO expression and evaluate all solutions are as follows:

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Var a = qbpp::var("a");
  qbpp::Var b = qbpp::var("b");
  qbpp::Var c = qbpp::var("c");
  qbpp::Expr f = (a + b + c - 2) * (a + b + c - 3);
  qbpp::Expr g = qbpp::simplify_as_binary(f);
  std::cout << "f = " << f << std::endl;
  std::cout << "g = " << g << std::endl;
  qbpp::exhaustive_solver::ExhaustiveSolver solver(g);
  auto all_sol = solver.search_all_solutions();
  std::cout << all_sol;
}
```

The header file `qbpp.hpp` is included to enable the use of the QUBO++ library, while `qbpp_exhaustive_solver.hpp` is included to access the ExhaustiveSolver in the library.
In this program, three variables, $a$, $b$, and $c$, are defined, along with the formula $f$.
The formula $f$ is automatically expanded, and the function `qbpp::simplify_as_binary` returns the corresponding QUBO expression, which is stored in $g$. 
ExhaustiveSolver evaluates the values of $g$ for all $2^3$ possible solutions, and the results are then displayed.

The output of this program follows.

````text
f = 6 -2*a -2*b -2*c -3*a -3*b -3*c +a*a +a*b +a*c +b*a +b*b +b*c +c*a +c*b +c*c
g = 6 -4*a -4*b -4*c +2*a*b +2*a*c +2*b*c
0:0:{{a,0},{b,1},{c,1}}
1:0:{{a,1},{b,0},{c,1}}
2:0:{{a,1},{b,1},{c,0}}
3:0:{{a,1},{b,1},{c,1}}
4:2:{{a,0},{b,0},{c,1}}
5:2:{{a,0},{b,1},{c,0}}
6:2:{{a,1},{b,0},{c,0}}
7:6:{{a,0},{b,0},{c,0}}
````

The resulting value of a QUBO expression for a given assignment of binary values to variables is referred to as the **energy**. The goal of the QUBO problem is to find the optimal assignment that minimizes this energy. For instance, in the case of $g$, the minimum value of 0 is achieved when the sum of the three variables equals 2 or 3. When the sum of the three variables is 2 or less, the energy value is 2 or greater. QUBO solvers are designed to find one of the optimal solutions that minimizes the energy.

{% endraw %}