---
layout: default
nav_exclude: true
title: "Adder Simulation"
nav_order: 50
---
<div class="lang-en" markdown="1">
# Adder Simulation

## Full adder and ripple carry adder
A full adder has three input bits: $a$, $b$, and $i$ (carry-in) and
$o$ (carry-out) and $s$ (sum).
The sum of the three input bits is represented using these two output bits.

A ripple-carry adder computes the sum of two multi-bit integers by cascading multiple full adders, as illustrated below:
<p align="center">
 <img src="images/adder.svg" alt="4-bit ripple carry adder" width="50%">
</p>

This ripple-carry adder computes the sum of two 4-bit integers $x_3x_2x_1x_0$ and $y_3y_2y_1y_0$
and outputs the 4-bit sum$z_3z_2z_1z_0$ using four full adders.
The corresponding 5-bit carry signals $c_4c_3c_2c_1c_0$ are also shown.

## QUBO formulation for full adder
A full adder can be formulated using the following expression:

$$
\begin{aligned}
fa(a,b,i,c,s) &=((a+b+i)-(2o+s))^2
\end{aligned}
$$

This expression attains its minimum value of 0 if and only if the five variables take values consistent with a valid full-adder operation.
The following QUBO++ program verifies this formulation using the exhaustive solver:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto i = qbpp::var("i");
  auto o = qbpp::var("o");
  auto s = qbpp::var("s");
  auto fa = (a + b + i) - (2 * o + s) == 0;
  fa.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(fa);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
In this QUBO program, the constraint $fa(a,b,i,c,s)$ is implemented using the equality operator `==`, which intuitively represents the constraint $a+b+i=2o+s$.
The program produces the following output, confirming that the expression correctly models a full adder:
{% raw %}
```
(0) 0:{{a,0},{b,0},{i,0},{o,0},{s,0}}
(1) 0:{{a,0},{b,0},{i,1},{o,0},{s,1}}
(2) 0:{{a,0},{b,1},{i,0},{o,0},{s,1}}
(3) 0:{{a,0},{b,1},{i,1},{o,1},{s,0}}
(4) 0:{{a,1},{b,0},{i,0},{o,0},{s,1}}
(5) 0:{{a,1},{b,0},{i,1},{o,1},{s,0}}
(6) 0:{{a,1},{b,1},{i,0},{o,1},{s,0}}
(7) 0:{{a,1},{b,1},{i,1},{o,1},{s,1}}
```
{% endraw %}

If some bits are fixed, the valid values of the remaining bits can be derived.
For example, the three input bits can be fixed using the `replace()` function:
{% raw %}
```cpp
  fa.replace({{a, 1}, {b, 1}, {i, 0}});
```
{% endraw %}


The program then produces the following output:

{% raw %}
```
(0) 0:{{o,1},{s,0}}
```
{% endraw %}

Conversely, if the two output bits are fixed:
{% raw %}
```cpp
  fa.replace({{o, 1}, {s, 0}});
```
{% endraw %}
the program produces all valid combinations of the input bits:
{% raw %}
```
(0) 0:{{a,0},{b,1},{i,1}}
(1) 0:{{a,1},{b,0},{i,1}}
(2) 0:{{a,1},{b,1},{i,0}}
```
{% endraw %}


## Simulating a ripple carry adder using multiple full adders
Using the QUBO expression for a full adder, we can construct a QUBO expression that simulates a ripple-carry adder.
The following QUBO++ program creates a QUBO expression for simulating a 4-bit adder by combining four full adders:
{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto i = qbpp::var("i");
  auto o = qbpp::var("o");
  auto s = qbpp::var("s");
  auto fa = (a + b + i) - (2 * o + s) == 0;

  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  auto c = qbpp::var("c", 5);
  auto z = qbpp::var("s", 4);

  auto fa0 = qbpp::replace(fa, {{a, x[0]}, {b, y[0]}, {i, c[0]}, {o, c[1]}, {s, z[0]}});
  auto fa1 = qbpp::replace(fa, {{a, x[1]}, {b, y[1]}, {i, c[1]}, {o, c[2]}, {s, z[1]}});
  auto fa2 = qbpp::replace(fa, {{a, x[2]}, {b, y[2]}, {i, c[2]}, {o, c[3]}, {s, z[2]}});
  auto fa3 = qbpp::replace(fa, {{a, x[3]}, {b, y[3]}, {i, c[3]}, {o, c[4]}, {s, z[3]}});
  auto adder = fa0 + fa1 + fa2 + fa3;
  adder.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(adder);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
{% endraw %}
In this QUBO++ program, four `qbpp::Expr` objects representing full adders are created using the `replace()` function and combined into a single expression, `adder`.
The Exhaustive Solver is then used to enumerate all optimal solutions.

This program produces 512 valid solutions, corresponding to all possible input combinations of a 4-bit adder:
{% raw %}
```
(0) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],0},{c[0],0},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],0},{s[1],0},{s[2],0},{s[3],0}}
(1) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],0},{c[0],1},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],1},{s[1],0},{s[2],0},{s[3],0}}
(2) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],1},{c[0],0},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],0},{s[1],0},{s[2],0},{s[3],1}}
(3) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],1},{c[0],1},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],1},{s[1],0},{s[2],0},{s[3],1}}

... omitted ...

(510) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[0],0},{c[1],1},{c[2],1},{c[3],1},{c[4],1},{s[0],0},{s[1],1},{s[2],1},{s[3],1}}
(511) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[0],1},{c[1],1},{c[2],1},{c[3],1},{c[4],1},{s[0],1},{s[1],1},{s[2],1},{s[3],1}}
```
{% endraw %}
Alternatively, we can define a C++ function `fa` to construct full-adder constraints in a more concise and readable manner:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

qbpp::Expr fa(qbpp::Var a, qbpp::Var b, qbpp::Var i, qbpp::Var o, qbpp::Var s) {
  return (a + b + i) - (2 * o + s) == 0;
}

int main() {
  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  auto c = qbpp::var("c", 5);
  auto z = qbpp::var("s", 4);
  auto fa0 = fa(x[0], y[0], c[0], c[1], z[0]);
  auto fa1 = fa(x[1], y[1], c[1], c[2], z[1]);
  auto fa2 = fa(x[2], y[2], c[2], c[3], z[2]);
  auto fa3 = fa(x[3], y[3], c[3], c[4], z[3]);
  auto adder = fa0 + fa1 + fa2 + fa3;
  adder.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(adder);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
This program produces the same 512 optimal solutions as the previous implementation.

If some of the binary variables are fixed, the valid values of the remaining variables can be derived using the Exhaustive Solver.
For example, the following `qbpp::MapList` object `ml` fixes the carry-in, carry-out, and sum bits:
{% raw %}
```cpp
  qbpp::MapList ml = {{c[4], 1}, {c[0], 0}, {z[3], 1},
                      {z[2], 1}, {z[1], 0}, {z[0], 1}};
  adder.replace(ml);
```
{% endraw %}
The resulting program produces the following output:
{% raw %}
```
(0) 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[1],0},{c[2],1},{c[3],1}}
(1) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],0},{y[1],1},{y[2],1},{y[3],1},{c[1],0},{c[2],1},{c[3],1}}
```
{% endraw %}
</div>

<div class="lang-ja" markdown="1">
# 加算器シミュレーション

## 全加算器とリプルキャリー加算器
全加算器は3つの入力ビット $a$、$b$、$i$（キャリー入力）と、
$o$（キャリー出力）および $s$（和）を持ちます。
3つの入力ビットの和は、これら2つの出力ビットを使って表されます。

リプルキャリー加算器は、以下に示すように複数の全加算器をカスケード接続することで、2つの多ビット整数の和を計算します:
<p align="center">
 <img src="images/adder.svg" alt="4-bit ripple carry adder" width="50%">
</p>

このリプルキャリー加算器は、4つの全加算器を使って2つの4ビット整数 $x_3x_2x_1x_0$ と $y_3y_2y_1y_0$ の和を計算し、4ビットの和 $z_3z_2z_1z_0$ を出力します。
対応する5ビットのキャリー信号 $c_4c_3c_2c_1c_0$ も示されています。

## 全加算器のQUBO定式化
全加算器は以下の式を用いて定式化できます:

$$
\begin{aligned}
fa(a,b,i,c,s) &=((a+b+i)-(2o+s))^2
\end{aligned}
$$

この式は、5つの変数が有効な全加算器の動作と整合する値を取るとき、かつそのときに限り、最小値0を達成します。
以下のQUBO++プログラムは、Exhaustive Solverを使ってこの定式化を検証します:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto i = qbpp::var("i");
  auto o = qbpp::var("o");
  auto s = qbpp::var("s");
  auto fa = (a + b + i) - (2 * o + s) == 0;
  fa.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(fa);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
このQUBOプログラムでは、制約 $fa(a,b,i,c,s)$ は等号演算子 `==` を使って実装されており、直感的に制約 $a+b+i=2o+s$ を表しています。
プログラムは以下の出力を生成し、式が全加算器を正しくモデル化していることを確認します:
{% raw %}
```
(0) 0:{{a,0},{b,0},{i,0},{o,0},{s,0}}
(1) 0:{{a,0},{b,0},{i,1},{o,0},{s,1}}
(2) 0:{{a,0},{b,1},{i,0},{o,0},{s,1}}
(3) 0:{{a,0},{b,1},{i,1},{o,1},{s,0}}
(4) 0:{{a,1},{b,0},{i,0},{o,0},{s,1}}
(5) 0:{{a,1},{b,0},{i,1},{o,1},{s,0}}
(6) 0:{{a,1},{b,1},{i,0},{o,1},{s,0}}
(7) 0:{{a,1},{b,1},{i,1},{o,1},{s,1}}
```
{% endraw %}

一部のビットを固定すると、残りのビットの有効な値を導出できます。
例えば、3つの入力ビットを `replace()` 関数で固定できます:
{% raw %}
```cpp
  fa.replace({{a, 1}, {b, 1}, {i, 0}});
```
{% endraw %}


プログラムは以下の出力を生成します:

{% raw %}
```
(0) 0:{{o,1},{s,0}}
```
{% endraw %}

逆に、2つの出力ビットを固定した場合:
{% raw %}
```cpp
  fa.replace({{o, 1}, {s, 0}});
```
{% endraw %}
プログラムは入力ビットのすべての有効な組み合わせを出力します:
{% raw %}
```
(0) 0:{{a,0},{b,1},{i,1}}
(1) 0:{{a,1},{b,0},{i,1}}
(2) 0:{{a,1},{b,1},{i,0}}
```
{% endraw %}


## 複数の全加算器を用いたリプルキャリー加算器のシミュレーション
全加算器のQUBO式を使って、リプルキャリー加算器をシミュレートするQUBO式を構築できます。
以下のQUBO++プログラムは、4つの全加算器を組み合わせて4ビット加算器をシミュレートするQUBO式を作成します:
{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto i = qbpp::var("i");
  auto o = qbpp::var("o");
  auto s = qbpp::var("s");
  auto fa = (a + b + i) - (2 * o + s) == 0;

  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  auto c = qbpp::var("c", 5);
  auto z = qbpp::var("s", 4);

  auto fa0 = qbpp::replace(fa, {{a, x[0]}, {b, y[0]}, {i, c[0]}, {o, c[1]}, {s, z[0]}});
  auto fa1 = qbpp::replace(fa, {{a, x[1]}, {b, y[1]}, {i, c[1]}, {o, c[2]}, {s, z[1]}});
  auto fa2 = qbpp::replace(fa, {{a, x[2]}, {b, y[2]}, {i, c[2]}, {o, c[3]}, {s, z[2]}});
  auto fa3 = qbpp::replace(fa, {{a, x[3]}, {b, y[3]}, {i, c[3]}, {o, c[4]}, {s, z[3]}});
  auto adder = fa0 + fa1 + fa2 + fa3;
  adder.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(adder);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
{% endraw %}
このQUBO++プログラムでは、全加算器を表す4つの `qbpp::Expr` オブジェクトが `replace()` 関数を使って作成され、単一の式 `adder` にまとめられます。
次にExhaustive Solverがすべての最適解を列挙します。

このプログラムは512個の有効な解を生成し、4ビット加算器のすべての可能な入力の組み合わせに対応します:
{% raw %}
```
(0) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],0},{c[0],0},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],0},{s[1],0},{s[2],0},{s[3],0}}
(1) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],0},{c[0],1},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],1},{s[1],0},{s[2],0},{s[3],0}}
(2) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],1},{c[0],0},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],0},{s[1],0},{s[2],0},{s[3],1}}
(3) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{y[0],0},{y[1],0},{y[2],0},{y[3],1},{c[0],1},{c[1],0},{c[2],0},{c[3],0},{c[4],0},{s[0],1},{s[1],0},{s[2],0},{s[3],1}}

... 省略 ...

(510) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[0],0},{c[1],1},{c[2],1},{c[3],1},{c[4],1},{s[0],0},{s[1],1},{s[2],1},{s[3],1}}
(511) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[0],1},{c[1],1},{c[2],1},{c[3],1},{c[4],1},{s[0],1},{s[1],1},{s[2],1},{s[3],1}}
```
{% endraw %}
あるいは、C++関数 `fa` を定義して、全加算器の制約をより簡潔で読みやすい形式で構築することもできます:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

qbpp::Expr fa(qbpp::Var a, qbpp::Var b, qbpp::Var i, qbpp::Var o, qbpp::Var s) {
  return (a + b + i) - (2 * o + s) == 0;
}

int main() {
  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  auto c = qbpp::var("c", 5);
  auto z = qbpp::var("s", 4);
  auto fa0 = fa(x[0], y[0], c[0], c[1], z[0]);
  auto fa1 = fa(x[1], y[1], c[1], c[2], z[1]);
  auto fa2 = fa(x[2], y[2], c[2], c[3], z[2]);
  auto fa3 = fa(x[3], y[3], c[3], c[4], z[3]);
  auto adder = fa0 + fa1 + fa2 + fa3;
  adder.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(adder);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
このプログラムは前の実装と同じ512個の最適解を生成します。

一部のバイナリ変数を固定すると、残りの変数の有効な値をExhaustive Solverで導出できます。
例えば、以下の `qbpp::MapList` オブジェクト `ml` はキャリー入力、キャリー出力、および和のビットを固定します:
{% raw %}
```cpp
  qbpp::MapList ml = {{c[4], 1}, {c[0], 0}, {z[3], 1},
                      {z[2], 1}, {z[1], 0}, {z[0], 1}};
  adder.replace(ml);
```
{% endraw %}
結果のプログラムは以下の出力を生成します:
{% raw %}
```
(0) 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],1},{y[0],1},{y[1],1},{y[2],1},{y[3],1},{c[1],0},{c[2],1},{c[3],1}}
(1) 0:{{x[0],1},{x[1],1},{x[2],1},{x[3],1},{y[0],0},{y[1],1},{y[2],1},{y[3],1},{c[1],0},{c[2],1},{c[3],1}}
```
{% endraw %}
</div>
