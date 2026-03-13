---
section: python
layout: default
title: "Expression Classes (Python)"
---

# Expression Classes
The most important feature of PyQUBO++ is its ability to create expressions for solving combinatorial optimization problems.
The following three classes are used for this purpose:

| Class | Contains | Details |
|------|-----|-----|
| `Var` | A variable  |  a 32-bit ID and a string to display |
| `Term` | A product term | Zero or more variables and an integer coefficient |
| `Expr` | An expression | Zero or more terms and an integer constant term |

## `Var` class
An instance of this class represents **a variable symbolically**.
Each `Var` instance consists of a unique ID and a string used for display.

```python
from pyqbpp import var

x = var("x")
print(x)
```
This simply prints `x`.

## `Term` class
An instance of this class represents **a product term** involving an integer coefficient and zero or more `Var` objects.

```python
from pyqbpp import var

x = var("x")
y = var("y")
t = 2 * x * y
print(t)
```
This program prints `2*x*y`.

## `Expr` class
An instance of this class represents **an expression** involving an integer constant term and zero or more `Term` objects.

```python
from pyqbpp import var

x = var("x")
y = var("y")
f = 3 + 2 * x * y + 3 * x
print(f)
```
This program prints `3 +2*x*y +3*x`.

Expressions can be written using basic operators such as **`+`**, **`-`**, and **`*`**, as well as parentheses.
Expressions are automatically expanded and stored as an `Expr` object:
```python
from pyqbpp import var

x = var("x")
y = var("y")
f = (x + y - 2) * (x - 2 * y + 3)
print("f =", f)
f.simplify()
print("simplified f =", f)
```
This program prints:
```
f = -6 +x*x +y*x -2*x*y -2*y*y +3*x +3*y -2*x +4*y
simplified f = -6 +x +7*y +x*x -x*y -2*y*y
```

## Using `toExpr()` for explicit construction
To explicitly create an `Expr` from an integer, a `Var`, or a `Term`, the **`toExpr()`** function is available.
This is useful when building up an expression incrementally:
```python
from pyqbpp import var, toExpr

x = var("x", 4)
f = toExpr(-1)
for i in range(len(x)):
    f += x[i]
print(f)
```
This program prints:
```
-1 +x[0] +x[1] +x[2] +x[3]
```
