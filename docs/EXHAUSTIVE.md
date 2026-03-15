---
layout: default
nav_exclude: true
title: "Exhaustive Solver"
nav_order: 20
---
<div class="lang-en" markdown="1">
# Exhaustive Solver Usage
The **Exhaustive Solver** is a complete-search solver for QUBO/HUBO expressions.
Since all possible assignments are examined, the optimality of the solutions is guaranteed.
The search is parallelized using CPU threads, and if a CUDA GPU is available, GPU acceleration is automatically enabled to further speed up the search.

Solving a problem with the Exhaustive Solver consists of the following three steps:
1. Create an Exhaustive Solver (or `qbpp::exhaustive_solver::ExhaustiveSolver`) object.
2. Set search options by calling member functions of the solver object.
3. Search for solutions by calling one of the search member functions.


## Creating Exhaustive Solver object
To use the Exhaustive Solver, an Exhaustive Solver object (or
`qbpp::exhaustive_solver::ExhaustiveSolver`) is constructed with an expression
(`or qbpp::Expr`) object as follows:
- **`qbpp::exhaustive_solver::ExhaustiveSolver(const qbpp::Expr& f)`**:
Here, `f` is the expression to be solved.
It must be simplified as a binary expression in advance by calling the
`simplify_as_binary()` function.
This function converts the given expression `f` into an internal format that is
used during the solution search.

## Setting Exhaustive Solver Options
- **`verbose()`**:
Displays the search progress as a percentage, which is helpful for estimating the total runtime.
- **`enable_default_callback()`**:
Enables the default callback function, which prints newly obtained best solutions.
- **`target_energy(energy_t energy)`**:
Sets a target energy value for early termination.
When the solver finds a solution with energy less than or equal to the target, the search terminates immediately.

## Searching Solutions
The Exhaustive Solver searches for solutions by calling one of the following
member functions of the solver object:
- **`search()`**: Returns the best solution found. If a CUDA GPU is available, the search is automatically accelerated using the GPU alongside CPU threads.
- **`search_optimal_solutions()`**: Returns a `Sol` object containing all optimal solutions in `sol.all_solutions()`.
- **`search_topk_solutions(size_t k)`**: Returns a `Sol` object containing the top-k solutions with the lowest energy in `sol.all_solutions()`, sorted in increasing order of energy.
- **`search_all_solutions()`**: Returns a `Sol` object containing all solutions in `sol.all_solutions()`, sorted in increasing order of energy.

# Program example
The following program searches for a solution to the
**Low Autocorrelation Binary Sequences (LABS)** problem using the Exhaustive
Solver:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  size_t size = 20;
  auto x = qbpp::var("x", size);
  auto f = qbpp::expr();
  for (size_t d = 1; d < size; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < size - d; ++i) {
      temp += (2 * x[i] - 1) * (2 * x[i + d] - 1);
    }
    f += qbpp::sqr(temp);
  }
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  solver.enable_default_callback();
  auto sol = solver.search();
  std::cout << sol.energy() << ": ";
  for (auto val : sol(x)) {
    std::cout << (val == 0 ? "-" : "+");
  }
  std::cout << std::endl;
}
```
The output of this program is as follows:
{% raw %}
```txt
TTS = 0.000s Energy = 1506
TTS = 0.000s Energy = 1030
TTS = 0.000s Energy = 502
TTS = 0.000s Energy = 446
TTS = 0.000s Energy = 234
TTS = 0.000s Energy = 110
TTS = 0.001s Energy = 106
TTS = 0.001s Energy = 74
TTS = 0.001s Energy = 66
TTS = 0.001s Energy = 42
TTS = 0.001s Energy = 34
TTS = 0.004s Energy = 26
26: --++-++----+----+-+-
```
{% endraw %}
All optimal solutions can be obtained by calling the
**`search_optimal_solutions()`** member function as follows:
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();
  for (const auto& s : sol.all_solutions()) {
    std::cout << s.energy() << ": ";
    for (auto val : s(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
The output is as follows:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
```
{% endraw %}
The top-k solutions with the lowest energy can be obtained by calling the
**`search_topk_solutions(k)`** member function as follows:
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_topk_solutions(10);
  for (const auto& s : sol.all_solutions()) {
    std::cout << s.energy() << ": ";
    for (auto val : s(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
The output is as follows:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
34: ++--++--+-++-+-+++++
34: +++---+-+-++++-++-++
```
{% endraw %}
Furthermore, all solutions, including non-optimal ones, can be obtained by calling
the **`search_all_solutions()`** member function as follows.
Note that this function stores all $2^n$ solutions in memory, where $n$ is the number of variables.
For example, with $n = 20$, over one million solutions are stored, and memory usage grows exponentially with $n$.
Use this function only when $n$ is small enough.
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_all_solutions();
  for (const auto& s : sol.all_solutions()) {
    std::cout << s.energy() << ": ";
    for (auto val : s(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
This prints all $2^{20}$ solutions in increasing order of energy, as
shown below:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
34: -----+-+--+-++--++--
34: ----+----++-++---+-+
34: ----+--+++--+-+++-+-
34: ---+++-+-+----+--+--
34: ---+++++-+++-++-+-++
34: --+--+----+-+-+++---
34: --+-+--+---+-----+++
34: --++--++-+--+-+-----
34: -+--+------+-+++---+
34: -+--+-+---+---+++++-
[omitted]
```
{% endraw %}
</div>

<div class="lang-ja" markdown="1">
# Exhaustive Solverの使い方
**Exhaustive Solver**はQUBO/HUBO式のための完全探索ソルバーです。
すべての可能な割り当てが検査されるため、解の最適性が保証されます。
探索はCPUスレッドを使用して並列化され、CUDA GPUが利用可能な場合は、探索をさらに高速化するためにGPUアクセラレーションが自動的に有効になります。

Exhaustive Solverを使って問題を解くには、以下の3つのステップで行います：
1. Exhaustive Solver（`qbpp::exhaustive_solver::ExhaustiveSolver`）オブジェクトを作成します。
2. ソルバーオブジェクトのメンバ関数を呼び出して探索オプションを設定します。
3. 探索メンバ関数のいずれかを呼び出して解を探索します。


## Exhaustive Solverオブジェクトの作成
Exhaustive Solverを使用するには、式（`qbpp::Expr`）オブジェクトを引数としてExhaustive Solverオブジェクト（`qbpp::exhaustive_solver::ExhaustiveSolver`）を以下のように構築します：
- **`qbpp::exhaustive_solver::ExhaustiveSolver(const qbpp::Expr& f)`**:
ここで、`f` は解くべき式です。
事前に `simplify_as_binary()` 関数を呼び出してバイナリ式として簡約化しておく必要があります。
この関数は与えられた式 `f` を解探索中に使用される内部フォーマットに変換します。

## Exhaustive Solverオプションの設定
- **`verbose()`**:
探索の進捗をパーセンテージで表示します。総実行時間の推定に役立ちます。
- **`enable_default_callback()`**:
新たに得られた最良解を出力するデフォルトコールバック関数を有効にします。
- **`target_energy(energy_t energy)`**:
早期終了のためのターゲットエネルギー値を設定します。
ソルバーがターゲット以下のエネルギーを持つ解を見つけると、探索は直ちに終了します。

## 解の探索
Exhaustive Solverは、ソルバーオブジェクトの以下のメンバ関数のいずれかを呼び出すことで解を探索します：
- **`search()`**: 見つかった最良解を返します。CUDA GPUが利用可能な場合、CPUスレッドと並行してGPUを使用した探索の高速化が自動的に行われます。
- **`search_optimal_solutions()`**: `sol.all_solutions()` にすべての最適解を含む `Sol` オブジェクトを返します。
- **`search_topk_solutions(size_t k)`**: `sol.all_solutions()` に最小エネルギーのtop-k解をエネルギーの昇順で含む `Sol` オブジェクトを返します。
- **`search_all_solutions()`**: `sol.all_solutions()` にすべての解をエネルギーの昇順で含む `Sol` オブジェクトを返します。

# プログラム例
以下のプログラムは、Exhaustive Solverを使用して**Low Autocorrelation Binary Sequences (LABS)**問題の解を探索します：
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/exhaustive_solver.hpp>

int main() {
  size_t size = 20;
  auto x = qbpp::var("x", size);
  auto f = qbpp::expr();
  for (size_t d = 1; d < size; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < size - d; ++i) {
      temp += (2 * x[i] - 1) * (2 * x[i + d] - 1);
    }
    f += qbpp::sqr(temp);
  }
  f.simplify_as_binary();

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  solver.enable_default_callback();
  auto sol = solver.search();
  std::cout << sol.energy() << ": ";
  for (auto val : sol(x)) {
    std::cout << (val == 0 ? "-" : "+");
  }
  std::cout << std::endl;
}
```
このプログラムの出力は以下のとおりです：
{% raw %}
```txt
TTS = 0.000s Energy = 1506
TTS = 0.000s Energy = 1030
TTS = 0.000s Energy = 502
TTS = 0.000s Energy = 446
TTS = 0.000s Energy = 234
TTS = 0.000s Energy = 110
TTS = 0.001s Energy = 106
TTS = 0.001s Energy = 74
TTS = 0.001s Energy = 66
TTS = 0.001s Energy = 42
TTS = 0.001s Energy = 34
TTS = 0.004s Energy = 26
26: --++-++----+----+-+-
```
{% endraw %}
すべての最適解は**`search_optimal_solutions()`**メンバ関数を以下のように呼び出すことで取得できます：
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();
  for (const auto& s : sol.all_solutions()) {
    std::cout << s.energy() << ": ";
    for (auto val : s(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
出力は以下のとおりです：
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
```
{% endraw %}
最小エネルギーのtop-k解は**`search_topk_solutions(k)`**メンバ関数を以下のように呼び出すことで取得できます：
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_topk_solutions(10);
  for (const auto& s : sol.all_solutions()) {
    std::cout << s.energy() << ": ";
    for (auto val : s(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
出力は以下のとおりです：
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
34: ++--++--+-++-+-+++++
34: +++---+-+-++++-++-++
```
{% endraw %}
さらに、非最適解を含むすべての解は**`search_all_solutions()`**メンバ関数を以下のように呼び出すことで取得できます。
この関数はすべての $2^n$ 個の解をメモリに格納することに注意してください。ここで $n$ は変数の数です。
例えば、$n = 20$ の場合、100万個以上の解が格納され、メモリ使用量は $n$ に対して指数的に増加します。
$n$ が十分小さい場合にのみこの関数を使用してください。
```cpp
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_all_solutions();
  for (const auto& s : sol.all_solutions()) {
    std::cout << s.energy() << ": ";
    for (auto val : s(x)) {
      std::cout << (val == 0 ? "-" : "+");
    }
    std::cout << std::endl;
  }
```
以下に示すように、すべての $2^{20}$ 個の解がエネルギーの昇順で出力されます：
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
34: -----+-+--+-++--++--
34: ----+----++-++---+-+
34: ----+--+++--+-+++-+-
34: ---+++-+-+----+--+--
34: ---+++++-+++-++-+-++
34: --+--+----+-+-+++---
34: --+-+--+---+-----+++
34: --++--++-+--+-+-----
34: -+--+------+-+++---+
34: -+--+-+---+---+++++-
[omitted]
```
{% endraw %}
</div>
