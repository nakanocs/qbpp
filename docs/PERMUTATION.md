---
layout: default
nav_exclude: true
title: "Permutation Matrix"
nav_order: 5
---
<div class="lang-en" markdown="1">
# Permutation matrix generation

Many combinatorial optimization problems are permutation-based in the sense that the objective is to find an optimal permutation.
As a fundamental technique for formulating such optimization problems, a matrix of binary variables is used in their QUBO formulation.

## Permutation matrix
Let $X=(x_{i,j})$ ($0\leq i,j\leq n-1$) is a matrix of $n\times n$ binary values.
The matrix $X$ is called a **permutation matrix** if and only if every row and every column has exactly one entry equal to 1, as shown below.

<p align="center">
  <img src="images/matrix.svg" alt="Permutation matrix" width="50%">
</p>

A **permutation matrix** represents a permutation of $n$ numbers $(0,1,\ldots,n-1)$, where $x_{i,j} = 1$ if and only if the $i$-th element is $j$.
For example, the above permutation matrix represents the permutation $(1,3,0,2)$.

## QUBO formulation for permutation matrices
A binary variable matrix $X=(x_{i,j})$ ($0\leq i,j\leq n-1$)
stores a permutation matrix if and only if the sum of each row and each column is 1.
Thus, the following QUBO function takes the minimum value 0 if and only if $X$ stores a permutation matrix:

$$
\begin{aligned}
f(X) &= \sum_{i=0}^{n-1}\left(1-\sum_{j=0}^{n-1}x_{i,j}\right)^2+\sum_{j=0}^{n-1}\left(1-\sum_{i=0}^{n-1}x_{i,j}\right)^2
\end{aligned}
$$

## QUBO++ program for generating permutation matrices
We can design a QUBO++ program based on the formula $f(X)$ above as follows:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::expr();

  for (size_t i = 0; i < 4; i++) {
    auto s = qbpp::expr();
    for (size_t j = 0; j < 4; j++) {
      s += x[i][j];
    }
    f += qbpp::sqr(1 - s);
  }

  for (size_t j = 0; j < 4; j++) {
    auto s = qbpp::expr();
    for (size_t i = 0; i < 4; i++) {
      s += x[i][j];
    }
    f += qbpp::sqr(1 - s);
  }

  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t k = 0; k < sols.size(); k++) {
    const auto& sol = sols[k];
    std::cout << "Solution " << k << " : " << sol(x) << std::endl;
  }
}
```

In this program, **`qbpp::var("x",4,4)`** returns a `qbpp::Vector<qbpp::Vector<qbpp::Var>>` object
of size $4\times 4$ named **`x`**.
For a `qbpp::Expr` object **`f`**, two double for-loops adds
formulas for $f(X)$.
Using the Exhaustive Solver, all optimal solutions are computed and stored in **`sols`**.
All solutions in `sols` are displayed one-by-one.
Here, `sol(x)` returns a matrix of values of `x` in `sol`.
This program outputs all 24 permutations as follows:
{% raw %}
```cpp
Solution 0 : {{0,0,0,1},{0,0,1,0},{0,1,0,0},{1,0,0,0}}
Solution 1 : {{0,0,0,1},{0,0,1,0},{1,0,0,0},{0,1,0,0}}
Solution 2 : {{0,0,0,1},{0,1,0,0},{0,0,1,0},{1,0,0,0}}
Solution 3 : {{0,0,0,1},{0,1,0,0},{1,0,0,0},{0,0,1,0}}
Solution 4 : {{0,0,0,1},{1,0,0,0},{0,0,1,0},{0,1,0,0}}
Solution 5 : {{0,0,0,1},{1,0,0,0},{0,1,0,0},{0,0,1,0}}
Solution 6 : {{0,0,1,0},{0,0,0,1},{0,1,0,0},{1,0,0,0}}
Solution 7 : {{0,0,1,0},{0,0,0,1},{1,0,0,0},{0,1,0,0}}
Solution 8 : {{0,0,1,0},{0,1,0,0},{0,0,0,1},{1,0,0,0}}
Solution 9 : {{0,0,1,0},{0,1,0,0},{1,0,0,0},{0,0,0,1}}
Solution 10 : {{0,0,1,0},{1,0,0,0},{0,0,0,1},{0,1,0,0}}
Solution 11 : {{0,0,1,0},{1,0,0,0},{0,1,0,0},{0,0,0,1}}
Solution 12 : {{0,1,0,0},{0,0,0,1},{0,0,1,0},{1,0,0,0}}
Solution 13 : {{0,1,0,0},{0,0,0,1},{1,0,0,0},{0,0,1,0}}
Solution 14 : {{0,1,0,0},{0,0,1,0},{0,0,0,1},{1,0,0,0}}
Solution 15 : {{0,1,0,0},{0,0,1,0},{1,0,0,0},{0,0,0,1}}
Solution 16 : {{0,1,0,0},{1,0,0,0},{0,0,0,1},{0,0,1,0}}
Solution 17 : {{0,1,0,0},{1,0,0,0},{0,0,1,0},{0,0,0,1}}
Solution 18 : {{1,0,0,0},{0,0,0,1},{0,0,1,0},{0,1,0,0}}
Solution 19 : {{1,0,0,0},{0,0,0,1},{0,1,0,0},{0,0,1,0}}
Solution 20 : {{1,0,0,0},{0,0,1,0},{0,0,0,1},{0,1,0,0}}
Solution 21 : {{1,0,0,0},{0,0,1,0},{0,1,0,0},{0,0,0,1}}
Solution 22 : {{1,0,0,0},{0,1,0,0},{0,0,0,1},{0,0,1,0}}
Solution 23 : {{1,0,0,0},{0,1,0,0},{0,0,1,0},{0,0,0,1}}
```
{% endraw %}
> **NOTE**
> A matrix of binary variables is implemented as a nested vector using `qbpp::Vector` class.
> For example, `qbpp::var("x",4,4)` returns a `qbpp::Vector<qbpp::Vector<qbpp::Var>>` object.
> Each `qbpp::Var` object is represented as `x[i][j]` and the value of `x[i][j]` for `sol` can be obtained by either `sol(x[i][j])` or `x[i][j](sol)`.


## QUBO formulation for a permutation matrix using vector functions and operations
Using `qbpp::vector_sum()`, we can compute the row-wise and column-wise sums of a matrix `x` of binary variables:
- **`qbpp::vector_sum(x, 1)`**: Computes the sum of each row of `x` and returns a vector of size `n` containing these sums.
- **`qbpp::vector_sum(x, 0)`**: Computes the sum of each column of `x` and returns a vector of size `n` containing these sums.

> **Note**:
> For a multi-dimensional array `x` and an axis `k`, `qbpp::vector_sum(x, k)` computes sums along axis `k` and returns a multi-dimensional array whose dimension is reduced by one.
> For a 2-dimensional array (matrix) `x`, axis `1` corresponds to the row direction, and axis `0` corresponds to the column direction.

A scalar–vector operation can be used to subtract 1 from each element:
- **`qbpp::vector_sum(x, 1) - 1`**: subtracts 1 from each row-wise sum.
- **`qbpp::vector_sum(x, 0) - 1`**: subtracts 1 from each column-wise sum.

For these two vectors of size `n`, `qbpp::sqr()` squares each element, and `qbpp::sum()` computes the sum of all elements.

The following QUBO++ program implements a QUBO formulation using these vector functions and operations:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::sum(qbpp::sqr(qbpp::vector_sum(x, 1) - 1)) +
           qbpp::sum(qbpp::sqr(qbpp::vector_sum(x, 0) - 1));
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t k = 0; k < sols.size(); k++) {
    const auto& sol = sols[k];
    const auto& row = qbpp::onehot_to_int(x(sol), 1);
    const auto& column = qbpp::onehot_to_int(x(sol), 0);
    std::cout << "Solution " << k << ": " << row << ", " << column << std::endl;
  }
}
```
In this program, `x(sol)` returns a matrix of assigned values to `x` in `sol`, which is a matrix of integers of size
.
`qbpp::onehot_to_int()` converts one-hot vectors along the axis to the corresponding integers.
- **`qbpp::onehot_to_int(x(sol), 1)`**: Computes the integer corresponding to each row and returns them as a vector of 4 integers, which represents the permutation.
- **`qbpp::onehot_to_int(x(sol), 0)`**: returns the integer corresponding to each column and returns them as a vector of 4 integers, which represents the inverse of the permutation.
This program outputs all permutations and their inverse as integer vectors as follows:
```
Solution 0: {3,2,1,0}, {3,2,1,0}
Solution 1: {3,2,0,1}, {2,3,1,0}
Solution 2: {3,1,2,0}, {3,1,2,0}
Solution 3: {3,1,0,2}, {2,1,3,0}
Solution 4: {3,0,2,1}, {1,3,2,0}
Solution 5: {3,0,1,2}, {1,2,3,0}
Solution 6: {2,3,1,0}, {3,2,0,1}
Solution 7: {2,3,0,1}, {2,3,0,1}
Solution 8: {2,1,3,0}, {3,1,0,2}
Solution 9: {2,1,0,3}, {2,1,0,3}
Solution 10: {2,0,3,1}, {1,3,0,2}
Solution 11: {2,0,1,3}, {1,2,0,3}
Solution 12: {1,3,2,0}, {3,0,2,1}
Solution 13: {1,3,0,2}, {2,0,3,1}
Solution 14: {1,2,3,0}, {3,0,1,2}
Solution 15: {1,2,0,3}, {2,0,1,3}
Solution 16: {1,0,3,2}, {1,0,3,2}
Solution 17: {1,0,2,3}, {1,0,2,3}
Solution 18: {0,3,2,1}, {0,3,2,1}
Solution 19: {0,3,1,2}, {0,2,3,1}
Solution 20: {0,2,3,1}, {0,3,1,2}
Solution 21: {0,2,1,3}, {0,2,1,3}
Solution 22: {0,1,3,2}, {0,1,3,2}
Solution 23: {0,1,2,3}, {0,1,2,3}
```

## Assignment problem and its QUBO formulation
Let $C = (c_{i,j})$ be a cost matrix of size $n \times n$.
The **assignment problem** for $C$ is to find a permutation
$p:\lbrace 0,1,\ldots, n-1\rbrace \rightarrow \lbrace 0,1,\ldots, n-1\rbrace$
that minimizes the total cost:

$$
\begin{aligned}
 g(p) &= \sum_{i=0}^{n-1}c_{i,p(i)}
\end{aligned}
$$

We can use a permutation matrix $X = (x_{i,j})$ of size $n \times n$ for a QUBO formulation of this problem by defining:

$$
\begin{aligned}
 g(X) &= \sum_{i=0}^{n-1}\sum_{j=0}^{n-1}c_{i,j}x_{i,j}
\end{aligned}
$$

Clearly, $g(p) = g(X)$ holds if and only if $X$ represents the permutation $p$.

We combine the QUBO formulation for the permutation matrix, $f(X)$, and the total cost, $g(X)$, to obtain a QUBO formulation of the assignment problem:

$$
\begin{aligned}
 h(X) &= P\cdot f(x)+g(x) \\
     &=P\left(\sum_{i=0}^{n-1}\left(1-\sum_{j=0}^{n-1}x_{i,j}\right)^2+\sum_{j=0}^{n-1}\left(1-\sum_{i=0}^{n-1}x_{i,j}\right)^2\right)+\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}c_{i,j}x_{i,j}
\end{aligned}
$$

Here, $P$ is a sufficiently large positive constant that prioritizes the permutation constraints encoded in $f(X)$.

## QUBO++ program for the assignment problem
We are now ready to design a QUBO++ program for the assignment problem.
In this program, a fixed matrix $C$ of size $4\times4$ is given as a nested `qbpp::Vector`.
The formulas for $f(X)$ and $g(X)$ are defined using vector functions and operations.
Here, `qbpp::vector_sum(x, 1) == 1` returns a QUBO expression that takes the minimum value 0 if the equality is satisfied.
In fact, it returns the same QUBO expression as `qbpp::sqr(qbpp::vector_sum(x, 1) - 1)`.
Also, `c * x` returns a matrix obtained by computing the element-wise product of `c` and `x`,
and therefore `qbpp::sum(c * x)` returns `g(X)`.

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  qbpp::Vector<qbpp::Vector<uint32_t>> c = {
      {58, 73, 91, 44}, {62, 15, 87, 39}, {78, 56, 23, 94}, {11, 85, 68, 72}};
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::sum(qbpp::vector_sum(x, 1) == 1) +
           qbpp::sum(qbpp::vector_sum(x, 0) == 1);
  auto g = qbpp::sum(c * x);
  auto h = 1000 * f + g;
  h.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(h);
  solver.time_limit(1.0);

  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  auto result = qbpp::onehot_to_int(x(sol), 1);
  std::cout << "Result : " << result << std::endl;
  for (size_t i = 0; i < result.size(); ++i) {
    std::cout << "c[" << i << "][" << result[i] << "] = " << c[i][result[i]]
              << std::endl;
  }
}
```

We use the Easy Solver to find a solution of `h`.
For an Easy Solver object `solver` for `h`, the time limit for searching a solution is set to 1.0 seconds by calling the `time_limit()` member function.
The resulting permutation is stored in `result`, and the selected `c[i][j]` values are printed in turn.
The output of this program is as follows:

{% raw %}
```
sol = 93:{{x[0][0],0},{x[0][1],0},{x[0][2],0},{x[0][3],1},{x[1][0],0},{x[1][1],1},{x[1][2],0},{x[1][3],0},{x[2][0],0},{x[2][1],0},{x[2][2],1},{x[2][3],0},{x[3][0],1},{x[3][1],0},{x[3][2],0},{x[3][3],0}}
Result : {3,1,2,0}
c[0][3] = 44
c[1][1] = 15
c[2][2] = 23
c[3][0] = 11
```
{% endraw %}
> **NOTE**
> For an expression `f` and an integer `m`, `f == m` returns an expression `qbpp::sqr(f - m)`,
> which takes the minimum value 0 if and only if the equality `f == m` is satisfied.

</div>

<div class="lang-ja" markdown="1">
# 置換行列の生成

多くの組合せ最適化問題は、最適な置換を見つけることが目的であるという意味で置換ベースです。
このような最適化問題を定式化する基本的な手法として、QUBO定式化ではバイナリ変数の行列が使用されます。

## 置換行列
$X=(x_{i,j})$ ($0\leq i,j\leq n-1$) を $n\times n$ のバイナリ値の行列とします。
行列 $X$ は、以下に示すように、すべての行とすべての列にちょうど1つの1のエントリがある場合にのみ**置換行列**と呼ばれます。

<p align="center">
  <img src="images/matrix.svg" alt="Permutation matrix" width="50%">
</p>

**置換行列**は $n$ 個の数 $(0,1,\ldots,n-1)$ の置換を表し、$x_{i,j} = 1$ であるのは $i$ 番目の要素が $j$ である場合に限ります。
例えば、上記の置換行列は置換 $(1,3,0,2)$ を表しています。

## 置換行列のQUBO定式化
バイナリ変数行列 $X=(x_{i,j})$ ($0\leq i,j\leq n-1$) が置換行列を格納するのは、各行と各列の和が1である場合に限ります。
したがって、以下のQUBO関数は $X$ が置換行列を格納する場合にのみ最小値0を取ります：

$$
\begin{aligned}
f(X) &= \sum_{i=0}^{n-1}\left(1-\sum_{j=0}^{n-1}x_{i,j}\right)^2+\sum_{j=0}^{n-1}\left(1-\sum_{i=0}^{n-1}x_{i,j}\right)^2
\end{aligned}
$$

## 置換行列を生成するQUBO++プログラム
上記の式 $f(X)$ に基づいて、以下のようにQUBO++プログラムを設計できます：
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::expr();

  for (size_t i = 0; i < 4; i++) {
    auto s = qbpp::expr();
    for (size_t j = 0; j < 4; j++) {
      s += x[i][j];
    }
    f += qbpp::sqr(1 - s);
  }

  for (size_t j = 0; j < 4; j++) {
    auto s = qbpp::expr();
    for (size_t i = 0; i < 4; i++) {
      s += x[i][j];
    }
    f += qbpp::sqr(1 - s);
  }

  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t k = 0; k < sols.size(); k++) {
    const auto& sol = sols[k];
    std::cout << "Solution " << k << " : " << sol(x) << std::endl;
  }
}
```

このプログラムでは、**`qbpp::var("x",4,4)`**は**`x`**という名前の $4\times 4$ サイズの `qbpp::Vector<qbpp::Vector<qbpp::Var>>` オブジェクトを返します。
`qbpp::Expr` オブジェクト**`f`**に対して、2つの二重forループが $f(X)$ の式を追加します。
Exhaustive Solverを使用して、すべての最適解が計算され**`sols`**に格納されます。
`sols` 内のすべての解が1つずつ表示されます。
ここで、`sol(x)` は `sol` における `x` の値の行列を返します。
このプログラムは以下のように24個すべての置換を出力します：
{% raw %}
```cpp
Solution 0 : {{0,0,0,1},{0,0,1,0},{0,1,0,0},{1,0,0,0}}
Solution 1 : {{0,0,0,1},{0,0,1,0},{1,0,0,0},{0,1,0,0}}
Solution 2 : {{0,0,0,1},{0,1,0,0},{0,0,1,0},{1,0,0,0}}
Solution 3 : {{0,0,0,1},{0,1,0,0},{1,0,0,0},{0,0,1,0}}
Solution 4 : {{0,0,0,1},{1,0,0,0},{0,0,1,0},{0,1,0,0}}
Solution 5 : {{0,0,0,1},{1,0,0,0},{0,1,0,0},{0,0,1,0}}
Solution 6 : {{0,0,1,0},{0,0,0,1},{0,1,0,0},{1,0,0,0}}
Solution 7 : {{0,0,1,0},{0,0,0,1},{1,0,0,0},{0,1,0,0}}
Solution 8 : {{0,0,1,0},{0,1,0,0},{0,0,0,1},{1,0,0,0}}
Solution 9 : {{0,0,1,0},{0,1,0,0},{1,0,0,0},{0,0,0,1}}
Solution 10 : {{0,0,1,0},{1,0,0,0},{0,0,0,1},{0,1,0,0}}
Solution 11 : {{0,0,1,0},{1,0,0,0},{0,1,0,0},{0,0,0,1}}
Solution 12 : {{0,1,0,0},{0,0,0,1},{0,0,1,0},{1,0,0,0}}
Solution 13 : {{0,1,0,0},{0,0,0,1},{1,0,0,0},{0,0,1,0}}
Solution 14 : {{0,1,0,0},{0,0,1,0},{0,0,0,1},{1,0,0,0}}
Solution 15 : {{0,1,0,0},{0,0,1,0},{1,0,0,0},{0,0,0,1}}
Solution 16 : {{0,1,0,0},{1,0,0,0},{0,0,0,1},{0,0,1,0}}
Solution 17 : {{0,1,0,0},{1,0,0,0},{0,0,1,0},{0,0,0,1}}
Solution 18 : {{1,0,0,0},{0,0,0,1},{0,0,1,0},{0,1,0,0}}
Solution 19 : {{1,0,0,0},{0,0,0,1},{0,1,0,0},{0,0,1,0}}
Solution 20 : {{1,0,0,0},{0,0,1,0},{0,0,0,1},{0,1,0,0}}
Solution 21 : {{1,0,0,0},{0,0,1,0},{0,1,0,0},{0,0,0,1}}
Solution 22 : {{1,0,0,0},{0,1,0,0},{0,0,0,1},{0,0,1,0}}
Solution 23 : {{1,0,0,0},{0,1,0,0},{0,0,1,0},{0,0,0,1}}
```
{% endraw %}
> **NOTE**
> バイナリ変数の行列は `qbpp::Vector` クラスを使用したネストされたベクトルとして実装されています。
> 例えば、`qbpp::var("x",4,4)` は `qbpp::Vector<qbpp::Vector<qbpp::Var>>` オブジェクトを返します。
> 各 `qbpp::Var` オブジェクトは `x[i][j]` として表され、`sol` における `x[i][j]` の値は `sol(x[i][j])` または `x[i][j](sol)` のいずれかで取得できます。


## ベクトル関数と演算を使った置換行列のQUBO定式化
`qbpp::vector_sum()` を使用して、バイナリ変数の行列 `x` の行方向と列方向の和を計算できます：
- **`qbpp::vector_sum(x, 1)`**: `x` の各行の和を計算し、それらの和を含むサイズ `n` のベクトルを返します。
- **`qbpp::vector_sum(x, 0)`**: `x` の各列の和を計算し、それらの和を含むサイズ `n` のベクトルを返します。

> **Note**:
> 多次元配列 `x` と軸 `k` に対して、`qbpp::vector_sum(x, k)` は軸 `k` に沿った和を計算し、次元が1つ減った多次元配列を返します。
> 2次元配列（行列）`x` の場合、軸 `1` は行方向に、軸 `0` は列方向に対応します。

スカラー-ベクトル演算を使用して、各要素から1を引くことができます：
- **`qbpp::vector_sum(x, 1) - 1`**: 行方向の各和から1を引きます。
- **`qbpp::vector_sum(x, 0) - 1`**: 列方向の各和から1を引きます。

これら2つのサイズ `n` のベクトルに対して、`qbpp::sqr()` は各要素を2乗し、`qbpp::sum()` はすべての要素の和を計算します。

以下のQUBO++プログラムは、これらのベクトル関数と演算を使用してQUBO定式化を実装しています：
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::sum(qbpp::sqr(qbpp::vector_sum(x, 1) - 1)) +
           qbpp::sum(qbpp::sqr(qbpp::vector_sum(x, 0) - 1));
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (size_t k = 0; k < sols.size(); k++) {
    const auto& sol = sols[k];
    const auto& row = qbpp::onehot_to_int(x(sol), 1);
    const auto& column = qbpp::onehot_to_int(x(sol), 0);
    std::cout << "Solution " << k << ": " << row << ", " << column << std::endl;
  }
}
```
このプログラムでは、`x(sol)` は `sol` における `x` に割り当てられた値の行列を返します。これは整数のサイズの行列です。
`qbpp::onehot_to_int()` は軸に沿ったone-hotベクトルを対応する整数に変換します。
- **`qbpp::onehot_to_int(x(sol), 1)`**: 各行に対応する整数を計算し、4つの整数のベクトルとして返します。これが置換を表します。
- **`qbpp::onehot_to_int(x(sol), 0)`**: 各列に対応する整数を返し、4つの整数のベクトルとして返します。これが置換の逆を表します。
このプログラムはすべての置換とその逆を整数ベクトルとして以下のように出力します：
```
Solution 0: {3,2,1,0}, {3,2,1,0}
Solution 1: {3,2,0,1}, {2,3,1,0}
Solution 2: {3,1,2,0}, {3,1,2,0}
Solution 3: {3,1,0,2}, {2,1,3,0}
Solution 4: {3,0,2,1}, {1,3,2,0}
Solution 5: {3,0,1,2}, {1,2,3,0}
Solution 6: {2,3,1,0}, {3,2,0,1}
Solution 7: {2,3,0,1}, {2,3,0,1}
Solution 8: {2,1,3,0}, {3,1,0,2}
Solution 9: {2,1,0,3}, {2,1,0,3}
Solution 10: {2,0,3,1}, {1,3,0,2}
Solution 11: {2,0,1,3}, {1,2,0,3}
Solution 12: {1,3,2,0}, {3,0,2,1}
Solution 13: {1,3,0,2}, {2,0,3,1}
Solution 14: {1,2,3,0}, {3,0,1,2}
Solution 15: {1,2,0,3}, {2,0,1,3}
Solution 16: {1,0,3,2}, {1,0,3,2}
Solution 17: {1,0,2,3}, {1,0,2,3}
Solution 18: {0,3,2,1}, {0,3,2,1}
Solution 19: {0,3,1,2}, {0,2,3,1}
Solution 20: {0,2,3,1}, {0,3,1,2}
Solution 21: {0,2,1,3}, {0,2,1,3}
Solution 22: {0,1,3,2}, {0,1,3,2}
Solution 23: {0,1,2,3}, {0,1,2,3}
```

## 割当問題とそのQUBO定式化
$C = (c_{i,j})$ をサイズ $n \times n$ のコスト行列とします。
$C$ に対する**割当問題**は、総コストを最小化する置換
$p:\lbrace 0,1,\ldots, n-1\rbrace \rightarrow \lbrace 0,1,\ldots, n-1\rbrace$
を見つけることです：

$$
\begin{aligned}
 g(p) &= \sum_{i=0}^{n-1}c_{i,p(i)}
\end{aligned}
$$

この問題のQUBO定式化には、サイズ $n \times n$ の置換行列 $X = (x_{i,j})$ を使用して以下のように定義できます：

$$
\begin{aligned}
 g(X) &= \sum_{i=0}^{n-1}\sum_{j=0}^{n-1}c_{i,j}x_{i,j}
\end{aligned}
$$

明らかに、$X$ が置換 $p$ を表す場合にのみ $g(p) = g(X)$ が成り立ちます。

置換行列のQUBO定式化 $f(X)$ と総コスト $g(X)$ を組み合わせて、割当問題のQUBO定式化を得ます：

$$
\begin{aligned}
 h(X) &= P\cdot f(x)+g(x) \\
     &=P\left(\sum_{i=0}^{n-1}\left(1-\sum_{j=0}^{n-1}x_{i,j}\right)^2+\sum_{j=0}^{n-1}\left(1-\sum_{i=0}^{n-1}x_{i,j}\right)^2\right)+\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}c_{i,j}x_{i,j}
\end{aligned}
$$

ここで、$P$ は $f(X)$ にエンコードされた置換制約を優先するための十分に大きな正の定数です。

## 割当問題のQUBO++プログラム
これで割当問題のQUBO++プログラムを設計する準備が整いました。
このプログラムでは、サイズ $4\times4$ の固定行列 $C$ がネストされた `qbpp::Vector` として与えられます。
$f(X)$ と $g(X)$ の式はベクトル関数と演算を使用して定義されます。
ここで、`qbpp::vector_sum(x, 1) == 1` は等式が満たされた場合に最小値0を取るQUBO式を返します。
実際には、`qbpp::sqr(qbpp::vector_sum(x, 1) - 1)` と同じQUBO式を返します。
また、`c * x` は `c` と `x` の要素ごとの積を計算して得られる行列を返すため、`qbpp::sum(c * x)` は `g(X)` を返します。

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  qbpp::Vector<qbpp::Vector<uint32_t>> c = {
      {58, 73, 91, 44}, {62, 15, 87, 39}, {78, 56, 23, 94}, {11, 85, 68, 72}};
  auto x = qbpp::var("x", 4, 4);
  auto f = qbpp::sum(qbpp::vector_sum(x, 1) == 1) +
           qbpp::sum(qbpp::vector_sum(x, 0) == 1);
  auto g = qbpp::sum(c * x);
  auto h = 1000 * f + g;
  h.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(h);
  solver.time_limit(1.0);

  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  auto result = qbpp::onehot_to_int(x(sol), 1);
  std::cout << "Result : " << result << std::endl;
  for (size_t i = 0; i < result.size(); ++i) {
    std::cout << "c[" << i << "][" << result[i] << "] = " << c[i][result[i]]
              << std::endl;
  }
}
```

Easy Solverを使用して `h` の解を求めます。
`h` に対するEasy Solverオブジェクト `solver` について、`time_limit()` メンバ関数を呼び出して解の探索の制限時間を1.0秒に設定します。
得られた置換は `result` に格納され、選択された `c[i][j]` の値が順に出力されます。
このプログラムの出力は以下のとおりです：

{% raw %}
```
sol = 93:{{x[0][0],0},{x[0][1],0},{x[0][2],0},{x[0][3],1},{x[1][0],0},{x[1][1],1},{x[1][2],0},{x[1][3],0},{x[2][0],0},{x[2][1],0},{x[2][2],1},{x[2][3],0},{x[3][0],1},{x[3][1],0},{x[3][2],0},{x[3][3],0}}
Result : {3,1,2,0}
c[0][3] = 44
c[1][1] = 15
c[2][2] = 23
c[3][0] = 11
```
{% endraw %}
> **NOTE**
> 式 `f` と整数 `m` に対して、`f == m` は式 `qbpp::sqr(f - m)` を返します。
> これは等式 `f == m` が満たされる場合にのみ最小値0を取ります。

</div>
