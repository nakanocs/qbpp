---
layout: default
nav_exclude: true
title: "Solving Expressions"
nav_order: 2
---
<div class="lang-en" markdown="1">
# Solving Expressions

QUBO++ provides three solvers for QUBO/HUBO expressions:

- **Easy Solver**
  - Runs a heuristic algorithm based on simulated annealing.
  - Runs in parallel on multicore CPUs using **Intel Threading Building Blocks (oneTBB)**.
  - Does not guarantee optimality.

- **Exhaustive Solver**
  - Explores all possible solutions.
  - Guarantees optimality of the returned solution.
  - Is computationally feasible only when the number of binary variables is about 30–40 or fewer.
  - If a CUDA GPU is available, GPU acceleration is automatically enabled alongside CPU threads.

- **ABS3 Solver**
  - A high-performance solver that uses CUDA GPUs and multicore CPUs.
  - Does not guarantee optimality, but is much more powerful than the Easy Solver.
  - If no GPU is available, falls back to CPU-only mode.

The Easy Solver and Exhaustive Solver are used in two steps:
1. Create a solver object, **`qbpp::easy_solver::EasySolver`** or **`qbpp::exhaustive_solver::ExhaustiveSolver`**.
2. Call the **`search()`** member function on the solver object. It returns a **`qbpp::Sol`** object that stores the obtained solution.

## Easy Solver
To use the **Easy Solver**, include the header file **`qbpp/easy_solver.hpp`**.
It is defined in the namespace **`qbpp::easy_solver`**.

We use the following expression $f(a,b,c,d)$ as an example:

$$
\begin{aligned}
f(a,b,c,d,e) &= (a+2b+3c+4d-5)^2
\end{aligned}
$$

Clearly, this expression attains its minimum value $f=0$
when $a+2b+3c+4d=5$.
Therefore, it has two optimal solutions, $(a,b,c,d)=(0,1,1,0)$ and $(1,0,0,1)$.

In the following program, expression `f` is created using the symbolic computation.
Note that the function **`qbpp::sqr()`** returns the square of the argument.
We then construct an instance of the class `qbpp::easy_solver::EasySolver`
by passing `f` to its constructor.
Before doing so, `f` must be simplified for binary variables by calling **`simplify_as_binary()`**.
The constructor returns an `EasySolver` object named **`solver`**.
Since we know that the optimal value is $f=0$, we set the target energy to $0$ by calling the **`target_energy()`** member function.
Calling the **`search()`** member function on `solver` returns a solution instance **`sol`**  of
class **`qbpp::Sol`**, which is printed using `std::cout`.

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << sol << std::endl;
}
```

The output of this program is as follows:
{% raw %}
```
f = 25 -9*a -16*b -21*c -24*d +4*a*b +6*a*c +8*a*d +12*b*c +16*b*d +24*c*d
0:{{a,1},{b,0},{c,0},{d,1}}
```
{% endraw %}
One of the optimal solutions is correctly output.

## Exhaustive Solver
To use the **Exhaustive Solver**, include the header file **`qbpp/exhaustive_solver.hpp`**.
It is defined in the namespace **`qbpp::exhaustive_solver`**.

We construct an instance **`solver`** of the class **`qbpp::exhaustive_solver::ExhaustiveSolver`**
by passing `f` to its constructor.
Calling the **`search()`** member function on `solver` returns a solution instance **`sol`** of
class **`qbpp::Sol`**, which is printed using `std::cout`.
Since the Exhaustive Solver explores all possible assignments, it is guaranteed that `sol`
stores an optimal solution.

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search();
  std::cout << sol << std::endl;
}
```
The output of this program is as follows:
{% raw %}
```
0:{{a,0},{b,1},{c,1},{d,0}}
```
{% endraw %}
All optimal solutions can be obtained by the **`search_optimal_solutions()`** member function as follows:
```cpp
  auto sol = solver.search_optimal_solutions();
```
The output is as follows:
{% raw %}
```
(0) 0:{{a,0},{b,1},{c,1},{d,0}}
(1) 0:{{a,1},{b,0},{c,0},{d,1}}
```
{% endraw %}
Furthermore, all solutions including non-optimal ones can be obtained by the
**`search_all_solutions()`** member function as follows:
```cpp
 auto sol = solver.search_all_solutions();
```
The output is as follows:
{% raw %}
```txt
(0) 0:{{a,0},{b,1},{c,1},{d,0}}
(1) 0:{{a,1},{b,0},{c,0},{d,1}}
(2) 1:{{a,0},{b,0},{c,0},{d,1}}
(3) 1:{{a,0},{b,1},{c,0},{d,1}}
(4) 1:{{a,1},{b,0},{c,1},{d,0}}
(5) 1:{{a,1},{b,1},{c,1},{d,0}}
(6) 4:{{a,0},{b,0},{c,1},{d,0}}
(7) 4:{{a,0},{b,0},{c,1},{d,1}}
(8) 4:{{a,1},{b,1},{c,0},{d,0}}
(9) 4:{{a,1},{b,1},{c,0},{d,1}}
(10) 9:{{a,0},{b,1},{c,0},{d,0}}
(11) 9:{{a,1},{b,0},{c,1},{d,1}}
(12) 16:{{a,0},{b,1},{c,1},{d,1}}
(13) 16:{{a,1},{b,0},{c,0},{d,0}}
(14) 25:{{a,0},{b,0},{c,0},{d,0}}
(15) 25:{{a,1},{b,1},{c,1},{d,1}}
```
{% endraw %}
The Exhaustive Solver is very useful for analyzing small expressions and for debugging.

## ABS3 Solver
To use the **ABS3 Solver**, include the header file **`qbpp/abs3_solver.hpp`**.
It is defined in the namespace **`qbpp::abs3`**.

The ABS3 Solver is a high-performance solver that uses CUDA GPUs and multicore CPUs.
If no GPU is available, it automatically falls back to CPU-only mode.

Usage involves three steps:
1. Create an **`qbpp::abs3::ABS3Solver`** object for the expression.
2. Create a **`qbpp::abs3::Params`** object and set search options.
3. Call the **`search()`** member function, which returns the obtained solution.

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  f.simplify_as_binary();

  auto solver = qbpp::abs3::ABS3Solver(f);
  auto params = qbpp::abs3::Params();
  params("time_limit", "5.0");
  params("target_energy", "0");
  params("enable_default_callback", "1");
  auto sol = solver.search(params);
  std::cout << sol << std::endl;
}
```
The output of this program is as follows:
{% raw %}
```
TTS = 0.000s Energy = 0
0:{{a,0},{b,1},{c,1},{d,0}}
```
{% endraw %}

For details on parameters, callbacks, multiple solution collection, and solution hints, see **[ABS3 Solver](ABS3)**.
</div>

<div class="lang-ja" markdown="1">
# 式の求解

QUBO++はQUBO/HUBO式を解くための3つのソルバーを提供しています：

- **Easy Solver**
  - シミュレーテッドアニーリングに基づくヒューリスティックアルゴリズムを実行します。
  - **Intel Threading Building Blocks (oneTBB)**を使用してマルチコアCPU上で並列に動作します。
  - 最適性は保証されません。

- **Exhaustive Solver**
  - すべての可能な解を探索します。
  - 返される解の最適性が保証されます。
  - バイナリ変数の数が約30～40以下の場合にのみ計算が現実的です。
  - CUDA GPUが利用可能な場合、CPUスレッドと並行してGPUアクセラレーションが自動的に有効になります。

- **ABS3 Solver**
  - CUDA GPUとマルチコアCPUを活用する高性能ソルバーです。
  - 最適性は保証されませんが、Easy Solverよりはるかに強力です。
  - GPUが利用できない場合はCPUのみモードにフォールバックします。

Easy SolverとExhaustive Solverは以下の2ステップで使用します：
1. ソルバーオブジェクト（**`qbpp::easy_solver::EasySolver`**または**`qbpp::exhaustive_solver::ExhaustiveSolver`**）を作成します。
2. ソルバーオブジェクトの**`search()`**メンバ関数を呼び出します。得られた解を格納する**`qbpp::Sol`**オブジェクトが返されます。

## Easy Solver
**Easy Solver**を使用するには、ヘッダファイル**`qbpp/easy_solver.hpp`**をインクルードします。
名前空間**`qbpp::easy_solver`**で定義されています。

以下の式 $f(a,b,c,d)$ を例として使用します：

$$
\begin{aligned}
f(a,b,c,d,e) &= (a+2b+3c+4d-5)^2
\end{aligned}
$$

明らかに、$a+2b+3c+4d=5$ のとき、この式は最小値 $f=0$ を取ります。
したがって、$(a,b,c,d)=(0,1,1,0)$ と $(1,0,0,1)$ の2つの最適解があります。

以下のプログラムでは、シンボリック計算を使って式 `f` を作成します。
関数**`qbpp::sqr()`**は引数の2乗を返すことに注意してください。
次に、`f` をコンストラクタに渡してクラス `qbpp::easy_solver::EasySolver` のインスタンスを構築します。
その前に、**`simplify_as_binary()`**を呼び出して `f` をバイナリ変数用に簡約化する必要があります。
コンストラクタは**`solver`**という名前の `EasySolver` オブジェクトを返します。
最適値が $f=0$ であることがわかっているので、**`target_energy()`**メンバ関数を呼び出してターゲットエネルギーを $0$ に設定します。
`solver` の**`search()`**メンバ関数を呼び出すと、クラス**`qbpp::Sol`**の解インスタンス**`sol`**が返され、`std::cout` で出力されます。

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << sol << std::endl;
}
```

このプログラムの出力は以下のとおりです：
{% raw %}
```
f = 25 -9*a -16*b -21*c -24*d +4*a*b +6*a*c +8*a*d +12*b*c +16*b*d +24*c*d
0:{{a,1},{b,0},{c,0},{d,1}}
```
{% endraw %}
最適解の1つが正しく出力されています。

## Exhaustive Solver
**Exhaustive Solver**を使用するには、ヘッダファイル**`qbpp/exhaustive_solver.hpp`**をインクルードします。
名前空間**`qbpp::exhaustive_solver`**で定義されています。

`f` をコンストラクタに渡してクラス**`qbpp::exhaustive_solver::ExhaustiveSolver`**のインスタンス**`solver`**を構築します。
`solver` の**`search()`**メンバ関数を呼び出すと、クラス**`qbpp::Sol`**の解インスタンス**`sol`**が返され、`std::cout` で出力されます。
Exhaustive Solverはすべての可能な割り当てを探索するため、`sol` に最適解が格納されることが保証されます。

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search();
  std::cout << sol << std::endl;
}
```
このプログラムの出力は以下のとおりです：
{% raw %}
```
0:{{a,0},{b,1},{c,1},{d,0}}
```
{% endraw %}
すべての最適解は**`search_optimal_solutions()`**メンバ関数で以下のように取得できます：
```cpp
  auto sol = solver.search_optimal_solutions();
```
出力は以下のとおりです：
{% raw %}
```
(0) 0:{{a,0},{b,1},{c,1},{d,0}}
(1) 0:{{a,1},{b,0},{c,0},{d,1}}
```
{% endraw %}
さらに、非最適解を含むすべての解は**`search_all_solutions()`**メンバ関数で以下のように取得できます：
```cpp
 auto sol = solver.search_all_solutions();
```
出力は以下のとおりです：
{% raw %}
```txt
(0) 0:{{a,0},{b,1},{c,1},{d,0}}
(1) 0:{{a,1},{b,0},{c,0},{d,1}}
(2) 1:{{a,0},{b,0},{c,0},{d,1}}
(3) 1:{{a,0},{b,1},{c,0},{d,1}}
(4) 1:{{a,1},{b,0},{c,1},{d,0}}
(5) 1:{{a,1},{b,1},{c,1},{d,0}}
(6) 4:{{a,0},{b,0},{c,1},{d,0}}
(7) 4:{{a,0},{b,0},{c,1},{d,1}}
(8) 4:{{a,1},{b,1},{c,0},{d,0}}
(9) 4:{{a,1},{b,1},{c,0},{d,1}}
(10) 9:{{a,0},{b,1},{c,0},{d,0}}
(11) 9:{{a,1},{b,0},{c,1},{d,1}}
(12) 16:{{a,0},{b,1},{c,1},{d,1}}
(13) 16:{{a,1},{b,0},{c,0},{d,0}}
(14) 25:{{a,0},{b,0},{c,0},{d,0}}
(15) 25:{{a,1},{b,1},{c,1},{d,1}}
```
{% endraw %}
Exhaustive Solverは、小さな式の解析やデバッグに非常に有用です。

## ABS3 Solver
**ABS3 Solver**を使用するには、ヘッダファイル**`qbpp/abs3_solver.hpp`**をインクルードします。
名前空間**`qbpp::abs3`**で定義されています。

ABS3 Solverは、CUDA GPUとマルチコアCPUを活用する高性能ソルバーです。
GPUが利用できない場合は、自動的にCPUのみモードにフォールバックします。

使用方法は以下の3ステップです：
1. 式に対して**`qbpp::abs3::ABS3Solver`**オブジェクトを作成します。
2. **`qbpp::abs3::Params`**オブジェクトを作成し、探索オプションを設定します。
3. **`search()`**メンバ関数を呼び出します。得られた解が返されます。

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  f.simplify_as_binary();

  auto solver = qbpp::abs3::ABS3Solver(f);
  auto params = qbpp::abs3::Params();
  params("time_limit", "5.0");
  params("target_energy", "0");
  params("enable_default_callback", "1");
  auto sol = solver.search(params);
  std::cout << sol << std::endl;
}
```
このプログラムの出力は以下のとおりです：
{% raw %}
```
TTS = 0.000s Energy = 0
0:{{a,0},{b,1},{c,1},{d,0}}
```
{% endraw %}

パラメータ、コールバック、複数解の収集、ヒント解の詳細については**[ABS3 Solver](ABS3)**をご覧ください。
</div>
