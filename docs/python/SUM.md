---
layout: default
nav_exclude: true
title: "Sum Functions"
nav_order: 18
---
<div class="lang-en" markdown="1">
# Sum Functions for Multi-dimensional Arrays
PyQBPP provides two sum functions for multi-dimensional arrays of variables or expressions:
- **`sum()`**: Computes the sum of all elements in the array.
- **`vector_sum()`**: Computes the sum along the lowest (innermost) dimension.
The resulting array has one fewer dimension than the input array.

The following program demonstrates the difference between `sum()` and `vector_sum()`:
```python
from pyqbpp import var, sum, vector_sum

x = var("x", 2, 3, 3)
y = x + 1
for i in range(2):
    for j in range(3):
        for k in range(3):
            print(f"y[{i}][{j}][{k}] =", y[i][j][k])

s = sum(y)
s.simplify()
print("sum(y) =", s)

vs = vector_sum(y)
vs.simplify()
for i in range(2):
    for j in range(3):
        print(f"vector_sum[{i}][{j}] =", vs[i][j])
```
First, an array `x` of variables with size $2 \times 3 \times 3$ is defined.
Next, an array `y` is created by adding 1 to every element of `x`.
Then, `sum(y)` computes the sum of all 18 elements.
After that, `vector_sum(y)` computes the sum along the innermost dimension, producing a $2 \times 3$ array.

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
However, for large arrays, it is recommended to use `sum()` and `vector_sum()`, since these functions internally exploit multithreading to accelerate computation.
</div>

<div class="lang-ja" markdown="1">
# 多次元配列の総和関数
PyQBPPは、変数や式の多次元配列に対する2つの総和関数を提供しています：
- **`sum()`**: 配列のすべての要素の総和を計算します。
- **`vector_sum()`**: 最も内側の次元に沿って総和を計算します。
結果の配列は入力配列より1次元少なくなります。

以下のプログラムは `sum()` と `vector_sum()` の違いを示しています：
```python
from pyqbpp import var, sum, vector_sum

x = var("x", 2, 3, 3)
y = x + 1
for i in range(2):
    for j in range(3):
        for k in range(3):
            print(f"y[{i}][{j}][{k}] =", y[i][j][k])

s = sum(y)
s.simplify()
print("sum(y) =", s)

vs = vector_sum(y)
vs.simplify()
for i in range(2):
    for j in range(3):
        print(f"vector_sum[{i}][{j}] =", vs[i][j])
```
まず、サイズ $2 \times 3 \times 3$ の変数配列 `x` を定義します。
次に、`x` のすべての要素に1を加えて配列 `y` を作成します。
そして、`sum(y)` で全18要素の総和を計算します。
その後、`vector_sum(y)` で最も内側の次元に沿って総和を計算し、$2 \times 3$ の配列を生成します。

このプログラムの出力は以下の通りです：
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
明示的なforループを使っても同じ結果が得られます。
しかし、大きな配列では `sum()` と `vector_sum()` の使用を推奨します。これらの関数は内部的にマルチスレッドを活用して計算を高速化するためです。
</div>
