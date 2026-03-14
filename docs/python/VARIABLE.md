---
layout: default
nav_exclude: true
title: "Variables and Expressions"
nav_order: 2
---
# Defining Variables and Creating Expressions

## Installing PyQBPP
To use PyQBPP, install it via pip:
```bash
pip install pyqbpp
```

## Importing the library
To use PyQBPP, import the necessary functions from the **`pyqbpp`** module:
```python
from pyqbpp import var, expr, simplify, simplify_as_binary, simplify_as_spin
```

## Defining variables and expressions
You can define a variable using **`var("name")`**.
The specified `name` is used when the variable is printed.

Expressions are constructed using standard arithmetic operators such as **`+`**, **`-`**, and **`*`**.

The following program defines three variables `a`, `b`, and `c`, and an expression `f`, which is printed:
```python
from pyqbpp import var

a = var("a")
b = var("b")
c = var("c")
f = (a + b - 1) * (b + c - 1)
print("f =", f)
```
The expression `(a + b - 1) * (b + c - 1)` is automatically expanded and stored in `f`.

In this program, the variables `a`, `b`, and `c` are objects of class **`Var`**, and the expression `f` is an object of class **`Expr`**.

Running the program prints the expanded expression:
```
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

> **NOTE**
> The variable name in `var()` may be omitted.
> If omitted, a default name such as `{0}`, `{1}`,... is automatically assigned.

> **WARNING**
> The textual output of expressions is not guaranteed to be stable and should not be used as input for subsequent computations, since its format may change in future releases.

## Simplifying expression
The expression stored in an **`Expr`** object can be simplified by calling the **`simplify()`** member function:
```python
print("f =", f.simplify())
```
With this change, the output of the program becomes:
```
f = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
The member function call **`f.simplify()`** simplifies the expression `f` in place and returns itself.

Assuming that all variables take **binary values (0 or 1)**, we can use the identity
**b^2=b** to further simplify the expression.
For this purpose, we use **`simplify_as_binary()`** instead:
```python
print("f =", f.simplify_as_binary())
```
Then the output becomes:
```
f = 1 -a -b -c +a*b +a*c +b*c
```

## Simplifying expressions with spin variables
If variables are assumed to take **spin values -1/+1**, the identity **b^2 = 1** can be used to further simplify the expression.
In this case, the expression can be simplified using the **`simplify_as_spin()`** member function:
```python
print("f =", f.simplify_as_spin())
```
Then the output becomes:
```
f = 2 -a -2*b -c +a*b +a*c +b*c
```

## Global functions for simplification
Member functions update the expression in place.
If you do not want to modify `f`, you can instead use the global functions
**`simplify(f)`**, **`simplify_as_binary(f)`**, and **`simplify_as_spin(f)`**, which return the simplified expressions without changing `f`.

```python
from pyqbpp import simplify_as_binary
g = simplify_as_binary(f)  # f is not modified, g is a new simplified expression
```

> **NOTE**
> In PyQBPP, most **member functions** update the object in place when possible, whereas **global functions** return a new value without modifying the original object.
