---
layout: default
nav_exclude: true
title: "NAE-SAT"
nav_order: 45
---
<div class="lang-en" markdown="1">
# NAE-SAT (Not-All-Equal Satisfiability)

The **Not-All-Equal Satisfiability (NAE-SAT)** problem is a variant of the Boolean satisfiability problem (SAT).
Given a set of Boolean variables $x_0, x_1, \ldots, x_{n-1}$ and a collection of clauses, each clause is **satisfied** if and only if at least one variable in the clause is True **and** at least one is False.
In other words, a clause is violated when all its variables have the same value (all True or all False).

NAE-SAT is NP-complete and arises in applications such as hypergraph coloring and constraint satisfaction.

## HUBO formulation

For $n$ binary variables $x_0, x_1, \ldots, x_{n-1}$ and $m$ clauses $C_0, C_1, \ldots, C_{m-1}$, the NAE-SAT constraint can be formulated as a HUBO (Higher-order Unconstrained Binary Optimization) expression.

### NAE constraint

For each clause $C_k = \lbrace x_{i_1}, x_{i_2}, \ldots, x_{i_s} \rbrace$, we define:

- **All-True penalty**: the product $x_{i_1} \cdot x_{i_2} \cdots x_{i_s}$ equals 1 only when all variables in the clause are True.
- **All-False penalty**: the product $\overline{x}_{i_1} \cdot \overline{x}_{i_2} \cdots \overline{x}_{i_s}$ equals 1 only when all variables are False, where $\overline{x}_i$ denotes the negated literal ($\overline{x}_i = 1 - x_i$).

The constraint for the entire instance is:

$$
\text{constraint} = \sum_{k=0}^{m-1} \Bigl( \prod_{j \in C_k} x_j + \prod_{j \in C_k} \overline{x}_j \Bigr)
$$

This expression equals 0 if and only if every clause is NAE-satisfied.

### Objective (optional)

As a secondary objective, we can balance the number of True and False variables:

$$
\text{objective} = \Bigl(2\sum_{i=0}^{n-1} x_i - n\Bigr)^2
$$

This is minimized (reaching 0 when $n$ is even, or 1 when $n$ is odd) when the True/False count is as balanced as possible.

### HUBO expression

The final HUBO expression combines the constraint and objective with a penalty weight $P$:

$$
f = \text{objective} + P \times \text{constraint}
$$

where $P$ must be large enough (e.g., $P = n^2 + 1$) to ensure that constraint satisfaction is prioritized over objective minimization.

## QUBO++ formulation

QUBO++ handles negated literals $\overline{x}_i$ (written as `~x[i]`) natively, which makes the NAE-SAT formulation natural and efficient.
The following program defines a simple NAE-SAT instance with 5 variables and 4 clauses of size 3, solves it using EasySolver, and verifies the result.

```cpp
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int n = 5;

  // Clauses: each clause is a set of variable indices
  std::vector<std::vector<int>> clauses = {
      {0, 1, 2},
      {1, 2, 3},
      {2, 3, 4},
      {0, 3, 4}
  };

  // Create binary variables
  auto x = qbpp::var("x", n);

  // NAE constraint: penalty if all-true or all-false
  auto constraint = qbpp::Expr(0);
  for (const auto& clause : clauses) {
    auto all_true = qbpp::Expr(1);
    auto all_false = qbpp::Expr(1);
    for (int idx : clause) {
      all_true *= x[idx];
      all_false *= ~x[idx];
    }
    constraint += all_true + all_false;
  }

  // Objective: balance True/False count
  auto s = qbpp::sum(x);
  auto objective = (2 * s - n) * (2 * s - n);

  // HUBO expression with penalty weight
  int penalty_weight = n * n + 1;
  auto f = (objective + penalty_weight * constraint).simplify_as_binary();

  // Solve
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(1);  // n=5 is odd, so best balance gives (2*s-n)^2 = 1
  auto sol = solver.search();

  // Print results
  std::cout << "Energy = " << sol.energy() << std::endl;
  std::cout << "Assignment: ";
  for (int i = 0; i < n; ++i) {
    std::cout << "x[" << i << "]=" << sol(x[i]) << " ";
  }
  std::cout << std::endl;

  std::cout << "constraint = " << sol(constraint) << std::endl;
  std::cout << "objective  = " << sol(objective) << std::endl;

  // Verify: check each clause
  bool all_satisfied = true;
  for (size_t k = 0; k < clauses.size(); ++k) {
    int sum_val = 0;
    for (int idx : clauses[k]) {
      sum_val += sol(x[idx]);
    }
    bool satisfied = (sum_val > 0) &&
                     (sum_val < static_cast<int>(clauses[k].size()));
    std::cout << "Clause " << k << ": "
              << (satisfied ? "satisfied" : "VIOLATED") << std::endl;
    if (!satisfied) all_satisfied = false;
  }
  std::cout << "All clauses NAE-satisfied: "
            << (all_satisfied ? "Yes" : "No") << std::endl;
}
```

### Example output
```
Energy = 1
Assignment: x[0]=1 x[1]=0 x[2]=1 x[3]=0 x[4]=1
constraint = 0
objective  = 1
Clause 0: satisfied
Clause 1: satisfied
Clause 2: satisfied
Clause 3: satisfied
All clauses NAE-satisfied: Yes
```

The solver finds an assignment where `constraint = 0`, meaning all four clauses are NAE-satisfied.
The objective value is 1 because $n = 5$ is odd, so a perfect True/False balance (e.g., 3 True and 2 False) gives $(2 \times 3 - 5)^2 = 1$.

### Key points
- **Negated literals**: `~x[i]` is used directly in QUBO++ to express $\overline{x}_i$ without expanding to $1 - x_i$. This keeps the HUBO expression compact.
- **Higher-order terms**: Each clause of size $s$ produces degree-$s$ terms (e.g., $x_0 x_1 x_2$ for a 3-literal clause). QUBO++ handles HUBO expressions natively without requiring quadratization.
- **Penalty weight**: $P = n^2 + 1$ ensures that any constraint violation outweighs the maximum possible objective value.

</div>

<div class="lang-ja" markdown="1">
# NAE-SAT (全不等充足可能性問題)

**全不等充足可能性問題 (NAE-SAT: Not-All-Equal Satisfiability)** はブール充足可能性問題 (SAT) の変種です。
ブール変数 $x_0, x_1, \ldots, x_{n-1}$ と節の集合が与えられたとき、各節は少なくとも1つの変数が True であり、**かつ**少なくとも1つが False である場合にのみ**充足**されます。
つまり、節内のすべての変数が同じ値（すべて True またはすべて False）のとき、その節は違反となります。

NAE-SAT は NP 完全であり、ハイパーグラフ彩色や制約充足問題などの応用があります。

## HUBO による定式化

$n$ 個のバイナリ変数 $x_0, x_1, \ldots, x_{n-1}$ と $m$ 個の節 $C_0, C_1, \ldots, C_{m-1}$ に対して、NAE-SAT 制約は HUBO（高次制約なしバイナリ最適化）式として定式化できます。

### NAE 制約

各節 $C_k = \lbrace x_{i_1}, x_{i_2}, \ldots, x_{i_s} \rbrace$ に対して以下を定義します：

- **全 True ペナルティ**: 積 $x_{i_1} \cdot x_{i_2} \cdots x_{i_s}$ は節内のすべての変数が True のときにのみ 1 になります。
- **全 False ペナルティ**: 積 $\overline{x}_{i_1} \cdot \overline{x}_{i_2} \cdots \overline{x}_{i_s}$ はすべての変数が False のときにのみ 1 になります。ここで $\overline{x}_i$ は否定リテラル（$\overline{x}_i = 1 - x_i$）を表します。

インスタンス全体の制約は以下の通りです：

$$
\text{constraint} = \sum_{k=0}^{m-1} \Bigl( \prod_{j \in C_k} x_j + \prod_{j \in C_k} \overline{x}_j \Bigr)
$$

この式はすべての節が NAE 充足されている場合にのみ 0 になります。

### 目的関数（オプション）

副次的な目的として、True と False の変数数のバランスを取ることができます：

$$
\text{objective} = \Bigl(2\sum_{i=0}^{n-1} x_i - n\Bigr)^2
$$

これは True/False の数がなるべく均等なとき最小化されます（$n$ が偶数のとき 0、$n$ が奇数のとき 1）。

### HUBO 式

最終的な HUBO 式は、制約と目的関数をペナルティ重み $P$ で組み合わせます：

$$
f = \text{objective} + P \times \text{constraint}
$$

ここで $P$ は十分大きく（例えば $P = n^2 + 1$）、制約の充足が目的関数の最小化よりも優先されるようにします。

## QUBO++ による定式化

QUBO++ は否定リテラル $\overline{x}_i$（`~x[i]` と記述）をネイティブに扱えるため、NAE-SAT の定式化が自然かつ効率的に行えます。
以下のプログラムは、5 変数・4 節（各節サイズ 3）の簡単な NAE-SAT インスタンスを定義し、EasySolver で解き、結果を検証します。

```cpp
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  const int n = 5;

  // 節: 各節は変数インデックスの集合
  std::vector<std::vector<int>> clauses = {
      {0, 1, 2},
      {1, 2, 3},
      {2, 3, 4},
      {0, 3, 4}
  };

  // バイナリ変数の作成
  auto x = qbpp::var("x", n);

  // NAE 制約: 全 True または全 False のときペナルティ
  auto constraint = qbpp::Expr(0);
  for (const auto& clause : clauses) {
    auto all_true = qbpp::Expr(1);
    auto all_false = qbpp::Expr(1);
    for (int idx : clause) {
      all_true *= x[idx];
      all_false *= ~x[idx];
    }
    constraint += all_true + all_false;
  }

  // 目的関数: True/False の数のバランス
  auto s = qbpp::sum(x);
  auto objective = (2 * s - n) * (2 * s - n);

  // ペナルティ重み付き HUBO 式
  int penalty_weight = n * n + 1;
  auto f = (objective + penalty_weight * constraint).simplify_as_binary();

  // 求解
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(1);  // n=5 は奇数なので最良バランスで (2*s-n)^2 = 1
  auto sol = solver.search();

  // 結果の出力
  std::cout << "Energy = " << sol.energy() << std::endl;
  std::cout << "Assignment: ";
  for (int i = 0; i < n; ++i) {
    std::cout << "x[" << i << "]=" << sol(x[i]) << " ";
  }
  std::cout << std::endl;

  std::cout << "constraint = " << sol(constraint) << std::endl;
  std::cout << "objective  = " << sol(objective) << std::endl;

  // 検証: 各節のチェック
  bool all_satisfied = true;
  for (size_t k = 0; k < clauses.size(); ++k) {
    int sum_val = 0;
    for (int idx : clauses[k]) {
      sum_val += sol(x[idx]);
    }
    bool satisfied = (sum_val > 0) &&
                     (sum_val < static_cast<int>(clauses[k].size()));
    std::cout << "Clause " << k << ": "
              << (satisfied ? "satisfied" : "VIOLATED") << std::endl;
    if (!satisfied) all_satisfied = false;
  }
  std::cout << "All clauses NAE-satisfied: "
            << (all_satisfied ? "Yes" : "No") << std::endl;
}
```

### 実行結果の例
```
Energy = 1
Assignment: x[0]=1 x[1]=0 x[2]=1 x[3]=0 x[4]=1
constraint = 0
objective  = 1
Clause 0: satisfied
Clause 1: satisfied
Clause 2: satisfied
Clause 3: satisfied
All clauses NAE-satisfied: Yes
```

ソルバーは `constraint = 0` となる割り当てを見つけ、4 つの節すべてが NAE 充足されています。
目的関数の値は 1 です。これは $n = 5$ が奇数であるため、最良の True/False バランス（例えば True が 3、False が 2）で $(2 \times 3 - 5)^2 = 1$ となるためです。

### 要点
- **否定リテラル**: QUBO++ では `~x[i]` を直接使用して $\overline{x}_i$ を表現でき、$1 - x_i$ に展開する必要がありません。これにより HUBO 式がコンパクトに保たれます。
- **高次項**: サイズ $s$ の節は次数 $s$ の項を生成します（例えば、3 リテラルの節では $x_0 x_1 x_2$）。QUBO++ は HUBO 式をネイティブに扱えるため、二次化（quadratization）は不要です。
- **ペナルティ重み**: $P = n^2 + 1$ により、制約違反が目的関数の最大値を必ず上回るようにします。

</div>
