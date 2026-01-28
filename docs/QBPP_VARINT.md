---
layout: default
title: "Integer Variables"
---

# Integer Variables
In QUBO++, an **integer variable** (a `qbpp::VarInt` object) can represent an integer value within a specified range.

## Defining an integer variable with a range
You can define an integer variable using `qbpp::var_int()` together with the range operator `<= ... <=`, which specifies the lower and upper bounds:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 10;
  std::cout << "x = " << x << std::endl;
}
```
The string argument `"x"` is used as the display name of the integer variable.
The following output shows that the integer variable is internally represented using multiple binary variables:
```
x = 1 +x[0] +2*x[1] +4*x[2] +2*x[3]
```
As shown above, the integer variable `x`, which takes values in `[1, 10]`, is represented as an expression over four binary variables: `x[0]`, `x[1]`, `x[2]`, and `x[3]`.
The expression `1 + x[0] + 2*x[1] + 4*x[2] + 2*x[3]` can represent any integer value from 1 to 10 using a binary-encoded form.
The coefficient of the last term, `2*x[3]`, is chosen so that the maximum representable value does not exceed the upper bound.

## Defining a constant integer variable and updating it with a range
You can define an integer variable with a fixed constant value.
You can also update the range of an existing integer variable.
This is useful when you want to declare an integer variable first and decide its range later:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var_int("x") == 0;
  std::cout << "x = " << x << std::endl;
  x = 1 <= qbpp::var_int() <= 10;
}
```
This program first defines an integer variable `x` with the fixed value `0`.
Then it updates `x` to represent an integer value in the range `[1, 10]`.

The program produces the following output:
```
(1) x = 0
(2) x = 1 +x[0] +2*x[1] +4*x[2] +2*x[3]
```
As shown above, `x` is updated to represent an integer in `[1, 10]`.

## Defining a vector of integers with the same range
You can define a vector of integer variables by passing its size to `qbpp::var_int()`:
```
#include "qbpp.hpp"

int main() {
  auto x = 1 <= qbpp::var_int("x", 3) <= 10;
  std::cout << "x = " << x << std::endl;
  for (size_t i = 0; i < x.size(); ++i) {
    std::cout << "x[" << i << "] = " << x[i] << std::endl;
  }
}
```
This program creates a vector `x` of three integer variables, each in the range `[1, 10]`.
Each integer variable can be accessed as `x[0]`, `x[1]`, and `x[2]`.

The program produces the following output:
```
x = {1 +x[0][0] +2*x[0][1] +4*x[0][2] +2*x[0][3],1 +x[1][0] +2*x[1][1] +4*x[1][2] +2*x[1][3],1 +x[2][0] +2*x[2][1] +4*x[2][2] +2*x[2][3]}
x[0] = 1 +x[0][0] +2*x[0][1] +4*x[0][2] +2*x[0][3]
x[1] = 1 +x[1][0] +2*x[1][1] +4*x[1][2] +2*x[1][3]
x[2] = 1 +x[2][0] +2*x[2][1] +4*x[2][2] +2*x[2][3]
```

## Defining a vector of integers with different ranges
You can define a vector of integer variables with element-wise ranges by providing vectors of lower and/or upper bounds.

The following example uses `qbpp::Vector<int>` to specify per-element bounds:
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Vector<int> lower = {-1, -2, 2};
  qbpp::Vector<int> upper = {4, 5, 10};
  auto x = 1 <= qbpp::var_int("x", 3) <= upper;
  std::cout << "x = " << x << std::endl;
  auto y = lower <= qbpp::var_int("y", 3) <= 5;
  std::cout << "y = " << y << std::endl;
  auto z = lower <= qbpp::var_int("z", 3) <= upper;
  std::cout << "z = " << z << std::endl;
}
```
In this program, the vectors `lower` and `upper` (each of length 3) specify per-element bounds.
The vector `x` uses a fixed lower bound of 1 and element-wise upper bounds given by `upper`.
The vector `y` uses element-wise lower bounds given by `lower` and a fixed upper bound of 5.
The vector `z` uses element-wise lower and upper bounds given by `lower` and `upper`, respectively.

This program produces the following output:
```
x = {1 +x[0][0] +2*x[0][1],1 +x[1][0] +2*x[1][1] +x[1][2],1 +x[2][0] +2*x[2][1] +4*x[2][2] +2*x[2][3]}
y = {-1 +y[0][0] +2*y[0][1] +3*y[0][2],-2 +y[1][0] +2*y[1][1] +4*y[1][2],2 +y[2][0] +2*y[2][1]}
z = {-1 +z[0][0] +2*z[0][1] +2*z[0][2],-2 +z[1][0] +2*z[1][1] +4*z[1][2],2 +z[2][0] +2*z[2][1] +4*z[2][2] +z[2][3]}
```

# Defining a multi-dimensional vector with different ranges
You can specify element-wise lower and upper bounds for a multi-dimensional array by using nested `qbpp::Vector` objects of integers.

The following program defines a $3 \times 2$ integer array `lower`, and then creates a $3 \times 2$ array `x` of integer variables.
Each `x[i][j]` takes an integer value in the range `[lower[i][j], lower[i][j] + 5]`:
{% raw %}
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Vector<qbpp::Vector<int>> lower = {{-1, 3}, {-1, -2}, {2, 1}};
  auto x = lower <= qbpp::var_int("x", lower.size(), lower[0].size()) <= lower + 5;
  std::cout << "x = " << x << std::endl;
}
```
{% endraw %}
Note that `lower + 5` is a $3 \times 2$ array obtained by adding 5 to each element of lower.
This program produces the following output:
{% raw %}
```
x = {{-1 +x[0][0][0] +2*x[0][0][1] +2*x[0][0][2],3 +x[0][1][0] +2*x[0][1][1] +2*x[0][1][2]},{-1 +x[1][0][0] +2*x[1][0][1] +2*x[1][0][2],-2 +x[1][1][0] +2*x[1][1][1] +2*x[1][1][2]},{2 +x[2][0][0] +2*x[2][0][1] +2*x[2][0][2],1 +x[2][1][0] +2*x[2][1][1] +2*x[2][1][2]}}
```
{% endraw %}
Alternatively, you can first define a $3 \times 2$ array `x` of integer variables with a fixed constant value, and then update each element with the desired range:
{% raw %}
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Vector<qbpp::Vector<int>> lower = {{-1, 3}, {-1, -2}, {2, 1}};
  auto x = qbpp::var_int("x", lower.size(), lower[0].size()) == 0;
  for (size_t i = 0; i < x.size(); ++i) {
    for (size_t j = 0; j < x[i].size(); ++j) {
      x[i][j] = lower[i][j] <= qbpp::var_int() <= lower[i][j] + 5;
    }
  }
  std::cout << "x = " << x << std::endl;
}
```
{% endraw %}
This program updates each `x[i][j]` using the nested for loops and produces the same output as the previous program.