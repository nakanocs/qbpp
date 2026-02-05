---
layout: default
title: "Basic Operators and Functions"
---

# Basic Operators and Functions

## Unary and Binary Operators
QUBO++ supports the following basic binary operators for constructing expressions (i.e., `qbpp::Expr` objects):
- **`+`**: Returns the sum of the operands.
- **`-`**: Returns the difference of the operands.
- **`*`**: Returns the product of the operands.
- **`/`**: Returns the quotient of the operands.
The divisor must be an integer, and both the constant term and all coefficients of the dividend must be divisible by the divisor.
- unary **`-`**: Returns the negation of the operand.

The precedence of these operators follows the standard C++ operator precedence rules.

The following program demonstrates how to construct expressions using these operators:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto f = 6 * -(x + 1) * (y - 1);
  auto g = f / 3;

  std::cout << "f = " << f << std::endl;
  std::cout << "g = " << g << std::endl;
}

```
This program produces the following output:
```
f = 6 -6*x*y +6*x -6*y
g = 2 -2*x*y +2*x -2*y
```

## Compound operators
Also the following compound operators to update qbpp::Expr objects are supported.
- **`+=`** : Adds the righthand size operand to the lefthand side. 
- **`-=`** : Subtract the righthand size operand from the lefthand side. 
- **`*=`** : Multiply the righthand side operand to the lefthand side.
- **`/=`** : Divides the lefthand side operand by the righthand side. The righthand side operand must be an integer and the constant term integer and all coefficiets of the lefthand side must be divisible.

The following program demonstrates how to construct expressions using these compound operators:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto f = 6 * x + 4;

  f += 3 * y;
  std::cout << "f = " << f << std::endl;
  
  f -= 12;
  std::cout << "f = " << f << std::endl;
  
  f *= 2 * y;
  std::cout << "f = " << f << std::endl;
  
  f /= 2;
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
f = 4 +6*x +3*y
f = -8 +6*x +3*y
f = 12*x*y +6*y*y -16*y
f = 6*x*y +3*y*y -8*y
```

## Square functions
QUBO++ provides both a global function **`qbpp::sqr()`** and a member function **`sqr()`** of the `qbpp::Expr` class to compute the square of an expression.

In the following program, for a `qbpp::Expr` object `f`, the global function **`qbpp::sqr(f)`** returns a new `qbpp::Expr` object representing the square of `f`,
whereas the member function **`f.sqr()`** updates `f` in place by replacing it with its square.

```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto f = x + 1;

  std::cout << "f = " << qbpp::sqr(f) << std::endl;
  std::cout << "f = " << f << std::endl;

  f.sqr();
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
f = 1 +x*x +x +x
f = 1 +x
f = 1 +x*x +x +x
```

## Simplify functions
After operators or functions are applied to `qbpp::Expr` objects, expressions are automatically expanded.
To sort terms and simplify the resulting expressions, simplify functions must be explicitly called.

QUBO++ provides the following three **global simplify functions**:
- **`qbpp::simplify()`**:
Returns a simplified expression by merging coefficients of identical terms.
- **`qbpp::simplify_as_binary()`**:
Returns a simplified expression under the assumption that all variables take binary values $0/1$
i.e., the identity $x^2=x$ holds.
- **`qbpp::simplify_as_spin()`**:
Returns a simplified expression under the assumption that all variables take spin values $−1/+1$, i.e., the identity $x^2=1$ holds.

The following program demonstrates the behavior of these simplify functions:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto f = qbpp::sqr(x - 1);

  std::cout << "f = " << f << std::endl;
  std::cout << "simplified(f) = " << qbpp::simplify(f) << std::endl;
  std::cout << "simplified_as_binary(f) = " << qbpp::simplify_as_binary(f) << std::endl;
  std::cout << "simplified_as_spin(f) = " << qbpp::simplify_as_spin(f) << std::endl;
}
```
This program produces the following output:
```
f = 1 +x*x -x -x
simplified(f) = 1 -2*x +x*x
simplified_as_binary(f) = 1 -x
simplified_as_spin(f) = 2 -2*x
```

**Member function** versions of these simplify functions are also provided for `qbpp::Expr` objects, and they update the object in place with the simplified result.

For example, the following program updates `f` by applying **`simplify()`**:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto f = qbpp::sqr(x - 1);
  
  f.simplify();
  std::cout << "f = " << f << std::endl;
}
```
This program prints the following output:
```
f = 1 -2*x +x*x
```