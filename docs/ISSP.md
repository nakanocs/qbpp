---
layout: default
nav_exclude: true
title: "Interval Subset Sum"
nav_order: 35
---
<div class="lang-en" markdown="1">

# Interval Subset Sum Problem (ISSP)
The **Interval Subset Sum Problem (ISSP)** is a generalization of the **Subset Sum Problem**.
Given $n$ integer **intervals $[l_i, u_i]$** $(0\leq i\leq n-1)$ and an **upper bound $T$**, the goal is to choose an integer value

$$
\begin{aligned}
v_i &\in \lbrace 0\rbrace \cup [l_i, u_i] && (i = 0,1,\dots,n-1),
\end{aligned}
$$

so as to satisfy the constraint

$$
\begin{aligned}
 \sum_{i=0}^{n-1} v_i \leq T,
\end{aligned}
$$

and maximize the objective:

$$
\begin{aligned}
 \sum_{i=0}^{n-1} v_i.
\end{aligned}
$$

## HUBO formulation of the ISPP
An integer variable can be represented by multiple binary variables using a binary encoding.
In QUBO++, such integer variables can be defined easily as shown in
[Integer Variables and Solving Simultaneous Equations](INTEGER).

Let $v_i$ $(0\leq i\leq n-1)$ be an integer variable that can take a value in $[l_i, u_i]$.
We also introduce a binary variable $s_i$  $(0\leq i\leq n-1)$ such that $s_i=1$ if and only if
interval $i$ is selected.

To model ISSP, we use the product $s_i v_i$ as the selected value:

$$
\begin{aligned}
s_iv_i &= 0 && \text{if } s_i= 0\\
       &\in [l_i,u_i] && \text{if } s_i= 1
\end{aligned}
$$

Let

$$
\begin{aligned}
\text{sum} &= \sum_{i=0}^{n-1} s_i v_i .
\end{aligned}
$$

In QUBO++, we impose this inequality constraint via a penalty term:

$$
\begin{aligned}
 \text{constraint} &= \sum_{i=0}^{n-1} \bigr(0\leq s_iv_i \leq T\bigl) \\
                &= (T-\sum_{i=0}^{n-1} s_iv_i)^2
\end{aligned}
$$

Since $s_i v_i$ is quadratic in binary variables, $\text{sum}$ is quadratic and $\text{constraint}$ becomes quartic.

Because the ISSP maximizes the sum under the upper bound $T$, we minimize the negative sum:

$$
\begin{aligned}
 \text{objective} &= -\sum_{i=0}^{n-1} s_iv_i
\end{aligned}
$$

Finally, we combine the objective and the constraint penalty into a single HUBO function:

$$
\begin{aligned}
f &= \text{objective} + P\times\text{constraint},
\end{aligned}
$$

where $P$ is a sufficiently large constant to prioritize feasibility.

## QUBO++ program of the HUBO formulation
The following QUBO++ program solves an ISSP instance with 8 intervals.
The lower and upper bounds $[l_i,u_i]$ are stored in the vectors `lower` and `upper`, and $T=100$.

```cpp
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  qbpp::Vector<int> lower = {18, 17, 21, 18, 20, 14, 14, 23};
  qbpp::Vector<int> upper = {19, 17, 22, 19, 20, 16, 15, 25};
  const int T = 100;

  auto v = lower <= qbpp::var_int("v", lower.size()) <= upper;
  auto s = qbpp::var("s", lower.size());

  auto sum = qbpp::sum(v * s);
  auto constraint = 0 <= sum <= T;
  auto f = -sum + 1000 * constraint;
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(-T);
  auto sol = solver.search();
  for (size_t i = 0; i < v.size(); ++i) {
    if (sol(s[i])) {
      std::cout << "Interval " << i << ": val = " << sol(v[i]) << std::endl;
    }
  }
  std::cout << "sum = " << sol(sum) << std::endl;
}
```
First, we define a vector `v` of integer variables where each `v[i]` takes an integer value in
`[lower[i], upper[i]]`.
We also define a vector `s` of binary variables, where `s[i] = 1` means
interval `i` is selected.
The expression `sum` represents $\sum_i v_i s_i$.

The inequality constraint `0 <= sum <= T` is stored in `constraint`. In QUBO++, such a constraint
is internally converted into a nonnegative penalty term that becomes zero when the constraint is satisfied.

Finally, we construct the HUBO objective function `f` as
`f = -sum + P * constraint` (with `P = 1000` in this example).
Minimizing `f` therefore maximizes `sum` while heavily penalizing any violation of the constraint.

We set the target energy to `-T` because if the solver finds a feasible solution with `sum = T`,
then the penalty term is zero and the objective term becomes `-T`, i.e., the global minimum reaches `-T`.

For the obtained solution, the selected intervals and their values are printed. For example:
```
Interval 0: val = 18
Interval 1: val = 17
Interval 2: val = 22
Interval 4: val = 20
Interval 7: val = 23
sum = 100
```
This output confirms that a feasible solution achieving the maximum possible `sum` ($=T$) was obtained.

## QUBO formulation for the ISSP
The HUBO formulation above contains quartic terms because it uses products $s_i v_i$.
We can avoid quartic terms by introducing auxiliary integer variables.

Let $a_i$ $(0\leq i\leq n-1)$ be an integer variable that can take a value in
$[0,\, u_i-l_i]$.
We also use a binary variable $s_i$ $(0\leq i\leq n-1$) such that $s_i=1$ if and only if
interval $i$ is selected

We define

$$
\begin{aligned}
  v_i &= l_is_i + a_i && (0\leq i\leq n-1) \\
\end{aligned}
$$

To ensure that $x_i$ becomes 0 when $s_i=0$, we add the following penalty term:

$$
\begin{aligned}
  \text{constraint1} &= \sum_{i=0}^{n-1}\sum_j (1-s_i)a_i
\end{aligned}
$$

Since $a_i \ge 0$ and $1-s_i \ge 0$, we have $\text{constraint1}\ge 0$.
Moreover, $\text{constraint1}=0$ holds if and only if $a_i=0$ whenever $s_i=0$.
Therefore, the selected value $v_i$ satisfies

$$
\begin{aligned}
  v_i & = 0 && \text{if } s_i=0,\\
      & \in  [l_i,u_i] &&\text{if } s_i=1.
\end{aligned}
$$

because $v_i = l_i + a_i$ and $a_i \in [0,u_i-l_i]$ when $s_i=1$.

Let

$$
\begin{aligned}
\text{sum} &= \sum_{i=0}^{n-1} x_i.
\end{aligned}
$$

The ISSP constraint is:

$$
\begin{aligned}
 \text{constraint2} &= \sum_{i=0}^{n-1} \bigr(0\leq v_i \leq T\bigl) \\
                &= (T-\sum_{i=0}^{n-1} v_i)^2
\end{aligned}
$$

Finally, since ISSP maximizes $\text{sum}$ under the upper bound $T$, we minimize

$$
\begin{aligned}
 \text{objective} &= -\sum_{i=0}^{n-1} v_i
\end{aligned}
$$

Combining the objective and the penalties, we obtain the QUBO expression:

$$
\begin{aligned}
f &= \text{objective} + P\times(\text{constraint1}+\text{constraint2}),
\end{aligned}
$$

where $P$ is a sufficiently large constant to prioritize feasibility.

## QUBO++ program of the QUBO formulation
The following QUBO++ program solves the same ISSP instance using the QUBO formulation:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  qbpp::Vector<int> lower = {18, 17, 21, 18, 20, 14, 14, 23};
  qbpp::Vector<int> upper = {19, 17, 22, 19, 20, 16, 15, 25};
  const int T = 100;

  auto a = 0 <= qbpp::var_int("a", lower.size()) <= (upper - lower);
  auto s = qbpp::var("s", lower.size());
  auto v = s * lower + a;

  auto sum = qbpp::sum(v);
  auto constraint1 = qbpp::sum((1 - s) * a);
  auto constraint2 = 0 <= sum <= T;
  auto f = -sum + 1000 * (constraint1 + constraint2);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(-T);
  auto sol = solver.search();
  for (size_t i = 0; i < v.size(); ++i) {
    if (sol(s[i])) {
      std::cout << "Interval " << i << ": val = " << (sol(v[i])) << std::endl;
    }
  }
  std::cout << "sum = " << sol(sum) << std::endl;
}
```
First, we define a vector `a` of integer variables, where each `a[i]` takes an integer value in
`[0, upper[i] - lower[i]]`.
We also define a vector `s` of binary variables.
Using `a` and `s`, we construct `x = s * lower + a`, which corresponds to
$v_i = s_i * l_i+a_i$.
The expression `constraint1 = sum((1 - s) * a)` penalizes any solution with `a[i] > 0` when `s[i] = 0`,
thereby enforcing `v[i] = 0` for unselected intervals.
The inequality constraint `constraint2 = 0 <= sum <= T`
ensures that the total selected sum does not exceed `T`.

Finally, we minimize `f = -sum + P * (constraint1 + constraint2)` with a sufficiently large penalty constant `P`.
As in the previous example, setting `target_energy(-T)` allows the solver to stop early if it finds a feasible solution achieving `sum = T` (in which case the penalty terms are zero and the objective term becomes `-T`).

</div>

<div class="lang-ja" markdown="1">

# 区間部分和問題 (ISSP)
**区間部分和問題 (Interval Subset Sum Problem, ISSP)** は**部分和問題**の一般化です。
$n$ 個の整数**区間 $[l_i, u_i]$** $(0\leq i\leq n-1)$ と**上限 $T$** が与えられたとき、整数値

$$
\begin{aligned}
v_i &\in \lbrace 0\rbrace \cup [l_i, u_i] && (i = 0,1,\dots,n-1)
\end{aligned}
$$

を選び、制約

$$
\begin{aligned}
 \sum_{i=0}^{n-1} v_i \leq T
\end{aligned}
$$

を満たしつつ、目的関数

$$
\begin{aligned}
 \sum_{i=0}^{n-1} v_i
\end{aligned}
$$

を最大化することが目標です。

## ISSPのHUBO定式化
整数変数はバイナリ符号化を用いて複数のバイナリ変数で表現できます。
QUBO++では、[整数変数と連立方程式の解法](INTEGER)で示されているように、このような整数変数を簡単に定義できます。

$v_i$ $(0\leq i\leq n-1)$ を $[l_i, u_i]$ の値をとる整数変数とします。
また、区間 $i$ が選択されるときかつそのときに限り $s_i=1$ となるバイナリ変数 $s_i$ $(0\leq i\leq n-1)$ を導入します。

ISSPをモデル化するために、選択された値として積 $s_i v_i$ を使用します：

$$
\begin{aligned}
s_iv_i &= 0 && \text{if } s_i= 0\\
       &\in [l_i,u_i] && \text{if } s_i= 1
\end{aligned}
$$

次を定義します：

$$
\begin{aligned}
\text{sum} &= \sum_{i=0}^{n-1} s_i v_i .
\end{aligned}
$$

QUBO++では、この不等式制約をペナルティ項で課します：

$$
\begin{aligned}
 \text{constraint} &= \sum_{i=0}^{n-1} \bigr(0\leq s_iv_i \leq T\bigl) \\
                &= (T-\sum_{i=0}^{n-1} s_iv_i)^2
\end{aligned}
$$

$s_i v_i$ はバイナリ変数の2次式であるため、$\text{sum}$ は2次、$\text{constraint}$ は4次になります。

ISSPは上限 $T$ のもとで和を最大化するため、負の和を最小化します：

$$
\begin{aligned}
 \text{objective} &= -\sum_{i=0}^{n-1} s_iv_i
\end{aligned}
$$

最後に、目的関数と制約ペナルティを単一のHUBO関数にまとめます：

$$
\begin{aligned}
f &= \text{objective} + P\times\text{constraint},
\end{aligned}
$$

ここで $P$ は実行可能性を優先するための十分大きな定数です。

## HUBO定式化のQUBO++プログラム
以下のQUBO++プログラムは、8個の区間を持つISSPインスタンスを解きます。
下限と上限 $[l_i,u_i]$ はベクトル `lower` と `upper` に格納され、$T=100$ です。

```cpp
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  qbpp::Vector<int> lower = {18, 17, 21, 18, 20, 14, 14, 23};
  qbpp::Vector<int> upper = {19, 17, 22, 19, 20, 16, 15, 25};
  const int T = 100;

  auto v = lower <= qbpp::var_int("v", lower.size()) <= upper;
  auto s = qbpp::var("s", lower.size());

  auto sum = qbpp::sum(v * s);
  auto constraint = 0 <= sum <= T;
  auto f = -sum + 1000 * constraint;
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(-T);
  auto sol = solver.search();
  for (size_t i = 0; i < v.size(); ++i) {
    if (sol(s[i])) {
      std::cout << "Interval " << i << ": val = " << sol(v[i]) << std::endl;
    }
  }
  std::cout << "sum = " << sol(sum) << std::endl;
}
```
まず、各 `v[i]` が `[lower[i], upper[i]]` の整数値をとる整数変数のベクトル `v` を定義します。
また、`s[i] = 1` が区間 `i` が選択されることを意味するバイナリ変数のベクトル `s` を定義します。
式 `sum` は $\sum_i v_i s_i$ を表します。

不等式制約 `0 <= sum <= T` は `constraint` に格納されます。QUBO++では、このような制約は内部的に非負のペナルティ項に変換され、制約が満たされると0になります。

最後に、HUBO目的関数 `f` を `f = -sum + P * constraint`（この例では `P = 1000`）として構築します。
`f` を最小化することで、制約違反に大きなペナルティを課しつつ `sum` を最大化します。

ターゲットエネルギーを `-T` に設定するのは、ソルバーが `sum = T` の実行可能解を見つけた場合、ペナルティ項が0になり目的関数項が `-T` になる、すなわち大域最小値が `-T` に達するためです。

得られた解について、選択された区間とその値が表示されます。例えば：
```
Interval 0: val = 18
Interval 1: val = 17
Interval 2: val = 22
Interval 4: val = 20
Interval 7: val = 23
sum = 100
```
この出力は、最大可能な `sum` ($=T$) を達成する実行可能解が得られたことを確認しています。

## ISSPのQUBO定式化
上記のHUBO定式化は積 $s_i v_i$ を使用するため4次項を含みます。
補助整数変数を導入することで4次項を避けることができます。

$a_i$ $(0\leq i\leq n-1)$ を $[0,\, u_i-l_i]$ の値をとる整数変数とします。
また、区間 $i$ が選択されるときかつそのときに限り $s_i=1$ となるバイナリ変数 $s_i$ $(0\leq i\leq n-1)$ を使用します。

次を定義します：

$$
\begin{aligned}
  v_i &= l_is_i + a_i && (0\leq i\leq n-1) \\
\end{aligned}
$$

$s_i=0$ のときに $x_i$ が0になることを保証するため、次のペナルティ項を追加します：

$$
\begin{aligned}
  \text{constraint1} &= \sum_{i=0}^{n-1}\sum_j (1-s_i)a_i
\end{aligned}
$$

$a_i \ge 0$ かつ $1-s_i \ge 0$ であるため、$\text{constraint1}\ge 0$ が成り立ちます。
さらに、$s_i=0$ のときに $a_i=0$ であるときかつそのときに限り $\text{constraint1}=0$ が成り立ちます。
したがって、選択された値 $v_i$ は次を満たします：

$$
\begin{aligned}
  v_i & = 0 && \text{if } s_i=0,\\
      & \in  [l_i,u_i] &&\text{if } s_i=1.
\end{aligned}
$$

$s_i=1$ のとき $v_i = l_i + a_i$ かつ $a_i \in [0,u_i-l_i]$ であるためです。

次を定義します：

$$
\begin{aligned}
\text{sum} &= \sum_{i=0}^{n-1} x_i.
\end{aligned}
$$

ISSPの制約は：

$$
\begin{aligned}
 \text{constraint2} &= \sum_{i=0}^{n-1} \bigr(0\leq v_i \leq T\bigl) \\
                &= (T-\sum_{i=0}^{n-1} v_i)^2
\end{aligned}
$$

最後に、ISSPは上限 $T$ のもとで $\text{sum}$ を最大化するため、次を最小化します：

$$
\begin{aligned}
 \text{objective} &= -\sum_{i=0}^{n-1} v_i
\end{aligned}
$$

目的関数とペナルティを組み合わせて、QUBO式を得ます：

$$
\begin{aligned}
f &= \text{objective} + P\times(\text{constraint1}+\text{constraint2}),
\end{aligned}
$$

ここで $P$ は実行可能性を優先するための十分大きな定数です。

## QUBO定式化のQUBO++プログラム
以下のQUBO++プログラムは、QUBO定式化を用いて同じISSPインスタンスを解きます：
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  qbpp::Vector<int> lower = {18, 17, 21, 18, 20, 14, 14, 23};
  qbpp::Vector<int> upper = {19, 17, 22, 19, 20, 16, 15, 25};
  const int T = 100;

  auto a = 0 <= qbpp::var_int("a", lower.size()) <= (upper - lower);
  auto s = qbpp::var("s", lower.size());
  auto v = s * lower + a;

  auto sum = qbpp::sum(v);
  auto constraint1 = qbpp::sum((1 - s) * a);
  auto constraint2 = 0 <= sum <= T;
  auto f = -sum + 1000 * (constraint1 + constraint2);
  f.simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(-T);
  auto sol = solver.search();
  for (size_t i = 0; i < v.size(); ++i) {
    if (sol(s[i])) {
      std::cout << "Interval " << i << ": val = " << (sol(v[i])) << std::endl;
    }
  }
  std::cout << "sum = " << sol(sum) << std::endl;
}
```
まず、各 `a[i]` が `[0, upper[i] - lower[i]]` の整数値をとる整数変数のベクトル `a` を定義します。
また、バイナリ変数のベクトル `s` を定義します。
`a` と `s` を用いて `x = s * lower + a` を構築し、これは $v_i = s_i * l_i+a_i$ に対応します。
式 `constraint1 = sum((1 - s) * a)` は、`s[i] = 0` のときに `a[i] > 0` となる解にペナルティを課し、選択されていない区間に対して `v[i] = 0` を強制します。
不等式制約 `constraint2 = 0 <= sum <= T` は、選択された合計が `T` を超えないことを保証します。

最後に、十分大きなペナルティ定数 `P` で `f = -sum + P * (constraint1 + constraint2)` を最小化します。
前の例と同様に、`target_energy(-T)` を設定することで、`sum = T` を達成する実行可能解が見つかった場合にソルバーを早期停止させることができます（この場合、ペナルティ項は0になり目的関数項は `-T` になります）。

</div>
