---
layout: default
title: "Variable and Expression Classes"
---


# Variable and Expression Classes

## qbpp::Var, qbpp::Term, and qbpp::Expr classes

QUBO++ provides the following fundamental classes:
- **`qbpp::Var`**: Represents a variable symbolically and is associated with a string used for display.
Internally, a 32-bit unsigned integer is used as its identifier.
- **`qbpp::Term`**: Represents a product term consisting of an integer coefficient and one or more `qbpp::Var` objects.
The data type of the integer coefficient is defined by the `COEFF_TYPE` macro, whose default value is `int32_t`.
- **`qbpp::Expr`**: Represents an expanded expression consisting of an integer constant term and zero or more `qbpp::Term` objects.
The data type of the integer constant term is defined by the `ENERGY_TYPE` macro, whose default value is `int64_t`.

In the following program, **`x`** and **`y`** are `qbpp::Var` objects, **`t`** is a `qbpp::Term` object, and **`f`** is a `qbpp::Expr` object:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto t = 2 * x * y;
  auto f = t - x + 1;

  std::cout << "x = " << x << std::endl;
  std::cout << "y = " << y << std::endl;
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
x = x
y = y
t = 2*x*y
f = 1 -x +2*x*y
```
If the data types are to be explicitly specified, the program can be rewritten as follows:
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Var x = qbpp::var("x");
  qbpp::Var y = qbpp::var("y");
  qbpp::Term t = 2 * x * y;
  qbpp::Expr f = t - x + 1;

  std::cout << "x = " << x << std::endl;
  std::cout << "y = " << y << std::endl;
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
`qbpp::Var` objects are **immutable** and cannot be updated after creation.
In contrast, `qbpp::Term` and `qbpp::Expr` objects are **mutable** and can be updated via assignment.

For example, as shown in the following program, compound assignment operators can be used to update `qbpp::Term` and `qbpp::Expr` objects:
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Var x = qbpp::var("x");
  qbpp::Var y = qbpp::var("y");
  qbpp::Term t = 2 * x * y;
  qbpp::Expr f = t - x + 1;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;

  t *= 3 * x;
  f += 2 * y;
  
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
This program prints the following output:
```
t = 2*x*y
f = 1 -x +2*x*y
t = 6*x*y*x
f = 1 -x +2*x*y +2*y
```
In most cases, there is no need to explicitly use `qbpp::Term` objects.
They should only be used when maximum performance optimization is required.

However, note that `auto` type deduction may create a `qbpp::Term` object, which cannot store general expressions.
For example, the following program results in a compilation error because an expression is assigned to a `qbpp::Term` object:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");

  auto t = 2 * x * y;
  t = x + 1;
}
```
If a `qbpp::Expr` object is intended, **`qbpp::toExpr()`** can be used to explicitly construct one, as shown below:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto t = qbpp::toExpr(2 * x * y);
  auto f = qbpp::toExpr(1);

  t += x + 1;
  f += t;
  
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
In this program, both **`t`** and **`f`** are `qbpp::Expr` objects and can store general expressions.
In particular, `f` is created as a `qbpp::Expr` object containing only a constant term with value `1` and no product terms.

## COEFF_TYPE and ENERGY_TYPE
The macros **`COEFF_TYPE`** and **`ENERGY_TYPE`** define the data types used for coefficients and energy values in expressions.
The `ENERGY_TYPE` macro is also used as the data type for the integer constant term of a `qbpp::Expr` object.
By default, `COEFF_TYPE` and `ENERGY_TYPE` are defined as **`int32_t`** and **`int64_t`**, respectively.
They can be changed either by compiler options or by using `#define` directives in the source code.

The following data types are supported:
- **Standard integer types**:
**`int8_t`**, **`int16_t`**, **`int32_t`**, and **`int64_t`**

- **Boost.Multiprecision integer types**:
**`qbpp::int128_t`**, **`qbpp::int256_t`**, **`qbpp::int512_t`**, **`qbpp::int1024_t`**, and **`qbpp::cpp_int`**

The type **`qbpp::cpp_int`** represents an integer with an arbitrary number of digits.
Constant values of this type can be specified using string literals.

For example, the following program creates a `qbpp::Expr` object with very large coefficient and constant terms:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto f = qbpp::cpp_int("123456789012345678901234567890") * x +
           qbpp::cpp_int("987654321098765432109876543210");
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```