---
layout: default
title: "Quick Reference: Operators and Functions"
---

# Quick Reference: Operators and Functions for Expressions

The table below summarizes the operators and functions available for `qbpp::Expr` objects.

| Operators/Functions           | Operator Symbols/Function Names                      | Function Type | Return Type    | Argument Type          |
|-------------------------------|------------------------------------------------------|---------------|----------------|------------------------|
| Type Conversion               | `toExpr()`                                             | Global        | `qbpp::Expr`     | `ExprType`               |
| Type Conversion               | `toInt()`                                              | Global        | `Int`            | `qbpp::Expr`                   |
| Assignment                    | `=`                                                  | Member        | `qbpp::Expr`     | `ExprType`               |
| Binary Operators              | `+`, `-`, `*`                                        | Global        | `qbpp::Expr`     | `ExprType`-`ExprType`      |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member        | `qbpp::Expr`     | `ExprType`               |
| Division                      | `/`                                                  | Global        | `qbpp::Expr`     | `ExprType`-`Int`           |
| Compound Division             | `/=`                                                 | Member        | `qbpp::Expr`     | `Int`                    |
| Unary Operators               | `+`, `-`                                             | Global        | `qbpp::Expr`     | `ExprType`               |
| Comparison (Equality)         | `==`                                                 | Global        | `qbpp::ExprExpr` | `ExprType`-`Int`           |
| Comparison (Range Comparison) | `<= <=`                                              | Global        | `qbpp::ExprExpr` | `IntInf`-`ExprType`-`IntInf` |
| Square                        | `sqr()`                                                | Global        | `qbpp::Expr`     | `ExprType`               |
| Square                        | `sqr()`                                                | Member        | `qbpp::Expr`     | -                      |
| GCD                           | `gcd()`                                                | Global        | `Int`            | `ExprType`               |
| Simplify                      | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | Global        | `qbpp::Expr`     | `ExprType`               |
| Simplify                      | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | Member        | `qbpp::Expr`     | -                      |
| Eval                          | `operator()`                                           | Member        | `Int`            | `ExprType`-`qbpp::MapList`       |
| Replace                       | `replace()`                                            | Global        | `qbpp::Expr`     | `ExprType`-`qbpp::MapList`       |
| Replace                       | `replace()`                                            | Member        | `qbpp::Expr`     | `qbpp::MapList`                |
| Reduce                        | `reduce()`                                             | Global        | `qbpp::Expr`     | `ExprType`               |
| Reduce                        | `reduce()`                                             | Member        | `qbpp::Expr`     | `qbpp::MapList`                |
| Binary/Spin Conversion        | `binary_to_spin()`, `spin_to_binary()`                   | Global        | `qbpp::Expr`     | `ExprType`               |
| Binary/Spin Conversion        | `binary_to_spin()`, `spin_to_binary()`                   | Member        | `qbpp::Expr`     | -                      |

## Type Conversion: **`qbpp::toExpr()`** and **`qbpp::toInt()`**
The global function **`qbpp::toExpr()`** converts its argument into a `qbpp::Expr` instance and returns it.
The argument may be:
- an integer
- a variable (`qbpp::Var`)
- a product term (`qbpp::Term`)
- an expression (`qbpp::Expr`) — in this case, no conversion is performed

We refer to these argument types collectively as `ExprType`.

The global function **`qbpp::toInt()`** extracts and returns the integer constant term of a `qbpp::Expr` object.
If the expression contains any product terms (`qbpp::Term` objects), an error is thrown.


## Expression-related type: **`ExprType`**
The term **`ExprType`** denotes a category of types that can be converted to a `qbpp::Expr` object.

## Integer-Related Types: **`Int`** and **`IntInf`**
- **`Int`**: ordinary integers
- **`IntInf`**: either an integer, `-qbpp::inf`, or `+qbpp::inf`, representing infinite bounds.

## Global and Member Functions
Operators and functions related to `qbpp::Expr` are provided in two forms:
- **Global functions**:
These take at least one ExprType argument and typically return a new `qbpp::Expr` object without modifying the inputs.
- **Member functions**:
These are member functions of the `qbpp::Expr` class.
In many cases, they update the calling object and also return the resulting `qbpp::Expr`.

### Example: `sqr()`
The `sqr()` function computes the square of an expression and is available in both forms:
- `sqr(f)` (global): returns the square of f without modifying f
- `f.sqr()` (member): updates f to its square and returns the updated expression

## Assignment Operator: `=`
The left-hand side must be a `qbpp::Expr` object.
The right-hand side must be an `ExprType`, which is first converted to a `qbpp::Expr`.
The converted expression is then assigned to the left-hand side.

## Binary Operators: `+`, `-`, `*`
These operators are defined as global functions.
They take two `ExprType` operands, compute the result, and return it.
If at least one operand is a `qbpp::Expr`, the result is always a `qbpp::Expr`.
If neither operand is a `qbpp::Expr`, the result may be a `qbpp::Term`.

### Example
For a variable `x` of type `qbpp::Var`:
- `2 + x`: `qbpp::Expr`
- `2 * x`: `qbpp::Term`

## Compound Assignment Operators: `+=`, `-=`, `*=`
These operators are defined as member functions.
The left-hand side must be a `qbpp::Expr`.
The specified operation is applied using the right-hand side operand.
The left-hand side expression is updated in place.

## Division `/` and Compound Division `/=`
The division operator `/` is defined as a global function.

It takes a non-integer `ExprType` operand as the **dividend** and an integer operand as the **divisor**, and returns the **quotient** as a `qbpp::Expr`.

The dividend expression must be divisible by the divisor; that is,
both the integer constant term and all integer coefficients in the expression must be divisible by the divisor.

The compound division operator `/=` is defined as a member function.
- The left-hand side must be a `qbpp::Expr`.
- The right-hand side must be an integer.

The same divisibility condition applies, and the division is performed in place, updating the left-hand side expression.

## Comparison (Equality): `==`
The equality comparison operator `==` takes:
- a non-integer `ExprType` on the left-hand side, and
- an integer on the right-hand side.

It returns an expression whose minimum value is 0 when the equality constraint is satisfied.
More specifically, for a non-integer `ExprType` object `f` and an integer `n`, the operator returns: `qbpp::sqr(f-n)`.

For the returned object `g`:
- **`g`** represents the constraint expression `qbpp::sqr(f - n)`, and
- **`*g`** returns the underlying expression `f`.

### `qbpp::ExprExpr` class

Here, `g` is a **`qbpp::ExprExpr`** object, which is a derived class of `qbpp::Expr`.
Dereferencing `g` using the `*` operator returns the associated underlying qbpp::Expr object.

## Comparison (Range Comparison): `<= <=`
The **range comparison operator** is written in the form:
```cpp
l <= f <= u
```
where:
- `f` is a non-integer ExprType, and
- `l` and `u` are integers.

This operator returns an expression whose minimum value is 0 when the range constraint is satisfied.

More specifically, an auxiliary integer variable `a` with unit gaps, taking values in the range `[l,u−1]`, is implicitly introduced, and the operator returns:
```cpp
(f - a)(f - (a + 1))
```

For the returned `qbpp::ExprExpr` object `g`:
- **`g`** represents the constraint expression `(f - a)(f - (a + 1))`, and
- **`*g`** returns the underlying expression `f`.

## Square functions: `sqr()`
For a qbpp::Expr object `f`:
- **`qbpp::sqr(f)`** (global function): Returns the expression `f * f`.
The argument `f` may be a non-integer `ExprType` object.
- **`f.sqr()`** (member function):
Updates `f` in place by replacing it with `f * f`, and returns the updated expression.

## Greatest Common Divisor function `gcd()`
The global function **`gcd()`** takes a `qbpp::Expr` object as its argument and returns the greatest common divisor (GCD) of all integer coefficients and the integer constant term.

Since the given `qbpp::Expr` object is divisible by the resulting GCD, all integer coefficients and the integer constant term can be reduced by dividing by the GCD without changing the structure of the expression or its optimal solutions.


## Simplify functions: `simplify()`, `simplify_as_binary()`, `simplify_as_spin()`
For a `qbpp::Expr` object `f`, the member function **`f.simplify()`** performs the following operations in place:
- Sort variables within each term according to their unique variable IDs
- Merge duplicated terms
- Sort terms such that:
  - lower-degree terms appear earlier, and
  - terms of the same degree are ordered lexicographically.

The global function **`qbpp::simplify(f)`** performs the same operations without modifying `f`.

### Binary and Spin Simplification
Two specialized variants of the simplification function are provided:
- **`simplify_as_binary()`**:
Simplification is performed under the assumption that all variables take binary values 
$\lbrace 0,1\rbrace$.
The identity $x^2=x$ is applied to all variables $x$.
- **`simplify_as_spin()`**
Simplification is performed under the assumption that all variables take spin values 
$\lbrace -1,+1\rbrace$.
The identity $x^2=1$ is applied to all variables $x$.

Both variants are available as member functions and global functions:
- Member functions: These perform simplification in place and update `f`.
  - `f.simplify_as_binary()`
  - `f.simplify_as_spin()`
- Global functions: These return a simplified expression without modifying f.
  - `qbpp::simplify_as_binary(f)`
  - `qbpp::simplify_as_spin(f)`

## Evaluation function
A **`qbpp::MapList`** object stores a list of pairs consisting of a `qbpp::Var` object and an integer.
Each pair defines a mapping from a variable to an integer value.

For a `qbpp::Expr` object `f` and a `qbpp::MapList` object `ml`, the evaluation function
```cpp
f(ml)
```
evaluates the value of `f` under the variable assignments specified by `ml` and returns the resulting integer value.

All variables appearing in `f` must have corresponding mappings defined in `ml`.

## Replace Functions: `replace()`
A **`qbpp::MapList`** object may also contain pairs consisting of a `qbpp::Var` object and an `ExprType` object.
Such pairs define mappings from variables to expressions.

For a `qbpp::Expr` object `f` and a `qbpp::MapList` object `ml`:
- **`qbpp::replace(f, ml)`**:
Returns a new `qbpp::Expr` object obtained by replacing variables in `f` according to the mappings in `ml`, without modifying `f`.
- **`f.replace(ml)`**:
Replaces variables in `f` according to the mappings in `ml` in place and returns the resulting `qbpp::Expr` object.

## Reduce Functions: `reduce()`
The **`reduce()`** function converts a `qbpp::Expr` object containing higher-degree terms into an equivalent `qbpp::Expr` object consisting only of linear and quadratic terms, resulting in a QUBO expression.

For a `qbpp::Expr` object `f`:
- **`qbpp::reduce(f)`**:
Returns a new `qbpp::Expr` object with linear and quadratic terms that is equivalent to `f`.
- **`f.reduce()`**:
Replaces `f` with the result of `qbpp::reduce(f)` and returns the updated expression.

## Binar_y/Spin Conversion functions: `spin_to_binary()`, `binary_to_spin()`
Let `x` be a binary variable and s be a spin variable.
We assume that `x = 1` if and only if `s = 1`.
Under this assumption, the following relations hold:

$$
\begin{aligned}
 s &= 2x-1 \\
 x &= (s+1)/2 
\end{aligned}
$$

Let $f(s)$ be a function of a spin variable $s$.
Then the function $g(x)=f(2x−1)$ is a function of the binary variable $x$ that yields the same value under the above relation.

The **`spin_to_binary()`** function uses this relation to convert a `qbpp::Expr` object representing a function of spin variables into an equivalent `qbpp::Expr` object representing a function of binary variables.
More specifically, it replaces all spin variables `s` in `f` by `2 * s - 1`.
- **`qbpp::spin_to_binary(f)`**:
Produces and returns a new `qbpp::Expr` object by replacing all spin variables `s` in `f` with `2 * s - 1`.
- **`f.spin_to_binary()`**:
Updates `f` in place using `qbpp::spin_to_binary(f)` and returns the updated expression.

Similarly, the **`binary_to_spin()`** function replaces all binary variables `x` in `f` by `(x + 1) / 2`.
The resulting expression may contain non-integer coefficients.
Therefore, the entire expression is multiplied by 
$2^d$ where $d$ is the maximum degree of all terms, so that all coefficients become integers.

As with `spin_to_binary()`, both global and member function variants of `binary_to_spin()` are provided.

