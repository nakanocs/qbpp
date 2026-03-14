---
layout: default
title: "Multi-dimensional Variables"
nav_order: 13
parent: "PyQBPP"
---
# Multi-dimensional Variables and Expressions

## Defining multi-dimensional variables
PyQBPP supports **multi-dimensional variables** and **multi-dimensional integer variables** of arbitrary depth using the functions `var()` and `var_int()`, respectively.
Their basic usage is as follows:
- `var("name", s1, s2, ..., sd)`: Creates an array of `Var` objects with the given `name` and shape $s_1\times s_2\times \cdots\times s_d$.
- `between(var_int("name", s1, s2, ..., sd), l, u)`: Creates an array of `VarInt` objects with the specified range and shape.

The following program creates a binary variable with dimension $2\times 3\times 4$:
```python
from pyqbpp import var

x = var("x", 2, 3, 4)
print("x =", x)
```
{% raw %}
Each `Var` object in **`x`** can be accessed as **`x[i][j][k]`**.
{% endraw %}

## Defining multi-dimensional expressions
PyQBPP allows you to define **multi-dimensional expressions** with arbitrary depth using the function `expr()`:
- **`expr(s1, s2, ..., sd)`**: Creates a multi-dimensional array of `Expr` objects with shape $s_1\times s_2\times \cdots\times s_d$.

The following program defines a 3-dimensional array **`x`** of `Var` objects with shape $2\times 3\times 4$ and
a 2-dimensional array `f` of size $2\times 3$.
Then, using nested loops, each `f[i][j]` accumulates the sum of `x[i][j][0]` through `x[i][j][3]`:
```python
from pyqbpp import var, expr

x = var("x", 2, 3, 4)
f = expr(2, 3)
for i in range(2):
    for j in range(3):
        for k in range(4):
            f[i][j] += x[i][j][k]
f.simplify_as_binary()

for i in range(2):
    for j in range(3):
        print(f"f[{i}][{j}] =", f[i][j])
```
This program produces the following output:
```
f[0][0] = x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3]
f[0][1] = x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3]
f[0][2] = x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]
f[1][0] = x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3]
f[1][1] = x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3]
f[1][2] = x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]
```

## Creating an array of expressions by operations
An array of `Expr` objects can be created without explicitly calling `expr()`.
When an arithmetic operation yields an array-shaped result, an array of `Expr` objects with the same shape is created automatically.

```python
from pyqbpp import var

x = var("x", 2, 3)
f = x + 1
f += x - 2
f.simplify_as_binary()
for i in range(2):
    for j in range(3):
        print(f"f[{i}][{j}] =", f[i][j])
```
This program outputs:
```
f[0][0] = -1 +2*x[0][0]
f[0][1] = -1 +2*x[0][1]
f[0][2] = -1 +2*x[0][2]
f[1][0] = -1 +2*x[1][0]
f[1][1] = -1 +2*x[1][1]
f[1][2] = -1 +2*x[1][2]
```

## Iterating over multi-dimensional arrays
Since PyQBPP vectors support Python iteration, nested for loops can be used:
```python
from pyqbpp import var

x = var("x", 2, 3)
f = x + 1
f += x - 2
f.simplify_as_binary()
for row in f:
    for element in row:
        print(f"({element})", end="")
    print()
```
This program outputs:
```
(-1 +2*x[0][0])(-1 +2*x[0][1])(-1 +2*x[0][2])
(-1 +2*x[1][0])(-1 +2*x[1][1])(-1 +2*x[1][2])
```
