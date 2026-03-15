---
layout: default
nav_exclude: true
title: "Sum Functions"
nav_order: 18
---
<div class="lang-en" markdown="1">
# Sum Functions for Multi-dimensional Arrays
QUBO++ provides two sum functions for multi-dimensional arrays of variables or expressions:
- **`qbpp::sum()`**: Computes the sum of all elements in the array.
- **`qbpp::vector_sum()`**: Computes the sum along the lowest (innermost) dimension.
The resulting array has one fewer dimension than the input array.
The input array must have a dimension of 2 or greater.

The following program demonstrates the difference between `qbpp::sum()` and `qbpp::vector_sum()`:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

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
</div>

<div class="lang-ja" markdown="1">
# 多次元配列の合計関数
QUBO++は、変数や式の多次元配列に対する2つの合計関数を提供しています:
- **`qbpp::sum()`**: 配列の全要素の合計を計算します。
- **`qbpp::vector_sum()`**: 最も内側（最低次元）の次元に沿った合計を計算します。
結果の配列は、入力配列より1つ少ない次元を持ちます。
入力配列の次元は2以上である必要があります。

以下のプログラムは、`qbpp::sum()` と `qbpp::vector_sum()` の違いを示しています:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

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
まず、サイズ $2 \times 3 \times 3$ の変数配列 `x` を定義します。
次に、`x` の全要素に1を加えた配列 `y` を作成し、`y` の全要素を出力します。
続いて、`qbpp::sum(y)` を計算して出力します。
その後、`qbpp::vector_sum()` 関数を `y` に適用し、結果をサイズ $2 \times 3$ の2次元式配列 `vector_sum` に格納します。
最後に、`vector_sum` の全要素を出力します。

このプログラムは以下の出力を生成します:
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
同じ結果は明示的なforループでも得ることができます。
ただし、大きな配列の場合は `qbpp::sum()` と `qbpp::vector_sum()` を使用することを推奨します。これらの関数は内部でマルチスレッドを活用して計算を高速化します。
</div>
