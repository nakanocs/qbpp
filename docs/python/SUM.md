---
layout: default
title: "Sum Functions"
nav_order: 18
parent: "PyQBPP (Python)"
---
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
