---
layout: default
nav_exclude: true
title: "Partitioning Problem"
nav_order: 4
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
Mathematically, $\overline{x_i} = 1 - x_i$, but QUBO++ handles negated literals natively using the `~` operator (e.g., `~x[i]`), which avoids expanding $1 - x_i$ and is more efficient.
For more details, see **[Negated Literals](NEGATIVE)**.

Clearly, $f(x)=f(L)^2$ holds.
The function $f(x)$ is a quadratic expression of $x$, and an optimal solution that minimizes $f(x)$ also gives an optimal solution to the original partitioning problem.

## QUBO++ program for the partitioning problem
The following QUBO++ program creates the QUBO formulation of the partitioning problem for a fixed set of 8 numbers and finds a solution using the Exhaustive Solver.

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  std::vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};

  auto x = qbpp::var("x", w.size());
  auto p = qbpp::expr();
  auto q = qbpp::expr();
  for (size_t i = 0; i < w.size(); ++i) {
    p += w[i] * x[i];
    q += w[i] * ~x[i];
  }
  auto f = qbpp::sqr(p - q);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search();

  std::cout << "Solution: " << sol << std::endl;
  std::cout << "f(sol) = " << f(sol) << std::endl;
  std::cout << "p(sol) = " << p(sol) << std::endl;
  std::cout << "q(sol) = " << q(sol) << std::endl;

  std::cout << "P :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](sol) == 1) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;

  std::cout << "Q :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](sol) == 0) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
}
```

In this program, **`w`** is defined as a `std::vector` object with 8 numbers.
A vector **`x`** of `w.size()=8` binary variables is defined.
Two `qbpp::Expr` objects **`p`** and **`q`** are defined, and the expressions for $P(x)$ and $Q(x)$
are constructed in the for-loop.
Here, **`~x[i]`** denotes the negated literal $\overline{x_i}$ of `x[i]`.
A `qbpp::Expr` object **`f`** stores the expression for $f(x)$.

An Exhaustive Solver object **`solver`** for `f` is created
and the solution **`sol`** (a `qbpp::Sol` object) is obtained by calling its `search()` member function.

The values of $f(x)$, $P(x)$, and $Q(x)$ are evaluated by calling **`f(sol)`**, **`p(sol)`** and **`q(sol)`**, respectively.
The numbers in the sets $L$ and $\overline{L}$ are displayed using the for loops.
In these loops, **`x[i](sol)`** returns the value of `x[i]` in `sol`.

This program outputs:
{% raw %}
```txt
f = 168100 -88576*x[0] -41364*x[1] -68244*x[2] -99456*x[3] -19104*x[4] -108564*x[5] -87444*x[6] -59200*x[7] +13824*x[0]*x[1] +24064*x[0]*x[2] +37888*x[0]*x[3] +6144*x[0]*x[4] +42496*x[0]*x[5] +32256*x[0]*x[6] +20480*x[0]*x[7] +10152*x[1]*x[2] +15984*x[1]*x[3] +2592*x[1]*x[4] +17928*x[1]*x[5] +13608*x[1]*x[6] +8640*x[1]*x[7] +27824*x[2]*x[3] +4512*x[2]*x[4] +31208*x[2]*x[5] +23688*x[2]*x[6] +15040*x[2]*x[7] +7104*x[3]*x[4] +49136*x[3]*x[5] +37296*x[3]*x[6] +23680*x[3]*x[7] +7968*x[4]*x[5] +6048*x[4]*x[6] +3840*x[4]*x[7] +41832*x[5]*x[6] +26560*x[5]*x[7] +20160*x[6]*x[7]
Solution: 0:{{x[0],0},{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],1},{x[7],0}}
f(sol) = 0
p(sol) = 205
q(sol) = 205
P : 47 12 83 63
Q : 64 27 74 40
```
{% endraw %}
> **NOTE**
> For a `qbpp::Expr` object `f` and a `qbpp::Sol` object `sol`, both **`f(sol)`** and **`sol(f)`** return the resulting value of `f` evaluated on `sol`.
> Likewise, for a `qbpp::Var` object `a`, both **`a(sol)`** and **`sol(a)`** return the value of `a` in the solution `sol`.
> The form **`f(sol)`** is natural from a **mathematical perspective**, as it corresponds to evaluating a function at a point.
> In contrast, **`sol(f)`** is natural from an **object-oriented programming perspective**, where the solution object evaluates an expression.
> You may use either form according to your preference.

## QUBO++ program using vector operations
QUBO++ has rich vector operations that can simplify the code.
For this purpose, **`qbpp::Vector`** class, which is similar to `std::vector` class shoud be used.
In the following code, **`w`** is defined as an object of **`qbpp::Vector<uint32_t>`** class.
Also, **`x`** is defined as an object of **`qbpp::Vector<qbpp::Var>`** class.
Since the overloaded operator `*` for `qbpp::Vector` class returns the element-wise product of two
`qbpp::Vector` objects, **`qbpp::sum(w * x)`** returns the `qbpp::Expr` object representing $P(L)$.
The `~` operator applied to a `qbpp::Vector` of variables returns a vector of their negated literals.
Thus, **`qbpp::sum(w * ~x)`** returns a `qbpp::Expr` object storing $Q(L)$.

```cpp
  qbpp::Vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
  auto x = qbpp::var("x", w.size());
  auto p = qbpp::sum(w * x);
  auto q = qbpp::sum(w * ~x);
  auto f = qbpp::sqr(p - q);
```

QUBO++ programs can be simplified by using these vector operations.
In addition, since vector operations for large vectors are parallelized by multithreading, they can accelerate the process of creating QUBO models.

> **NOTE**
> The operators `+`, `-`, and `*` are overloaded both for two `qbpp::Vector` objects and for a scalar and a `qbpp::Vector` object.
> For two `qbpp::Vector` objects, the overloaded operators perform element-wise operations.
> For a scalar and a `qbpp::Vector` object, the overloaded operators apply the scalar operation to each element of the vector.

</div>

<div class="lang-ja" markdown="1">

# 変数ベクトルを用いた分割問題の解法

## 分割問題
$w=(w_0, w_1, \ldots, w_{n-1})$ を $n$ 個の正の数とします。
**分割問題**は、これらの数を2つの集合 $P$ と $Q$ ($=\overline{P}$) に分割し、2つの集合の要素の和ができるだけ近くなるようにする問題です。
より具体的には、次を最小化する部分集合 $L \subseteq \lbrace 0,1,\ldots, n-1\rbrace$ を見つける問題です：

$$
\begin{aligned}
P(L) &= \sum_{i\in L}w_i \\
Q(L) &= \sum_{i\not\in L}w_i \\
f(L) &= \left| P(L)-Q(L) \right|
\end{aligned}
$$

この問題はQUBO問題として定式化できます。
$x=(x_0, x_1, \ldots, x_{n-1})$ を集合 $L$ を表すバイナリ変数とし、$i\in L$ は $x_i=1$ のときかつそのときに限ります。
$x$ を用いて $P(L)$、$Q(L)$、$f(L)$ を次のように書き換えることができます：

$$
\begin{aligned}
P(x) &= \sum_{i=0}^{n-1} w_ix_i \\
Q(x) &= \sum_{i=0}^{n-1} w_i \overline{x_i} \\
f(x)    &= \left( P(x)-Q(x) \right)^2
\end{aligned}
$$

ここで $\overline{x_i}$ は $x_i$ の**否定リテラル**であり、$x_i=0$ のとき $1$、$x_i=1$ のとき $0$ を取ります。
数学的には $\overline{x_i} = 1 - x_i$ ですが、QUBO++は否定リテラルを `~` 演算子（例: `~x[i]`）でネイティブに扱えるため、$1 - x_i$ への展開を避けて効率的に処理できます。
詳細は**[否定リテラル](NEGATIVE)**を参照してください。

明らかに $f(x)=f(L)^2$ が成り立ちます。
関数 $f(x)$ は $x$ の2次式であり、$f(x)$ を最小化する最適解は元の分割問題の最適解も与えます。

## 分割問題のQUBO++プログラム
以下のQUBO++プログラムは、固定された8個の数の分割問題のQUBO定式化を作成し、Exhaustive Solverを用いて解を求めます。

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  std::vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};

  auto x = qbpp::var("x", w.size());
  auto p = qbpp::expr();
  auto q = qbpp::expr();
  for (size_t i = 0; i < w.size(); ++i) {
    p += w[i] * x[i];
    q += w[i] * ~x[i];
  }
  auto f = qbpp::sqr(p - q);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search();

  std::cout << "Solution: " << sol << std::endl;
  std::cout << "f(sol) = " << f(sol) << std::endl;
  std::cout << "p(sol) = " << p(sol) << std::endl;
  std::cout << "q(sol) = " << q(sol) << std::endl;

  std::cout << "P :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](sol) == 1) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;

  std::cout << "Q :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](sol) == 0) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
}
```

このプログラムでは、**`w`** は8個の数を持つ `std::vector` オブジェクトとして定義されています。
`w.size()=8` 個のバイナリ変数のベクトル **`x`** が定義されます。
2つの `qbpp::Expr` オブジェクト **`p`** と **`q`** が定義され、$P(x)$ と $Q(x)$ の式がforループ内で構築されます。
ここで **`~x[i]`** は `x[i]` の否定リテラル $\overline{x_i}$ を表します。
`qbpp::Expr` オブジェクト **`f`** は $f(x)$ の式を格納します。

`f` に対するExhaustive Solverオブジェクト **`solver`** が作成され、`search()` メンバ関数の呼び出しにより解 **`sol`**（`qbpp::Sol` オブジェクト）が得られます。

$f(x)$、$P(x)$、$Q(x)$ の値はそれぞれ **`f(sol)`**、**`p(sol)`**、**`q(sol)`** の呼び出しにより評価されます。
集合 $L$ と $\overline{L}$ の数はforループを使って表示されます。
これらのループでは、**`x[i](sol)`** が `sol` における `x[i]` の値を返します。

このプログラムの出力は以下のとおりです：
{% raw %}
```txt
f = 168100 -88576*x[0] -41364*x[1] -68244*x[2] -99456*x[3] -19104*x[4] -108564*x[5] -87444*x[6] -59200*x[7] +13824*x[0]*x[1] +24064*x[0]*x[2] +37888*x[0]*x[3] +6144*x[0]*x[4] +42496*x[0]*x[5] +32256*x[0]*x[6] +20480*x[0]*x[7] +10152*x[1]*x[2] +15984*x[1]*x[3] +2592*x[1]*x[4] +17928*x[1]*x[5] +13608*x[1]*x[6] +8640*x[1]*x[7] +27824*x[2]*x[3] +4512*x[2]*x[4] +31208*x[2]*x[5] +23688*x[2]*x[6] +15040*x[2]*x[7] +7104*x[3]*x[4] +49136*x[3]*x[5] +37296*x[3]*x[6] +23680*x[3]*x[7] +7968*x[4]*x[5] +6048*x[4]*x[6] +3840*x[4]*x[7] +41832*x[5]*x[6] +26560*x[5]*x[7] +20160*x[6]*x[7]
Solution: 0:{{x[0],0},{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],1},{x[7],0}}
f(sol) = 0
p(sol) = 205
q(sol) = 205
P : 47 12 83 63
Q : 64 27 74 40
```
{% endraw %}
> **注記**
> `qbpp::Expr` オブジェクト `f` と `qbpp::Sol` オブジェクト `sol` に対して、**`f(sol)`** と **`sol(f)`** はどちらも `sol` 上で `f` を評価した値を返します。
> 同様に、`qbpp::Var` オブジェクト `a` に対して、**`a(sol)`** と **`sol(a)`** はどちらも解 `sol` における `a` の値を返します。
> **`f(sol)`** の形式は、関数をある点で評価することに対応するため、**数学的な観点**からは自然です。
> 一方、**`sol(f)`** は解オブジェクトが式を評価するという、**オブジェクト指向プログラミングの観点**からは自然です。
> 好みに応じてどちらの形式を使用しても構いません。

## ベクトル演算を用いたQUBO++プログラム
QUBO++にはコードを簡潔にするための豊富なベクトル演算があります。
このために、`std::vector` クラスに似た **`qbpp::Vector`** クラスを使用します。
以下のコードでは、**`w`** は **`qbpp::Vector<uint32_t>`** クラスのオブジェクトとして定義されています。
また、**`x`** は **`qbpp::Vector<qbpp::Var>`** クラスのオブジェクトとして定義されています。
`qbpp::Vector` クラスのオーバーロードされた演算子 `*` は2つの `qbpp::Vector` オブジェクトの要素ごとの積を返すため、**`qbpp::sum(w * x)`** は $P(L)$ を表す `qbpp::Expr` オブジェクトを返します。
変数のベクトルに対する `~` 演算子は否定リテラルのベクトルを返します。
したがって、**`qbpp::sum(w * ~x)`** は $Q(L)$ を格納する `qbpp::Expr` オブジェクトを返します。

```cpp
  qbpp::Vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
  auto x = qbpp::var("x", w.size());
  auto p = qbpp::sum(w * x);
  auto q = qbpp::sum(w * ~x);
  auto f = qbpp::sqr(p - q);
```

これらのベクトル演算を使用することで、QUBO++プログラムを簡潔にできます。
さらに、大きなベクトルに対するベクトル演算はマルチスレッドにより並列化されるため、QUBOモデルの作成プロセスを高速化できます。

> **注記**
> 演算子 `+`、`-`、`*` は、2つの `qbpp::Vector` オブジェクト間、およびスカラーと `qbpp::Vector` オブジェクト間の両方でオーバーロードされています。
> 2つの `qbpp::Vector` オブジェクトの場合、オーバーロードされた演算子は要素ごとの演算を行います。
> スカラーと `qbpp::Vector` オブジェクトの場合、オーバーロードされた演算子はベクトルの各要素にスカラー演算を適用します。

</div>
