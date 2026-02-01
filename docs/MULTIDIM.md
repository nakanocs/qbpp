---
layout: default
title: "MULTIDIM"
---

# Multi-dimensional Variables and Expressions

## Defining multi-dimensional variables
QUBO++ supports **multi-dimensional variables** (or `qbpp::Var` objects) and **multi-dimensional integer variables** (or `qbpp::VarInt` objects) of arbitrary depth using the functions `qbpp::var()` and `qbpp::var_int()`, respectively.
Their basic usage is as follows:
- `qbpp::var("name",s1,s2,...,sd)`: Creates an array of `qbpp::Var` objects with the given `name` and shape $s1\times s2\times \cdots\times sd$.
- `l <= qbpp::var_int("name",s1,s2,...,sd) <= u`: Creates an array of `qbpp::VarInt` objects with the specified range and shape $s1\times s2\times \cdots\times sd$.

The following QUBO++ program creates a binary variable and an integer variable, each with dimension $2\times 3\times 4$.
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 3, 4);
  auto y = 1 <= qbpp::var_int("y", 2, 3, 4) <= 8;
  std::cout << "x : " << x << std::endl;
  std::cout << "y : " << y << std::endl;
}
```
This program outputs the following:
{% raw %}
```
x : {{{x[0][0][0],x[0][0][1],x[0][0][2],x[0][0][3]},{x[0][1][0],x[0][1][1],x[0][1][2],x[0][1][3]},{x[0][2][0],x[0][2][1],x[0][2][2],x[0][2][3]}},{{x[1][0][0],x[1][0][1],x[1][0][2],x[1][0][3]},{x[1][1][0],x[1][1][1],x[1][1][2],x[1][1][3]},{x[1][2][0],x[1][2][1],x[1][2][2],x[1][2][3]}}}
y : {{{1 +y[0][0][0][0] +2*y[0][0][0][1] +4*y[0][0][0][2],1 +y[0][0][1][0] +2*y[0][0][1][1] +4*y[0][0][1][2],1 +y[0][0][2][0] +2*y[0][0][2][1] +4*y[0][0][2][2],1 +y[0][0][3][0] +2*y[0][0][3][1] +4*y[0][0][3][2]},{1 +y[0][1][0][0] +2*y[0][1][0][1] +4*y[0][1][0][2],1 +y[0][1][1][0] +2*y[0][1][1][1] +4*y[0][1][1][2],1 +y[0][1][2][0] +2*y[0][1][2][1] +4*y[0][1][2][2],1 +y[0][1][3][0] +2*y[0][1][3][1] +4*y[0][1][3][2]},{1 +y[0][2][0][0] +2*y[0][2][0][1] +4*y[0][2][0][2],1 +y[0][2][1][0] +2*y[0][2][1][1] +4*y[0][2][1][2],1 +y[0][2][2][0] +2*y[0][2][2][1] +4*y[0][2][2][2],1 +y[0][2][3][0] +2*y[0][2][3][1] +4*y[0][2][3][2]}},{{1 +y[1][0][0][0] +2*y[1][0][0][1] +4*y[1][0][0][2],1 +y[1][0][1][0] +2*y[1][0][1][1] +4*y[1][0][1][2],1 +y[1][0][2][0] +2*y[1][0][2][1] +4*y[1][0][2][2],1 +y[1][0][3][0] +2*y[1][0][3][1] +4*y[1][0][3][2]},{1 +y[1][1][0][0] +2*y[1][1][0][1] +4*y[1][1][0][2],1 +y[1][1][1][0] +2*y[1][1][1][1] +4*y[1][1][1][2],1 +y[1][1][2][0] +2*y[1][1][2][1] +4*y[1][1][2][2],1 +y[1][1][3][0] +2*y[1][1][3][1] +4*y[1][1][3][2]},{1 +y[1][2][0][0] +2*y[1][2][0][1] +4*y[1][2][0][2],1 +y[1][2][1][0] +2*y[1][2][1][1] +4*y[1][2][1][2],1 +y[1][2][2][0] +2*y[1][2][2][1] +4*y[1][2][2][2],1 +y[1][2][3][0] +2*y[1][2][3][1] +4*y[1][2][3][2]}}}
```
{% endraw %}
Each `qbpp::Var` object in **`x`** can be accessed as **`x[i][j][k]`**.
Each `qbpp::VarInt` object in **`y`** can be accessed as **`y[i][j][k]`**,
which is internally represented by three binary variables:
- **`y[i][j][k][0]`**
- **`y[i][j][k][1]`**
- **`y[i][j][k][2]`**

corresponding to the binary encoding of integers in the specified range.

## Defining multi-dimensional expressions
QUBO++ allows you to define **multi-dimensional expressions** (or `qbpp::Expr` objects) with arbitrary depth using the function `qbpp::expr()` as follows:
- **`qbpp::expr(s1,s2,...,sd)`**: Creates a multi-dimensional array of `qbpp::Expr` objects with shape $s1\times s2\times \cdots\times sd$.

The following program defines a 3-dimensional array **`x`** of `qbpp::Var` objects with shape $2\times 3\times 4$ and
a 2-dimensional array `f` of  size $2\times 3$.
Then, using a triple for-loop, each `f[i][j]` accumulates the sum of `x[i][j][0]`, `x[i][j][1]`, `x[i][j][2]`, and `x[i][j][3]`:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 3, 4);
  auto f = qbpp::expr(2, 3);
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      for (size_t k = 0; k < 4; ++k) {
        f[i][j] += x[i][j][k];
      }
    }
  }
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
Note that the `simplify_as_binary()` member function can be applied to a multi-dimensional array of `qbpp::Expr` objects.
When called on such an array, it applies `simplify_as_binary()` to each element individually (element-wise).

This program produces the following output:
{% raw %}
```
f = {{x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3],x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3],x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]},{x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3],x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3],x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]}}
f[0][0] = x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3]
f[0][1] = x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3]
f[0][2] = x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]
f[1][0] = x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3]
f[1][1] = x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3]
f[1][2] = x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]
```
{% endraw %}
## Creating an array of expressions by auto type deduction
An array of **`qbpp::Expr`** objects can be created without explicitly calling `qbpp::expr()`.
When a function call or an arithmetic operation yields an array-shaped result, an array of `qbpp::Expr` objects with the same shape can be defined using auto type deduction.

The following QUBO++ program creates an array **`f`** of `qbpp::Expr` objects with the same dimensions as an array **`x`** of `qbpp::Var` objects:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 3);
  auto f = x + 1;
  f += x - 2;
  f.simplify_as_binary();
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
In this program, `x` is defined as a $2 \times 3$ array of `qbpp::Var` objects.
The expression `x + 1` produces a $2 \times 3$ array of `qbpp::Expr` objects, which is used to initialize `f` via auto type deduction.
After that, the expression `x - 2` is added element-wise to `f`.

This program outputs:
```
f[0][0] = -1 +2*x[0][0]
f[0][1] = -1 +2*x[0][1]
f[0][2] = -1 +2*x[0][2]
f[1][0] = -1 +2*x[1][0]
f[1][1] = -1 +2*x[1][1]
f[1][2] = -1 +2*x[1][2]
```

## Implementation of Vectors and Arrays

QUBO++ implements vectors (i.e., one-dimensional arrays) as **`qbpp::Vector<T>`** objects, which are largely compatible with `std::vector<T>`.
The template parameter `T` can be `qbpp::Expr`, `qbpp::Var`, or an integer type.

The `qbpp::Vector<T>` class provides the following member functions for compatibility with `std::vector<T>`.
For details, please refer to the documentation for `std::vector<T>`.

### Element Access

| function | description |
|:---|:---|
| `operator[]` | Returns the element at the specified index. |
| `at()` | Returns the element at the specified index with bounds checking. |
| `front()` | Returns a reference to the first element. |
| `back()` | Returns a reference to the last element. |
| `data()` | Returns a pointer to the underlying array. |

### Capacity

| function | description |
|:---|:---|
| `size()` | Returns the number of elements. |
| `empty()` | Returns `true` if the vector contains no elements. |
| `capacity()` | Returns the number of elements that can be held in the currently allocated storage. |
| `reserve()` | Reserves storage for at least the specified number of elements. |
| `shrink_to_fit()` | Requests the removal of unused capacity to reduce memory usage. |

### Modifiers

| function | description |
|:---|:---|
| `clear()` | Removes all elements. |
| `resize()` | Changes the number of elements. |
| `push_back()` | Appends an element to the end. |
| `emplace_back()` | Constructs an element in place at the end. |
| `pop_back()` | Removes the last element. |
| `insert()` | Inserts elements at the specified position. |
| `emplace()` | Constructs an element in place at the specified position. |
| `erase()` | Removes elements at the specified position or in the specified range. |
| `swap()` | Swaps the contents with another vector. |
| `assign()` | Replaces the contents with the specified elements. |

### Iterators

| function | description |
|:---|:---|
| `begin()`, `end()` | Returns iterators to the beginning/end. |
| `cbegin()`, `cend()` | Returns const iterators to the beginning/end. |
| `rbegin()`, `rend()` | Returns reverse iterators to the beginning/end. |
| `crbegin()`, `crend()` | Returns const reverse iterators to the beginning/end. |

### Additional operators
In addition, unlike `std::vector<T>`, **`qbpp::Vector<T>`** supports the following operators for element-wise operations:

| operator | description |
|:---|:---|
| **`+`** | Performs element-wise addition between two vectors, or between a vector and a scalar. |
| **`-`** | Performs element-wise subtraction between two vectors, or between a vector and a scalar. |
| **`*`** | Performs element-wise multiplication between two vectors, or between a vector and a scalar. |
| unary **`-`** | Negates all elements of the vector. |

Furthermore, multi-dimensional arrays are implemented as nested instances of `qbpp::Vector<T>`.
For example, the data type of `x` in the following code is **`qbpp::Vector<qbpp::Vector<qbpp::Var>>`**:
```cpp
  auto x = qbpp::var("x", 2, 3);
```
The element-wise operations described above are supported for multi-dimensional arrays only when the operands have the same shape.

Since `qbpp::Vector<T>` supports iterators, range-based `for` loops with `auto` type deduction can be used as follows:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 3);
  auto f = x + 1;
  f += x - 2;
  f.simplify_as_binary();
  for (const auto& vec : f) {
    for (const auto& element : vec) {
      std::cout << "(" << element << ")";
    }
    std::cout << std::endl;
  }
}
```
In the outer `for` loop, each `qbpp::Vector<qbpp::Expr>` object contained in the `qbpp::Vector<qbpp::Vector<qbpp::Expr>>` object `f` is referenced in turn by `vec`.
In the inner `for` loop, each `qbpp::Expr` object contained in `vec` is referenced in turn by `element`.

This program outputs:
```
(-1 +2*x[0][0])(-1 +2*x[0][1])(-1 +2*x[0][2])
(-1 +2*x[1][0])(-1 +2*x[1][1])(-1 +2*x[1][2])
```

### Additional Functions

Most QUBO++ functions—such as `qbpp::sqr()` and `qbpp::simplify()`—that work on `qbpp::Expr`, `qbpp::Term`, and integer types also work on `qbpp::Vector` of these types.
When a `qbpp::Vector` is passed, the function is applied element-wise to the vector.
For details, please refer to the documentation for each function.