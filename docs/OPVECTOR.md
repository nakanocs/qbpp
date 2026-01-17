---
layout: default
title: "Basic Operations and Functions for Vectors"
---

# Basic Operators and Functions for Vectors
Basically, operators and functions for vectors operate element-wise.


## Basic operators for vectors
The basic operators **`+`**, **`-`**, **`*`**, and **`/`** work for vectors of variables and expressions.

In QUBO++, these operators are applied element-wise.

### Vector–Scalar Operations
When you combine a vector and a scalar, the scalar is applied to each element of the vector.
For example, if `x` is a vector of size 3, then:
- `2 * x` produces `{2*x[0], 2*x[1], 2*x[2]}`
- `x + 1` produces `{x[0] + 1, x[1] + 1, x[2] + 1}`

The following program illustrates this behavior:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto f = 2 * x + 1;

  std::cout << "f = " << f << std::endl;
  for (size_t i = 0; i < f.size(); ++i) {
    std::cout << "f[" << i << "] = " << f[i] << std::endl;
  }
}
```
This program creates a vector `x = {x[0], x[1], x[2]}` of binary variables.
Then `2 * x` multiplies each element by `2`, and `+ 1` adds `1` to each element, so `f` becomes:
`{1 + 2*x[0], 1 + 2*x[1], 1 + 2*x[2]}`.
This program produces the following output:
```
f = {1 +2*x[0],1 +2*x[1],1 +2*x[2]}
f[0] = 1 +2*x[0]
f[1] = 1 +2*x[1]
f[2] = 1 +2*x[2]
```

### Vector–Vector Operations
When you combine two vectors of the same size, the operation is performed element-wise at each index.

The following example uses two vectors `x` and `y`, both of size 3:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto y = qbpp::var("y", 3);
  auto f = 2 * x + 3 * y + 1;

  std::cout << "f = " << f << std::endl;
  for (size_t i = 0; i < f.size(); ++i) {
    std::cout << "f[" << i << "] = " << f[i] << std::endl;
  }
}
```
Here:
- `2 * x` becomes `{2*x[0], 2*x[1], 2*x[2]}`
- `3 * y` becomes `{3*y[0], 3*y[1], 3*y[2]}`
- adding them is element-wise, so the `i`-th element is `2*x[i] + 3*y[i]`
- `+ 1` is again applied element-wise

Therefore, `f` becomes: `{1 + 2*x[0] + 3*y[0], 1 + 2*x[1] + 3*y[1], 1 + 2*x[2] + 3*y[2]}`
which matches the output:
```
f = {1 +2*x[0] +3*y[0],1 +2*x[1] +3*y[1],1 +2*x[2] +3*y[2]}
f[0] = 1 +2*x[0] +3*y[0]
f[1] = 1 +2*x[1] +3*y[1]
f[2] = 1 +2*x[2] +3*y[2]
```
Vector–vector operations require the same vector size.

The next example demonstrates a more complex element-wise expression involving vector–scalar operations, vector–vector operations, unary minus, and parentheses:

```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto y = qbpp::var("y", 3);
  auto f = 6 * -(x + 1) * (y - 1);
  auto g = f / 3;

  std::cout << "f = " << f << std::endl;
  for (size_t i = 0; i < x.size(); ++i) {
    std::cout << "f[" << i << "] = " << f[i] << std::endl;
  }

  std::cout << "g = " << g << std::endl;
  for (size_t i = 0; i < x.size(); ++i) {
    std::cout << "g[" << i << "] = " << g[i] << std::endl;
  }
}
```
In this example, all operations are still applied element-wise.

- First, `x + 1` and `y - 1` add/subtract the scalar to/from each element, producing two vectors
`{x[i]+1}` and `{y[i]−1}`.
- The unary minus -(x + 1) also works element-wise, so it becomes `{−(x[i]+1)}`.
- The multiplication `6 * -(x + 1) * (y - 1)` is then performed element-wise as well, so for each index `i`, `f[i]=6⋅(−(x[i]+1))⋅(y[i]−1)`.
Expanding this expression yields `f[i]=6−6x[i]y[i]+6x[i]−6y[i]`, which matches the printed form in the output.
- Finally, `g = f / 3` divides each element by 3, so `g[i]=f[i]/3=2−2x[i]y[i]+2x[i]−2y[i]`,
again matching the output.
```
f = {6 -6*x[0]*y[0] +6*x[0] -6*y[0],6 -6*x[1]*y[1] +6*x[1] -6*y[1],6 -6*x[2]*y[2] +6*x[2] -6*y[2]}
f[0] = 6 -6*x[0]*y[0] +6*x[0] -6*y[0]
f[1] = 6 -6*x[1]*y[1] +6*x[1] -6*y[1]
f[2] = 6 -6*x[2]*y[2] +6*x[2] -6*y[2]
g = {2 -2*x[0]*y[0] +2*x[0] -2*y[0],2 -2*x[1]*y[1] +2*x[1] -2*y[1],2 -2*x[2]*y[2] +2*x[2] -2*y[2]}
g[0] = 2 -2*x[0]*y[0] +2*x[0] -2*y[0]
g[1] = 2 -2*x[1]*y[1] +2*x[1] -2*y[1]
g[2] = 2 -2*x[2]*y[2] +2*x[2] -2*y[2]
```

## Compound opertors for vectors
Similarly, the compound operators **`+=`**, **`-=`**, **`*=`**, and **`/=`** work for vectors of variables and expressions.
The following example demonstrates how these operators work for a vector of size 3:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto y = qbpp::var("y", 3);
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
f = {4 +6*x[0] +3*y[0],4 +6*x[1] +3*y[1],4 +6*x[2] +3*y[2]}
f = {-8 +6*x[0] +3*y[0],-8 +6*x[1] +3*y[1],-8 +6*x[2] +3*y[2]}
f = {12*x[0]*y[0] +6*y[0]*y[0] -16*y[0],12*x[1]*y[1] +6*y[1]*y[1] -16*y[1],12*x[2]*y[2] +6*y[2]*y[2] -16*y[2]}
f = {6*x[0]*y[0] +3*y[0]*y[0] -8*y[0],6*x[1]*y[1] +3*y[1]*y[1] -8*y[1],6*x[2]*y[2] +3*y[2]*y[2] -8*y[2]}
```

## Square functions for vectors
Square functions also work for vectors, as demonstrated below:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto f = x + 1;

  std::cout << "f = " << qbpp::sqr(f) << std::endl;
  std::cout << "f = " << f << std::endl;
  f.sqr();
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
f = {1 +x[0]*x[0] +x[0] +x[0],1 +x[1]*x[1] +x[1] +x[1],1 +x[2]*x[2] +x[2] +x[2]}
f = {1 +x[0],1 +x[1],1 +x[2]}
f = {1 +x[0]*x[0] +x[0] +x[0],1 +x[1]*x[1] +x[1] +x[1],1 +x[2]*x[2] +x[2] +x[2]}
```

## Simplfy functions for vectors
Simplify functions also work for vectors, as demonstrated below:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 3);
  auto f = qbpp::sqr(x - 1);
  std::cout << "f = " << f << std::endl;
  std::cout << "simplified(f) = " << qbpp::simplify(f) << std::endl;
  std::cout << "simplified_as_binary(f) = " << qbpp::simplify_as_binary(f) << std::endl;
  std::cout << "simplified_as_spin(f) = " << qbpp::simplify_as_spin(f) << std::endl;
}
```
This program produces the following output:
```
f = {1 +x[0]*x[0] -x[0] -x[0],1 +x[1]*x[1] -x[1] -x[1],1 +x[2]*x[2] -x[2] -x[2]}
simplified(f) = {1 -2*x[0] +x[0]*x[0],1 -2*x[1] +x[1]*x[1],1 -2*x[2] +x[2]*x[2]}
simplified_as_binary(f) = {1 -x[0],1 -x[1],1 -x[2]}
simplified_as_spin(f) = {2 -2*x[0],2 -2*x[1],2 -2*x[2]}
```

> **NOTE**
> These operators and functions also work for **multi-dimensional arrays**.