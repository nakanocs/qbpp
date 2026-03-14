---
layout: default
title: "PyQBPP Reference Variables"
---

# Quick Reference: Variables and Expressions (PyQBPP)

## Data types in PyQBPP
PyQBPP uses Python's native `int` type for coefficients, energy values, and constants.
Since Python integers have unlimited precision, there is no need to specify `coeff_t` or `energy_t` as in the C++ version.

## Printing objects
All PyQBPP objects can be printed using `print()` or converted to strings using `str()`:
```python
print(obj)
s = str(obj)
```

## Variable classes
- **`pyqbpp.Var`**:
  A class that holds a unique 32-bit integer ID.
  The variable name can be retrieved via the `name` property.

> **NOTE**
> A `pyqbpp.Var` object represents a variable symbolically.
> No specific data type is associated with it.
> It can be used to represent binary, spin, or other types of variables.

### Variable creation functions
The following functions are provided to create variables:

- **`pyqbpp.var("name")`**:
  Creates a `pyqbpp.Var` object with the given name `"name"`.

- **`pyqbpp.var("name", s1)`**:
  Creates a one-dimensional array (vector) of `pyqbpp.Var` objects with the base name `"name"`.
  Each element is represented as `name[i]`.
  The resulting type is `pyqbpp.Vector`.

- **`pyqbpp.var("name", s1, s2)`**:
  Creates a two-dimensional array (matrix) of `pyqbpp.Var` objects with the base name `"name"`.
  Each element is represented as `name[i][j]`.
  The resulting type is a nested `pyqbpp.Vector`.

- **`pyqbpp.var("name", s1, s2, ...)`**:
  Creates a higher-dimensional array of `pyqbpp.Var` objects with the base name `"name"`.
  Each element is represented as `name[i][j]...`.
  The resulting type is a nested `pyqbpp.Vector`.

> **NOTE**
> If `"name"` is omitted, numbered names such as `"{0}"`, `"{1}"`, ... are automatically assigned in creation order.

### Examples
```python
from pyqbpp import var

x = var("x")          # Single variable named "x"
y = var("y", 3)       # Vector: y[0], y[1], y[2]
z = var("z", 2, 3)    # 2x3 matrix: z[0][0], ..., z[1][2]
a = var()             # Single unnamed variable
b = var(5)            # Vector of 5 unnamed variables
```

## `pyqbpp.Var` properties and methods
For a `pyqbpp.Var` instance `x`, the following are available:

- **`x.name`** (property):
  Returns the name of `x` as a string.

- **`x.index`** (property):
  Returns the unique integer ID of `x`.

Usually, there is no need to use these properties explicitly in PyQBPP programs.

## Integer variable class
- **`pyqbpp.VarInt`**:
  A class derived from `pyqbpp.Expr` that represents an integer variable with a specified range.

### Integer variable creation functions
The following functions are provided to create integer variables:

- **`pyqbpp.var_int("name")`**:
  Returns an internally used helper object (`pyqbpp.VarIntCore`) and does not create a `pyqbpp.VarInt` by itself.
  To define a `pyqbpp.VarInt`, the range must be specified using the `between()` function, as shown below.

- **`pyqbpp.between(pyqbpp.var_int("name"), l, u)`**:
  Here, `l` and `u` must be integers.
  This expression creates a `pyqbpp.VarInt` object with the name `"name"`,
  which internally contains a `pyqbpp.Expr` object representing all integers in the range `[l, u]`.
  Internally, this also creates `pyqbpp.Var` objects used in the underlying expression.

- **`pyqbpp.between(pyqbpp.var_int("name", s1), l, u)`**:
  Creates a one-dimensional array (vector) of `pyqbpp.VarInt` objects with the base name `"name"`
  and the same range `[l, u]`.
  Each element is represented as `name[i]`.
  The resulting type is `pyqbpp.Vector`.
  Higher-dimensional arrays of `pyqbpp.VarInt` objects can be created in the same way as `pyqbpp.Var` objects.

### Examples
```python
from pyqbpp import var_int, between

x = between(var_int("x"), 0, 10)       # Integer variable x in [0, 10]
y = between(var_int("y", 3), -5, 5)    # Vector of 3 integer variables in [-5, 5]
z = between(var_int("z", 2, 3), 1, 8)  # 2x3 matrix of integer variables in [1, 8]
```

### Integer variable properties
For a `pyqbpp.VarInt` instance `x`, the following are available:

- **`x.min_val`** (property):
  Returns the minimum value `l` of `x`.

- **`x.max_val`** (property):
  Returns the maximum value `u` of `x`.

- **`x.int_vars`** (property):
  Returns the list of `pyqbpp.Var` objects used to represent the integer variable.

- **`x.coeffs`** (property):
  Returns a list of integer coefficients.

The following expression is equivalent to the expression stored in `x`:
```python
x.min_val + sum(c * v for c, v in zip(x.coeffs, x.int_vars))
```

### Comparison with C++ QUBO++

| C++ QUBO++                                          | PyQBPP                                        |
|-----------------------------------------------------|-------------------------------------------------|
| `l <= qbpp::var_int("name") <= u`                   | `between(var_int("name"), l, u)`                |
| `l <= qbpp::var_int("name", s1) <= u`               | `between(var_int("name", s1), l, u)`            |
| `x.name()`                                          | `x.name` (property)                              |
| `x.str()`                                           | `str(x)`                                         |
| `x.min_val()`                                       | `x.min_val` (property)                           |
| `x.max_val()`                                       | `x.max_val` (property)                           |
| `x.vars()`                                          | `x.int_vars` (property)                          |
| `x.coeffs()`                                        | `x.coeffs` (property)                            |
