---
layout: default
title: "HUBO and QUBO"
---

# HUBO and QUBO
A **High-Order Unconstrained Binary Optimization (HUBO)** problem is defined by a polynomial over binary variables.
The goal is to find an assignment of binary values ${0,1}$ to all variables that minimizes the value of the polynomial.

The following polynomial is an example of a HUBO instance:

$$
\begin{aligned}
f(a,b,c,d) &=1 -2a +45c +8d +4ab -13ac +2ad -10bc -12bd +2abc +5acd
\end{aligned}
$$

This polynomial attains its minimum value of $-3$ when $(a,b,c,d) = (0,1,0,1)$.
Finding such an assignment constitutes the HUBO problem for this polynomial.

A **Quadratic Unconstrained Binary Optimization (QUBO)** problem is a special case of HUBO in which the polynomial is restricted to degree at most two.

Usually, optimization problems consist of an objective function and a set of constraints, both expressed as functions of variables.
They aim to find an assignment of variable values that minimizes (or maximizes) the objective function while satisfying all constraints.

In contrast, HUBO and QUBO problems consist only of an objective function and have no explicit constraints.
This simple problem structure enables solvers to efficiently explore solutions by leveraging highly accelerated SIMD-style parallelism.
Furthermore, because constraints can be encoded into the objective function using penalty terms, many constrained optimization problems can be reformulated as equivalent HUBO or QUBO problems.


**QUBO++** is a model-and-solve tool implemented in C++.
It allows users to formulate combinatorial optimization problems as HUBO or QUBO polynomials and then solve them.
QUBO++ also includes three built-in solvers.
