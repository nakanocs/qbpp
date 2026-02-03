---
layout: default
title: "Sum Functions for Multi-dimensional Arrays"
---


# Sum Functions for Multi-dimensional Arrays
QUBO++ provides two sum functions for multi-dimensional arrays of variables or expressions:

| function | description |
|:---|:---|
|**`qbpp::sum()`**| Computes the sum of all elements in the array.|
|**`qbpp::vector_sum()`**| Computes the sum along the lowest (innermost) dimension. The resulting array has one fewer dimension than the input array. The input array must have a dimension of 2 or greater.|

The following program demonstrates the difference between `qbpp::sum()` and `qbpp::vector_sum()`:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 3, 3);
  auto y = x + 1;
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      for (size_t k = 0; k < 3; ++k) {
        std::cout << "y[" << i << "][" << j << "][" << k << "] = " << y[i][j][k]
                  << std::endl;
      }
    }
  }
  auto sum = qbpp::sum(y).simplify();
  std::cout << "sum(y) = " << sum << std::endl;
  auto vector_sum = qbpp::vector_sum(y).simplify();
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "vector_sum[" << i << "][" << j << "] = " << vector_sum[i][j]
                << std::endl;
    }
  }
}
```
First, an array `x` of variables with size $2 \times 3 \times 3$ is defined.
Next, an array `y` is created by adding 1 to every element of `x`,
and all elements of `y` are printed.
Then, `qbpp::sum(y)` is computed and printed.
After that, the `qbpp::vector_sum()` function is applied to `y` and the result is stored in `vector_sum`, which is a two-dimensional array of expressions with size $2 \times 3$.
Finally, all elements of `vector_sum` are printed.

This program produces the following output:
```
y[0][0][0] = 1 +x[0][0][0]
y[0][0][1] = 1 +x[0][0][1]
y[0][0][2] = 1 +x[0][0][2]
y[0][1][0] = 1 +x[0][1][0]
y[0][1][1] = 1 +x[0][1][1]
y[0][1][2] = 1 +x[0][1][2]
y[0][2][0] = 1 +x[0][2][0]
y[0][2][1] = 1 +x[0][2][1]
y[0][2][2] = 1 +x[0][2][2]
y[1][0][0] = 1 +x[1][0][0]
y[1][0][1] = 1 +x[1][0][1]
y[1][0][2] = 1 +x[1][0][2]
y[1][1][0] = 1 +x[1][1][0]
y[1][1][1] = 1 +x[1][1][1]
y[1][1][2] = 1 +x[1][1][2]
y[1][2][0] = 1 +x[1][2][0]
y[1][2][1] = 1 +x[1][2][1]
y[1][2][2] = 1 +x[1][2][2]
sum(y) = 18 +x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][2][0] +x[0][2][1] +x[0][2][2] +x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][2][0] +x[1][2][1] +x[1][2][2]
vector_sum[0][0] = 3 +x[0][0][0] +x[0][0][1] +x[0][0][2]
vector_sum[0][1] = 3 +x[0][1][0] +x[0][1][1] +x[0][1][2]
vector_sum[0][2] = 3 +x[0][2][0] +x[0][2][1] +x[0][2][2]
vector_sum[1][0] = 3 +x[1][0][0] +x[1][0][1] +x[1][0][2]
vector_sum[1][1] = 3 +x[1][1][0] +x[1][1][1] +x[1][1][2]
vector_sum[1][2] = 3 +x[1][2][0] +x[1][2][1] +x[1][2][2]
```
The same results can be obtained using explicit for-loops.
However, for large arrays, it is recommended to use `qbpp::sum()` and `qbpp::vector_sum()`, since these functions internally exploit multithreading to accelerate computation.

## Sum Along a Specified Axis

You can specify an axis in **`qbpp::vector_sum()`** to indicate the dimension along which the sum is computed.
An *axis* is a dimension index of an array.

For example, let `x` be a $2\times 3\times 3$ array of binary variables.
Each element is accessed as `x[i][j][k]`.
Axis 0, 1, and 2 correspond to the dimensions indexed by `i`, `j`, and `k`, respectively.
By passing the axis as the second argument to **`qbpp::vector_sum()`**, the function computes the sum along the specified dimension.

The following QUBO++ program computes the vector sum along axis 0:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 2, 3, 3);
  auto f = qbpp::vector_sum(x, 0);
  std::cout << "f= " << f << std::endl;
  for (size_t i = 0; i < f.size(); i++) {
    for (size_t j = 0; j < f[i].size(); j++) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
This program produces the following output:
{% raw %}
```
f= {{x[1][0][0] +x[0][0][0],x[1][0][1] +x[0][0][1],x[1][0][2] +x[0][0][2]},{x[1][1][0] +x[0][1][0],x[1][1][1] +x[0][1][1],x[1][1][2] +x[0][1][2]},{x[1][2][0] +x[0][2][0],x[1][2][1] +x[0][2][1],x[1][2][2] +x[0][2][2]}}
f[0][0] = x[1][0][0] +x[0][0][0]
f[0][1] = x[1][0][1] +x[0][0][1]
f[0][2] = x[1][0][2] +x[0][0][2]
f[1][0] = x[1][1][0] +x[0][1][0]
f[1][1] = x[1][1][1] +x[0][1][1]
f[1][2] = x[1][1][2] +x[0][1][2]
f[2][0] = x[1][2][0] +x[0][2][0]
f[2][1] = x[1][2][1] +x[0][2][1]
f[2][2] = x[1][2][2] +x[0][2][2]
```
{% endraw %}
The vector sum along axis 1 can be computed by passing 1 as the second argument:
```cpp
  auto f = qbpp::vector_sum(x, 1);
```
The modified program produces the following output:
{% raw %}
```
f= {{x[0][2][0] +x[0][1][0] +x[0][0][0],x[0][2][1] +x[0][1][1] +x[0][0][1],x[0][2][2] +x[0][1][2] +x[0][0][2]},{x[1][2][0] +x[1][1][0] +x[1][0][0],x[1][2][1] +x[1][1][1] +x[1][0][1],x[1][2][2] +x[1][1][2] +x[1][0][2]}}
f[0][0] = x[0][2][0] +x[0][1][0] +x[0][0][0]
f[0][1] = x[0][2][1] +x[0][1][1] +x[0][0][1]
f[0][2] = x[0][2][2] +x[0][1][2] +x[0][0][2]
f[1][0] = x[1][2][0] +x[1][1][0] +x[1][0][0]
f[1][1] = x[1][2][1] +x[1][1][1] +x[1][0][1]
f[1][2] = x[1][2][2] +x[1][1][2] +x[1][0][2]
```
{% endraw %}
Finally, the vector sum along axis 2 can be computed as follows:
```cpp
  auto f = qbpp::vector_sum(x, 2);
```
This produces the following output:
{% raw %}
```
f= {{x[0][0][2] +x[0][0][1] +x[0][0][0],x[0][1][2] +x[0][1][1] +x[0][1][0],x[0][2][2] +x[0][2][1] +x[0][2][0]},{x[1][0][2] +x[1][0][1] +x[1][0][0],x[1][1][2] +x[1][1][1] +x[1][1][0],x[1][2][2] +x[1][2][1] +x[1][2][0]}}
f[0][0] = x[0][0][2] +x[0][0][1] +x[0][0][0]
f[0][1] = x[0][1][2] +x[0][1][1] +x[0][1][0]
f[0][2] = x[0][2][2] +x[0][2][1] +x[0][2][0]
f[1][0] = x[1][0][2] +x[1][0][1] +x[1][0][0]
f[1][1] = x[1][1][2] +x[1][1][1] +x[1][1][0]
f[1][2] = x[1][2][2] +x[1][2][1] +x[1][2][0]
```
{% endraw %}
Note that this output is the same as when the axis is omitted.
Similar to NumPy, axis `-1` refers to the innermost (last) axis.
Thus, for the 3-dimensional array `x`, the following calls produce the same result:
```cpp
  auto f = qbpp::vector_sum(x);
  auto f = qbpp::vector_sum(x, 2);
  auto f = qbpp::vector_sum(x, -1);
```

## Application to the one-hot matrix
Using sum functions, we can create an expression for a `one-hot` matrix very easily.
Let $X=(x_{i,j})$ $(0\leq i,j\leq N-1)$ denote an $N\times N$ matrix of binary variables.
$X$ is a one-hot matrix if every row and every column contains exactly one 1. 
More specifically, the following constraints must be satisfied:

$$
\begin{aligned}
\text{row-wise}&&\sum_{j=0}^{N-1}x_{i,j} &= 1 && (0\leq i\leq N-1)\\
\text{column-wise}&&\sum_{i=0}^{N-1}x_{i,j} &= 1 && (0\leq j\leq N-1)\\
\end{aligned}
$$

The row-wise and column-wise sums can be computed using `qbpp::vector_sum()` with axis 0 and axis 1, respectively.
Using this idea, the following QUBO++ program creates an expression `f` that attains its minimum value of `0` if and only if `x` (for `N = 3`) represents a one-hot matrix:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  const size_t N = 3;
  auto x = qbpp::var("x", N, N);
  auto f = qbpp::sum(qbpp::vector_sum(x, 0) == 1) +
           qbpp::sum(qbpp::vector_sum(x, 1) == 1);
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  std::cout << sols << std::endl;
}
```
In this program, `f` is constructed using sum functions.
The Exhaustive Solver is used to list all optimal solutions.
This program produces the following output:
{% raw %}
```
(0) 0:{{x[0][0],0},{x[0][1],0},{x[0][2],1},{x[1][0],0},{x[1][1],1},{x[1][2],0},{x[2][0],1},{x[2][1],0},{x[2][2],0}}
(1) 0:{{x[0][0],0},{x[0][1],0},{x[0][2],1},{x[1][0],1},{x[1][1],0},{x[1][2],0},{x[2][0],0},{x[2][1],1},{x[2][2],0}}
(2) 0:{{x[0][0],0},{x[0][1],1},{x[0][2],0},{x[1][0],0},{x[1][1],0},{x[1][2],1},{x[2][0],1},{x[2][1],0},{x[2][2],0}}
(3) 0:{{x[0][0],0},{x[0][1],1},{x[0][2],0},{x[1][0],1},{x[1][1],0},{x[1][2],0},{x[2][0],0},{x[2][1],0},{x[2][2],1}}
(4) 0:{{x[0][0],1},{x[0][1],0},{x[0][2],0},{x[1][0],0},{x[1][1],0},{x[1][2],1},{x[2][0],0},{x[2][1],1},{x[2][2],0}}
(5) 0:{{x[0][0],1},{x[0][1],0},{x[0][2],0},{x[1][0],0},{x[1][1],1},{x[1][2],0},{x[2][0],0},{x[2][1],0},{x[2][2],1}}
```
{% endraw %}
The output confirms that all six optimal solutions are obtained.

