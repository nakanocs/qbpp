---
layout: default
title: "PyQBPP (Python)"
nav_order: 5
---
<div class="lang-en" markdown="1">
# PyQBPP Document
This documentation for PyQBPP (Python binding of QUBO++) is currently under development.
Some pages may contain incomplete or provisional information.

## Getting Started
1. [HUBO and QUBO](../HUBO_QUBO)
2. [Quick Start](QUICK)
3. [Installation](INSTALL)

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
9. [Negated Literals](NEGATIVE)
10. [Sum Functions for Multi-dimensional Arrays](SUM)
11. [Easy Solver Usage](EASYSOLVER)
12. [Exhaustive Solver Usage](EXHAUSTIVE)
13. [ABS3 Solver Usage](ABS3)

> **NOTE**
> The Gurobi Solver is not available in the Python binding.
> For Gurobi usage, please refer to the [QUBO++ (C++) documentation](../GUROBI).

## Case Studies
This section provides examples of solving optimization, mathematical, and graph problems using PyQBPP.
See the [Case Studies](CASE_STUDIES) page for the full list.

## Quick References
1. [Variables and Expressions](QR_VARIABLE)
2. [Operators and Functions](QR_OPERATION)
</div>

<div class="lang-ja" markdown="1">
# PyQBPP ドキュメント
PyQBPP（QUBO++のPythonバインディング）のドキュメントは現在作成中です。
一部のページには不完全または暫定的な情報が含まれている場合があります。

## はじめに
1. [HUBO と QUBO](../HUBO_QUBO)
2. [クイックスタート](QUICK)
3. [インストール](INSTALL)

## 基礎
このセクションでは、PyQBPPの段階的な入門を提供します。
ページを順番に読むことで、変数と式の定義方法、最適化問題のモデル化、
PyQBPPを使った求解方法を学ぶことができます。
このチュートリアルを完了すれば、一般的な用途でPyQBPPを使えるようになります。

1. [変数の定義と式の作成](VARIABLE)
2. [式の求解](SOLVE)
3. [変数のベクトルとベクトル関数](VECTOR)
4. [変数ベクトルを用いた分割問題の求解](PARTITION)
5. [順列行列と割当問題の求解](PERMUTATION)
6. [整数変数と連立方程式の求解](INTEGER)
7. [HUBO式による因数分解](FACTORIZATION)
8. [範囲制約と整数線形計画法の求解](RANGE)

## トピック
このセクションでは、PyQBPPの特定の機能についてトピックごとに解説します。
各ページは特定のトピックに焦点を当て、設計上の判断、使用パターン、
および必要に応じて内部実装について、より深い知見を提供します。

1. [変数と式のデータ型](VAREXPR)
2. [基本演算子と関数](OPERATOR)
3. [ベクトル用の基本演算子と関数](OPVECTOR)
4. [多次元変数と式](MULTIDIM)
5. [比較演算子](COMPARISON)
6. [式のクラス](EXPRESSION)
7. [式の評価](EVAL)
8. [置換関数](REPLACE)
9. [否定リテラル](NEGATIVE)
10. [多次元配列の和関数](SUM)
11. [Easy Solverの使い方](EASYSOLVER)
12. [Exhaustive Solverの使い方](EXHAUSTIVE)
13. [ABS3 Solverの使い方](ABS3)

> **NOTE**
> Gurobi SolverはPythonバインディングでは利用できません。
> Gurobiの使い方については、[QUBO++ (C++) ドキュメント](../GUROBI)を参照してください。

## ケーススタディ
このセクションでは、PyQBPPを使った最適化問題、数学問題、グラフ問題の解法例を提供します。
全リストは[ケーススタディ](CASE_STUDIES)ページを参照してください。

## クイックリファレンス
1. [変数と式](QR_VARIABLE)
2. [演算子と関数](QR_OPERATION)
</div>
