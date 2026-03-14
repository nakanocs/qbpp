---
layout: default
title: "PyQBPP (Python)"
nav_order: 50
---
# PyQBPP Document
This documentation for PyQBPP (Python binding of QUBO++) is currently under development.
Some pages may contain incomplete or provisional information.

## Basics
This section provides a step-by-step introduction to PyQBPP.
By reading the pages in order, you will learn how to define variables and expressions,
model optimization problems, and solve them using PyQBPP.
After completing this tutorial, you should be able to use PyQBPP for most typical applications.

1. [Defining Variables and Creating Expressions](VARIABLE)
2. [Solving Expressions](SOLVE)
3. [Vector of Variables and Vector Functions](VECTOR)
4. [Solving Partitioning Problem Using Vector of Variables](PARTITION)
5. [Permutation Matrix and Solving Assignment Problem](PERMUTATION)
6. [Integer Variables and Solving Simultaneous Equations](INTEGER)
7. [Factorization Through HUBO Expression](FACTORIZATION)
8. [Range Constraints and Solving Integer Linear Programming](RANGE)

## Topics
This section provides topic-wise explanations of selected features of PyQBPP.
Each page focuses on a specific topic and offers deeper insights into design decisions,
usage patterns, and, where appropriate, internal implementations.

1. [Data Types of Variables and Expressions](VAREXPR)
2. [Basic Operators and Functions](OPERATOR)
3. [Basic Operators and Functions for Vectors](OPVECTOR)
4. [Multi-dimensional Variables and Expressions](MULTIDIM)
5. [Comparison Operators](COMPARISON)
6. [Expression Classes](EXPRESSION)
7. [Evaluating Expressions](EVAL)
8. [Replace Functions](REPLACE)
9. [Sum Functions for Multi-dimensional Arrays](SUM)
10. [Easy Solver Usage](EASYSOLVER)
11. [Exhaustive Solver Usage](EXHAUSTIVE)
12. [ABS3 Solver Usage](ABS3)

> **NOTE**
> The Gurobi Solver is not available in the Python binding.
> For Gurobi usage, please refer to the [QUBO++ (C++) documentation](../GUROBI).

## Quick References
1. [Variables and Expressions](QR_VARIABLE)
2. [Operators and Functions](QR_OPERATION)
