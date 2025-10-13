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
For example, we can derive a QUBO model @f$f@f$ of three binary variables @f$a@f$, @f$b@f$, and @f$c@f$ as follows:
@f{eqnarray*}{
f(a,b,c) &=&(a+b+c-2)(a+b+c-3)\\
         &=&  6 -2a -2b -2c -3a -3b -3c +a^2 +ab +ac +ba +b^2 +bc +ca +cb +c^2 \\
         &=& 6 -5a -5b -5c +a +2ab +2ac +b +2bc +c \\
         & =& 6 -4a -4b -4c +2ab +2ac +2bc
@f}
By expanding @f$(a+b+c-2)(a+b+c-3)@f$, we obtain an equivalent quadratic formula.
We can simplify it using the fact that @f$x^2=x@f$ holds for all binary variables @f$x@f$.
The resulting quadratic formula has:

* constant term: @f$6@f$
* linear terms: @f$-4a  -4b  -4c@f$
* quadratic terms: @f$2ab + 2ac + 2bc@f$

In this document, we term @f$(a+b+c-2)(a+b+c-3)@f$ as **a formula** and
@f$6 -4a -4b -4c +2ab +2ac +2bc@f$ as **a QUBO expression**.
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
In this program, three variables, @f$a@f$, @f$b@f$, and @f$c@f$, are defined, along with the formula @f$f@f$.
The formula @f$f@f$ is automatically expanded, and the function `qbpp::simplify_as_binary` returns the corresponding QUBO expression, which is stored in @f$g@f$. 
ExhaustiveSolver evaluates the values of @f$g@f$ for all @f$2^3@f$ possible solutions, and the results are then displayed.

The output of this program follows.

```text
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
```

The resulting value of a QUBO expression for a given assignment of binary values to variables is referred to as the **energy**. The goal of the QUBO problem is to find the optimal assignment that minimizes this energy. For instance, in the case of @f$g@f$, the minimum value of 0 is achieved when the sum of the three variables equals 2 or 3. When the sum of the three variables is 2 or less, the energy value is 2 or greater. QUBO solvers are designed to find one of the optimal solutions that minimizes the energy.

# Getting Started: QUBO++ Program Example for Solving the Partitioning Problem

Let's begin with a simple example that solves the partitioning problem through a QUBO model using the QUBO++ library.
This will allow us to quickly see how a combinatorial optimization problem is handled by the QUBO++ library.
Specifically, we will use the QUBO++ EasySolver, included in the QUBO++ library, to find solutions for a QUBO model derived from an instance of the partitioning problem.

## Partitioning Problem

Suppose that a list of @f$n@f$ numbers @f$w_0, w_1, \ldots, w_{n-1}@f$ is given.
The partitioning problem is to partition these numbers into two lists, @f$L@f$ and @f$\overline{L}@f$, so that their sums are as equal as possible.
More specifically, @f$L@f$ and @f$\overline{L}@f$ are determined so that
@f{eqnarray*}{
{\rm Partition}(L) &=&\left(\sum_{i\in L}w_i - \sum_{i\in \overline{L}}w_i\right)^2
@f}
is minimized.

## QUBO Expression for the Partitioning Problem

Let @f$X=\{x_0, x_1, \ldots, x_{n-1}\}@f$ be a set of @f$n@f$ binary variables to denote elements in @f$L@f$ such that @f$x_i=1@f$ if and only if @f$w_i@f$ is in @f$L@f$.
We have the following objective function:
@f{eqnarray*}{
{\rm Partition}(X) &=& \left(\sum_{i=0}^{n-1} w_i x_i - \sum_{i=0}^{n-1} w_i (1 - x_i)\right)^2 = \left(2 \sum_{i=0}^{n-1} w_i x_i - \sum_{i=0}^{n-1} w_i \right)^2
@f}
It should be clear that @f${\rm Partition}(X)={\rm Partition}(L)@f$ and we can obtain a QUBO expression for the partitioning problem by expanding @f${\rm Partition}(X)@f$.

## QUBO++ Program Examples

Below is an example of a QUBO++ program that generates a QUBO expression for a partitioning problem with 8 input elements, stored in `qbpp::Vector<int>`, an integer vector class in QUBO++ compatible with `std::vector<int>`.

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Vector<int> w = {15, 10, 24, 1, 14, 10, 8, 6};

  int sum = 0;
  for (size_t i = 0; i < w.size(); i++) {
    sum += w[i];
  }

  auto x = qbpp::var("x", w.size());

  auto g = qbpp::expr();
  for (size_t i = 0; i < w.size(); i++) {
    g += w[i] * x[i];
  }

  auto f = (sum - 2 * g) * (sum - 2 * g);
  f.simplify_as_binary();

  std::cout << "f = " << f << std::endl;

  qbpp::exhaustive_solver::ExhaustiveSolver solver(f);
  auto sol = solver.search();

  std::cout << "Solution: " << sol << std::endl;
}
```
To use the QUBO++ ExhaustiveSolver, the header file qbpp_exhaustive_solver.hpp is required.
This program performs the following steps:
1. Computes the sum of all elements in w.
2. Creates a vector x of 8 variables.
3. Initializes an empty expression g and iteratively adds w[i] * x[i] using a for loop.
4. Computes the objective function f using the sum and g. The function is simplified into a QUBO expression with the simplify_as_binary() function.
5. Creates a QUBO++ ExhaustiveSolver object solver using the objective function f. Parameters such as time limits and target energy can also be set if needed.
6. Executes the solver and stores the solution in the variable sol.
Upon executing this program, the following results are obtained:
An optimal solution with an objective value of 0 is achieved.

```cpp
f : 7744 -4380*x[0] -3120*x[1] -6144*x[2] -348*x[3] -4144*x[4] -3120*x[5] -2560*x[6] -1968*x[7] +1200*x[0]*x[1] +2880*x[0]*x[2] +120*x[0]*x[3] +1680*x[0]*x[4] +1200*x[0]*x[5] +960*x[0]*x[6] +720*x[0]*x[7] +1920*x[1]*x[2] +80*x[1]*x[3] +1120*x[1]*x[4] +800*x[1]*x[5] +640*x[1]*x[6] +480*x[1]*x[7] +192*x[2]*x[3] +2688*x[2]*x[4] +1920*x[2]*x[5] +1536*x[2]*x[6] +1152*x[2]*x[7] +112*x[3]*x[4] +80*x[3]*x[5] +64*x[3]*x[6] +48*x[3]*x[7] +1120*x[4]*x[5] +896*x[4]*x[6] +672*x[4]*x[7] +640*x[5]*x[6] +480*x[5]*x[7] +384*x[6]*x[7]
Solution: 0:{{x[0],1},{x[1],1},{x[2],0},{x[3],1},{x[4],0},{x[5],1},{x[6],1},{x[7],0}}
```

Alternatively, the same QUBO expression can be defined more concisely using the extensive vector operations available in the QUBO++ library:

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Vector<int> w = {15, 10, 24, 1, 14, 10, 8, 6};

  auto x = qbpp::var("x", w.size());
  auto f = qbpp::sqr(qbpp::sum(w) - 2 * qbpp::sum(w * x));
  f.simplify_as_binary();

  std::cout << "f = " << f << std::endl;

  qbpp::exhaustive_solver::ExhaustiveSolver solver(f);
  auto sol = solver.search();

  std::cout << "Solution: " << sol << std::endl;
}
```

This program constructs a QUBO expression for the partitioning problem in a single formula using vector operations.
By leveraging the vector-based features of the QUBO++ library, the formula is defined more concisely and efficiently, eliminating the need for explicit loops and manual construction of the objective function.
The qbpp::sum and qbpp::sqr functions streamline the process by handling summation and square QUBO expressions, thereby simplifying the definition of the partitioning problem.

Users of the QUBO++ library can choose between these two approaches based on their preferences and familiarity with the library.
The first method offers more explicit control over the construction of the QUBO expression, making it easier for those new to the library to follow the steps.
The second method, on the other hand, takes advantage of the powerful vector operations available in QUBO++, providing a more concise and efficient way to define the problem for advanced users who are comfortable with these features.

# QUBO++ Classes

The QUBO++ library provides the following core classes for designing polynomial expressions involving variables:

* qbpp::Var: Stores a symbolic variable in polynomial expressions.
* qbpp::Expr: Stores a polynomial expression of qbpp::Var objects.
* qbpp::Sol: Stores a solution for a qbpp::Expr object, mapping qbpp::Var objects to binary values.

These three classes form the core components of the QUBO++ library.

In addition, the following supplementary classes are provided:

* qbpp::VarInt: Stores an integer symbolic variable with a qbpp::Expr object to represent an integer in a specified range.

To handle vectors or multi-dimensional arrays of integers, qbpp::Var, qbpp::VarInt, and qbpp::Expr objects, use the qbpp::Vector class:

* qbpp::Vector: A composite class built upon std::vector, inheriting most of its functionality.
Multi-dimensional arrays are implemented as nested qbpp::Vector objects.
For example, a two-dimensional array of qbpp::Var can be created as a qbpp::Vector<qbpp::Vector<qbpp::Var>>.

A qbpp::Expr object is internally converted into a qbpp::Model, and subsequently into a qbpp::QuadModel.
QUBO solvers find solutions for the QUBO model stored in qbpp::QuadModel and return the results as qbpp::Sol objects:

* qbpp::Model: Stores a qbpp::Expr object with variable indices.
* qbpp::QuadModel: The derived class of qbp::Model with a constant term, linear terms, and quadratic terms of qbpp::Var objects.

Each product term in qbpp::Expr objects is represented by:

* qbpp::Term: Stores a term consisting of an integer coefficient and a vector of qbpp::Var objects representing a product.

Lastly, to store or exchange the best solutions obtained by solvers, the following class is provided:

* qbpp::SolHolder: Stores a solution along with attributes such as the solver’s name, time-to-solution (TTS), and other details.

Below is a summary of these classes and their roles:

| Classes         | Role                                                                                 |
|-----------------|--------------------------------------------------------------------------------------|
| qbpp::Var       | Stores a single symbolic variable.                                                   |
| qbpp::Expr      | Stores a polynomial expression of qbpp::Var objects.                                 |
| qbpp::Sol       | Stores a solution for qbpp::Expr objects.                                            |
| qbpp::VarInt    | Stores a set of qbpp::Var objects representing an integer.                           |
| qbpp::Vector    | Implements a vector and nested vector of objects.                                    |
| qbpp::Term      | Stores a single term included in qbpp::Expr objects.                                 |
| qbpp::Model     | Stores a qbpp::Expr object with indexing of qbpp::Var objects.                       |
| qbpp::QuadModel | Stores a qbpp::Model object with a constant term, linear terms, and quadratic terms. |
| qbpp::SolHolder | Stores the best solution with attributes obtained so far.                            |

## String Representation of QUBO++ Classes  

Objects of these classes can be converted to strings using the str() global functions, which return std::string representing objects.
In other words, `str(x)` returns the std::string representation of the object `x`.
Additionally, these objects can be displayed using the standard C++ output style, as in `std::cout << x`.
This functionality is particularly useful when designing QUBO++ programs and for debugging purposes.

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  std::cout << "a = " << a << std::endl;  // print qbpp::Var a
  auto x = 1 <= qbpp::var_int("x") <= 10;
  std::cout << "x = " << x << std::endl;  // print qbpp::VarInt x
  auto t = qbpp::Term(a, 2);
  std::cout << "t = " << t << std::endl;  // print qbpp::Term t
  auto f = (a + 1) * x;
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;  // print qbpp::Expr f
  auto model = qbpp::Model(f);
  std::cout << "model = " << model << std::endl;  // print qbpp::Model model
  auto quad_model = qbpp::QuadModel(model);
  std::cout << "quad_model = " << quad_model
            << std::endl;  // print qbpp::QuadModel quad_model
}
```

The output of this code is:

```cpp
a = a
x = 1 +x[0] +2*x[1] +4*x[2] +2*x[3]
t = 2*a
f = 1 +a +x[0] +2*x[1] +4*x[2] +2*x[3] +a*x[0] +2*a*x[1] +4*a*x[2] +2*a*x[3]
model = 1 +<0> +<1> +2*<2> +4*<3> +2*<4> +<0>*<1> +2*<0>*<2> +4*<0>*<3> +2*<0>*<4>
quad_model = 1 +<0>*(1 +<1>/2 +2*<2>/2 +4*<3>/2 +2*<4>/2) +<1>*(1 +<0>/2) +<2>*(2 +2*<0>/2) +<3>*(4 +4*<0>/2) +<4>*(2 +2*<0>/2)
```

Note that the outputs are provided for observation and debugging purposes only and should not be used as inputs for other programs, as they may change in future updates.
Additionally, the output from the latest version of the QUBO++ library may differ from the example shown above.

# Variable Objects: qbpp::Var

qbpp::Var objects can be created using `auto` type deduction and the qbpp::var() function.
A qbpp::Var object represents a single variable.
Unlike conventional programming languages, it does not store a variable's value;
instead, it symbolically represents a variable within qbpp::Expr objects.

We can use the qbpp::var function to create a qbpp::Var object `x` that stores a variable with the label string  `X` as follows:

```cpp
auto x = qbpp::var("X");
```

Note that the function qbpp::var() uses all lowercase letters, while the class qbpp::Var begins with a capital letter.
The argument `"X"` specifies the label string for the qbpp::Var object, which is used to represent the object as a string.
This example uses qbpp::Var object `x` with the label string `X` to distinguish them easily, but it is recommended to use the same letter or string for them.
The label strings are used only for displaying expressions during debugging and are not referenced in any internal computations.
Additionally, the library does not check for duplicate label strings, so no error will occur even if the same label string is assigned to multiple objects.

A vector of qbpp::Var objects can be defined with a specified size as follows:

```cpp
auto x = qbpp::var("X", 3);
```

This creates a vector `x` with three qbpp::Var objects labeled `X`.
Each object can be accessed using `x[0]`, `x[1]`, and `x[2]`, and their label strings, `X[0]`, `X[1]`, and `X[2]`, combine the base label with the corresponding indices for display purposes.

Multi-dimensional qbpp::Var objects can be defined similarly.
For example, the following code creates a @f$3\times 2@f$ matrix `x` of qbpp::Var objects.

```cpp
auto x = qbpp::var("X", 3, 2);
```

Each object can be accessed as `x[0][0]`, `x[0][1]`, `x[1][0]`, `x[1][1]`, `x[2][0]`, and `x[2][1]` and
they are displayed as `X[0][0]`, `X[0][1]`, `X[1][0]`, `X[1][1]`, `X[2][0]`, and `X[2][1]`.

Higher-dimensional qbpp::Var objects can be defined similarly, with no limitation on the number of dimensions.
Additionally, a variable can be created without specifying a label string:

```cpp
auto x = qbpp:var();
```

For such variables, default numbered label string such as `"{0}"`, `"{1}"`, and so on are assigned.

The qbpp::var() functions shown above call the constructor of the qbpp::Var class.
However, the constructor should not be called directly, as a set of all qbpp::Var objects is maintained internally.

Unique integer IDs, starting from 0, are internally assigned to all created qbpp::Var objects in the order of their creation.
By default, these unique IDs are stored as uint32_t, but they can be configured to use uint64_t through a compiler option.

# Integer Variable Objects: qbpp::VarInt

A qbpp::VarInt object is used to represent an integer value within a specified range.
It corresponds to a qbpp::Expr object composed of multiple qbpp::Var objects.
qbpp::VarInt objects can be created using the qbpp::var_int() global function.

A single qbpp::VarInt object `x`, representing an integer value in the range @f$[1,10]@f$, can be created and displayed with the following code:

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

If a string is not provided, default names such as {0}, {1}, and so on are assigned, similarly to qbpp::Var objects.

In most cases, qbpp::VarInt objects are implicitly converted to their corresponding qbpp::Expr objects.

# Expression Objects: qbpp:Expr

## Data Structure of qbpp::Expr Objects

A qbpp::Expr object consists of a constant term and a list of qbpp::Term objects, representing an expanded polynomial expression.
A qbpp::Term object consists of a constant coefficient and a vector of qbpp::Var objects, representing a product term.
For example, a qbpp::Expr object represented as @f$2 + 4a + ab + 3abc@f$ has:

* a constant term @f$2@f$, and
* three qbpp::Term objects represented as @f$4a@f$, @f$ab@f$, and @f$3abc@f$, respectively.

A qbpp::Term object represented as @f$3abc@f$ has a coefficient @f$3@f$ and the qbpp::Var objects @f$a@f$, @f$b@f$, and @f$c@f$.

The number of qbpp::Var objects in a qbpp::Term object is called **the degree** of the qbpp::Term object.
The maximum degree of all qbpp::Term objects in a qbpp::Expr object is called **the degree** of the qbpp::Expr object.

QUBO and Ising expressions are defined using qbpp::Expr objects with degree 2.

Note that a qbpp::Expr object simply stores an expanded polynomial expression.
It does not have any attributes related to the variables used, and thus, it can represent a QUBO expression, an Ising expression, or neither.
Additionally, the QUBO++ library stores expressions as expanded polynomial forms in qbpp::Expr objects, automatically expanding any input formula
The original, non-expanded form of the formula is not preserved.

## Type Conversion to qbpp::Expr Object

The qbpp::Expr class includes constructors for int, qbpp::Var, and qbpp::VarInt.
Collectively, we refer to these types, along with qbpp::Expr, as **ExprType**.
Since these constructors allow implicit conversion of ExprType objects to qbpp::Expr, operators and functions defined for qbpp::Expr will work seamlessly with any of these types.

Additionally, the global function qbpp::toExpr() can be used to explicitly convert ExprType object into qbpp::Expr object.
The following code demonstrates how to use qbpp::toExpr() to create qbpp::Expr objects:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = 1 <= qbpp::var_int("b") <= 10;
  auto f = qbpp::toExpr(a);
  auto g = qbpp::toExpr(b);
  auto h = qbpp::toExpr(1);
}
```

In this example, `f`, `g`, and `h` are qbpp::Expr objects.
However, if qbpp::toExpr() is omitted, `f`, `g`, and `h` would be deduced as qbpp::Var, qbpp::VarInt, and int, respectively, by the auto type deduction.
This would lead to a compilation error if these variables are subsequently updated by qbpp::Expr objects.

## Operators and Functions for qbpp::Expr

Operators and functions for qbpp::Expr are divided into two categories: global functions and member functions as follows:

* **Global functions**: Take at least one ExprType object as an argument. In many cases, they return a qbpp::Expr object.
* **Member functions**: Defined as member functions of the qbpp::Expr class, and they update the calling object based on the result of the function.

For example, the sqr() function, which computes the square of a qbpp::Expr object, is available as both a global and a member function.
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
| Eval                          | operator()                                           | Global        | Int            | ExprType-MapList       |
| Replace                       | replace()                                            | Global        | qbpp::Expr     | ExprType-MapList       |
| Replace                       | replace()                                            | Member        | qbpp::Expr     | MapList                |
| Reduce                        | reduce()                                             | Global        | qbpp::Expr     | ExprType               |
| Reduce                        | reduce()                                             | Member        | qbpp::Expr     | MapList                |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Global        | qbpp::Expr     | ExprType               |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Member        | qbpp::Expr     | -                      |

In this table, **Int** denotes an integer, while **IntInf** represents either an integer, -qbpp::inf, or +qbpp::inf, which signify infinite values.
Additionally, `qbpp::ExprExpr` is a derived class of `qbpp::Expr` that encapsulates another `qbpp::Expr` object.

## Creation and Assignment of qbpp::Expr Objects

qbpp::Expr object can be created using the auto type deduction and qbpp::expr() global function.

### Empty qbpp::Expr Object Creation

A qbpp::Expr object with an empty expression, which is an expression with a constant term 0 and no variables, can be created as shown below:

```cpp
  auto f = qbpp::expr();
```

Unlike qbpp::Var objects, qbpp::Expr objects cannot be assigned label strings.

Similarly to variable definition, a multi-dimensional array of `qbpp::Expr` with empty expressions can be created.
For example, the following code creates a @f$3\times 2@f$ matrix `f` of `qbpp::Var` objects, with each object being represented as `f[0][0]`, `f[0][1]`, `f[1][0]`, `f[1][1]`, `f[2][0]`, and `f[2][1]`:

```cpp
  auto f = qbpp::expr(3,2);
```

Be mindful of the capitalization when using qbpp::expr. For example:

```cpp
  auto f = qbpp::Expr(3);
```

creates a single qbpp::Expr object initialized with the integer 3 by calling the constructor of qbpp::Expr class, while

```cpp
  auto f = qbpp::expr(3);
```

creates three qbpp::Expr objects `f[0]`, `f[1]` and `f[2]` by the global function qbpp::expr.

### qbpp::Expr Object Creation with Explicit Initialization

A qbpp::Expr object with an initial expression can be easily created using auto type deduction, as shown below:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto f = a + 2;
  std::cout << "f = " << f << std::endl;
}
```

In this example, f is a qbpp::Expr object that stores the expression `a + 2`.
However, if the right-hand side of the expression does not involve operations that return a qbpp::Expr object, a compilation error will occur. For example:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto f = a;
  f += 2;
  std::cout << "f = " << f << std::endl;
}
```

In this case, you must use the toExpr() function to explicitly return a qbpp::Expr object, as shown below:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto f = qbpp::toExpr(a);
  f += 2;
  std::cout << "f = " << f << std::endl;
}
```

While not recommended, the following workaround uses a no-op multiplication to force the return of a qbpp::Expr object:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto f = a * 1;
  f += 2;
  std::cout << "f = " << f << std::endl;
}
```

## Operators and Basic Functions for qbpp::Expr

The following C++ operators are overloaded to work with qbpp::Expr objects:

* Assignment operator: `=`
* Binary operators: `+`, `*`, `-`, `/`
* Unary operators: `+`, `-`
* Compound assignment operators: `+=`, `-=`, `*=`, `/=`

Additionally, the following basic functions can be used:

* qbpp::sqr() : Returns the square of the qbpp::Expr object.
* qbpp::gcd(): Returns the GCD(Greatest Common Divisor) of a constant and all the coefficients of the qbpp::Expr object.

Since implicit conversions from qbpp::Var and qbpp::VarInt objects to qbpp::Expr objects are defined, these operators can also be used with them.
However, the left-hand side of the assignment and compound assignment operators must be a qbpp::Expr object, while qbpp::Var and qbpp::VarInt objects can only be placed on the right-hand side.
The division operator and the compound assignment operator for division only accept numbers as divisors, and all dividends, including constants and coefficients, must be divisible.
Additionally, all constants and coefficients of the resulting qbpp::Expr object after division must be integers.

The following example demonstrates the use of qbpp::Expr objects with these operators and functions:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto f = qbpp::sqr(2 * a - 4 * b);
  std::cout << "f = " << f << std::endl;
  f /= gcd(f);
  std::cout << "f = " << f << std::endl;
}
```

The output of this code is as follows:

```cpp
f = 4*a*a -8*a*b -8*b*a +16*b*b
f = a*a -2*a*b -2*b*a +4*b*b
```


## Simplification Functions for qbpp::Expr

Simplification functions reduce expressions by sorting variables within terms, merging redundant terms, and ordering all terms. The following simplification functions are available:

* simplify(): Sorts variables within terms, merges redundant terms, and orders all terms.
* simplify_as_binary(): Simplifies expressions under the assumption that all variables take binary values (@f$0/1@f$), such that @f$x^2 = x@f$ for all variables @f$x@f$.
* simplify_as_spin(): Simplifies expressions assuming all variables are spin variables (@f$-1/+1@f$), such that @f$x^2 = 1@f$ for all variables @f$x@f$.

Therefore, after applying either simplify_as_binary() or simplify_as_spin(), no terms will contain duplicated variables.

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

Comparison operators, equality (==), and range comparison (<= <=) are supported between qbpp::Expr objects and integers.
In addition to integers, infinite values represented as +qbpp::inf and -qbpp::inf can also be specified.
* Expr == Int: Returns a qbpp::Expr object that evaluates to a minimum value of 0 if the equality is satisfied.
* IntInf <= Expr <= IntInf: Returns a qbpp::Expr object that evaluates to a minimum value of 0 if the range comparison is satisfied.

Single inequalities are intentionally not supported to prevent potential misuse caused by misunderstandings.
Instead, qbpp::inf can be used to represent single inequalities. Specifically:
* Int <= Expr can be expressed as Int <= Expr <= +qbpp::inf, and
* Expr <= Int can be expressed as -qbpp::inf <= Expr <= Int.

Other comparison operators such as !=, <, >=, and > are not supported.

### Example of Equality
As an example of using equality operators, we present a QUBO++ program that solves the following linear equations:
@f{eqnarray*}{
x + y &=& 10 \\
2x+4y &=& 24 
@f}

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
  std::cout << "x = " << sol.get(x) << std::endl;
  std::cout << "y = " << sol.get(y) << std::endl;
  std::cout << "f = " << sol.get(*f) << std::endl;
  std::cout << "g = " << sol.get(*g) << std::endl;
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
f = 10
g = 24
```

### Example of Inequality

The following code computes qbpp::Expr object `f` that takes minimum value of 0 if
@f$
2\leq a+2b+3c +4d\leq 4
@f$
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
  for (const auto sol : sols) {
    std::cout << sol << " *f = " << sol.get(*f) << std::endl;
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
@f$
8\leq a+2b+3c +4d
@f$
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
  for (const auto sol : sols) {
    std::cout << sol << " *f = " << sol.get(*f) << std::endl;
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
  auto f = 2 <= a + 2 * b + 3 * c <= 3
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  qbpp::MapList var_map = {{a, 0}, {b, 0}, {c, 0}};
  std::cout << "f(0, 0, 0) = " << qbpp::eval(f, var_map) << std::endl;
  std::cout << "f(0, 1, 0) = " << qbpp::eval(f, {{a, 0}, {b, 1}, {c, 0}}) << std::endl;
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

Since the operator() for a qbpp::Expr object simply calls the qbpp::eval function, the values can be evaluated in a more intuitive way as follows:
```cpp
  std::cout << "f(0, 0, 0) = " << f(var_map) << std::endl;
  std::cout << "f(0, 1, 0) = " << f({{a, 0}, {b, 1}, {c, 0}}) << std::endl;
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

# Vectors of qbpp::Expr Objects

The QUBO++ library supports vectors and nested vectors of qbpp::Expr objects, which can be treated as multi-dimensional arrays of qbpp::Expr.
There is no limitation on the depth of nested vectors.
These structures are implemented using the qbpp::Vector class, which is a wrapper around std::vector.
In this context, **Vector** refers to any instance of qbpp::Vector and its nested forms, containing ExprType objects.
A **Scalar** refers to a single ExprType object.

These operators and functions are defined as either **Global** and/or **Member** functions of the qbpp::Vector class:

* **Global functions**: These functions return a new object based on the operation performed on the given Vector objects.
* **Member functions**: These functions modify the current object in-place, updating it with the result of the operation.

We categorize operators and functions for the Vectors into three types:

* **Element-wise**: The operation is applied to each element independently.
* **Vector-wise**: The operation is applied to each vector independently, reducing the depth of the result by one level.
* **Object-wise**: The operation is applied to all elements collectively, producing a Scalar result.

Element-wise functions can be defined as both global and member functions.
However, Vector-wise and Object-wise functions cannot be member functions because the resulting values are not of the same type as the original objects.

The following table summarizes the operators and functions defined for Vectors of qbpp::Expr objects:

| Operators/Functions           | Operator Symbols/Function Names                      | Global/Member Function | Operation Type | Arguments                                   |
|-------------------------------|------------------------------------------------------|------------------------|----------------|---------------------------------------------|
| Type Conversion               | toExpr()                                             | Global                 | Element-wise   | Vector                                      |
| Assignment                    | `=`                                                  | Member                 | Element-wise   | Vector                                      |
| Binary Operators              | `+`, `-`, `*`                                        | Global                 | Element-wise   | Vector-Vector, Scalar-Vector, Vector-Scaler |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member                 | Element-wise   | Vector, Scalar                              |
| Division                      | `/`                                                  | Global                 | Element-wise   | Vector-Int                                  |
| Compound Division             | `/=`                                                 | Member                 | Element-wise   | Int                                         |
| Unary Operators               | `+`, `-`                                             | Global                 | Element-wise   | Vector                                      |
| Comparison (Equality)         | `==`                                                 | Global                 | Element-wise   | Vector-Int                                  |
| Comparison (range comparison) | `<= <=`                                              | Global                 | Element-wise   | IntInf-Vector-IntInf                        |
| Square                        | sqr()                                                | Global                 | Element-wise   | Vector                                      |
| Square                        | sqr()                                                | Member                 | Element-wise   | -                                           |
| Simplify                      | simplify(), simplify_as_binary(), simplify_as_spin() | Global                 | Element-wise   | Vector                                      |
| Simplify                      | simplify(), simplify_as_binary(), simplify_as_spin() | Member                 | Element-wise   | -                                           |
| Reduce                        | reduce()                                             | Global                 | Element-wise   | Vector                                      |
| Reduce                        | reduce()                                             | Member                 | Element-wise   | -                                           |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Global                 | Element-wise   | Vector                                      |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Member                 | Element-wise   | -                                           |
| Eval/Replace                  | eval(),replace()                                     | Global                 | Element-wise   | Vector-MapList                              |
| Replace                       | replace()                                            | Member                 | Element-wise   | MapList                                     |
| Sum                           | sum()                                                | Global                 | Ojbect-wise    | Vector (1-d)                                |
| Total                         | total_sum(), total_product()                         | Global                 | Object-wise    | Vector (2-d or higher)                      | s |
| Vector Sum/Vector Product     | vector_sum(), vector_product()                       | Global                 | Vector-wise    | Vector (2-d or higher)                      |
| GCD                           | gcd()                                                | Global                 | Object-wise    | Int                                         |
| Transpose                     | transpose()                                          | Global                 | Element-wise   | Vector (2-d)                                |
| Transpose                     | transpose()                                          | Member                 | Element-wise   | -                                           |
| Row/Column Read               | get_row(), get_col()                                 | Global                 | Vector         | Vector,Int                                  |

We will provide several example programs to demonstrate how to use these operators and functions with Vectors of qbpp::Expr objects.

## Creating a Vector of qbpp::Expr objects

Recall that the qbpp::expr() function can create a Vector of qbpp::Expr objects, with each object storing an expression.
The following program demonstrates how to create Vectors and assign Scalars, such as the integer 1 and a qbpp::Var object x, to them:

```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto f = qbpp::expr(2, 2);
  auto g = qbpp::expr(2, 2);
  f = 1;
  std::cout << qbpp::str(f, "f") << std::endl;
  g = x;
  std::cout << qbpp::str(g, "g") << std::endl;
}
```

Since qbpp::Expr objects do not have labels, the label strings must be specified in the str() function to display them.
If label strings are not needed, conventional stream output, such as std::cout << f, can be used.

The following output shows that all elements of the Vectors hold the same scalar value:

```cpp
{f[0][0],1},{f[0][1],1},{f[1][0],1},{f[1][1],1}
{g[0][0],x},{g[0][1],x},{g[1][0],x},{g[1][1],x}
```

A Vector of qbpp::Expr objects can be created without using the qbpp::toExpr() function.
The following program demonstrates how to create Vectors using the qbpp::toExpr() function or operators:

```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 2);
  auto f = qbpp::toExpr(x);
  f += 1;
  auto g = x + 1;
  std::cout << qbpp::str(f, "f") << std::endl;
  std::cout << qbpp::str(g, "g") << std::endl;
}
```

The qbpp::toExpr() function is required to explicitly convert the Vector `x` of qbpp::Var objects to a Vector of qbpp::Expr objects.
If the qbpp::toExpr() function is omitted, `f` will be defined as a Vector of qbpp::Var objects, and the following compound assignment `+=` will cause a compilation error.
However, qbpp::toExpr() is not necessary if the right-hand side involves an operator that returns a Vector of qbpp::Expr objects.
The output of this code is:

```cpp
{f[0][0],1 +x[0][0]},{f[0][1],1 +x[0][1]},{f[1][0],1 +x[1][0]},{f[1][1],1 +x[1][1]}
{g[0][0],1 +x[0][0]},{g[0][1],1 +x[0][1]},{g[1][0],1 +x[1][0]},{g[1][1],1 +x[1][1]}
```

The qbpp::toExpr() function can create a Vector of qbpp::Expr objects using an initializer list as follows:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto f = qbpp::toExpr({{1, a}, {a + 1, 2 * a}});
  std::cout << qbpp::str(f, "f") << std::endl;
}
```

The output of this code is:

```cpp
{f[0][0],1},{f[0][1],a},{f[1][0],1 +a},{f[1][1],2*a}
```

Although the Vector class has no inherent limitation on depth, for technical reasons, the depth is restricted to four levels when using initializer lists.

To create and initialize large vectors, expressions can be assigned one by one as follows:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto f = qbpp::expr(100, 100);
  for (size_t i = 0; i < 100; i++) {
    for (size_t j = 0; j < 100; j++) {
      f[i][j] = (i + 1) * a + (j + 1);
    }
  }
  std::cout << qbpp::str(f, "f") << std::endl;
}
```

The output of this code is:

```cpp
{f[0][0],1 +a},{f[0][1],2 +a},{f[0][2],3 +a},...,{f[99][99],100 +100*a}
```

## Basic Functions for Vectors

Basic functions such as sqr(), simplify(), simplify_as_binary(), simplify_as_spin(), reduce(), replace(), eval(), binary_to_spin(), and spin_to_binary() perform element-wise operations on qbpp::Expr objects.
Their global versions return a vector of qbpp::Expr objects with the same dimensions, storing the results of the operations.
Their member versions update the current object and return the modified object.
Note that for the eval() function, only the global version is available.

The following example demonstrates how these functions are used:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");

  auto f = qbpp::toExpr({{(a + b) * a, (a - b) * a}, {(a + b) * (a - b), a + b}});
  std::cout << qbpp::str(f, "f") << std::endl;

  f.sqr();
  std::cout << "f.sqr()\n" << qbpp::str(f, "f") << std::endl;

  f.simplify_as_binary();
  std::cout << "f.simplify_as_binary()\n" << qbpp::str(f, "f") << std::endl;

  f.replace({{a, 1}}).simplify_as_binary();
  std::cout << "f.replace({{a, 1}}).simplify_as_binary()\n" << str(f, "f") << std::endl;

  std::cout << "qbpp::eval(f, {{b, 1}})\n" << qbpp::str(qbpp::eval(f, {{b, 1}}), "f") << std::endl;
}
```

From the output of this code, we can confirm that the operations are performed element-wise:

```cpp
{f[0][0],a*a +b*a},{f[0][1],a*a -b*a},{f[1][0],a*a +a*b -b*a -b*b},{f[1][1],a +b}
f.sqr()
{f[0][0],a*a*a*a +a*a*b*a +b*a*a*a +b*a*b*a},{f[0][1],a*a*a*a -a*a*b*a -b*a*a*a +b*a*b*a},{f[1][0],a*a*a*a +a*a*a*b -a*a*b*a -a*a*b*b +a*b*a*a +a*b*a*b -a*b*b*a -a*b*b*b -b*a*a*a -b*a*a*b +b*a*b*a +b*a*b*b -b*b*a*a -b*b*a*b +b*b*b*a +b*b*b*b},{f[1][1],a*a +a*b +b*a +b*b}
f.simplify_as_binary()
{f[0][0],a +3*a*b},{f[0][1],a -a*b},{f[1][0],a +b -2*a*b},{f[1][1],a +b +2*a*b}
f.replace({{a, 1}}).simplify_as_binary()
{f[0][0],1 +3*b},{f[0][1],1 -b},{f[1][0],1 -b},{f[1][1],1 +3*b}
qbpp::eval(f, {{b, 1}})
{f[0][0],4},{f[0][1],0},{f[1][0],0},{f[1][1],4}
```

## Basic Operators for Vectors

The operators for Vectors in the QUBO++ library support element-wise operations on Vectors of the same dimension.
They also allow element-wise operations between a Vector and a scalar, where the operation is applied to all qbpp::Expr objects within the Vector using the scalar.
The following code demonstrates how these operators are used:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = 0 <= qbpp::var_int("a", 2, 2) <= 3;
  auto b = qbpp::var("b");
  auto f = a + b;
  std::cout << "f = a + b\n" << str(f, "f") << std::endl;
  f = 2 * f + 1;
  f.simplify_as_binary();
  std::cout << "f = 2 * f + 1\n" << str(f, "f") << std::endl;
  f = 3 <= f <= 4;
  f.simplify_as_binary();
  std::cout << "f = 3 <= f <= 4\n" << str(f, "f") << std::endl;
}
```

The output of this code is as follows:

```cpp
f = a + b
{f[0][0],a[0][0][0] +2*a[0][0][1] +b},{f[0][1],a[0][1][0] +2*a[0][1][1] +b},{f[1][0],a[1][0][0] +2*a[1][0][1] +b},{f[1][1],a[1][1][0] +2*a[1][1][1] +b}
f = 2 * f + 1
{f[0][0],1 +2*a[0][0][0] +4*a[0][0][1] +2*b},{f[0][1],1 +2*a[0][1][0] +4*a[0][1][1] +2*b},{f[1][0],1 +2*a[1][0][0] +4*a[1][0][1] +2*b},{f[1][1],1 +2*a[1][1][0] +4*a[1][1][1] +2*b}
f = 3 <= f <= 4
{f[0][0],3 -3*a[0][0][0] -2*a[0][0][1] -3*b +8*a[0][0][0]*a[0][0][1] +4*a[0][0][0]*b +8*a[0][0][1]*b},{f[0][1],3 -3*a[0][1][0] -2*a[0][1][1] -3*b +8*a[0][1][0]*a[0][1][1] +4*a[0][1][0]*b +8*a[0][1][1]*b},{f[1][0],3 -3*a[1][0][0] -2*a[1][0][1] -3*b +8*a[1][0][0]*a[1][0][1] +4*a[1][0][0]*b +8*a[1][0][1]*b},{f[1][1],3 -3*a[1][1][0] -2*a[1][1][1] -3*b +8*a[1][1][0]*a[1][1][1] +4*a[1][1][0]*b +8*a[1][1][1]*b}
```

## Sum and Product Functions
The sum() and product() functions in the QUBO++ library compute the sum and product of elements in a Vector, respectively.
Compilation errors are intentionally triggered if a scalar or a nested vector is passed as an argument.

The vector_sum() and vector_product() functions in the QUBO++ library operate on Vector objects at the lowest dimensional level.
- For two-dimensional Vector objects, the sum or product of each row is computed, resulting in a one-dimensional Vector.
- For three-dimensional Vector objects, the sum or product of elements at the lowest dimensional level is computed, resulting in a two-dimensional Vector.
Compilation errors are intentionally triggered if a scalar or a non-nested vector is passed as an argument.

The following code demonstrates how vector_sum() and vector_product() work on a @f$2\times 2\times 2@f$ Vector:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a", 2, 2, 2);
  auto sum = qbpp::vector_sum(a + 1).simplify_as_binary();
  auto product = qbpp::vector_product(a + 1).simplify_as_binary();
  std::cout << str(sum, "sum") << std::endl;
  std::cout << str(product, "product") << std::endl;
}
```

This code outputs @f$2\times 2\times 2@f$ Vectors as follows:

```cpp
{sum[0][0],2 +a[0][0][0] +a[0][0][1]},{sum[0][1],2 +a[0][1][0] +a[0][1][1]},{sum[1][0],2 +a[1][0][0] +a[1][0][1]},{sum[1][1],2 +a[1][1][0] +a[1][1][1]}
{product[0][0],1 +a[0][0][0] +a[0][0][1] +a[0][0][0]*a[0][0][1]},{product[0][1],1 +a[0][1][0] +a[0][1][1] +a[0][1][0]*a[0][1][1]},{product[1][0],1 +a[1][0][0] +a[1][0][1] +a[1][0][0]*a[1][0][1]},{product[1][1],1 +a[1][1][0] +a[1][1][1] +a[1][1][0]*a[1][1][1]}
```

The total_sum() and total_product() functions compute the sum and product of all qbpp::Expr objects in the Vector and return the result as a scalar qbpp::Expr object.
The following code demonstrates their usage:

```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a", 2, 2, 2);
  auto total_sum = qbpp::total_sum(a).simplify_as_binary();
  auto total_product = qbpp::total_product(a).simplify_as_binary();
  std::cout << "total_sum = " << total_sum << std::endl;
  std::cout << "total_product = " << total_product << std::endl;
}
```

The output of this code is as follows:

```cpp
total_sum = a[0][0][0] +a[0][0][1] +a[0][1][0] +a[0][1][1] +a[1][0][0] +a[1][0][1] +a[1][1][0] +a[1][1][1]
total_product = a[0][0][0]*a[0][0][1]*a[0][1][0]*a[0][1][1]*a[1][0][0]*a[1][0][1]*a[1][1][0]*a[1][1][1]
```

## Functions for Matrices

Matrices can be represented using qbpp::Vector<qbpp::Vector<qbpp::Expr>> objects. The following functions are designed to manipulate matrices:

* transpose(): Transposes the matrix.
* get_row(): Returns a vector of qbpp::Expr objects from the specified row.
* get_col(): Returns a vector of qbpp::Expr objects from the specified column.

The following code demonstrates the usage of these functions:

```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3, 2);
  auto f = qbpp::transpose(x) * 2 + 1;
  std::cout << str(f, "f") << std::endl;
  f.transpose();
  std::cout << "f.transpose()\n" << str(f, "f") << std::endl;
  std::cout << str(get_row(f, 1), "get_row(f,1)") << std::endl;
  std::cout << str(get_col(f, 1), "get_col(f,1)") << std::endl;
}
```

The output of this code is:

```cpp
{f[0][0],1 +2*x[0][0]},{f[0][1],1 +2*x[1][0]},{f[0][2],1 +2*x[2][0]},{f[1][0],1 +2*x[0][1]},{f[1][1],1 +2*x[1][1]},{f[1][2],1 +2*x[2][1]}
f.transpose()
{f[0][0],1 +2*x[0][0]},{f[0][1],1 +2*x[0][1]},{f[1][0],1 +2*x[1][0]},{f[1][1],1 +2*x[1][1]},{f[2][0],1 +2*x[2][0]},{f[2][1],1 +2*x[2][1]}
{get_row(f,1)[0],1 +2*x[1][0]},{get_row(f,1)[1],1 +2*x[1][1]}
{get_col(f,1)[0],1 +2*x[0][1]},{get_col(f,1)[1],1 +2*x[1][1]},{get_col(f,1)[2],1 +2*x[2][1]}
```

The following elegant code generates a QUBO expression for an @f$n\times n@f$ matrix.
The expression reaches a minimum value of 0 when the sum of each row and column equals 1.

```cpp
#include "qbpp.hpp"

int main() {
  const int n = 2;
  auto x = qbpp::var("x", n, n);
  auto f = qbpp::sum((qbpp::vector_sum(x) == 1) + (qbpp::vector_sum(qbpp::transpose(x)) == 1));
  f.simplify_as_binary();
  f /= gcd(f);
  std::cout << "f = " << f << std::endl;
}
```

The output of this code is:

```cpp
f = 2 -x[0][0] -x[0][1] -x[1][0] -x[1][1] +x[0][0]*x[0][1] +x[0][0]*x[1][0] +x[0][1]*x[1][1] +x[1][0]*x[1][1]
```

# Model objects: qbpp::Model and qbpp::QuadModel

The QUBO++ library contains three classes: qbpp::Expr, qbpp::Model, and qbpp::QuadModel, for storing polynomial expressions.

## qbpp::Model object

A qbpp::Model object is created from a qbpp::Expr object.
It has a "has-a" relationship with qbpp::Expr, holding a const qbpp::Expr object as a member variable.
Additionally, it maintains a vector of variables extracted from the qbpp::Expr object, representing indices from 0.
Since all member variables are const, they cannot be modified after creation.
The degree of a qbpp::Expr object can be arbitrarily large, while the corresponding qbpp::QuadModel must be quadratic.
When operating on QUBO models, there is no need to explicitly create a qbpp::Model object.
These qbpp::Model objects are intended to handle High-order Unconstrained Binary Optimization (HUBO) in future extensions of the QUBO++ library.

## qbpp::QuadModel objects

The qbpp::QuadModel class is derived from qbpp::Model and follows an "is-a" relationship with it.
This class can store qbpp::Expr objects with quadratic expressions, whereas a qbpp::Model object can store any qbpp::Expr object.
It includes member variables to store a constant term, as well as vectors of linear and quadratic terms, allowing convenient access to these elements for QUBO solvers.
Similarly, all member variables are const and cannot be modified after creation.

## Implicit Conversion from qbpp::Expr

Implicit conversions from a qbpp::Expr object to both qbpp::Model and qbpp::QuadModel are defined.
Additionally, an implicit conversion from qbpp::Model to qbpp::QuadModel is defined.
Therefore, since QUBO solvers accept qbpp::QuadModel objects, they can also accept qbpp::Expr and qbpp::Model objects.
However, if the same QUBO model is repeatedly passed to QUBO solvers, it is recommended to explicitly create a qbpp::QuadModel object from a qbpp::Expr object to avoid repeated implicit conversions.
Both qbpp::Model and qbpp::QuadModel objects are immutable, and since their data is stored via shared pointers (std::shared_ptr), the copy overhead is minimal.


# Solving QUBO Problems

The following solvers are available through the QUBO++ library:

* [QUBO++ EasySolver]
  * A basic QUBO solver included in the QUBO++ library, designed to run on a CPU.
  * The algorithm employs PositiveMin search combined with Tabu search.
  * As it requires neither a license nor GPUs, this solver is ideal for learning and debugging purposes.
* [QUBO++ ExhaustiveSolver]
  * A straightforward QUBO solver included in the QUBO++ library, designed to run on a CPU.
  * Since it examines all @f$2^n@f$ possible solutions for @f$n@f$ variables, the running time becomes impractical for @f$n>40@f$.
  * This solver is capable of listing all optimal solutions, making it useful for validating the correctness of QUBO formulations for solving optimization problems.
* [ABS2 QUBO Solver](@ref qbpp_abs2)
  * A QUBO solver that runs on Linux servers with CUDA-enabled GPUs.
  * It is available for non-commercial research and evaluation purposes but comes without any guarantees.
* [Gurobi Optimizer](@ref qbpp_grb):
  * A commercial MIP solver that runs on multi-core CPUs.
  * Since this solver supports quadratic objectives, it can be used for QUBO problems.
  * A valid license is required to use this solver.

Please refer to [Sample Programs](SAMPLE.md) for examples of how to call these solvers from the QUBO++ library.

To quickly compare how different QUBO solvers handle a QUBO expression `f`, the following table outlines how to set the QUBO expression, configure parameters, execute the solver, and obtain the solution.

| QUBO Solvers     | Setting QUBO expression `f`                                 | Setting parameter examples                                | Running the solver and obtaining the solution                       |
|------------------|-------------------------------------------------------------|-----------------------------------------------------------|---------------------------------------------------------------------|
| EasySolver       | auto solver = qbpp::easy_solver::EasySolver(f);             | model.time_limit(5);                                      | auto sol = solver.search();                                         |
| ExhaustiveSOlver | auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f); | -                                                         | auto sol = solver.search();                                         |
| Gurobi Optimizer | auto model = qbpp_grb::QuadModel(f);                        | model.set_time_limit(5);                                  | auto sol = model.optimize();                                        |
| ABS2 QUBO Solver | auto model = qbpp_abs2::QuadModel(f)                        | auto param = qbpp_abs2::Param(); param.set_time_limit(5); | auto solver = qbpp_abs2::Solver(); auto sol = solver(model, param); |

## Callback functions

All solvers support callback functions, which are triggered by events such as the discovery of a new best solution.
Typically, callback functions allow users to interrupt solver execution and perform customized operations.
The default callback functions simply display the newly found best solutions.

The following table summarizes how to enable default callback functions and define user-defined custom callback functions.

| QUBO Solvers     | Enabling default callback                           | Customizing callback function                                                            |
|------------------|-----------------------------------------------------|------------------------------------------------------------------------------------------|
| EasySolver       | solver.enable_default_callback();                   | Define callback() function in derived class of qbpp::easy_solver::EasySolver             |
| ExhaustiveSolver | solver.enable_default_callback();                   | Define callback() function in derived class of qbpp::exhaustive_solver::ExhaustiveSolver |
| Gurobi Optimizer | auto cb = qbpp_grb::Callback(model); model.set(cb); | Define callback() function in derived class of qbpp_grb::Callback                        |
| ABS2 QUBO Solver | auto cb = qbpp_abs2::Callback(); param.set(cb);     | Define callback() function in derived class of qbpp_abs2::Callback                       |

The following code demonstrates the use of the default callback function of the ExhaustiveSolver.
The QUBO expression is simply the negative sum of 20 variables, and the default callback is enabled.
As expected, the minimum energy of -20 is achieved when all variables are set to 1.

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto x = qbpp::var("x", 20);
  auto f = -qbpp::sum(x);
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  solver.enable_default_callback();
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
}
```

The callback function in this code displays the TTS (Time to Solution) and energy of new best solutions as follows:

```cpp
f = -x[0] -x[1] -x[2] -x[3] -x[4] -x[5] -x[6] -x[7] -x[8] -x[9] -x[10] -x[11] -x[12] -x[13] -x[14] -x[15] -x[16] -x[17] -x[18] -x[19]
TTS = 0.000s Energy = -1
TTS = 0.000s Energy = -2
TTS = 0.000s Energy = -3
TTS = 0.000s Energy = -4
TTS = 0.001s Energy = -5
TTS = 0.001s Energy = -6
TTS = 0.001s Energy = -7
TTS = 0.001s Energy = -8
TTS = 0.001s Energy = -9
TTS = 0.002s Energy = -10
TTS = 0.002s Energy = -11
TTS = 0.002s Energy = -12
TTS = 0.002s Energy = -13
TTS = 0.003s Energy = -14
TTS = 0.003s Energy = -15
TTS = 0.004s Energy = -16
TTS = 0.004s Energy = -17
TTS = 0.006s Energy = -18
TTS = 0.009s Energy = -19
TTS = 0.014s Energy = -20
sol = -20:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{x[4],1},{x[5],1},{x[6],1},{x[7],1},{x[8],1},{x[9],1},{x[10],1},{x[11],1},{x[12],1},{x[13],1},{x[14],1},{x[15],1},{x[16],1},{x[17],1},{x[18],1},{x[19],1}}
```

# Solution Objects: qbpp::Sol

Solutions of QUBO expressions, stored as qbpp::Expr, qbpp::Model, or qbpp::QuadModel, represent a mapping from qbpp::Var objects to binary (0/1) values.
The APIs for the QUBO solvers supported by the QUBO++ library are designed to output solutions as qbpp::Sol objects.
qbpp::Sol objects contain:

* A const QuadModel object,
* An qbpp::impl::BitVector object that stores a bit vector representing a solution for the QuadModel object, and
* The energy value of the solution.

Note that a solution stored in a qbpp::Sol object may not necessarily be the optimal solution, even though the goal of QUBO is to find the solution with the minimum energy among all possible solutions.

Once a solution stored in a qbpp::Sol object is obtained, next step is to read binary values for each variable.
Reading the solution can be done using get() member function for the qbpp::Sol object.
The argument of get() member function can be:

* qbpp::Var object: Returns the binary value for the object.
* qbpp::VarInt object: Returns the integer value for the object.
* Vector and nested Vector of qbpp::Var or qbpp::VarInt object: Returns the binary/integer values of a Vector of the same size.

The energy() member function simply returns the energy value of the solution.

The following code demonstrates an example of reading the energy value and the solution of each qbpp::Var object.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (a + b == 1) + (b + c == 1);
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.set_target_energy(0);
  auto sol = solver.search();
  std::cout << "Energy = " << sol.get_energy() << std::endl;
  std::cout << "a = " << sol.get(a) << std::endl;
  std::cout << "b = " << sol.get(b) << std::endl;
  std::cout << "c = " << sol.get(c) << std::endl;
}
```

The output of this code is:

```cpp
f = 2 -a -2*b -c +2*a*b +2*b*c
Energy = 0
a = 1
b = 0
c = 1
```

The following code demonstrates how to read the energy value and the solution for each qbpp::VarInt object:

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = qbpp::var_int("x") <= 10;
  auto y = qbpp::var_int("y") <= 10;
  auto f = (x + 2 * y == 11) + (2 * x - y == 7);
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.set_target_energy(0);
  auto sol = solver.search();
  std::cout << "Energy = " << sol.get_energy() << std::endl;
  std::cout << "x = " << sol.get(x) << std::endl;
  std::cout << "y = " << sol.get(y) << std::endl;
}
```

The output of this code is:

```cpp
f = 170 -45*x[0] -80*x[1] -120*x[2] -105*x[3] -25*y[0] -40*y[1] -40*y[2] -45*y[3] +20*x[0]*x[1] +40*x[0]*x[2] +30*x[0]*x[3] +80*x[1]*x[2] +60*x[1]*x[3] +120*x[2]*x[3] +20*y[0]*y[1] +40*y[0]*y[2] +30*y[0]*y[3] +80*y[1]*y[2] +60*y[1]*y[3] +120*y[2]*y[3]
Energy = 0
x = 5
y = 3
```

## Reading Vector Solutions and Onehot Function
The get() member function of the qbpp::Sol class can retrieve solutions for a Vector or a nested Vector of qbpp::Var objects as the same Vector or nested Vector, respectively.
Additionally, binary Vector values are often used to represent integers using one-hot encoding, where the integer @f$i@f$ is represented if the @f$i@f$-th bit is 1 and all other bits are 0. Furthermore, a Vector of Vectors of binary values is used to represent a sequence of integers.

The QUBO++ library provides the global function onehot_to_int() to obtain the integer values represented by such Vectors.

In the following example code, a QUBO expression `f` is created with a Vector of `x`, which takes its minimum value of 0 when the sum of `x` equals 1.
Therefore, the optimal solution is one-hot, and onehot_to_int() returns the corresponding integer value of the solution.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = qbpp::var("x", 10);
  auto f = qbpp::sum(x) == 1;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.set_target_energy(0);
  auto sol = solver.search();
  std::cout << sol << std::endl;
  std::cout << qbpp::onehot_to_int(sol.get(x)) << std::endl;
}
```

Due to the random nature of EasySolver, the output may vary each time the program is run.
Below is an example of the output:

```cpp
f = 1 -x[0] -x[1] -x[2] -x[3] -x[4] -x[5] -x[6] -x[7] -x[8] -x[9] +2*x[0]*x[1] +2*x[0]*x[2] +2*x[0]*x[3] +2*x[0]*x[4] +2*x[0]*x[5] +2*x[0]*x[6] +2*x[0]*x[7] +2*x[0]*x[8] +2*x[0]*x[9] +2*x[1]*x[2] +2*x[1]*x[3] +2*x[1]*x[4] +2*x[1]*x[5] +2*x[1]*x[6] +2*x[1]*x[7] +2*x[1]*x[8] +2*x[1]*x[9] +2*x[2]*x[3] +2*x[2]*x[4] +2*x[2]*x[5] +2*x[2]*x[6] +2*x[2]*x[7] +2*x[2]*x[8] +2*x[2]*x[9] +2*x[3]*x[4] +2*x[3]*x[5] +2*x[3]*x[6] +2*x[3]*x[7] +2*x[3]*x[8] +2*x[3]*x[9] +2*x[4]*x[5] +2*x[4]*x[6] +2*x[4]*x[7] +2*x[4]*x[8] +2*x[4]*x[9] +2*x[5]*x[6] +2*x[5]*x[7] +2*x[5]*x[8] +2*x[5]*x[9] +2*x[6]*x[7] +2*x[6]*x[8] +2*x[6]*x[9] +2*x[7]*x[8] +2*x[7]*x[9] +2*x[8]*x[9]
0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{x[4],0},{x[5],0},{x[6],0},{x[7],0},{x[8],1},{x[9],0}}
8
```

The get() and onehot_to_int() functions also support nested Vectors.
The following code demonstrates an example using a Vector of Vectors.
It creates a QUBO expression f for a Vector of Vectors of qbpp::Var objects, where each element of `x` takes binary values.
This expression f reaches its minimum value of 0 if each row and column of `x` contains exactly one 1.
By applying the onehot_to_int() function to the resulting value of `x`, we can obtain a sequence of distinct integers.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  const int n = 10;
  auto x = qbpp::var("x", n, n);
  auto f = qbpp::sum((qbpp::vector_sum(x) == 1) +
                     (qbpp::vector_sum(qbpp::transpose(x)) == 1))
               .simplify_as_binary();
  f /= gcd(f);

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.set_target_energy(0);
  auto sol = solver.search();
  auto result = qbpp::onehot_to_int(sol.get(x));

  for (size_t i = 0; i < n; ++i) std::cout << " " << result[i];
  std::cout << std::endl;
}
```

An example output of this code is:

```cpp
 3 7 0 1 2 8 5 6 9 4
```

# Solution Holder Objects: qbpp::SolHolder

The qbpp::SolHolder class is designed to store the best solution obtained during computation.
It holds the solution along with the TTS (Time To Solution) and the solver’s name.
The EasySolver and ExhaustiveSolver classes in the QUBO++ library use qbpp::SolHolder to store and display the best solution in their callback functions.
It is also used in the factorization sample program (factorization.cpp) to exchange the best solution found by concurrently executed QUBO solvers.

# Compiler Options for QUBO++ Library

* **MAXDEG**: This compiler option specifies the implementation of the qbpp::Var vector in the qbpp::Term class, which determines the degree of the qbpp::Expr class.
  * **MAXDEG=0** (default): There is no limit on the size of the qbpp::Var vector, allowing the degree of qbpp::Expr to be arbitrarily large. However, the first two qbpp::Var objects are stored in a fixed reserved space. Operations on qbpp::Expr are efficient if the maximum degree is 2, or if most terms have a degree of 2 or less.
  * **MAXDEG>=1** : The size of the qbpp::Var vector is fixed to MAXDEG, and qbpp::Term objects with a degree larger than MAXDEG cannot be handled. If it is guaranteed that qbpp::Term objects will not exceed this degree, this option can be the most efficient. 
  
* **COEFF_TYPE** and **ENERGY_TYPE**: These compiler options specify the integer data types for coefficients and energy values.
By default, they are set to `int32_t` and `int64_t`, respectively.
Other possible data types include `int8_t`, `int16_t`, `int32_t`, `int64_t`, `int128_t`, `int256_t`, `int512_t`, `int1024_t` and `cpp_int`.
Data types from `int128_t` to `cpp_int` utilize the Boost Multiprecision library, which allows EasySolver and ExhaustiveSolver to handle QUBO models with these types.

# Tips

## Estimating Constant and Coefficient Bit Counts
In QUBO++, two integer data types, qbpp::energy_t and qbpp::coeff_t, are used as follows:
* qbpp::energy_t: Represents the resulting energy value and the constant term value of qbpp::Expr objects. The default type is int64_t.
* qbpp::coeff_t: Represents the coefficient values of qbpp::Term objects. The default type is int32_t.

If you are unsure how large the coefficients of terms can be, you can initially specify `cpp_int` for `qbpp::energy_t` and `qbpp::coeff_t`.
This enables computations with an unlimited number of digits.
To achieve this, define `ENERGY_TYPE` and `COEFF_TYPE` as `cpp_int`, or use compiler options such as `-DENERGY_TYPE=cpp_int` and `-DCOEFF_TYPE=cpp_int`.

An example source code is provided below:
```cpp
./te

#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  qbpp::Vector<qbpp::cpp_int> input = {qbpp::cpp_int("48371620594832719402"),
                                       qbpp::cpp_int("90536217483910257649"),
                                       qbpp::cpp_int("32785041960278314958"),
                                       qbpp::cpp_int("57491836204712038956"),
                                       qbpp::cpp_int("21980475632419876532"),
                                       qbpp::cpp_int("86321504789214367501"),
                                       qbpp::cpp_int("30284715609382147658"),
                                       qbpp::cpp_int("74961830529487162034"),
                                       qbpp::cpp_int("61839427501038476925"),
                                       qbpp::cpp_int("58274310984651293847")};
  auto x = qbpp::var("x", input.size());
  auto f = qbpp::sqr(qbpp::sum(x * input) * 2 - qbpp::sum(input));
  auto model = qbpp::QuadModel(simplify_as_binary(f));
  std::cout << "f = " << f << std::endl;
  qbpp::exhaustive_solver::ExhaustiveSolver solver(model);
  auto sol = solver.search();
  std::cout << "Solution: " << sol << std::endl;
  std::cout << "constant = " << model.get_constant()
            << " min_coeff = " << model.get_min_coeff()
            << " max_coeff = " << model.get_max_coeff() << std::endl;
}
```
The output of this code is:

```cpp
f = 316796724347183046560485185991381254433444 -54451641263806484987544949395176353347448*x[0] -101916073416454334651924314933858546257676*x[1] -36905923797612458117204531816915974001192*x[2] -64718212913274169775880553710639468355344*x[3] -24743288714048638032000867985227486835568*x[4] -97171596782026506575492602682245353880924*x[5] -34091321519929326785695023593469152415992*x[6] -84384080050977826883636353625167370259416*x[7] -69612270187313558986270647143279424428700*x[8] -65599040048922788325320527096783379084628*x[9] -54451641263806484987544949395176353347448*x[0] +9359254715881779547145226204713868950416*x[0]*x[0] +17517534248891870189699032471508564823592*x[0]*x[1] +6343462443553013641318875644949573660464*x[0]*x[2] +11123893152778392923472198474102564097248*x[0]*x[3] +4252924911141520168938131005626563495456*x[0]*x[4] +16702044315355651679759482558996963817608*x[0]*x[5] +5859683093117765694166585745756981842064*x[0]*x[6] +14504100901866005207975156884526518334672*x[0]*x[7] +11965093299527590551601643633335907195400*x[0]*x[8] +11275291445499380371013566166835200477976*x[0]*x[9] -101916073416454334651924314933858546257676*x[1] +17517534248891870189699032471508564823592*x[1]*x[0] +32787226705175589696962740495686252028804*x[1]*x[1] +11872934756539524021853557204293402454968*x[1]*x[2] +20820373544716619407876714124200339897776*x[1]*x[3] +7960116489026223213384912455795554373072*x[1]*x[4] +31260890124538851435145582922805929060596*x[1]*x[5] +10967454395397575558614147603534967744168*x[1]*x[6] +27147042367238693303956768198843003592264*x[1]*x[7] +22394831429258082268501913235357164997300*x[1]*x[8] +21103742772125640207853261155691913542812*x[1]*x[9] -36905923797612458117204531816915974001192*x[2] +6343462443553013641318875644949573660464*x[2]*x[0] +11872934756539524021853557204293402454968*x[2]*x[1] +4299435905348839107009314938303386167056*x[2]*x[2] +7539489049379728739473087617136534015392*x[2]*x[3] +2882523263663042734368384976381475062624*x[2]*x[4] +11320216626355034239740209350016149519832*x[2]*x[5] +3971542688034757479981251466045544273456*x[2]*x[6] +9830507037314034493125705135612127618288*x[2]*x[7] +8109632901684540988913249366647061376600*x[2]*x[8] +7642102923352400788625427934446693853704*x[2]*x[9] -64718212913274169775880553710639468355344*x[3] +11123893152778392923472198474102564097248*x[3]*x[0] +20820373544716619407876714124200339897776*x[3]*x[1] +7539489049379728739473087617136534015392*x[3]*x[2] +13221244920757751934699781302091446279744*x[3]*x[3] +5054791619042991234613357012669976722368*x[3]*x[4] +19851127257143112945445656017998449475824*x[3]*x[5] +6964495637283537504322958313821760660192*x[3]*x[6] +17238773129626633021318401780024368786016*x[3]*x[7] +14221048947531476946684842874902028361200*x[3]*x[8] +13401188568288094893854109904331468414928*x[3]*x[9] -24743288714048638032000867985227486835568*x[4] +4252924911141520168938131005626563495456*x[4]*x[0] +7960116489026223213384912455795554373072*x[4]*x[1] +2882523263663042734368384976381475062624*x[4]*x[2] +5054791619042991234613357012669976722368*x[4]*x[3] +1932565236109615884737302531000497388096*x[4]*x[4] +7589550930292568301391759821051573546128*x[4]*x[5] +2662689813947160674140674050709411848224*x[4]*x[6] +6590786757259923743921061395285431944352*x[4]*x[7] +5437040117237487284417433014759324096400*x[4]*x[8] +5123588290376742824088719707175165194416*x[4]*x[9] -97171596782026506575492602682245353880924*x[5] +16702044315355651679759482558996963817608*x[5]*x[0] +31260890124538851435145582922805929060596*x[5]*x[1] +11320216626355034239740209350016149519832*x[5]*x[2] +19851127257143112945445656017998449475824*x[5]*x[3] +7589550930292568301391759821051573546128*x[5]*x[4] +29805608756297435940197603512013939940004*x[5]*x[5] +10456888894061104684779324309237833850632*x[5]*x[6] +25883272052237607374051766602251242628136*x[5]*x[7] +21352289748772670207565287148113033657700*x[5]*x[8] +20121304858998976342908533122005592265388*x[5]*x[9] -34091321519929326785695023593469152415992*x[6] +5859683093117765694166585745756981842064*x[6]*x[0] +10967454395397575558614147603534967744168*x[6]*x[1] +3971542688034757479981251466045544273456*x[6]*x[2] +6964495637283537504322958313821760660192*x[6]*x[3] +2662689813947160674140674050709411848224*x[6]*x[4] +10456888894061104684779324309237833850632*x[6]*x[5] +3668655998164618828669987297106059539856*x[6]*x[6] +9080790876576876315169713630473438465488*x[6]*x[7] +7491157901263822472926865875399103166600*x[6]*x[8] +7059283742011434358618538894853203441304*x[6]*x[9] -84384080050977826883636353625167370259416*x[7] +14504100901866005207975156884526518334672*x[7]*x[0] +27147042367238693303956768198843003592264*x[7]*x[1] +9830507037314034493125705135612127618288*x[7]*x[2] +17238773129626633021318401780024368786016*x[7]*x[3] +6590786757259923743921061395285431944352*x[7]*x[4] +25883272052237607374051766602251242628136*x[7]*x[5] +9080790876576876315169713630473438465488*x[7]*x[6] +22477104145326214142027916795277484068624*x[7]*x[7] +18542386737493416400171713939424180261800*x[7]*x[8] +17473396097016249765554502888616944819192*x[7]*x[9] -69612270187313558986270647143279424428700*x[8] +11965093299527590551601643633335907195400*x[8]*x[0] +22394831429258082268501913235357164997300*x[8]*x[1] +8109632901684540988913249366647061376600*x[8]*x[2] +14221048947531476946684842874902028361200*x[8]*x[3] +5437040117237487284417433014759324096400*x[8]*x[4] +21352289748772670207565287148113033657700*x[8]*x[5] +7491157901263822472926865875399103166600*x[8]*x[6] +18542386737493416400171713939424180261800*x[8]*x[7] +15296459174624775548115997237295029822500*x[8]*x[8] +14414600117233255303642347961326015921900*x[8]*x[9] -65599040048922788325320527096783379084628*x[9] +11275291445499380371013566166835200477976*x[9]*x[0] +21103742772125640207853261155691913542812*x[9]*x[1] +7642102923352400788625427934446693853704*x[9]*x[2] +13401188568288094893854109904331468414928*x[9]*x[3] +5123588290376742824088719707175165194416*x[9]*x[4] +20121304858998976342908533122005592265388*x[9]*x[5] +7059283742011434358618538894853203441304*x[9]*x[6] +17473396097016249765554502888616944819192*x[9]*x[7] +14414600117233255303642347961326015921900*x[9]*x[8] +13583581282943401794482046458284560237636*x[9]*x[9]
Solution: 2791397863231656792222775232303104:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{x[4],1},{x[5],0},{x[6],1},{x[7],0},{x[8],0},{x[9],0}}
constant = 316796724347183046560485185991381254433444 min_coeff = -171044920127733079606885889372030840486548 max_coeff = 62521780249077702870291165845611858121192
```
From this output, you can estimate the required bit count for `energy_t` and `coeff_t`.

{% endraw %}