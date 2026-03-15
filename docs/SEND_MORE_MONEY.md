---
layout: default
nav_exclude: true
title: "SEND+MORE=MONEY"
nav_order: 42
---
<div class="lang-en" markdown="1">
# Math Puzzle: SEND MORE MONEY

**SEND + MORE = MONEY** is a famous alphametic puzzle: assign a decimal digit to each letter so that
$$
\text{SEND}+\text{MORE}=\text{MONEY}
$$

The constraints are:
- The digits assigned to letters are all distinct.
- `S` and `M` must not be 0.

## QUBO++ formulation

We assign a unique index to each letter as follows:

| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| letter | S | E | N | D | M | O | R | Y |

Let $I(\alpha)$ denote the index of letter $\alpha$ ($\in \lbrace S,E,N,D,M,O,R,Y\rbrace$).
We use an $8\times 10$ binary matrix $X=(x_{i,j})$ $(0\leq i\leq 7, 0\leq j\leq 9)$ to represent the digit assigned to each letter:  $x_{I(\alpha),j}=1$ if and only if letter $\alpha$ is assigned digit $j$.

### One-hot constraints (each letter takes exactly one digit)
Each row of $X$ must be one-hot:

$$
\begin{aligned}
\text{onehot} &=\sum_{i=0}^{7}\Bigl(\sum_{j=0}^{9}x_{i,j}=1\Bigr) \\
              &=\sum_{i=0}^{7}\Bigl(1-\sum_{j=0}^{9}x_{i,j}\Bigr)^2
\end{aligned}
$$

The value of $\text{onehot}$ is minimized to 0 if and only if every row is one-hot.

### All-different constraints (no two letters share the same digit)
Digits must be distinct across letters, i.e., no two rows choose the same column:
$$
\begin{aligned}
\text{different} &=\sum_{0\leq i<j\leq 7}\sum_{k=0}^9x_{i,k}x_{j,k}
\end{aligned}
$$

### Encoding the words as linear expressions
The values of $\text{SEND}$, $\text{MORE}$, and $\text{MONEY}$ are represented
by:

$$
\begin{aligned}
\text{SEND} &= 1000\sum_{k=0}^9 kx_{I(S),k}+ 100\sum_{k=0}^9 kx_{I(E),k}+ 10\sum_{k=0}^9 kx_{I(N),k}+\sum_{k=0}^9 kx_{I(D),k}\\
       &= \sum_{k=0}^9k(1000x_{I(S),k}+100x_{I(E),k}+10x_{I(N),k}+x_{I(D),k})\\
\text{MORE} &= 1000\sum_{k=0}^9 kx_{I(M),k}+ 100\sum_{k=0}^9 kx_{I(O),k}+ 10\sum_{k=0}^9 kx_{I(R),k}+\sum_{k=0}^9 kx_{I(E),k}\\
       &= \sum_{k=0}^9k(1000x_{I(M),k}+100x_{I(O),k}+10x_{I(R),k}+x_{I(E),k})\\
\text{MONEY} &= 10000\sum_{k=0}^9 kx_{I(M),k}+1000\sum_{k=0}^9 kx_{I(O),k}+ 100\sum_{k=0}^9 kx_{I(N),k}+ 10\sum_{k=0}^9 kx_{I(E),k}+\sum_{k=0}^9 kx_{I(Y),k}\\
       &= \sum_{k=0}^9k(10000x_{I(M),k}+ 1000x_{I(O),k}+100x_{I(N),k}+10x_{I(E),k}+x_{I(Y),k})
\end{aligned}
$$

### Equality constraint
We enforce the equation by penalizing the residual:

$$
\begin{aligned}
\text{equal} &= \Bigl(\text{SEND}+\text{MORE} = \text{MONEY}\Bigr) \\
             &= \Bigl(\text{SEND}+\text{MORE} - \text{MONEY}\Bigr)^2
\end{aligned}
$$

### Combined objective
All constraints are combined into a single objective:

$$
\begin{aligned}
f & = P\cdot (\text{onehot}+\text{different})+\text{equal}
\end{aligned}
$$

where
`P` is a sufficiently large constant to prioritize feasibility (`onehot` and `different`).
In principle, if all terms are nonnegative and each becomes 0 exactly when its constraint holds, then any solution with $f=0$ satisfies all constraints.
In practice, choosing a larger `P` often helps heuristic solvers.

In this case, there is no need to prioritize them and we can set $P=1$,
because $\text{equal}\geq 0$ always holds and $f$ takes a minimum value of 0
only if $\text{onehot}=\text{different}=\text{equal}=0$ holds.
However, a large constant $P$ helps solvers to find the optimal solution.

Finally, since $\text{S}$ and $\text{M}$ must not be 0, we fix
the binary variables as follows:
$$
x_{I(S),0} = x_{I(M),0}= 0
$$

## QUBO++ program for SEND+MORE=MONEY
The following QUBO++ program implements the QUBO formulation above and finds a solution using EasySolver:
{% raw %}
```cpp
#define COEFF_TYPE qbpp::int128_t
#define ENERGY_TYPE qbpp::int128_t

#include <string_view>

#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

constexpr std::string_view LETTERS = "SENDMORY";
constexpr size_t L = LETTERS.size();

constexpr size_t I(char c) {
  for (size_t i = 0; i < LETTERS.size(); ++i) {
    if (LETTERS[i] == c) return i;
  }
  return L;
}

const qbpp::Vector<int> K = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

int main() {
  auto x = qbpp::var("x", L, 10);


  auto onehot = qbpp::sum(qbpp::vector_sum(x) == 1);

  auto different = qbpp::toExpr(0);
  for (size_t i = 0; i < L - 1; ++i) {
    for (size_t j = i + 1; j < L; ++j) {
      different += qbpp::sum(x[i] * x[j]);
    }
  }

  auto send = qbpp::sum((x[I('S')] * 1000 + x[I('E')] * 100 + x[I('N')] * 10 + x[I('D')]) * K);
  auto more = qbpp::sum((x[I('M')] * 1000 + x[I('O')] * 100 + x[I('R')] * 10 + x[I('E')]) * K);
  auto money = qbpp::sum((x[I('M')] * 10000 + x[I('O')] * 1000 + x[I('N')] * 100 + x[I('E')] * 10 + x[I('Y')]) * K);


  auto equal = send + more - money == 0;

  qbpp::coeff_t P = 10000;
  auto f = P * (onehot + different) + equal;

  f.simplify_as_binary();

  qbpp::MapList ml = {{x[I('S')][0], 0}, {x[I('M')][0], 0}};
  auto g = qbpp::replace(f, ml);

  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.target_energy(0);

  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f).set(sol).set(ml);

  std::cout << "onehot = " << full_sol(onehot) << std::endl;
  std::cout << "different = " << full_sol(different) << std::endl;
  std::cout << "equal = " << full_sol(equal) << std::endl;

  auto val = qbpp::onehot_to_int(full_sol(x));

  auto str = [](int d) -> std::string {
    return (d < 0) ? "*" : std::to_string(d);
  };

  std::cout << "SEND + MORE = MONEY" << std::endl;
  std::cout << str(val[I('S')]) << str(val[I('E')]) << str(val[I('N')])
            << str(val[I('D')]) << " + " << str(val[I('M')]) << str(val[I('O')])
            << str(val[I('R')]) << str(val[I('E')]) << " = " << str(val[I('M')])
            << str(val[I('O')]) << str(val[I('N')]) << str(val[I('E')])
            << str(val[I('Y')]) << std::endl;
}
```
{% endraw %}
In this program, `LETTERS` assigns an integer index to each letter in `"SENDMORY"`, which is used to implement $I(\alpha)$.
We define an `L`$\times$`10` matrix `x` of binary variables (here $L=8$).
The expressions `onehot`, `different`, and `equal` are computed according to the formulation and combined into a single objective `f` with a penalty weight `P`.

We use a `qbpp::MapList` object `ml` to fix `x[I('S')][0]` and `x[I('M')][0]` to 0, and create a reduced expression g by applying this replacement.
The solver is run on `g`, and the resulting assignment `sol` is merged with the fixed mapping `ml` to produce `full_sol` for the original objective f.

Finally, `qbpp::onehot_to_int(full_sol(x))` converts the one-hot rows into digits, and the program prints the obtained solution.
This program produces the following output:
```
onehot = 0
different = 0
equal = 0
SEND + MORE = MONEY
9567 + 1085 = 10652
```
This confirms that all constraints are satisfied and the correct solution is obtained.
</div>

<div class="lang-ja" markdown="1">
# 数学パズル: SEND MORE MONEY

**SEND + MORE = MONEY** は有名な覆面算パズルです。各文字に10進数の数字を割り当てて、次の等式を成り立たせます:
$$
\text{SEND}+\text{MORE}=\text{MONEY}
$$

制約は以下の通りです:
- 各文字に割り当てられる数字はすべて異なる。
- `S` と `M` は0であってはならない。

## QUBO++による定式化

各文字に以下のように一意のインデックスを割り当てます:

| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| letter | S | E | N | D | M | O | R | Y |

$I(\alpha)$ を文字 $\alpha$ ($\in \lbrace S,E,N,D,M,O,R,Y\rbrace$) のインデックスとします。
各文字に割り当てられる数字を表すために、$8\times 10$ のバイナリ行列 $X=(x_{i,j})$ $(0\leq i\leq 7, 0\leq j\leq 9)$ を使用します。文字 $\alpha$ に数字 $j$ が割り当てられるとき、かつそのときに限り $x_{I(\alpha),j}=1$ とします。

### ワンホット制約（各文字はちょうど1つの数字を取る）
$X$ の各行はワンホットでなければなりません:

$$
\begin{aligned}
\text{onehot} &=\sum_{i=0}^{7}\Bigl(\sum_{j=0}^{9}x_{i,j}=1\Bigr) \\
              &=\sum_{i=0}^{7}\Bigl(1-\sum_{j=0}^{9}x_{i,j}\Bigr)^2
\end{aligned}
$$

$\text{onehot}$ の値は、すべての行がワンホットであるとき、かつそのときに限り、最小値0になります。

### 全異なり制約（2つの文字が同じ数字を共有しない）
数字は文字間で異なっていなければなりません。すなわち、2つの行が同じ列を選んではいけません:
$$
\begin{aligned}
\text{different} &=\sum_{0\leq i<j\leq 7}\sum_{k=0}^9x_{i,k}x_{j,k}
\end{aligned}
$$

### 単語の線形式としての符号化
$\text{SEND}$、$\text{MORE}$、$\text{MONEY}$ の値は以下のように表されます:

$$
\begin{aligned}
\text{SEND} &= 1000\sum_{k=0}^9 kx_{I(S),k}+ 100\sum_{k=0}^9 kx_{I(E),k}+ 10\sum_{k=0}^9 kx_{I(N),k}+\sum_{k=0}^9 kx_{I(D),k}\\
       &= \sum_{k=0}^9k(1000x_{I(S),k}+100x_{I(E),k}+10x_{I(N),k}+x_{I(D),k})\\
\text{MORE} &= 1000\sum_{k=0}^9 kx_{I(M),k}+ 100\sum_{k=0}^9 kx_{I(O),k}+ 10\sum_{k=0}^9 kx_{I(R),k}+\sum_{k=0}^9 kx_{I(E),k}\\
       &= \sum_{k=0}^9k(1000x_{I(M),k}+100x_{I(O),k}+10x_{I(R),k}+x_{I(E),k})\\
\text{MONEY} &= 10000\sum_{k=0}^9 kx_{I(M),k}+1000\sum_{k=0}^9 kx_{I(O),k}+ 100\sum_{k=0}^9 kx_{I(N),k}+ 10\sum_{k=0}^9 kx_{I(E),k}+\sum_{k=0}^9 kx_{I(Y),k}\\
       &= \sum_{k=0}^9k(10000x_{I(M),k}+ 1000x_{I(O),k}+100x_{I(N),k}+10x_{I(E),k}+x_{I(Y),k})
\end{aligned}
$$

### 等式制約
残差にペナルティを課すことで等式を強制します:

$$
\begin{aligned}
\text{equal} &= \Bigl(\text{SEND}+\text{MORE} = \text{MONEY}\Bigr) \\
             &= \Bigl(\text{SEND}+\text{MORE} - \text{MONEY}\Bigr)^2
\end{aligned}
$$

### 結合目的関数
すべての制約を単一の目的関数にまとめます:

$$
\begin{aligned}
f & = P\cdot (\text{onehot}+\text{different})+\text{equal}
\end{aligned}
$$

ここで `P` は実行可能性（`onehot` と `different`）を優先するための十分に大きな定数です。
原理的には、すべての項が非負であり、各項がその制約が成り立つときにちょうど0になるならば、$f=0$ となる任意の解はすべての制約を満たします。
実際には、より大きな `P` を選ぶことがヒューリスティックソルバーに有効なことが多いです。

この場合、優先順位をつける必要はなく $P=1$ と設定できます。
なぜなら $\text{equal}\geq 0$ が常に成り立ち、$f$ は $\text{onehot}=\text{different}=\text{equal}=0$ のときにのみ最小値0を取るからです。
ただし、大きな定数 $P$ はソルバーが最適解を見つけるのに役立ちます。

最後に、$\text{S}$ と $\text{M}$ は0であってはならないため、バイナリ変数を以下のように固定します:
$$
x_{I(S),0} = x_{I(M),0}= 0
$$

## SEND+MORE=MONEYのQUBO++プログラム
以下のQUBO++プログラムは、上記のQUBO定式化を実装し、EasySolverを使って解を求めます:
{% raw %}
```cpp
#define COEFF_TYPE qbpp::int128_t
#define ENERGY_TYPE qbpp::int128_t

#include <string_view>

#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

constexpr std::string_view LETTERS = "SENDMORY";
constexpr size_t L = LETTERS.size();

constexpr size_t I(char c) {
  for (size_t i = 0; i < LETTERS.size(); ++i) {
    if (LETTERS[i] == c) return i;
  }
  return L;
}

const qbpp::Vector<int> K = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

int main() {
  auto x = qbpp::var("x", L, 10);


  auto onehot = qbpp::sum(qbpp::vector_sum(x) == 1);

  auto different = qbpp::toExpr(0);
  for (size_t i = 0; i < L - 1; ++i) {
    for (size_t j = i + 1; j < L; ++j) {
      different += qbpp::sum(x[i] * x[j]);
    }
  }

  auto send = qbpp::sum((x[I('S')] * 1000 + x[I('E')] * 100 + x[I('N')] * 10 + x[I('D')]) * K);
  auto more = qbpp::sum((x[I('M')] * 1000 + x[I('O')] * 100 + x[I('R')] * 10 + x[I('E')]) * K);
  auto money = qbpp::sum((x[I('M')] * 10000 + x[I('O')] * 1000 + x[I('N')] * 100 + x[I('E')] * 10 + x[I('Y')]) * K);


  auto equal = send + more - money == 0;

  qbpp::coeff_t P = 10000;
  auto f = P * (onehot + different) + equal;

  f.simplify_as_binary();

  qbpp::MapList ml = {{x[I('S')][0], 0}, {x[I('M')][0], 0}};
  auto g = qbpp::replace(f, ml);

  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.target_energy(0);

  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f).set(sol).set(ml);

  std::cout << "onehot = " << full_sol(onehot) << std::endl;
  std::cout << "different = " << full_sol(different) << std::endl;
  std::cout << "equal = " << full_sol(equal) << std::endl;

  auto val = qbpp::onehot_to_int(full_sol(x));

  auto str = [](int d) -> std::string {
    return (d < 0) ? "*" : std::to_string(d);
  };

  std::cout << "SEND + MORE = MONEY" << std::endl;
  std::cout << str(val[I('S')]) << str(val[I('E')]) << str(val[I('N')])
            << str(val[I('D')]) << " + " << str(val[I('M')]) << str(val[I('O')])
            << str(val[I('R')]) << str(val[I('E')]) << " = " << str(val[I('M')])
            << str(val[I('O')]) << str(val[I('N')]) << str(val[I('E')])
            << str(val[I('Y')]) << std::endl;
}
```
{% endraw %}
このプログラムでは、`LETTERS` が `"SENDMORY"` の各文字に整数インデックスを割り当てており、$I(\alpha)$ の実装に使用されています。
`L`$\times$`10` のバイナリ変数行列 `x` を定義します（ここで $L=8$）。
式 `onehot`、`different`、`equal` は定式化に従って計算され、ペナルティ重み `P` とともに単一の目的関数 `f` にまとめられます。

`qbpp::MapList` オブジェクト `ml` を使用して `x[I('S')][0]` と `x[I('M')][0]` を0に固定し、この置換を適用して縮小された式 `g` を作成します。
ソルバーは `g` に対して実行され、得られた割り当て `sol` は固定マッピング `ml` とマージされて、元の目的関数 `f` に対する `full_sol` が生成されます。

最後に、`qbpp::onehot_to_int(full_sol(x))` がワンホット行を数字に変換し、プログラムは得られた解を出力します。
このプログラムは以下の出力を生成します:
```
onehot = 0
different = 0
equal = 0
SEND + MORE = MONEY
9567 + 1085 = 10652
```
これにより、すべての制約が満たされ、正しい解が得られたことが確認できます。
</div>
