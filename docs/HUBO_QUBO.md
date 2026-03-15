---
layout: default
title: "HUBO and QUBO"
nav_order: 2
---
<div class="lang-en" markdown="1">

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


**QUBO++** is a **model-and-solve tool** implemented in C++.
It allows users to formulate combinatorial optimization problems as HUBO or QUBO polynomials and then solve them.
QUBO++ also includes three built-in solvers.

**PyQBPP** is a Python wrapper for the QUBO++ library, providing the same modeling and solving capabilities from Python.

</div>

<div class="lang-ja" markdown="1">

# HUBO と QUBO
**高次制約なしバイナリ最適化 (High-Order Unconstrained Binary Optimization, HUBO)** 問題は、バイナリ変数上の多項式によって定義されます。
目標は、多項式の値を最小化するようなバイナリ値 ${0,1}$ の割り当てを全変数に対して見つけることです。

以下の多項式は HUBO のインスタンスの例です：

$$
\begin{aligned}
f(a,b,c,d) &=1 -2a +45c +8d +4ab -13ac +2ad -10bc -12bd +2abc +5acd
\end{aligned}
$$

この多項式は $(a,b,c,d) = (0,1,0,1)$ のとき最小値 $-3$ を取ります。
このような割り当てを見つけることが、この多項式に対する HUBO 問題です。

**2次制約なしバイナリ最適化 (Quadratic Unconstrained Binary Optimization, QUBO)** 問題は、多項式の次数が2以下に制限された HUBO の特殊ケースです。

通常、最適化問題は目的関数と制約条件の集合で構成され、いずれも変数の関数として表現されます。
すべての制約条件を満たしつつ、目的関数を最小化（または最大化）する変数値の割り当てを見つけることが目的です。

一方、HUBO および QUBO 問題は目的関数のみで構成され、明示的な制約条件を持ちません。
このシンプルな問題構造により、ソルバーは高度に加速された SIMD スタイルの並列処理を活用して効率的に解を探索できます。
さらに、ペナルティ項を用いて制約条件を目的関数に組み込めるため、多くの制約付き最適化問題を等価な HUBO または QUBO 問題として再定式化できます。


**QUBO++** は C++ で実装された**モデリング＆求解ツール**です。
組合せ最適化問題を HUBO または QUBO の多項式として定式化し、それを解くことができます。
QUBO++ には3つの組み込みソルバーが含まれています。

**PyQBPP** は QUBO++ ライブラリの Python ラッパーであり、Python から同じモデリングおよび求解機能を提供します。

</div>
