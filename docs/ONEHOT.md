---
layout: default
title: "One-hot function"
---


# One-hot function qbpp::onehot_to_int()

## A one-hot vector representing an integer

A length-$n$ binary vector is called one-hot if it contains exactly one 1 and all other elements are 0.
The position of the 1 represents an integer from $0$ to $n−1$.

Many QUBO/HUBO formulations for combinatorial optimization use one-hot vectors to represent integer-valued decisions.
Therefore, QUBO++ provides the function `qbpp::onehot_to_int()`, which converts a one-hot vector into the corresponding integer.

## Converting a one-hot vector into an integer
The following QUBO++ program demonstrates qbpp::onehot_to_int() for vectors:
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Vector<int> onehot = {0, 0, 1, 0};
  qbpp::Vector<int> nononehot = {0, 1, 1, 0};

  std::cout << "onehot: " << onehot
            << " integer: " << qbpp::onehot_to_int(onehot) << std::endl;

  std::cout << "nononehot: " << nononehot
            << " integer: " << qbpp::onehot_to_int(nononehot) << std::endl;
}
```
This program produces the following output:
```
onehot: {0,0,1,0} integer: 2
nononehot: {0,1,1,0} integer: -1
```
For the vector `{0,0,1,0}`, `qbpp::onehot_to_int()` returns `2`, which is the index of the `1`.
For the vector `{0,1,1,0}`, it returns `-1` because the input is not one-hot.

## One-hot conversion for a matrix
`qbpp::onehot_to_int()` also works for a matrix of binary values.
The second argument specifies the axis, and the function is applied to each vector (1D slice) along that axis.

For a matrix `x`:
- `qbpp::onehot_to_int(x, 0)` applies the conversion column-wise (i.e., to each column), and
- `qbpp::onehot_to_int(x, 1)` applies the conversion row-wise (i.e., to each row).

The following program demonstrates these behaviors for a 
$4\times 4$ matrix `x`:
```cpp
#include "qbpp.hpp"

int main() {
  qbpp::Vector<qbpp::Vector<int>> x = {
      {0, 0, 1, 0},
      {1, 0, 0, 0},
      {0, 0, 0, 1},
      {0, 1, 0, 0}
  };

  for (size_t i = 0; i < 4; ++i) {
    for (size_t j = 0; j < 4; ++j) {
      std::cout << " " << x[i][j];
    }
    std::cout << std::endl;
  }

  std::cout << "rows: " << qbpp::onehot_to_int(x, 1) << std::endl;
  std::cout << "cols: " << qbpp::onehot_to_int(x, 0) << std::endl;
}
```
This program produces the following output:
```
 0 0 1 0
 1 0 0 0
 0 0 0 1
 0 1 0 0
rows: {2,0,3,1}
cols: {1,3,0,2}
```

`qbpp::onehot_to_int()` can also be applied to higher-dimensional arrays.
More specifically, `qbpp::onehot_to_int(x, k)` computes the integer values for all one-hot vectors taken along axis `k` of a binary array `x`.