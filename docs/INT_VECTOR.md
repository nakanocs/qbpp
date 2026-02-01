---
layout: default
title: "Integers with Vector Operations and Functions "
---

# Integers with Vector Operations and Functions

## Creating an array of integers
In C and C++, a multi-dimensional array of integers can be defined as follows:
```cpp
  int x[2][3][4];
```
This creates a $2\times 3\times 4$ array of int.

Similarly, in QUBO++, an array of integers can be created using `qbpp::integer()` with the desired dimensions:
```cpp
  auto x = qbpp::integer(2,3,4)
```
This creates a $2\times 3\times 4$ array of integers (i.e., `qbpp::energy_t`).
All elements are initialized to 0, and the values are mutable, so they can be updated.
In other words, they behave like plain integer containers.

Do not confuse these with the integer variable class `qbpp::VarInt`, which represents an integer expression with a specified range.

The following QUBO++ program creates a $2\times 3\times 4$ integer array `x`, and updates each element `x[i][j][k]` to `i * 100 + j * 10 + k`:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::integer(2, 3, 4);
  for (size_t i = 0; i < x.size(); i++) {
    for (size_t j = 0; j < x[i].size(); j++) {
      for (size_t k = 0; k < x[i][j].size(); k++) {
        x[i][j][k] = static_cast<int>(i * 100 + j * 10 + k);
      }
    }
  }
  std::cout << "x = " << x << std::endl;
}
```
This program produces the following output:
{% raw %}
```
x = {{{0,1,2,3},{10,11,12,13},{20,21,22,23}},{{100,101,102,103},{110,111,112,113},{120,121,122,123}}}
```
{% endraw %}

## Operations and functions for integer arrays
Most operators and functions that work for arrays of expressions (i.e., `qbpp::Expr` objects) also work for integer arrays.
For example, the following program applies the `+` operator, `qbpp::vector_sum()`, and `qbpp::sum()` to an integer array:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::integer(2, 3, 4);
  for (size_t i = 0; i < x.size(); i++) {
    for (size_t j = 0; j < x[i].size(); j++) {
      for (size_t k = 0; k < x[i][j].size(); k++) {
        x[i][j][k] = static_cast<int>(i * 100 + j * 10 + k);
      }
    }
  }

  auto f = x + 1;
  std::cout << "f = " << f << std::endl;
  auto g = qbpp::vector_sum(f);
  std::cout << "g = " << g << std::endl;
  auto h = qbpp::sum(g);
  std::cout << "h = " << h << std::endl;
}
```
- The `+` operator adds 1 to all elements.
- `qbpp::vector_sum()` computes the sum over the innermost dimension and returns the result as a $2\times 3$ array.
- `qbpp::sum()` computes the sum of all elements.

This program produces the following output:
{% raw %}
```
f = {{{1,2,3,4},{11,12,13,14},{21,22,23,24}},{{101,102,103,104},{111,112,113,114},{121,122,123,124}}}
g = {{10,50,90},{410,450,490}}
h = 1500
```
{% endraw %}

## The applications of integer arrays
Integer arrays are useful for creating `qbpp::VarInt` objects.
The following example creates an array of integer variables whose ranges are specified by
the integer array `x`:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::integer(2, 3);
  for (size_t i = 0; i < x.size(); i++) {
    for (size_t j = 0; j < x[i].size(); j++) {
      x[i][j] = static_cast<int>(i * 10 + j);
    }
  }

  std::cout << "x = " << x << std::endl;
  auto v = x <= qbpp::var_int("v", 2, 3) <= x + 3;
  std::cout << "v = " << v << std::endl;
}
```
This program produces the following output:
{% raw %}
```
v = {{v[0][0][0] +2*v[0][0][1],1 +v[0][1][0] +2*v[0][1][1],2 +v[0][2][0] +2*v[0][2][1]},{10 +v[1][0][0] +2*v[1][0][1],11 +v[1][1][0] +2*v[1][1][1],12 +v[1][2][0] +2*v[1][2][1]}}
```
{% endraw %}