---
layout: default
title: "Quick Reference: Operators and Functions"
---

# Quick Reference: Operators and Functions for expressions

The table below summarizes the operators and functions available for `qbpp::Expr` objects.

| Operators/Functions           | Operator Symbols/Function Names                      | Function Type | Return Type    | Argument Type          |
|-------------------------------|------------------------------------------------------|---------------|----------------|------------------------|
| Type Conversion               | toExpr()                                             | Global        | qbpp::Expr     | ExprType               |
| Type Conversion               | toInt()                                              | Global        | Int            | Expr                   |
| Assignment                    | `=`                                                  | Member        | qbpp::Expr     | ExprType               |
| Binary Operators              | `+`, `-`, `*`                                        | Global        | qbpp::Expr     | ExprType-ExprType      |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member        | qbpp::Expr     | ExprType               |
| Division                      | `/`                                                  | Global        | qbpp::Expr     | ExprType-Int           |
| Compound Division             | `/=`                                                 | Member        | qbpp::Expr     | Int                    |
| Unary Operators               | `+`, `-`                                             | Global        | qbpp::Expr     | ExprType               |
| Comparison (Equality)         | `==`                                                 | Global        | qbpp::ExprExpr | ExprType-Int           |
| Comparison (Range Comparison) | `<= <=`                                              | Global        | qbpp::ExprExpr | IntInf-ExprType-IntInf |
| Square                        | sqr()                                                | Global        | qbpp::Expr     | ExprType               |
| Square                        | sqr()                                                | Member        | qbpp::Expr     | -                      |
| GCD                           | gcd()                                                | Global        | Int            | ExprType               |
| Simplify                      | simplify(), simplify_as_binary(), simplify_as_spin() | Global        | qbpp::Expr     | ExprType               |
| Simplify                      | simplify(), simplify_as_binary(), simplify_as_spin() | Member        | qbpp::Expr     | -                      |
| Eval                          | operator()                                           | Member        | Int            | ExprType-MapList       |
| Replace                       | replace()                                            | Global        | qbpp::Expr     | ExprType-MapList       |
| Replace                       | replace()                                            | Member        | qbpp::Expr     | MapList                |
| Reduce                        | reduce()                                             | Global        | qbpp::Expr     | ExprType               |
| Reduce                        | reduce()                                             | Member        | qbpp::Expr     | MapList                |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Global        | qbpp::Expr     | ExprType               |
| Binary/Spin Conversion        | binary_to_spin(), spin_to_binary()                   | Member        | qbpp::Expr     | -                      |

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
