---
layout: default
title: "Reference Variables"
---

# Quick Reference: Operators and Functions for qbpp::Expr

The table below summarizes the Operators and Functions for qbpp::Expr objects.

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

## Type Conversion: `qbpp::toExpr()` and `qbpp::toInt()`
`qbpp::toExpr()` global function converts an argment into `qbpp::Expr` instance and returns it.
The argument can be
- an integer
- a variable (qbpp::Var)
- a product term (qbpp::Term)
- a expression (qbpp::Expr) (no conversion performed)

`qbpp::toInt()` global function with an argument `qbpp::Expr()` picks its integer constant term and returns it as an intger.
It throws error if it has product terms (`qbpp::Term` objects).


We term these classes `ExprType` that can be an argument of `qbpp::toExpr()`

## `ExprType`
`ExprType` denotes a category of classes that can be converted to a `qbpp::Expr` object.
This category includes:
- integers
- variables (qbpp::Var)
- terms (qbpp::Term)
- expressions (qbpp::Expr)

## `Int` and `IntInf`
- `Int`: integers
- `IntInf`: either an integer, `-qbpp::inf`, or `+qbpp::inf`, which signify infinite values.

## Global and member functions:
Operators and functions for `qbpp::Expr` have following types:
- **Global functions**: Take at least one `ExprType` object as an argument. In many cases, they return a `qbpp::Expr` object.
- **Member functions**: Defined as member functions of the `qbpp::Expr` class. In many cases, they update the calling object based on the result of the function.

For example, the `sqr()` function, which computes the square of a qbpp::Expr object, is available as both a global and a member function.
The global version, `sqr(f)`, returns the square of `f` without updating `f`.
In contrast, the member function `f.sqr()` returns the square of `f` and updates `f` with this value.

The following table summarizes the available operators and functions:

## Assignment: Operator `=`
The lefthand side must be an `qbpp::Expr` object and the righthand side must be `ExprType`, which is converted to `qbpp::Expr` object.
The converted object is and assigned to the lefthand side.



