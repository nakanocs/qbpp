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

## Dual-Matrix Domain Wall

The **Dual-Matrix Domain Wall** method uses two separate $n \times n$ binary matrices:
`x` with column-wise domain walls and `y` with row-wise domain walls.
The adjacent differences of each form one-hot vectors, and requiring these to match yields a permutation-like structure.
For details, see: [https://arxiv.org/abs/2308.01024](https://arxiv.org/abs/2308.01024)

{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const size_t n = 6;
  auto x = qbpp::var("x", n, n);
  auto y = qbpp::var("y", n, n);

  // x: column-wise domain wall, guard rows along dim=0
  auto xg = qbpp::concat(1, qbpp::concat(x, 0, 0), 0);
  auto x_diff = qbpp::head(xg, n + 1, 0) - qbpp::tail(xg, n + 1, 0);
  auto x_dw = qbpp::sum(qbpp::sqr(x_diff));

  // y: row-wise domain wall, guard columns along dim=1
  auto yg = qbpp::concat(1, qbpp::concat(y, 0, 1), 1);
  auto y_diff = qbpp::head(yg, n + 1, 1) - qbpp::tail(yg, n + 1, 1);
  auto y_dw = qbpp::sum(qbpp::sqr(y_diff));

  // Match: transpose(x_diff) == y_diff
  auto match = qbpp::sum(qbpp::sqr(qbpp::transpose(x_diff) - y_diff));

  // Fix total to get a non-trivial solution
  auto sum_constraint = qbpp::sum(x) == 21;

  auto f = x_dw + y_dw + match + sum_constraint;
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(static_cast<int64_t>(2 * n));
  auto sol = solver.search();

  std::cout << "energy = " << sol.energy() << std::endl;
  std::cout << "x:" << std::endl;
  for (size_t i = 0; i < n; ++i) {
    for (size_t j = 0; j < n; ++j) std::cout << sol(x[i][j]);
    std::cout << std::endl;
  }
  std::cout << "y:" << std::endl;
  for (size_t i = 0; i < n; ++i) {
    for (size_t j = 0; j < n; ++j) std::cout << sol(y[i][j]);
    std::cout << std::endl;
  }
}
```
{% endraw %}

### Key operations

- **`concat(1, concat(x, 0, 0), 0)`** (`dim=0`): Adds a guard row of 1s at the top and 0s at the bottom, making each column a domain wall.
- **`concat(1, concat(y, 0, 1), 1)`** (`dim=1`): Adds guard bits 1 and 0 to each row, making each row a domain wall.
- **`transpose(x_diff) - y_diff`**: Matches the column-wise one-hot matrix with the row-wise one-hot matrix.

### Output

```
energy = 12
x:
111111
011110
001110
001110
001110
001010
y:
100000
110000
111111
111110
111111
100000
```

The optimal energy is $2n = 12$. Each column of `x` is a domain wall (top to bottom), and each row of `y` is a domain wall (left to right). The one-hot matrices derived from their differences are transposes of each other.

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

## Dual-Matrix Domain Wall

**Dual-Matrix Domain Wall** 法は、2つの別々の $n \times n$ バイナリ行列を使用します:
`x` は列方向のドメインウォール、`y` は行方向のドメインウォール。
それぞれの隣接差分がone-hotベクトルを形成し、これらを一致させることで置換行列的な構造が得られます。
詳細は [https://arxiv.org/abs/2308.01024](https://arxiv.org/abs/2308.01024) を参照してください。

{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const size_t n = 6;
  auto x = qbpp::var("x", n, n);
  auto y = qbpp::var("y", n, n);

  // x: 列方向ドメインウォール、dim=0 でガード行追加
  auto xg = qbpp::concat(1, qbpp::concat(x, 0, 0), 0);
  auto x_diff = qbpp::head(xg, n + 1, 0) - qbpp::tail(xg, n + 1, 0);
  auto x_dw = qbpp::sum(qbpp::sqr(x_diff));

  // y: 行方向ドメインウォール、dim=1 でガードビット追加
  auto yg = qbpp::concat(1, qbpp::concat(y, 0, 1), 1);
  auto y_diff = qbpp::head(yg, n + 1, 1) - qbpp::tail(yg, n + 1, 1);
  auto y_dw = qbpp::sum(qbpp::sqr(y_diff));

  // 一致制約: transpose(x_diff) == y_diff
  auto match = qbpp::sum(qbpp::sqr(qbpp::transpose(x_diff) - y_diff));

  // 非自明な解のために総和を固定
  auto sum_constraint = qbpp::sum(x) == 21;

  auto f = x_dw + y_dw + match + sum_constraint;
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(static_cast<int64_t>(2 * n));
  auto sol = solver.search();

  std::cout << "energy = " << sol.energy() << std::endl;
  std::cout << "x:" << std::endl;
  for (size_t i = 0; i < n; ++i) {
    for (size_t j = 0; j < n; ++j) std::cout << sol(x[i][j]);
    std::cout << std::endl;
  }
  std::cout << "y:" << std::endl;
  for (size_t i = 0; i < n; ++i) {
    for (size_t j = 0; j < n; ++j) std::cout << sol(y[i][j]);
    std::cout << std::endl;
  }
}
```
{% endraw %}

### 主要な操作

- **`concat(1, concat(x, 0, 0), 0)`**（`dim=0`）: 全1のガード行を上に、全0のガード行を下に追加。各列がドメインウォールになります。
- **`concat(1, concat(y, 0, 1), 1)`**（`dim=1`）: 各行にガードビット 1 と 0 を追加。各行がドメインウォールになります。
- **`transpose(x_diff) - y_diff`**: 列方向のone-hot行列と行方向のone-hot行列を一致させます。

### 出力

```
energy = 12
x:
111111
011110
001110
001110
001110
001010
y:
100000
110000
111111
111110
111111
100000
```

最適エネルギーは $2n = 12$ です。`x` の各列は上から $1\cdots 1\, 0\cdots 0$ のドメインウォール、`y` の各行は左から $1\cdots 1\, 0\cdots 0$ のドメインウォールで、それぞれの差分から得られるone-hot行列が互いに転置の関係にあります。

</div>
