---
layout: default
title: "Operators and Functions"
nav_order: 11
parent: "PyQBPP"
---
# Basic Operators and Functions

## Unary and Binary Operators
PyQBPP supports the following basic operators for constructing expressions:
- **`+`**: Returns the sum of the operands.
- **`-`**: Returns the difference of the operands.
- **`*`**: Returns the product of the operands.
- **`/`**: Returns the quotient of the operands (integer division; all coefficients must be divisible).
- unary **`-`**: Returns the negation of the operand.

The following program demonstrates how to construct expressions using these operators:
```python
from pyqbpp import var

x = var("x")
y = var("y")
f = 6 * -(x + 1) * (y - 1)
g = f / 3

print("f =", f)
print("g =", g)
```
This program produces the following output:
```
f = 6 -6*x*y +6*x -6*y
g = 2 -2*x*y +2*x -2*y
```

## Compound operators
The following compound operators are supported for updating `Expr` objects:
- **`+=`**: Adds the right-hand side operand to the left-hand side.
- **`-=`**: Subtracts the right-hand side operand from the left-hand side.
- **`*=`**: Multiplies the left-hand side by the right-hand side.
- **`/=`**: Divides the left-hand side by the right-hand side (integer division).

The following program demonstrates these compound operators:
```python
from pyqbpp import var, toExpr

x = var("x")
y = var("y")
f = toExpr(6 * x + 4)

f += 3 * y
print("f =", f)

f -= 12
print("f =", f)

f *= 2 * y
print("f =", f)

f /= 2
print("f =", f)
```
This program produces the following output:
```
f = 4 +6*x +3*y
f = -8 +6*x +3*y
f = 12*x*y +6*y*y -16*y
f = 6*x*y +3*y*y -8*y
```

## Square function
PyQBPP provides both a global function **`sqr()`** and a member function **`sqr()`** of the `Expr` class to compute the square of an expression.

The global function **`sqr(f)`** returns a new `Expr` object representing the square of `f` without modifying `f`,
whereas the member function **`f.sqr()`** updates `f` in place.

```python
from pyqbpp import var, sqr

x = var("x")
f = x + 1

print("sqr(f) =", sqr(f))
print("f =", f)

f.sqr()
print("f =", f)
```
This program produces the following output:
```
sqr(f) = 1 +x*x +x +x
f = 1 +x
f = 1 +x*x +x +x
```

## Simplify functions
After operators or functions are applied, expressions are automatically expanded but not simplified.
To simplify the resulting expressions, simplify functions must be explicitly called.

PyQBPP provides the following three **global simplify functions**:
- **`simplify(f)`**: Returns a simplified expression by merging coefficients of identical terms.
- **`simplify_as_binary(f)`**: Returns a simplified expression under the assumption that all variables take binary values $0/1$ (i.e., $x^2=x$).
- **`simplify_as_spin(f)`**: Returns a simplified expression under the assumption that all variables take spin values $-1/+1$ (i.e., $x^2=1$).

**Member function** versions are also available. They update the object in place.

```python
from pyqbpp import var, sqr, simplify, simplify_as_binary, simplify_as_spin

x = var("x")
f = sqr(x - 1)

print("f =", f)
print("simplify(f) =", simplify(f))
print("simplify_as_binary(f) =", simplify_as_binary(f))
print("simplify_as_spin(f) =", simplify_as_spin(f))
```
This program produces the following output:
```
f = 1 +x*x -x -x
simplify(f) = 1 -2*x +x*x
simplify_as_binary(f) = 1 -x
simplify_as_spin(f) = 2 -2*x
```

> **NOTE**
> In PyQBPP, **member functions** update the object in place, whereas **global functions** return a new value without modifying the original object.
