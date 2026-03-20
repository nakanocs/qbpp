---
layout: default
nav_exclude: true
title: "Slice and Concat Functions"
nav_order: 19
---
<div class="lang-en" markdown="1">
# Slice and Concat Functions

QUBO++ provides slice and concat functions for manipulating vectors of variables and expressions.
This page demonstrates these functions through a practical example: **domain wall encoding**.

## Domain Wall Encoding

A **domain wall** is a binary pattern of the form $1\cdots 1\, 0\cdots 0$,
where all 1s appear before all 0s.
For $n$ binary variables, there are exactly $n+1$ domain wall patterns
(including the all-1 and all-0 patterns),
so a domain wall can represent an integer in the range $[0, n]$.

Using `concat`, `head`, `tail`, and `sqr`, we can construct a QUBO expression
whose minimum-energy solutions are exactly the domain wall patterns.

## QUBO++ program

{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  const size_t n = 8;
  auto x = qbpp::var("x", n);

  // y = (1, x[0], x[1], ..., x[n-1], 0)
  auto y = qbpp::concat(1, qbpp::concat(x, 0));

  // Adjacent difference
  auto diff = qbpp::head(y, n + 1) - qbpp::tail(y, n + 1);

  // Penalty: minimum value 1 iff domain wall
  auto f = qbpp::sum(qbpp::sqr(diff));
  f.simplify_as_binary();

  std::cout << "f = " << f << std::endl;

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();

  std::cout << "energy = " << sol.energy() << std::endl;
  std::cout << "solutions = " << sol.all_solutions().size() << std::endl;
  for (const auto& s : sol.all_solutions()) {
    for (size_t i = 0; i < n; ++i) std::cout << s(x[i]);
    std::cout << "  (sum = " << s(qbpp::sum(x)) << ")" << std::endl;
  }
}
```
{% endraw %}

### How it works

**Step 1: Guard bits with `concat`**

`concat(1, concat(x, 0))` constructs the extended vector:

$$
y = (1,\; x_0,\; x_1,\; \ldots,\; x_{n-1},\; 0)
$$

The guard bit 1 at the beginning and 0 at the end ensure that the domain wall pattern is bounded.

**Step 2: Adjacent difference with `head` and `tail`**

`head(y, n+1) - tail(y, n+1)` computes the element-wise difference between consecutive elements:

$$
\text{diff}_i = y_i - y_{i+1} \quad (0 \le i \le n)
$$

**Step 3: Penalty with `sqr` and `sum`**

`sum(sqr(diff))` computes $\sum_{i=0}^{n} (y_i - y_{i+1})^2$.
Since each $y_i \in \{0, 1\}$, each squared difference is either 0 or 1.
The sum counts the number of transitions (changes from 0 to 1 or 1 to 0) in $y$.

A domain wall pattern has exactly **one** transition (from 1 to 0),
so the minimum energy is **1**, and all $n+1$ domain wall patterns achieve this minimum.

### Output

```
f = 1 +2*x[1] +2*x[2] +2*x[3] +2*x[4] +2*x[5] +2*x[6] +2*x[7] -2*x[0]*x[1] -2*x[1]*x[2] -2*x[2]*x[3] -2*x[3]*x[4] -2*x[4]*x[5] -2*x[5]*x[6] -2*x[6]*x[7]
energy = 1
solutions = 9
00000000  (sum = 0)
10000000  (sum = 1)
11000000  (sum = 2)
11100000  (sum = 3)
11110000  (sum = 4)
11111000  (sum = 5)
11111100  (sum = 6)
11111110  (sum = 7)
11111111  (sum = 8)
```

All 9 optimal solutions are domain wall patterns, representing integers 0 through 8.

## 2D Domain Wall Encoding

The same technique extends to two-dimensional arrays using `concat` with a dimension parameter.
Given a 2D variable array `x` of size $m \times n$, we can enforce domain wall constraints on every row
using `concat(1, concat(x, 0, 1), 1)` where `dim=1` applies guard bits to each row simultaneously.

For column-wise domain walls, we `transpose` the array and apply the same pattern.

{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  const size_t rows = 3, cols = 3;
  auto x = qbpp::var("x", rows, cols);

  // Row domain wall: guard bits along dim=1
  auto yr = qbpp::concat(1, qbpp::concat(x, 0, 1), 1);
  auto row_diff = qbpp::head(yr, cols + 1, 1) - qbpp::tail(yr, cols + 1, 1);
  auto row_dw = qbpp::sum(qbpp::sqr(row_diff));

  // Column domain wall: transpose, then same pattern
  auto xt = qbpp::transpose(x);
  auto yc = qbpp::concat(1, qbpp::concat(xt, 0, 1), 1);
  auto col_diff = qbpp::head(yc, rows + 1, 1) - qbpp::tail(yc, rows + 1, 1);
  auto col_dw = qbpp::sum(qbpp::sqr(col_diff));

  auto f = row_dw + col_dw;
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();

  std::cout << "energy = " << sol.energy() << std::endl;
  std::cout << "solutions = " << sol.all_solutions().size() << std::endl;
  for (const auto& s : sol.all_solutions()) {
    for (size_t i = 0; i < rows; ++i) {
      for (size_t j = 0; j < cols; ++j) std::cout << s(x[i][j]);
      std::cout << std::endl;
    }
    std::cout << std::endl;
  }
}
```
{% endraw %}

Each row must be a domain wall ($1\cdots 1\, 0\cdots 0$) and each column must independently be a domain wall ($1\cdots 1\, 0\cdots 0$ from top to bottom).

### Key operations

- **`concat(1, concat(x, 0, 1), 1)`**: Adds guard bits to each row of a 2D array. The `dim=1` parameter means the operation applies along the column (innermost) dimension.
- **`head(yr, cols+1, 1)` / `tail(yr, cols+1, 1)`**: Extracts the first/last `cols+1` elements from each row.
- **`transpose(x)`**: Transposes the matrix so that column-wise operations become row-wise.

### Output

```
energy = 6
solutions = 20
000
000
000

100
000
000

110
000
000

111
000
000

100
100
000

110
100
000

111
100
000

110
110
000

111
110
000

111
111
000

100
100
100

110
100
100

111
100
100

110
110
100

111
110
100

111
111
100

110
110
110

111
110
110

111
111
110

111
111
111
```

All 20 optimal solutions satisfy both row and column domain wall constraints.
Each row is $1\cdots 1\, 0\cdots 0$ (left to right) and each column is $1\cdots 1\, 0\cdots 0$ (top to bottom).

</div>

<div class="lang-ja" markdown="1">
# スライス関数と連結関数

QUBO++ は、変数や式のベクトルを操作するためのスライス関数と連結関数を提供しています。
このページでは、実用的な例である**ドメインウォール符号化**を通じてこれらの関数を紹介します。

## ドメインウォール符号化

**ドメインウォール**とは、$1\cdots 1\, 0\cdots 0$ の形をしたバイナリパターンで、
すべての1がすべての0の前に現れます。
$n$ 個のバイナリ変数に対して、ドメインウォールパターンは正確に $n+1$ 個あり
（全1パターンと全0パターンを含む）、
$[0, n]$ の範囲の整数を表現できます。

`concat`、`head`、`tail`、`sqr` を使って、
最小エネルギー解がちょうどドメインウォールパターンとなるQUBO式を構築できます。

## QUBO++ プログラム

{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  const size_t n = 8;
  auto x = qbpp::var("x", n);

  // y = (1, x[0], x[1], ..., x[n-1], 0)
  auto y = qbpp::concat(1, qbpp::concat(x, 0));

  // 隣接差分
  auto diff = qbpp::head(y, n + 1) - qbpp::tail(y, n + 1);

  // ペナルティ: ドメインウォールのとき最小値 1
  auto f = qbpp::sum(qbpp::sqr(diff));
  f.simplify_as_binary();

  std::cout << "f = " << f << std::endl;

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();

  std::cout << "energy = " << sol.energy() << std::endl;
  std::cout << "solutions = " << sol.all_solutions().size() << std::endl;
  for (const auto& s : sol.all_solutions()) {
    for (size_t i = 0; i < n; ++i) std::cout << s(x[i]);
    std::cout << "  (sum = " << s(qbpp::sum(x)) << ")" << std::endl;
  }
}
```
{% endraw %}

### 仕組み

**ステップ 1: `concat` によるガードビット**

`concat(1, concat(x, 0))` で拡張ベクトルを構築します:

$$
y = (1,\; x_0,\; x_1,\; \ldots,\; x_{n-1},\; 0)
$$

先頭のガードビット 1 と末尾の 0 により、ドメインウォールパターンが境界で正しく制約されます。

**ステップ 2: `head` と `tail` による隣接差分**

`head(y, n+1) - tail(y, n+1)` で連続する要素間の差分を計算します:

$$
\text{diff}_i = y_i - y_{i+1} \quad (0 \le i \le n)
$$

**ステップ 3: `sqr` と `sum` によるペナルティ**

`sum(sqr(diff))` は $\sum_{i=0}^{n} (y_i - y_{i+1})^2$ を計算します。
各 $y_i \in \{0, 1\}$ なので、各二乗差分は 0 または 1 です。
この和は $y$ における遷移（0 から 1 または 1 から 0 への変化）の回数を数えます。

ドメインウォールパターンは遷移が正確に**1回**（1 から 0 への変化）なので、
最小エネルギーは **1** であり、$n+1$ 個すべてのドメインウォールパターンがこの最小値を達成します。

### 出力

```
f = 1 +2*x[1] +2*x[2] +2*x[3] +2*x[4] +2*x[5] +2*x[6] +2*x[7] -2*x[0]*x[1] -2*x[1]*x[2] -2*x[2]*x[3] -2*x[3]*x[4] -2*x[4]*x[5] -2*x[5]*x[6] -2*x[6]*x[7]
energy = 1
solutions = 9
00000000  (sum = 0)
10000000  (sum = 1)
11000000  (sum = 2)
11100000  (sum = 3)
11110000  (sum = 4)
11111000  (sum = 5)
11111100  (sum = 6)
11111110  (sum = 7)
11111111  (sum = 8)
```

9つの最適解はすべてドメインウォールパターンで、整数 0 から 8 を表現しています。

## 2次元ドメインウォール符号化

次元パラメータ付きの `concat` を使えば、同じ手法を2次元配列に拡張できます。
サイズ $m \times n$ の2次元変数配列 `x` に対して、
`concat(1, concat(x, 0, 1), 1)` で各行に同時にガードビットを追加できます（`dim=1` は列方向への操作）。

列方向のドメインウォールには、`transpose` で転置してから同じパターンを適用します。

{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  const size_t rows = 3, cols = 3;
  auto x = qbpp::var("x", rows, cols);

  // 行ドメインウォール: dim=1 でガードビット追加
  auto yr = qbpp::concat(1, qbpp::concat(x, 0, 1), 1);
  auto row_diff = qbpp::head(yr, cols + 1, 1) - qbpp::tail(yr, cols + 1, 1);
  auto row_dw = qbpp::sum(qbpp::sqr(row_diff));

  // 列ドメインウォール: 転置して同じパターン
  auto xt = qbpp::transpose(x);
  auto yc = qbpp::concat(1, qbpp::concat(xt, 0, 1), 1);
  auto col_diff = qbpp::head(yc, rows + 1, 1) - qbpp::tail(yc, rows + 1, 1);
  auto col_dw = qbpp::sum(qbpp::sqr(col_diff));

  auto f = row_dw + col_dw;
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();

  std::cout << "energy = " << sol.energy() << std::endl;
  std::cout << "solutions = " << sol.all_solutions().size() << std::endl;
  for (const auto& s : sol.all_solutions()) {
    for (size_t i = 0; i < rows; ++i) {
      for (size_t j = 0; j < cols; ++j) std::cout << s(x[i][j]);
      std::cout << std::endl;
    }
    std::cout << std::endl;
  }
}
```
{% endraw %}

各行がドメインウォール（$1\cdots 1\, 0\cdots 0$）であり、各列も独立にドメインウォール（上から $1\cdots 1\, 0\cdots 0$）となる制約です。

### 主要な操作

- **`concat(1, concat(x, 0, 1), 1)`**: 2次元配列の各行にガードビットを追加します。`dim=1` パラメータにより、列（内側の）次元に沿って操作が適用されます。
- **`head(yr, cols+1, 1)` / `tail(yr, cols+1, 1)`**: 各行から先頭/末尾の `cols+1` 要素を抽出します。
- **`transpose(x)`**: 行列を転置し、列方向の操作を行方向の操作に変換します。

### 出力

```
energy = 6
solutions = 20
000
000
000

100
000
000

110
000
000

111
000
000

100
100
000

110
100
000

111
100
000

110
110
000

111
110
000

111
111
000

100
100
100

110
100
100

111
100
100

110
110
100

111
110
100

111
111
100

110
110
110

111
110
110

111
111
110

111
111
111
```

20個の最適解はすべて行・列のドメインウォール制約を満たしています。
各行は $1\cdots 1\, 0\cdots 0$（左から右）、各列は $1\cdots 1\, 0\cdots 0$（上から下）となっています。

</div>
