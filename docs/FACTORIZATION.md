---
layout: default
nav_exclude: true
title: "Factorization Through HUBO Expression"
nav_order: 7
---
<div class="lang-en" markdown="1">

# Factorization Through HUBO Expression

## HUBO for factorizing the product of two prime numbers
We consider the **factorization of integers** that are products of two prime numbers.
For example, when the product $pq = 35$ is given, the goal is to recover the two prime factors $p=5$ and $q=7$.

Since $\sqrt{15}=5.91$ and $35/2=17.5$, we can restrict the search ranges $p$ and $q$ as follows:

$$
\begin{aligned}
  2 \leq &p \leq 5 \\
  6 \leq &q \leq 17
\end{aligned}
$$

For such integer variables, the factorization problem for $35$ can be formulated using the penalty expression:

$$
\begin{aligned}\
f(p,q) &= (pq-35)^2
\end{aligned}
$$

Because the integer variables $p$ and $q$ are implemented as linear expressions of binary variables, their product
$pq$ becomes a quadratic expression, and therefore
$f(p,q)$ becomes quartic.
Clearly, $f(p,q)$ attains the minimum value 0 exactly when $p$ and $q$ are the correct factors of 35.

## QUBO++ program for factorization
The following QUBO++ program constructs the HUBO expression $f(p,q)$, and solves the optimization problem using the Easy Solver:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 5;
  auto q = 6 <= qbpp::var_int("q") <= 17;

  auto f = p * q == 35;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  std::cout << "sol = " << sol << std::endl;
  std::cout << "p = " << sol(p) << std::endl;
  std::cout << "q = " << sol(q) << std::endl;
}
```

In this program, the expression `p * q == 35` is automatically converted into `qbpp::sqr(p * q - 35)`, which achieves an energy value of 0 when the equality is satisfied.
The output of this program is as follows:

{% raw %}
```cpp
f = 529 -240*p[0] -408*p[1] -88*q[0] -168*q[1] -304*q[2] -304*q[3] +144*p[0]*p[1] -5*p[0]*q[0] +40*p[0]*q[2] +40*p[0]*q[3] +16*p[1]*q[0] +56*p[1]*q[1] +208*p[1]*q[2] +208*p[1]*q[3] +16*q[0]*q[1] +32*q[0]*q[2] +32*q[0]*q[3] +64*q[1]*q[2] +64*q[1]*q[3] +128*q[2]*q[3] +52*p[0]*p[1]*q[0] +112*p[0]*p[1]*q[1] +256*p[0]*p[1]*q[2] +256*p[0]*p[1]*q[3] +20*p[0]*q[0]*q[1] +40*p[0]*q[0]*q[2] +40*p[0]*q[0]*q[3] +80*p[0]*q[1]*q[2] +80*p[0]*q[1]*q[3] +160*p[0]*q[2]*q[3] +48*p[1]*q[0]*q[1] +96*p[1]*q[0]*q[2] +96*p[1]*q[0]*q[3] +192*p[1]*q[1]*q[2] +192*p[1]*q[1]*q[3] +384*p[1]*q[2]*q[3] +16*p[0]*p[1]*q[0]*q[1] +32*p[0]*p[1]*q[0]*q[2] +32*p[0]*p[1]*q[0]*q[3] +64*p[0]*p[1]*q[1]*q[2] +64*p[0]*p[1]*q[1]*q[3] +128*p[0]*p[1]*q[2]*q[3]
sol = 0:{{p[0],1},{p[1],1},{q[0],1},{q[1],0},{q[2],0},{q[3],0}}
p = 5
q = 7
```
{% endraw %}
From the output, we can observe that the expression `f` contains quartic terms, confirming that it is a HUBO expression.
The solver correctly finds the prime factors $p=5$ and $q=7$.

## Unlimited large coefficients for prime factorization of large numbers
By default, the data types of expression coefficients and energy values in QUBO++ are `int32_t` and `int64_t`, respectively.
These types can be changed by defining the macros **`COEFF_TYPE`** and **`ENERGY_TYPE`**.

Furthermore, QUBO++ supports expressions with arbitrarily large coefficients and energy values.
To enable this option, both macros can be set to **`qbpp::cpp_int`**.
The following QUBO++ program factorizes the product of two large prime numbers:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto p = 2 <= qbpp::var_int("p") <= qbpp::cpp_int("2000000");
  auto q = 2 <= qbpp::var_int("q") <= qbpp::cpp_int("2000000");

  auto f = p * q == qbpp::cpp_int("1000039") * qbpp::cpp_int("1000079");
  std::cout << "f = " << f.simplify_as_binary() << std::endl;

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  std::cout << "sol = " << sol << std::endl;
  std::cout << "p = " << sol(p) << std::endl;
  std::cout << "q = " << sol(q) << std::endl;
}
```

Before including `qbpp/qbpp.hpp`, the macros `COEFF_TYPE` and `ENERGY_TYPE` are set to `qbpp::cpp_int`.
Constant integers are specified using **`qbpp::cpp_int()`** with a string literal as its argument.

This program outputs the following result:
{% raw %}
```txt
f = 1000236020078726181467929 -4000472012304*p[0] -8000944024600*p[1] -16001888049168*p[2] -32003776098208*p[3] -64007552195904*p[4] -128015104389760*p[5] -256030208771328*p[6] -512060417509888*p[7] -1024120834888704*p[8] -2048241669253120*p[9] -4096483336409088*p[10] -8192966664429568*p[11] -16385933295304704*p[12] -32771866456391680*p[13] -65543732375912448*p[14] -131087462604341248*p[15] -262174916618747904*p[16] -524349798877757440*p[17] -1048699460316561408*p[18] -2097398370877308928*p[19] -3806137462543214568*p[20] -4000472012304*q[0] -8000944024600*q[1] -16001888049168*q[2] -32003776098208*q[3] -64007552195904*q[4] -128015104389760*q[5] -256030208771328*q[6] -512060417509888*q[7] -1024120834888704*q[8] -2048241669253120*q[9] -4096483336409088*q[10] -8192966664429568*q[11] -16385933295304704*q[12] -32771866456391680*q[13] -65543732375912448*q[14] -131087462604341248*q[15] -262174916618747904*q[16] -524349798877757440*q[17] -1048699460316561408*q[18] -2097398370877308928*q[19] -3806137462543214568*q[20]

[omitted]

+17139313073086877663232*p[19]*p[20]*q[14]*q[19] +31102631877776215375872*p[19]*p[20]*q[14]*q[20] +4284828268271719415808*p[19]*p[20]*q[15]*q[16] +8569656536543438831616*p[19]*p[20]*q[15]*q[17] +17139313073086877663232*p[19]*p[20]*q[15]*q[18] +34278626146173755326464*p[19]*p[20]*q[15]*q[19] +62205263755552430751744*p[19]*p[20]*q[15]*q[20] +17139313073086877663232*p[19]*p[20]*q[16]*q[17] +34278626146173755326464*p[19]*p[20]*q[16]*q[18] +68557252292347510652928*p[19]*p[20]*q[16]*q[19] +124410527511104861503488*p[19]*p[20]*q[16]*q[20] +68557252292347510652928*p[19]*p[20]*q[17]*q[18] +137114504584695021305856*p[19]*p[20]*q[17]*q[19] +248821055022209723006976*p[19]*p[20]*q[17]*q[20] +274229009169390042611712*p[19]*p[20]*q[18]*q[19] +497642110044419446013952*p[19]*p[20]*q[18]*q[20] +995284220088838892027904*p[19]*p[20]*q[19]*q[20]
sol = 0:{{p[0],0},{p[1],1},{p[2],1},{p[3],1},{p[4],0},{p[5],0},{p[6],0},{p[7],0},{p[8],0},{p[9],1},{p[10],1},{p[11],1},{p[12],1},{p[13],1},{p[14],0},{p[15],1},{p[16],0},{p[17],0},{p[18],0},{p[19],0},{p[20],1},{q[0],0},{q[1],1},{q[2],1},{q[3],0},{q[4],0},{q[5],1},{q[6],1},{q[7],1},{q[8],1},{q[9],0},{q[10],1},{q[11],1},{q[12],1},{q[13],1},{q[14],0},{q[15],1},{q[16],0},{q[17],0},{q[18],0},{q[19],0},{q[20],1}}
p = 1000079
q = 1000039
```
{% endraw %}
We can see that the expression `f` contains very large coefficients, and the factorization of the large composite number is correctly obtained.

>**TIP**
> **`COEFF_TYPE`** and **`ENERGY_TYPE`** can be set to **`int8_t`**, **`int16_t`**, **`int32_t`**, **`int64_t`**, **`qbpp::int128_t`**, **`qbpp::int256_t`**, **`qbpp::int512_t`**, **`qbpp::int1024_t`**, or **`qbpp::cpp_int`**.
</div>

<div class="lang-ja" markdown="1">

# HUBO式による因数分解

## 2つの素数の積を因数分解するためのHUBO
2つの素数の積である**整数の因数分解**を考えます。
例えば、積 $pq = 35$ が与えられたとき、2つの素因数 $p=5$ と $q=7$ を求めることが目標です。

$\sqrt{15}=5.91$ かつ $35/2=17.5$ であるため、$p$ と $q$ の探索範囲を以下のように制限できます:

$$
\begin{aligned}
  2 \leq &p \leq 5 \\
  6 \leq &q \leq 17
\end{aligned}
$$

このような整数変数に対して、$35$ の因数分解問題は以下のペナルティ式を用いて定式化できます:

$$
\begin{aligned}\
f(p,q) &= (pq-35)^2
\end{aligned}
$$

整数変数 $p$ と $q$ はバイナリ変数の線形式として実装されるため、その積 $pq$ は2次式となり、したがって $f(p,q)$ は4次式になります。
明らかに、$f(p,q)$ は $p$ と $q$ が35の正しい因数であるとき、かつそのときに限り最小値0を達成します。

## 因数分解のためのQUBO++プログラム
以下のQUBO++プログラムはHUBO式 $f(p,q)$ を構築し、Easy Solverを使って最適化問題を解きます:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 5;
  auto q = 6 <= qbpp::var_int("q") <= 17;

  auto f = p * q == 35;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  std::cout << "sol = " << sol << std::endl;
  std::cout << "p = " << sol(p) << std::endl;
  std::cout << "q = " << sol(q) << std::endl;
}
```

このプログラムでは、式 `p * q == 35` が自動的に `qbpp::sqr(p * q - 35)` に変換され、等式が満たされたときにエネルギー値0を達成します。
このプログラムの出力は以下の通りです:

{% raw %}
```cpp
f = 529 -240*p[0] -408*p[1] -88*q[0] -168*q[1] -304*q[2] -304*q[3] +144*p[0]*p[1] -5*p[0]*q[0] +40*p[0]*q[2] +40*p[0]*q[3] +16*p[1]*q[0] +56*p[1]*q[1] +208*p[1]*q[2] +208*p[1]*q[3] +16*q[0]*q[1] +32*q[0]*q[2] +32*q[0]*q[3] +64*q[1]*q[2] +64*q[1]*q[3] +128*q[2]*q[3] +52*p[0]*p[1]*q[0] +112*p[0]*p[1]*q[1] +256*p[0]*p[1]*q[2] +256*p[0]*p[1]*q[3] +20*p[0]*q[0]*q[1] +40*p[0]*q[0]*q[2] +40*p[0]*q[0]*q[3] +80*p[0]*q[1]*q[2] +80*p[0]*q[1]*q[3] +160*p[0]*q[2]*q[3] +48*p[1]*q[0]*q[1] +96*p[1]*q[0]*q[2] +96*p[1]*q[0]*q[3] +192*p[1]*q[1]*q[2] +192*p[1]*q[1]*q[3] +384*p[1]*q[2]*q[3] +16*p[0]*p[1]*q[0]*q[1] +32*p[0]*p[1]*q[0]*q[2] +32*p[0]*p[1]*q[0]*q[3] +64*p[0]*p[1]*q[1]*q[2] +64*p[0]*p[1]*q[1]*q[3] +128*p[0]*p[1]*q[2]*q[3]
sol = 0:{{p[0],1},{p[1],1},{q[0],1},{q[1],0},{q[2],0},{q[3],0}}
p = 5
q = 7
```
{% endraw %}
出力から、式 `f` が4次の項を含んでおり、HUBO式であることが確認できます。
ソルバーは素因数 $p=5$ と $q=7$ を正しく求めています。

## 大きな数の素因数分解のための無制限の大きな係数
QUBO++では、式の係数とエネルギー値のデータ型はデフォルトでそれぞれ `int32_t` と `int64_t` です。
これらの型はマクロ **`COEFF_TYPE`** と **`ENERGY_TYPE`** を定義することで変更できます。

さらに、QUBO++は任意の大きさの係数とエネルギー値を持つ式をサポートしています。
このオプションを有効にするには、両方のマクロを **`qbpp::cpp_int`** に設定します。
以下のQUBO++プログラムは、2つの大きな素数の積を因数分解します:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto p = 2 <= qbpp::var_int("p") <= qbpp::cpp_int("2000000");
  auto q = 2 <= qbpp::var_int("q") <= qbpp::cpp_int("2000000");

  auto f = p * q == qbpp::cpp_int("1000039") * qbpp::cpp_int("1000079");
  std::cout << "f = " << f.simplify_as_binary() << std::endl;

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  std::cout << "sol = " << sol << std::endl;
  std::cout << "p = " << sol(p) << std::endl;
  std::cout << "q = " << sol(q) << std::endl;
}
```

`qbpp/qbpp.hpp` をインクルードする前に、マクロ `COEFF_TYPE` と `ENERGY_TYPE` を `qbpp::cpp_int` に設定します。
定数整数は **`qbpp::cpp_int()`** に文字列リテラルを引数として指定します。

このプログラムは以下の結果を出力します:
{% raw %}
```txt
f = 1000236020078726181467929 -4000472012304*p[0] -8000944024600*p[1] -16001888049168*p[2] -32003776098208*p[3] -64007552195904*p[4] -128015104389760*p[5] -256030208771328*p[6] -512060417509888*p[7] -1024120834888704*p[8] -2048241669253120*p[9] -4096483336409088*p[10] -8192966664429568*p[11] -16385933295304704*p[12] -32771866456391680*p[13] -65543732375912448*p[14] -131087462604341248*p[15] -262174916618747904*p[16] -524349798877757440*p[17] -1048699460316561408*p[18] -2097398370877308928*p[19] -3806137462543214568*p[20] -4000472012304*q[0] -8000944024600*q[1] -16001888049168*q[2] -32003776098208*q[3] -64007552195904*q[4] -128015104389760*q[5] -256030208771328*q[6] -512060417509888*q[7] -1024120834888704*q[8] -2048241669253120*q[9] -4096483336409088*q[10] -8192966664429568*q[11] -16385933295304704*q[12] -32771866456391680*q[13] -65543732375912448*q[14] -131087462604341248*q[15] -262174916618747904*q[16] -524349798877757440*q[17] -1048699460316561408*q[18] -2097398370877308928*q[19] -3806137462543214568*q[20]

[omitted]

+17139313073086877663232*p[19]*p[20]*q[14]*q[19] +31102631877776215375872*p[19]*p[20]*q[14]*q[20] +4284828268271719415808*p[19]*p[20]*q[15]*q[16] +8569656536543438831616*p[19]*p[20]*q[15]*q[17] +17139313073086877663232*p[19]*p[20]*q[15]*q[18] +34278626146173755326464*p[19]*p[20]*q[15]*q[19] +62205263755552430751744*p[19]*p[20]*q[15]*q[20] +17139313073086877663232*p[19]*p[20]*q[16]*q[17] +34278626146173755326464*p[19]*p[20]*q[16]*q[18] +68557252292347510652928*p[19]*p[20]*q[16]*q[19] +124410527511104861503488*p[19]*p[20]*q[16]*q[20] +68557252292347510652928*p[19]*p[20]*q[17]*q[18] +137114504584695021305856*p[19]*p[20]*q[17]*q[19] +248821055022209723006976*p[19]*p[20]*q[17]*q[20] +274229009169390042611712*p[19]*p[20]*q[18]*q[19] +497642110044419446013952*p[19]*p[20]*q[18]*q[20] +995284220088838892027904*p[19]*p[20]*q[19]*q[20]
sol = 0:{{p[0],0},{p[1],1},{p[2],1},{p[3],1},{p[4],0},{p[5],0},{p[6],0},{p[7],0},{p[8],0},{p[9],1},{p[10],1},{p[11],1},{p[12],1},{p[13],1},{p[14],0},{p[15],1},{p[16],0},{p[17],0},{p[18],0},{p[19],0},{p[20],1},{q[0],0},{q[1],1},{q[2],1},{q[3],0},{q[4],0},{q[5],1},{q[6],1},{q[7],1},{q[8],1},{q[9],0},{q[10],1},{q[11],1},{q[12],1},{q[13],1},{q[14],0},{q[15],1},{q[16],0},{q[17],0},{q[18],0},{q[19],0},{q[20],1}}
p = 1000079
q = 1000039
```
{% endraw %}
式 `f` が非常に大きな係数を含んでおり、大きな合成数の因数分解が正しく得られたことがわかります。

>**TIP**
> **`COEFF_TYPE`** と **`ENERGY_TYPE`** は **`int8_t`**、**`int16_t`**、**`int32_t`**、**`int64_t`**、**`qbpp::int128_t`**、**`qbpp::int256_t`**、**`qbpp::int512_t`**、**`qbpp::int1024_t`**、または **`qbpp::cpp_int`** に設定できます。
</div>
