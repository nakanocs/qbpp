---
layout: default
nav_exclude: true
title: "Partitioning Problem (PyQBPP)"
nav_order: 5
---
<div class="lang-en" markdown="1">
# Solving Partitioning Problem Using Vector of variables

## Partitioning problem
Let $w=(w_0, w_1, \ldots, w_{n-1})$ be $n$ positive numbers.
The **partitioning problem** is to partition these numbers into two sets $P$ and $Q$ ($=\overline{P}$) such that the sums of the elements in the two sets are as close as possible.
More specifically, the problem is to find a subset $L \subseteq \lbrace 0,1,\ldots, n-1\rbrace$ that minimizes:

$$
\begin{aligned}
P(L) &= \sum_{i\in L}w_i \\
Q(L) &= \sum_{i\not\in L}w_i \\
f(L) &= \left| P(L)-Q(L) \right|
\end{aligned}
$$

This problem can be formulated as a QUBO problem.
Let $x=(x_0, x_1, \ldots, x_{n-1})$ be binary variables representing the set $L$,
that is, $i\in L$ if and only if $x_i=1$.
We can rewrite $P(L)$, $Q(L)$ and $f(L)$ using $x$ as follows:

$$
\begin{aligned}
P(x) &= \sum_{i=0}^{n-1} w_ix_i \\
Q(x) &= \sum_{i=0}^{n-1} w_i \overline{x_i} \\
f(x)    &= \left( P(x)-Q(x) \right)^2
\end{aligned}
$$

where $\overline{x_i}$ denotes the **negated literal** of $x_i$, which takes the value $1$ when $x_i=0$ and $0$ when $x_i=1$.
Mathematically, $\overline{x_i} = 1 - x_i$, but PyQBPP handles negated literals natively using the `~` operator (e.g., `~x[i]`), which avoids expanding $1 - x_i$ and is more efficient.
For more details, see **[Negated Literals](NEGATIVE)**.

Clearly, $f(x)=f(L)^2$ holds.
The function $f(x)$ is a quadratic expression of $x$, and an optimal solution that minimizes $f(x)$ also gives an optimal solution to the original partitioning problem.

## PyQBPP program for the partitioning problem
The following program creates the QUBO formulation of the partitioning problem for a fixed set of 8 numbers and finds a solution using the Exhaustive Solver.

```python
import pyqbpp as qbpp

w = [64, 27, 47, 74, 12, 83, 63, 40]

x = qbpp.var("x", len(w))
p = qbpp.expr()
q = qbpp.expr()
for i in range(len(w)):
    p += w[i] * x[i]
    q += w[i] * ~x[i]
f = qbpp.sqr(p - q)
print("f =", f.simplify_as_binary())

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print("Solution:", sol)
print("f(sol) =", sol(f))
print("p(sol) =", sol(p))
print("q(sol) =", sol(q))

print("P :", end="")
for i in range(len(w)):
    if sol(x[i]) == 1:
        print(f" {w[i]}", end="")
print()

print("Q :", end="")
for i in range(len(w)):
    if sol(x[i]) == 0:
        print(f" {w[i]}", end="")
print()
```

In this program, **`w`** is a Python list with 8 numbers.
A vector **`x`** of `len(w)=8` binary variables is defined.
Two `Expr` objects **`p`** and **`q`** are defined, and the expressions for $P(x)$ and $Q(x)$
are constructed in the for-loop.
Here, **`~x[i]`** denotes the negated literal $\overline{x_i}$ of `x[i]`.
An `Expr` object **`f`** stores the expression for $f(x)$.

An Exhaustive Solver object **`solver`** for `f` is created
and the solution **`sol`** (a `Sol` object) is obtained by calling its `search()` method.

The values of $f(x)$, $P(x)$, and $Q(x)$ are evaluated by calling **`sol(f)`**, **`sol(p)`** and **`sol(q)`**, respectively.
The numbers in the sets $L$ and $\overline{L}$ are displayed using the for loops.
In these loops, **`sol(x[i])`** returns the value of `x[i]` in `sol`.

This program outputs:
```
f = 168100 -88576*x[0] ...
Solution: Sol(energy=0, x[0]=0, x[1]=0, x[2]=1, x[3]=0, x[4]=1, x[5]=1, x[6]=1, x[7]=0)
f(sol) = 0
p(sol) = 205
q(sol) = 205
P : 47 12 83 63
Q : 64 27 74 40
```

> **NOTE**
> For a `Sol` object `sol` and an `Expr` object `f`, **`sol(f)`** returns the evaluated value of `f` on `sol`.
> For a `Var` object `a`, **`sol(a)`** returns the value of `a` in the solution `sol`.

## PyQBPP program using vector operations
PyQBPP has rich vector operations that can simplify the code.
Since the overloaded operator `*` for `Vector` performs element-wise multiplication,
**`sum(Vector(w) * x)`** returns the `Expr` object representing $P(L)$.
The `~` operator applied to a `Vector` of variables returns a vector of their negated literals.
Thus, **`sum(Vector(w) * ~x)`** returns an `Expr` object storing $Q(L)$.

```python
import pyqbpp as qbpp

w = qbpp.Vector([64, 27, 47, 74, 12, 83, 63, 40])
x = qbpp.var("x", len(w))
p = qbpp.sum(w * x)
q = qbpp.sum(w * ~x)
f = qbpp.sqr(p - q)
```

PyQBPP programs can be simplified by using these vector operations.

> **NOTE**
> The operators `+`, `-`, and `*` are overloaded both for two `Vector` objects and for a scalar and a `Vector` object.
> For two `Vector` objects, the overloaded operators perform element-wise operations.
> For a scalar and a `Vector` object, the overloaded operators apply the scalar operation to each element of the vector.
</div>

<div class="lang-ja" markdown="1">
# 変数ベクトルを用いた分割問題の求解

## 分割問題
$w=(w_0, w_1, \ldots, w_{n-1})$ を $n$ 個の正の数とします。
**分割問題**とは、これらの数を2つの集合 $P$ と $Q$ ($=\overline{P}$) に分割し、両集合の要素の和ができるだけ近くなるようにする問題です。
より具体的には、以下を最小化する部分集合 $L \subseteq \lbrace 0,1,\ldots, n-1\rbrace$ を求めます：

$$
\begin{aligned}
P(L) &= \sum_{i\in L}w_i \\
Q(L) &= \sum_{i\not\in L}w_i \\
f(L) &= \left| P(L)-Q(L) \right|
\end{aligned}
$$

この問題はQUBO問題として定式化できます。
集合 $L$ を表すバイナリ変数 $x=(x_0, x_1, \ldots, x_{n-1})$ を導入します。
すなわち、$i\in L$ であることと $x_i=1$ であることは同値です。
$P(L)$、$Q(L)$、$f(L)$ を $x$ を用いて次のように書き換えられます：

$$
\begin{aligned}
P(x) &= \sum_{i=0}^{n-1} w_ix_i \\
Q(x) &= \sum_{i=0}^{n-1} w_i \overline{x_i} \\
f(x)    &= \left( P(x)-Q(x) \right)^2
\end{aligned}
$$

ここで $\overline{x_i}$ は $x_i$ の**否定リテラル**であり、$x_i=0$ のとき $1$、$x_i=1$ のとき $0$ を取ります。
数学的には $\overline{x_i} = 1 - x_i$ ですが、PyQBPPは否定リテラルを `~` 演算子（例: `~x[i]`）でネイティブに扱えるため、$1 - x_i$ への展開を避けて効率的に処理できます。
詳細は**[否定リテラル](NEGATIVE)**を参照してください。

明らかに $f(x)=f(L)^2$ が成り立ちます。
関数 $f(x)$ は $x$ の2次式であり、$f(x)$ を最小化する最適解は元の分割問題の最適解を与えます。

## 分割問題のPyQBPPプログラム
以下のプログラムは、8個の数の固定集合に対する分割問題のQUBO定式化を作成し、Exhaustive Solverを用いて解を求めます。

```python
import pyqbpp as qbpp

w = [64, 27, 47, 74, 12, 83, 63, 40]

x = qbpp.var("x", len(w))
p = qbpp.expr()
q = qbpp.expr()
for i in range(len(w)):
    p += w[i] * x[i]
    q += w[i] * ~x[i]
f = qbpp.sqr(p - q)
print("f =", f.simplify_as_binary())

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()

print("Solution:", sol)
print("f(sol) =", sol(f))
print("p(sol) =", sol(p))
print("q(sol) =", sol(q))

print("P :", end="")
for i in range(len(w)):
    if sol(x[i]) == 1:
        print(f" {w[i]}", end="")
print()

print("Q :", end="")
for i in range(len(w)):
    if sol(x[i]) == 0:
        print(f" {w[i]}", end="")
print()
```

このプログラムでは、**`w`** は8個の数を持つPythonリストです。
`len(w)=8` 個のバイナリ変数のベクトル **`x`** を定義します。
2つの `Expr` オブジェクト **`p`** と **`q`** を定義し、forループで $P(x)$ と $Q(x)$ の式を構築します。
ここで **`~x[i]`** は `x[i]` の否定リテラル $\overline{x_i}$ を表します。
`Expr` オブジェクト **`f`** に $f(x)$ の式を格納します。

`f` に対するExhaustive Solverオブジェクト **`solver`** を作成し、
`search()` メソッドを呼び出して解 **`sol`**（`Sol` オブジェクト）を取得します。

$f(x)$、$P(x)$、$Q(x)$ の値は、それぞれ **`sol(f)`**、**`sol(p)`**、**`sol(q)`** で評価します。
集合 $L$ と $\overline{L}$ に含まれる数はforループで表示します。
これらのループでは、**`sol(x[i])`** が `sol` における `x[i]` の値を返します。

このプログラムの出力は以下の通りです：
```
f = 168100 -88576*x[0] ...
Solution: Sol(energy=0, x[0]=0, x[1]=0, x[2]=1, x[3]=0, x[4]=1, x[5]=1, x[6]=1, x[7]=0)
f(sol) = 0
p(sol) = 205
q(sol) = 205
P : 47 12 83 63
Q : 64 27 74 40
```

> **NOTE**
> `Sol` オブジェクト `sol` と `Expr` オブジェクト `f` に対して、**`sol(f)`** は `sol` における `f` の評価値を返します。
> `Var` オブジェクト `a` に対して、**`sol(a)`** は解 `sol` における `a` の値を返します。

## ベクトル演算を用いたPyQBPPプログラム
PyQBPPにはコードを簡潔にするための豊富なベクトル演算があります。
`Vector` に対するオーバーロード演算子 `*` は要素ごとの乗算を行うため、
**`sum(Vector(w) * x)`** は $P(L)$ を表す `Expr` オブジェクトを返します。
変数のベクトルに対する `~` 演算子は否定リテラルのベクトルを返します。
したがって、**`sum(Vector(w) * ~x)`** は $Q(L)$ を格納する `Expr` オブジェクトを返します。

```python
import pyqbpp as qbpp

w = qbpp.Vector([64, 27, 47, 74, 12, 83, 63, 40])
x = qbpp.var("x", len(w))
p = qbpp.sum(w * x)
q = qbpp.sum(w * ~x)
f = qbpp.sqr(p - q)
```

これらのベクトル演算を使用することで、PyQBPPプログラムを簡潔に記述できます。

> **NOTE**
> 演算子 `+`、`-`、`*` は、2つの `Vector` オブジェクト間、およびスカラーと `Vector` オブジェクト間の両方でオーバーロードされています。
> 2つの `Vector` オブジェクトの場合、オーバーロード演算子は要素ごとの演算を行います。
> スカラーと `Vector` オブジェクトの場合、オーバーロード演算子はベクトルの各要素にスカラー演算を適用します。
</div>
