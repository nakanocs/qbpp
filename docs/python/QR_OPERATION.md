---
layout: default
title: "PyQUBO++ Quick Reference: Operators and Functions"
---

# Quick Reference: Operators and Functions for Expressions (PyQUBO++)

The table below summarizes the operators and functions available for `pyqbpp.Expr` objects.

| Operators/Functions           | Operator Symbols/Function Names                      | Function Type | Return Type       | Argument Type            |
|-------------------------------|------------------------------------------------------|---------------|-------------------|--------------------------|
| Assignment                    | `=`                                                  | —             | `pyqbpp.Expr`     | `ExprType`               |
| Binary Operators              | `+`, `-`, `*`                                        | Global        | `pyqbpp.Expr`     | `ExprType`-`ExprType`    |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member        | `pyqbpp.Expr`     | `ExprType` or `int`      |
| Division                      | `/`                                                  | Global        | `pyqbpp.Expr`     | `ExprType`-`int`         |
| Compound Division             | `/=`                                                 | Member        | `pyqbpp.Expr`     | `int`                    |
| Unary Operators               | `+`, `-`                                             | Global        | `pyqbpp.Expr`     | `ExprType`               |
| Comparison (Equality)         | `==`                                                 | Member        | `pyqbpp.ExprExpr` | `ExprType`-`int`         |
| Comparison (Range)            | `between()`                                          | Global        | `pyqbpp.ExprExpr` | `ExprType`-`int`-`int`   |
| Square                        | `sqr()`                                              | Global        | `pyqbpp.Expr`     | `ExprType`               |
| Type Conversion               | `toExpr()`                                           | Global        | `pyqbpp.Expr`     | `ExprType`               |
| Type Conversion               | `toInt()`                                            | Global        | `int`             | `pyqbpp.Expr`            |
| GCD                           | `gcd()`                                              | Global        | `int`             | `ExprType`               |
| Simplify                      | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | Global/Member | `pyqbpp.Expr`     | `ExprType`               |
| Eval                          | `f(ml)`                                              | Member        | `int`             | `pyqbpp.Expr`-`MapList`  |
| Replace                       | `replace()`                                          | Global/Member | `pyqbpp.Expr`     | `ExprType`-`MapList`     |
| Reduce                        | `reduce()`                                           | Global/Member | `pyqbpp.Expr`     | `ExprType`               |
| Binary/Spin Conversion        | `binary_to_spin()`, `spin_to_binary()`               | Global/Member | `pyqbpp.Expr`     | `ExprType`               |

## Expression-related type: **`ExprType`**
The term **`ExprType`** denotes a category of types that can be converted to a `pyqbpp.Expr` object.
In PyQUBO++, this includes:
- `int` — integer constant
- `pyqbpp.Var` — binary variable
- `pyqbpp.Term` — polynomial term
- `pyqbpp.Expr` — expression

## Global and Member Functions
Operators and functions related to `pyqbpp.Expr` are provided in two forms:
- **Global functions**:
These take at least one `ExprType` argument and typically return a new `pyqbpp.Expr` object without modifying the inputs.
- **Member functions**:
These are methods of the `pyqbpp.Expr` class.
In many cases, they update the calling object in place and also return the resulting `pyqbpp.Expr`.

### Example: `sqr()`
The `sqr()` function computes the square of an expression:
- `sqr(f)` (global): returns the square of `f` without modifying `f`

## Type Conversion: **`toExpr()`** and **`toInt()`**
The global function **`pyqbpp.toExpr()`** converts its argument into a `pyqbpp.Expr` instance and returns it.
The argument may be:
- an integer
- a variable (`pyqbpp.Var`)
- a product term (`pyqbpp.Term`)
- an expression (`pyqbpp.Expr`) — in this case, no conversion is performed

The global function **`pyqbpp.toInt()`** extracts and returns the integer constant term of a `pyqbpp.Expr` object.
If the expression contains any product terms, an error is thrown.

### Example
```python
from pyqbpp import toExpr, toInt, Expr

e = toExpr(5)       # Expr with constant 5
n = toInt(Expr(42)) # 42
```

## Assignment
In Python, the `=` operator rebinds the variable name to a new object.
To copy an expression, use the `Expr` constructor:
```python
f = Expr(g)  # f is a copy of g
```

## Binary Operators: `+`, `-`, `*`
These operators take two `ExprType` operands, compute the result, and return it.
If at least one operand is a `pyqbpp.Expr`, the result is always a `pyqbpp.Expr`.
If neither operand is a `pyqbpp.Expr`, the result may be a `pyqbpp.Term`.

### Example
For a variable `x` of type `pyqbpp.Var`:
- `2 + x`: `pyqbpp.Expr`
- `2 * x`: `pyqbpp.Term`

## Compound Assignment Operators: `+=`, `-=`, `*=`
The left-hand side must be a `pyqbpp.Expr`.
The specified operation is applied using the right-hand side operand.
The left-hand side expression is updated in place.

> **NOTE**
> `*=` only accepts `int` operands in PyQUBO++.

## Division `/` and Compound Division `/=`
The division operator `/` takes a `pyqbpp.Expr` as the **dividend** and an integer as the **divisor**, and returns the **quotient** as a new `pyqbpp.Expr`.

The dividend expression must be divisible by the divisor; that is,
both the integer constant term and all integer coefficients in the expression must be divisible by the divisor.

The compound division operator `/=` divides the expression in place.

### Example
```python
from pyqbpp import var, Expr

x = var("x")
y = var("y")
f = 6 * x + 4 * y + 2
g = f / 2          # g = 3*x + 2*y + 1
f = Expr(f)
f /= 2             # f = 3*x + 2*y + 1
```

## Comparison (Equality): `==`
The equality comparison operator `==` takes:
- a `pyqbpp.Expr` (or `ExprType` that creates one) on the left-hand side, and
- an integer on the right-hand side.

It returns an expression whose minimum value is 0 when the equality constraint is satisfied.
More specifically, for a `pyqbpp.Expr` object `f` and an integer `n`, the operator returns: `sqr(f - n)`.

For the returned object `g`:
- **`g`** represents the constraint expression `sqr(f - n)`, and
- **`g.body`** returns the underlying expression `f`.

### `pyqbpp.ExprExpr` class
Here, `g` is a **`pyqbpp.ExprExpr`** object, which is a derived class of `pyqbpp.Expr`.
The `body` property returns the associated underlying `pyqbpp.Expr` object.

### Comparison with C++ QUBO++
In C++ QUBO++, `*g` (dereference operator) is used to access the underlying expression.
In PyQUBO++, `g.body` property is used instead.

## Comparison (Range): `between()`
In C++ QUBO++, the range comparison is written as `l <= f <= u`.
In PyQUBO++, the `between()` function is used instead:
```python
g = between(f, l, u)
```
where:
- `f` is a non-integer `ExprType`, and
- `l` and `u` are integers.

This function returns an expression whose minimum value is 0 when the range constraint `l <= f <= u` is satisfied.

More specifically, an auxiliary integer variable `a` with unit gaps, taking values in the range `[l, u-1]`, is implicitly introduced, and the function returns:
```python
(f - a) * (f - (a + 1))
```

For the returned `pyqbpp.ExprExpr` object `g`:
- **`g`** represents the constraint expression `(f - a) * (f - (a + 1))`, and
- **`g.body`** returns the underlying expression `f`.

### Comparison with C++ QUBO++

| C++ QUBO++       | PyQUBO++            |
|------------------|---------------------|
| `l <= f <= u`    | `between(f, l, u)`  |
| `*g`             | `g.body`            |

## Square function: `sqr()`
For a `pyqbpp.Expr` object `f`:
- **`pyqbpp.sqr(f)`** (global function): Returns the expression `f * f`.
The argument `f` may be any `ExprType` object.

For a `pyqbpp.Vector` object `v`:
- **`pyqbpp.sqr(v)`**: Returns a new `pyqbpp.Vector` with each element squared.

### Example
```python
from pyqbpp import var, sqr

x = var("x")
f = sqr(x)       # x * x
```

## Greatest Common Divisor function: `gcd()`
The global function **`pyqbpp.gcd()`** takes a `pyqbpp.Expr` object as its argument and returns the greatest common divisor (GCD) of all integer coefficients and the integer constant term.

Since the given expression is divisible by the resulting GCD, all integer coefficients and the integer constant term can be reduced by dividing by the GCD.

### Example
```python
from pyqbpp import var, gcd

x = var("x")
y = var("y")
f = 6 * x + 4 * y + 2
print(gcd(f))    # 2
g = f / gcd(f)   # 3*x + 2*y + 1
```

## Simplify functions: `simplify()`, `simplify_as_binary()`, `simplify_as_spin()`
For a `pyqbpp.Expr` object `f`, the member function **`f.simplify()`** performs the following operations in place:
- Sort variables within each term according to their unique variable IDs
- Merge duplicated terms
- Sort terms such that:
  - lower-degree terms appear earlier, and
  - terms of the same degree are ordered lexicographically.

The global function **`pyqbpp.simplify(f)`** performs the same operations without modifying `f`.

### Binary and Spin Simplification
Two specialized variants of the simplification function are provided:
- **`simplify_as_binary()`**:
Simplification is performed under the assumption that all variables take binary values
$\lbrace 0,1\rbrace$.
The identity $x^2=x$ is applied to all variables $x$.
- **`simplify_as_spin()`**:
Simplification is performed under the assumption that all variables take spin values
$\lbrace -1,+1\rbrace$.
The identity $x^2=1$ is applied to all variables $x$.

Both variants are available as member functions and global functions:
- Member functions (in-place): `f.simplify_as_binary()`, `f.simplify_as_spin()`
- Global functions (non-destructive): `simplify_as_binary(f)`, `simplify_as_spin(f)`

### Example
```python
from pyqbpp import var, Expr, simplify_as_binary, simplify_as_spin

x = var("x")
f = Expr(x * x + x)
f.simplify_as_binary()  # 2*x (since x^2 = x)

g = Expr(x * x + x)
g.simplify_as_spin()    # 1 + x (since x^2 = 1)
```

## Evaluation function: `f(ml)`
A **`pyqbpp.MapList`** object stores a list of pairs consisting of a `pyqbpp.Var` object and an integer.
Each pair defines a mapping from a variable to an integer value.

For a `pyqbpp.Expr` object `f` and a `pyqbpp.MapList` object `ml`, the evaluation function `f(ml)` evaluates the value of `f` under the variable assignments specified by `ml` and returns the resulting integer value.

All variables appearing in `f` must have corresponding mappings defined in `ml`.

### Example
```python
from pyqbpp import var, MapList

x = var("x")
y = var("y")
f = 3 * x + 2 * y + 1

ml = MapList([(x, 1), (y, 0)])
print(f(ml))  # 4  (= 3*1 + 2*0 + 1)
```

### Comparison with C++ QUBO++

| C++ QUBO++       | PyQUBO++           |
|------------------|--------------------|
| `f(ml)`          | `f(ml)`            |

## Replace functions: `replace()`
A **`pyqbpp.MapList`** object may also contain pairs consisting of a `pyqbpp.Var` object and a `pyqbpp.Expr` object.
Such pairs define mappings from variables to expressions.

For a `pyqbpp.Expr` object `f` and a `pyqbpp.MapList` object `ml`:
- **`pyqbpp.replace(f, ml)`** (global function):
Returns a new `pyqbpp.Expr` object obtained by replacing variables in `f` according to the mappings in `ml`, without modifying `f`.
- **`f.replace(ml)`** (member function):
Replaces variables in `f` according to the mappings in `ml` in place and returns the resulting `pyqbpp.Expr` object.

### Creating a MapList
```python
from pyqbpp import MapList, Expr

ml = MapList()                      # Empty MapList
ml.add(x, 0)                       # Add mapping: x -> 0
ml.add(y, Expr(z))                  # Add mapping: y -> z

ml = MapList([(x, 0), (y, 1)])      # Create from list of pairs
```

### Example
```python
from pyqbpp import var, Expr, MapList, replace

x = var("x")
y = var("y")
f = 2 * x + 3 * y + 1

ml = MapList([(x, 1), (y, 0)])
g = replace(f, ml)   # g = 2*1 + 3*0 + 1 = 3 (new Expr)
f.replace(ml)         # f is modified in place
```

### Comparison with C++ QUBO++

| C++ QUBO++                    | PyQUBO++                          |
|-------------------------------|-----------------------------------|
| `qbpp::MapList ml;`           | `ml = MapList()`                  |
| `ml.push_back({x, 0});`      | `ml.add(x, 0)`                   |
| `qbpp::replace(f, ml)`       | `replace(f, ml)`                  |
| `f.replace(ml)`              | `f.replace(ml)`                   |

## Reduce function: `reduce()`
The **`reduce()`** function converts a `pyqbpp.Expr` object containing higher-degree terms into an equivalent `pyqbpp.Expr` object consisting only of linear and quadratic terms, resulting in a QUBO expression.

For a `pyqbpp.Expr` object `f`:
- **`pyqbpp.reduce(f)`** (global function):
Returns a new `pyqbpp.Expr` object with linear and quadratic terms that is equivalent to `f`.
- **`f.reduce()`** (member function):
Replaces `f` with the reduced expression in place and returns the updated expression.

### Example
```python
from pyqbpp import var, Expr, reduce

x = var("x")
y = var("y")
z = var("z")
f = Expr(x * y * z)
f.simplify_as_binary()
g = reduce(f)   # Reduced to linear and quadratic terms
```

## Binary/Spin Conversion functions: `spin_to_binary()`, `binary_to_spin()`
Let `x` be a binary variable and `s` be a spin variable.
We assume that `x = 1` if and only if `s = 1`.
Under this assumption, the following relations hold:

$$
\begin{aligned}
 s &= 2x-1 \\
 x &= (s+1)/2
\end{aligned}
$$

The **`spin_to_binary()`** function converts a spin-variable expression to a binary-variable expression
by replacing all spin variables `s` with `2 * s - 1`.

The **`binary_to_spin()`** function converts a binary-variable expression to a spin-variable expression
by replacing all binary variables `x` with `(x + 1) / 2`.
The resulting expression is multiplied by $2^d$ (where $d$ is the maximum degree) so that all coefficients remain integers.

Both functions are available as member functions (in-place) and global functions (non-destructive).

### Example
```python
from pyqbpp import var, Expr, spin_to_binary, binary_to_spin

s = var("s")
f = 3 * s + 1
g = spin_to_binary(f)   # -2 + 6*s  (replaced s with 2*s-1)

b = var("b")
h = 2 * b + 1
k = binary_to_spin(h)   # 2 + 2*b  (replaced b with (b+1)/2, multiplied by 2)
```

### Comparison with C++ QUBO++

| C++ QUBO++                      | PyQUBO++                       |
|---------------------------------|--------------------------------|
| `qbpp::spin_to_binary(f)`      | `spin_to_binary(f)`            |
| `f.spin_to_binary()`           | `f.spin_to_binary()`           |
| `qbpp::binary_to_spin(f)`      | `binary_to_spin(f)`            |
| `f.binary_to_spin()`           | `f.binary_to_spin()`           |
