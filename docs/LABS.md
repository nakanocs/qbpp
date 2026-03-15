---
layout: default
nav_exclude: true
title: "LABS Problem"
nav_order: 32
---
<div class="lang-en" markdown="1">

# Low-Autocorrelation Binary Sequence (LABS) Problem

The **Low Autocorrelation Binary Sequence (LABS)** problem aims to find a spin sequence$S=(s_i)$ ($s_i=\pm, 0\leq i\leq n-1$)
that minimizes its autocorrelation.
The autocorrelation of $S$ with alignment $d$ is defined as

$$
\left(\sum_{i=0}^{n-d-1}s_is_{i+d}\right)^2
$$


The LABS objective function is the sum of these autocorrelations over all alignments:

$$
\begin{aligned}
\text{LABS}(S) &= \sum_{d=1}^{n-1}\left(\sum_{i=0}^{n-d-1}s_is_{i+d}\right)^2
\end{aligned}
$$

The LABS problem is to find a sequence $S$ that minimizes $\text{LABS}(S)$.

## Spin-to-binary conversion
Since the solvers bundled with QUBO++ do not support spin variables directly,
we convert the spin variables to binary variables using the following transformation:

$$
\begin{aligned}
 s_i &\leftarrow 2s_i - 1
\end{aligned}
$$

After this conversion, each $s_i$ can be treated as a binary variable,
and HUBO solvers for binary variables can be used to find a solution to $\text{LABS}(S)$.

QUBO++ provides this conversion through the spin_to_binary() function.

## QUBO++ program for the LABS
The following QUBO++ program formulates and solves the LABS problem:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int n = 30;

  auto s = qbpp::var("s", n);
  auto labs = qbpp::expr();
  for (size_t d = 1; d < n; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < n - d; ++i) {
      temp += s[i] * s[i + d];
    }
    labs += qbpp::sqr(temp);
  }

  labs.spin_to_binary();
  labs.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(labs);
  solver.time_limit(10.0);
  solver.enable_best_energy_sols();
  auto sols = solver.search();
  size_t i = 0;
  for (const auto& sol : sols.best_sols()) {
    std::cout << i++ << ": LABS = ";
    std::cout << sol.energy() << " : ";
    for (size_t j = 0; j < n; ++j) {
      std::cout << (sol(s[j]) ? "+" : "-");
    }
    std::cout << std::endl;
  }
}
```
In this program, `s` stores a vector of `n` variables.
The `qbpp::Expr` object `labs` is constructed using a nested loop,
directly following the mathematical definition of the LABS objective.

Afterward, `labs` is converted into an expression over binary variables
using the `spin_to_binary()` function and simplified by
`simplify_as_binary()`.

The Easy Solver is then executed with a time limit of 10 seconds.
Since `enable_best_energy_sols()` is enabled, all solutions achieving
the minimum energy are stored in sols.

Using a range-based for loop, all best-energy solutions are printed.
A typical output of this program is:
```
0: LABS = 59 : -----+++++-++-++-+-+-+++--+++-
1: LABS = 59 : -+-++-+-+---+++-------+--++-++
2: LABS = 59 : -+-+--+-+---+++-------+--++-++
3: LABS = 59 : +-+-++-+-+++---+++++++-++--+--
4: LABS = 59 : --+--++-+++++++---+++-+-++-+-+
5: LABS = 59 : ----++++++-++-++-+-+-+++--+++-
6: LABS = 59 : +-+--+-+-+++---+++++++-++--+--
7: LABS = 59 : ++-++--+-------+++---+-+-++-+-
8: LABS = 59 : -+++--+++-+-+-++-++-++++++----
9: LABS = 59 : +---++---+-+-+--+--+-----+++++
```
In this run, multiple solutions achieving the same minimum LABS value are obtained.

</div>

<div class="lang-ja" markdown="1">

# 低自己相関バイナリ列 (LABS) 問題

**低自己相関バイナリ列 (Low Autocorrelation Binary Sequence, LABS)** 問題は、自己相関を最小化するスピン列 $S=(s_i)$ ($s_i=\pm, 0\leq i\leq n-1$) を見つけることを目的とします。
アラインメント $d$ における $S$ の自己相関は次のように定義されます：

$$
\left(\sum_{i=0}^{n-d-1}s_is_{i+d}\right)^2
$$


LABSの目的関数は、すべてのアラインメントにわたるこれらの自己相関の和です：

$$
\begin{aligned}
\text{LABS}(S) &= \sum_{d=1}^{n-1}\left(\sum_{i=0}^{n-d-1}s_is_{i+d}\right)^2
\end{aligned}
$$

LABS問題は $\text{LABS}(S)$ を最小化する列 $S$ を見つける問題です。

## スピンからバイナリへの変換
QUBO++に同梱されているソルバーはスピン変数を直接サポートしていないため、以下の変換を用いてスピン変数をバイナリ変数に変換します：

$$
\begin{aligned}
 s_i &\leftarrow 2s_i - 1
\end{aligned}
$$

この変換の後、各 $s_i$ はバイナリ変数として扱うことができ、バイナリ変数用のHUBOソルバーを使用して $\text{LABS}(S)$ の解を求めることができます。

QUBO++ではこの変換を spin_to_binary() 関数を通じて提供しています。

## LABSのQUBO++プログラム
以下のQUBO++プログラムはLABS問題を定式化し、求解します：
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int n = 30;

  auto s = qbpp::var("s", n);
  auto labs = qbpp::expr();
  for (size_t d = 1; d < n; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < n - d; ++i) {
      temp += s[i] * s[i + d];
    }
    labs += qbpp::sqr(temp);
  }

  labs.spin_to_binary();
  labs.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(labs);
  solver.time_limit(10.0);
  solver.enable_best_energy_sols();
  auto sols = solver.search();
  size_t i = 0;
  for (const auto& sol : sols.best_sols()) {
    std::cout << i++ << ": LABS = ";
    std::cout << sol.energy() << " : ";
    for (size_t j = 0; j < n; ++j) {
      std::cout << (sol(s[j]) ? "+" : "-");
    }
    std::cout << std::endl;
  }
}
```
このプログラムでは、`s` に `n` 個の変数のベクトルが格納されます。
`qbpp::Expr` オブジェクト `labs` はネストされたループを使用して構築され、LABSの目的関数の数学的定義に直接従っています。

その後、`labs` は `spin_to_binary()` 関数によりバイナリ変数上の式に変換され、`simplify_as_binary()` により簡略化されます。

次に、Easy Solver が時間制限10秒で実行されます。
`enable_best_energy_sols()` が有効になっているため、最小エネルギーを達成するすべての解が sols に格納されます。

範囲ベースのforループを使用して、最良エネルギーのすべての解が表示されます。
このプログラムの典型的な出力は以下のとおりです：
```
0: LABS = 59 : -----+++++-++-++-+-+-+++--+++-
1: LABS = 59 : -+-++-+-+---+++-------+--++-++
2: LABS = 59 : -+-+--+-+---+++-------+--++-++
3: LABS = 59 : +-+-++-+-+++---+++++++-++--+--
4: LABS = 59 : --+--++-+++++++---+++-+-++-+-+
5: LABS = 59 : ----++++++-++-++-+-+-+++--+++-
6: LABS = 59 : +-+--+-+-+++---+++++++-++--+--
7: LABS = 59 : ++-++--+-------+++---+-+-++-+-
8: LABS = 59 : -+++--+++-+-+-++-++-++++++----
9: LABS = 59 : +---++---+-+-+--+--+-----+++++
```
この実行では、同じ最小LABS値を達成する複数の解が得られています。

</div>
