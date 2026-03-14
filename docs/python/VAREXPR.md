---
layout: default
nav_exclude: true
title: "Data Types"
nav_order: 10
---
# Variable and Expression Classes

## Var, Term, and Expr classes

PyQBPP provides the following fundamental classes:
- **`Var`**: Represents a variable symbolically and is associated with a string used for display.
- **`Term`**: Represents a product term consisting of an integer coefficient and one or more `Var` objects.
- **`Expr`**: Represents an expanded expression consisting of an integer constant term and zero or more `Term` objects.

In the following program, **`x`** and **`y`** are `Var` objects, **`t`** is a `Term` object, and **`f`** is an `Expr` object:
```python
from pyqbpp import var

x = var("x")
y = var("y")
t = 2 * x * y
f = t - x + 1

print("x =", x)
print("y =", y)
print("t =", t)
print("f =", f)
```
This program produces the following output:
```
x = x
y = y
t = 2*x*y
f = 1 -x +2*x*y
```

`Var` objects are **immutable** and cannot be updated after creation.
In contrast, `Term` and `Expr` objects are **mutable** and can be updated via assignment.

For example, compound assignment operators can be used to update `Expr` objects:
```python
from pyqbpp import var, toExpr

x = var("x")
y = var("y")
f = toExpr(2 * x * y)
f += 3 * y

print("f =", f)
```
This program prints:
```
f = 2*x*y +3*y
```

## Differences from C++

In Python, there is no risk of accidentally creating a `Term` when you meant to create an `Expr`,
because Python's dynamic typing handles conversions automatically.
However, to explicitly create an `Expr` from an integer, a `Var`, or a `Term`, the **`toExpr()`** function is available:
```python
from pyqbpp import var, toExpr

x = var("x")
f = toExpr(0)    # Expr with constant 0
g = toExpr(x)    # Expr containing variable x
h = toExpr(3 * x)  # Expr from Term

f += 2 * x + 1
print("f =", f)
print("g =", g)
print("h =", h)
```
This program prints:
```
f = 1 +2*x
g = x
h = 3*x
```

## Coefficient and Energy Types
In PyQBPP, coefficients and energy values use **arbitrary-precision integers** by default.
Unlike the C++ version, there is no need to specify `COEFF_TYPE` or `ENERGY_TYPE` macros.

For example, the following program creates an expression with very large coefficients:
```python
from pyqbpp import var

x = var("x")
f = 123456789012345678901234567890 * x + 987654321098765432109876543210
print("f =", f)
```
This program produces the following output:
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```
