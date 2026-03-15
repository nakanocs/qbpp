---
layout: default
nav_exclude: true
title: "Cutting Stock"
nav_order: 33
---
<div class="lang-en" markdown="1">

# Cutting Stock Problem
Suppose that we are given $M$ identical bars of fixed length $L$, and a set of $N$ orders specified by pairs $(l_j,c_j)$
($0\leq j\leq N−1$), where $l_j$ is the required length and $c_j$ is the required quantity of order $j$.
The **cutting stock problem** aims to determine how the $M$ bars can be cut to satisfy all orders.

In general, the cutting stock problem is formulated as a minimization problem that seeks to minimize the number of used bars.
For simplicity, in this example we consider the feasibility problem of determining whether the $M$ bars can fulfill all $N$ orders.


Let $x_{i,j}$ ($0\leq i\leq M-1, 0\leq j\leq N-1$) denote the number of pieces of order $j$ cut from bar $i$.
The following constraints must be satisfied.

### Order Constraint:
For each order $j$, the total number of pieces assigned across all bars must equal $c_j$:

$$
\begin{aligned}
 \sum_{i=0}^{M-1}x_{i,j} &= c_j & &(0\leq j\leq N-1)
\end{aligned}
$$

### Bar Constraint
For each bar $i$, the total length of the assigned pieces must not exceed $L$:

$$
\begin{aligned}
 \sum_{j=0}^{N-1}l_jx_{i,j} &\leq  L & &(0\leq i\leq M-1)
\end{aligned}
$$

## QUBO++ program
The following QUBO++ program finds a feasible cutting plan using
$M=6$ bars of length $L=60$ and the following $N=4$ orders:

| Order $j$ | 0 | 1 | 2 | 3|
|---|---|---|---|---|
| Length $l_j$ | 13 | 23 | 8 | 11 |
| Quantity $c_j$ | 10 | 4 | 8 | 6 |

The QUBO++ program for this cutting stock problem is as follows:
```cpp

#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int L = 60;
  const qbpp::Vector<int> l = {13, 23, 8, 11};
  const qbpp::Vector<int> c = {10, 4, 8, 6};
  const size_t N = l.size();
  const size_t M = 6;

  auto x = qbpp::var_int("x", M, N) == 0;
  for (size_t i = 0; i < M; i++) {
    for (size_t j = 0; j < N; j++) {
      x[i][j] = 0 <= qbpp::var_int() <= c[j];
    }
  }

  auto order_fulfilled_count = qbpp::vector_sum(x, 0);
  auto order_constraint = order_fulfilled_count == c;

  auto bar_length_used = qbpp::expr(M);
  for (size_t i = 0; i < M; i++) {
    bar_length_used[i] = qbpp::sum(x[i] * l);
  }
  auto bar_constraint = 0 <= bar_length_used <= L;

  auto f = qbpp::sum(order_constraint) + qbpp::sum(bar_constraint);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(10.0);
  solver.target_energy(0);
  auto sol = solver.search();
  for (size_t i = 0; i < M; i++) {
    std::cout << "Bar " << i << ":  ";
    for (size_t j = 0; j < N; j++) {
      std::cout << sol(x[i][j]) << "  ";
    }
    std::cout << " used = " << sol(bar_length_used[i])
              << ", waste = " << L - sol(bar_length_used[i]) << std::endl;
  }
  for (size_t j = 0; j < N; j++) {
    std::cout << "Order " << j
              << " fulfilled = " << sol(order_fulfilled_count[j])
              << ", required = " << c[j] << std::endl;
  }
}
```
The program creates an `M`$\times$`N` matrix `x` of integer variables, initialized to the constant value 0.
The nested for loops assign to each entry `x[i][j]` a bounded integer variable, `0 <= qbpp::var_int(...) <= c[j]`, so that `x[i][j]` takes a non-negative integer value no greater than `c[j]`.

The constraints are defined as follows:
- `order_fulfilled_count`: a vector of $N$ expressions where `order_fulfilled_count[j]` represents the total number of pieces produced for order $j$.
- `order_constraint`: a vector of $N$ constraint expressions enforcing `order_fulfilled_count[j] == c[j]` for all $j$.
- `bar_length_used`: a vector of $M$ expressions where `bar_length_used[i]` represents the total length used in bar $i$.
- `bar_constraint`: a vector of $M$ constraint expressions enforcing `0 <= bar_length_used[i] <= L` for all $i$.
- `f`: the sum of all constraint expressions. After calling `f.simplify_as_binary()`, the Easy Solver searches for a solution with target energy 0 (i.e., all constraints satisfied).

The following output is an example feasible solution:
```
Bar 0:  2  0  0  3   used = 59, waste = 1
Bar 1:  4  0  1  0   used = 60, waste = 0
Bar 2:1  1  3  0   used = 60, waste = 0
Bar 3:  0  0  4  2   used = 54, waste = 6
Bar 4:  2  1  0  1   used = 60, waste = 0
Bar 5:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 10, required = 10
Order 1 fulfilled = 4, required = 4
Order 2 fulfilled = 8, required = 8
Order 3 fulfilled = 6, required = 6
```
We observe that all $N=4$ orders are fulfilled using $M=6$ bars.

If we set $M=5$, the solver returns the following infeasible solution, in which not all orders are satisfied:
```
Bar 0:  4  0  1  0   used = 60, waste = 0
Bar 1:  0  0  6  1   used = 59, waste = 1
Bar 2:  2  1  0  1   used = 60, waste = 0
Bar 3:  2  0  0  3   used = 59, waste = 1
Bar 4:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 9, required = 10
Order 1 fulfilled = 3, required = 4
Order 2 fulfilled = 7, required = 8
Order 3 fulfilled = 5, required = 6
```

</div>

<div class="lang-ja" markdown="1">

# 切出し問題（Cutting Stock Problem）
固定長 $L$ の同一の棒が $M$ 本と、ペア $(l_j,c_j)$ ($0\leq j\leq N-1$) で指定される $N$ 件の注文が与えられるとします。ここで $l_j$ は必要な長さ、$c_j$ は注文 $j$ の必要数量です。
**切出し問題**は、$M$ 本の棒をどのように切断すればすべての注文を満たせるかを決定することを目的とします。

一般に、切出し問題は使用する棒の本数を最小化する最小化問題として定式化されます。
簡単のため、この例では $M$ 本の棒で $N$ 件のすべての注文を満たせるかどうかを判定する実行可能性問題を考えます。


$x_{i,j}$ ($0\leq i\leq M-1, 0\leq j\leq N-1$) を棒 $i$ から切り出す注文 $j$ のピース数とします。
以下の制約を満たす必要があります。

### 注文制約：
各注文 $j$ について、すべての棒に割り当てられたピースの合計は $c_j$ に等しくなければなりません：

$$
\begin{aligned}
 \sum_{i=0}^{M-1}x_{i,j} &= c_j & &(0\leq j\leq N-1)
\end{aligned}
$$

### 棒制約
各棒 $i$ について、割り当てられたピースの合計長は $L$ を超えてはなりません：

$$
\begin{aligned}
 \sum_{j=0}^{N-1}l_jx_{i,j} &\leq  L & &(0\leq i\leq M-1)
\end{aligned}
$$

## QUBO++プログラム
以下のQUBO++プログラムは、長さ $L=60$ の $M=6$ 本の棒と以下の $N=4$ 件の注文に対する実行可能な切断計画を求めます：

| 注文 $j$ | 0 | 1 | 2 | 3|
|---|---|---|---|---|
| 長さ $l_j$ | 13 | 23 | 8 | 11 |
| 数量 $c_j$ | 10 | 4 | 8 | 6 |

この切出し問題のQUBO++プログラムは以下のとおりです：
```cpp

#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int L = 60;
  const qbpp::Vector<int> l = {13, 23, 8, 11};
  const qbpp::Vector<int> c = {10, 4, 8, 6};
  const size_t N = l.size();
  const size_t M = 6;

  auto x = qbpp::var_int("x", M, N) == 0;
  for (size_t i = 0; i < M; i++) {
    for (size_t j = 0; j < N; j++) {
      x[i][j] = 0 <= qbpp::var_int() <= c[j];
    }
  }

  auto order_fulfilled_count = qbpp::vector_sum(x, 0);
  auto order_constraint = order_fulfilled_count == c;

  auto bar_length_used = qbpp::expr(M);
  for (size_t i = 0; i < M; i++) {
    bar_length_used[i] = qbpp::sum(x[i] * l);
  }
  auto bar_constraint = 0 <= bar_length_used <= L;

  auto f = qbpp::sum(order_constraint) + qbpp::sum(bar_constraint);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(10.0);
  solver.target_energy(0);
  auto sol = solver.search();
  for (size_t i = 0; i < M; i++) {
    std::cout << "Bar " << i << ":  ";
    for (size_t j = 0; j < N; j++) {
      std::cout << sol(x[i][j]) << "  ";
    }
    std::cout << " used = " << sol(bar_length_used[i])
              << ", waste = " << L - sol(bar_length_used[i]) << std::endl;
  }
  for (size_t j = 0; j < N; j++) {
    std::cout << "Order " << j
              << " fulfilled = " << sol(order_fulfilled_count[j])
              << ", required = " << c[j] << std::endl;
  }
}
```
このプログラムは整数変数の `M`$\times$`N` 行列 `x` を作成し、定数値0で初期化します。
ネストされたforループにより、各エントリ `x[i][j]` に上限付き整数変数 `0 <= qbpp::var_int(...) <= c[j]` が割り当てられ、`x[i][j]` は `c[j]` 以下の非負整数値をとります。

制約は以下のように定義されます：
- `order_fulfilled_count`: $N$ 個の式のベクトルで、`order_fulfilled_count[j]` は注文 $j$ について生産されたピースの合計数を表します。
- `order_constraint`: すべての $j$ に対して `order_fulfilled_count[j] == c[j]` を強制する $N$ 個の制約式のベクトルです。
- `bar_length_used`: $M$ 個の式のベクトルで、`bar_length_used[i]` は棒 $i$ で使用された合計長を表します。
- `bar_constraint`: すべての $i$ に対して `0 <= bar_length_used[i] <= L` を強制する $M$ 個の制約式のベクトルです。
- `f`: すべての制約式の和です。`f.simplify_as_binary()` を呼び出した後、Easy Solverはターゲットエネルギー0（すなわちすべての制約が満たされた状態）の解を探索します。

以下の出力は実行可能解の例です：
```
Bar 0:  2  0  0  3   used = 59, waste = 1
Bar 1:  4  0  1  0   used = 60, waste = 0
Bar 2:1  1  3  0   used = 60, waste = 0
Bar 3:  0  0  4  2   used = 54, waste = 6
Bar 4:  2  1  0  1   used = 60, waste = 0
Bar 5:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 10, required = 10
Order 1 fulfilled = 4, required = 4
Order 2 fulfilled = 8, required = 8
Order 3 fulfilled = 6, required = 6
```
$N=4$ 件のすべての注文が $M=6$ 本の棒で満たされていることがわかります。

$M=5$ に設定すると、ソルバーは以下の実行不可能な解を返し、すべての注文が満たされていません：
```
Bar 0:  4  0  1  0   used = 60, waste = 0
Bar 1:  0  0  6  1   used = 59, waste = 1
Bar 2:  2  1  0  1   used = 60, waste = 0
Bar 3:  2  0  0  3   used = 59, waste = 1
Bar 4:  1  2  0  0   used = 59, waste = 1
Order 0 fulfilled = 9, required = 10
Order 1 fulfilled = 3, required = 4
Order 2 fulfilled = 7, required = 8
Order 3 fulfilled = 5, required = 6
```

</div>
