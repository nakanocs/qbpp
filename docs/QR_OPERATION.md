---
layout: default
title: "Reference Variables"
---

# Quick Reference: Operators and Functions for qbpp::Expr

The table below summarizes the Operators and Functions for qbpp::Expr objects

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

## `qbpp::toExpr()`
This global function converts an argment into `qbpp::Expr` instance and returns it.
The argument can be
- an integer
- a variable (qbpp::Var)
- a product term (qbpp::Term)
- a expression (qbpp::Expr) (no conversion performed)

We term these classes `ExprType` that can be an argument of `qbpp::toExpr()`



## `ExprType`
Here, ExprType is a category of classes that can be converted to




Operators and functions for `qbpp::Expr` have following types:
- **Global functions**: Take at least one `ExprType` object as an argument. In many cases, they return a `qbpp::Expr` object.
- **Member functions**: Defined as member functions of the `qbpp::Expr` class. In many cases, they update the calling object based on the result of the function.

For example, the `sqr()` function, which computes the square of a qbpp::Expr object, is available as both a global and a member function.
The global version, `sqr(f)`, returns the square of `f` without updating `f`.
In contrast, the member function `f.sqr()` returns the square of `f` and updates `f` with this value.

The following table summarizes the available operators and functions:


In this table, **Int** denotes an integer, while **IntInf** represents either an integer, `-qbpp::inf`, or `+qbpp::inf`, which signify infinite values.

