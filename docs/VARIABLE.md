---
layout: default
title: "Variables and Expressions"
nav_order: 1
parent: "C++ Document"
---
# Defining Variables and Expressions

## Header file and namespace
To use QUBO++, you need to include the header file **`qbpp/qbpp.hpp`** and use the **`qbpp`** namespace.

## Defining variables and expressions
You can define a variable using **`qbpp::var("name")`** with auto type deduction.
The specified `name` is used when the variable is printed with `std::cout`.

Expressions are constructed using standard arithmetic operators such as **`+`**, **`-`**, and **`*`**.

The following sample program defines three variables `a`, `b`, and `c`, and an expression `f`, which is printed using `std::cout`:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (a + b - 1) * (b + c - 1);
  std::cout << "f = " << f << std::endl;
}
```
The expression `(a + b - 1) * (b + c - 1)` is automatically expanded and stored in `f`.

In this QUBO++ program, the variables `a`, `b`, and `c` are objects of class **`qbpp::Var`**, and the expression `f` is an object of class **`qbpp::Expr`**.

Assuming the header and library paths are properly set up, this program (saved as **`test.cpp`**) can be compiled with `g++` as follows:
```bash
g++ test.cpp -o test -std=c++17 -lqbpp -ltbb
```
Running the executable prints the expanded expression:
```bash
./test
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

> **NOTE**
> The variable name in `qbpp::var()` may be omitted.
> If omitted, a default name such as `{0}`, `{1}`,... is automatically assigned.

> **WARNING**
> Most QUBO++ class instances, such as `qbpp::Expr`, can be printed as text using `std::cout`.
> However, this textual output is not guaranteed to be stable and should not be used as input for subsequent computations, since its format may change in future releases.
> In addition, the output shown in the QUBO++ documentation may have been generated with an older version of QUBO++, so the output produced by the latest version may differ.

## Simplifying expression
The expression stored in a **`qbpp::Expr`** object can be simplified by calling the **`simplify()`** member function:
```cpp
  std::cout << "f = " << f.simplify() << std::endl;
```
With this change, the output of the program becomes:
```
f.simplify() = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
The member function call **`f.simplify()`** simplifies the expression `f` and returns the resulting value,
which is then printed by `std::cout`.

Assuming that all variables take **binary values (0 or 1)**, we can use the identity 
**$b^2=b$** to further simplify the expression.
For this purpose, we use **`simplify_as_binary()`** instead:
```cpp
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
```
Then the output becomes:
```
f = 1 -a -b -c +a*b +a*c +b*c
```

The simplify functions reorder the variables within each term and the terms within the expression so that lower-degree terms appear first, and terms of the same degree are sorted in the lexicographical order of their variables.
The variables themselves are ordered according to the order in which they were defined.

## Simplifying expressions with spin variables
If variables are assumed to take **spin values $-1$/$+1$**, the identity **$b^2 = 1$** can be used to further simplify the expression.
In this case, the expression can be simplified using the **`simplify_as_spin()`** member function:
```cpp
  std::cout << "f = " << f.simplify_as_spin() << std::endl;
```
Then the output becomes:
```
f = 2 -a -2*b -c +a*b +a*c +b*c
```

## Global functions for simplification
Member functions update the expression stored in `f`.
If you do not want to modify `f`, you can instead use the global functions
**`qbpp::simplify(f)`**, **`qbpp::simplify_as_binary(f)`**, and **`qbpp::simplify_as_spin(f)`**, which return the simplified expressions without changing `f`.

> **NOTE**
> In QUBO++, most **member functions** update the object in place when possible, whereas **global functions** return a new value without modifying the original object.

## Negated literals
QUBO++ natively supports **negated literals** using the `~` operator.
For a binary variable `x`, the expression `~x` represents $1 - x$.

```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = ~a * ~b * ~c * ~d + a * b;
  std::cout << "f = " << f << std::endl;
  std::cout << "f = " << qbpp::simplify_as_binary(f) << std::endl;
}
```
Output:
```
f = ~a*~b*~c*~d +a*b
f = a*b +~a*~b*~c*~d
```

The negated literal `~x` is stored internally as a single variable with a negation flag, **not** expanded as `1 - x`.
This is important for performance: if `~x` were naively expanded, a product of $k$ negated literals such as `~x1 * ~x2 * ... * ~xk` would produce up to $2^k$ terms after expanding $(1-x_1)(1-x_2)\cdots(1-x_k)$.
For example, the term `~a*~b*~c*~d` above is stored as a single quartic term, whereas its expanded form $(1-a)(1-b)(1-c)(1-d)$ produces 16 terms:
```
1 -a -b -c -d +a*b +a*c +a*d +b*c +b*d +c*d -a*b*c -a*b*d -a*c*d -b*c*d +a*b*c*d
```

All solvers bundled with QUBO++ (EasySolver, ExhaustiveSolver, ABS3 GPU Solver) handle negated literals natively, so it is not necessary to expand them before solving.