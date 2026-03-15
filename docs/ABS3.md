---
layout: default
nav_exclude: true
title: "ABS3 Solver"
nav_order: 21
---
<div class="lang-en" markdown="1">
# ABS3 Solver Usage
Solving an expression `f` using the ABS3 Solver involves the following three steps:
1. Create an ABS3 Solver (or **`qbpp::abs3::ABS3Solver`**) object for the expression `f`.
2. Create a parameter object (or **`qbpp::abs3::Params`**) and set the options for solution search.
3. Call the **`search()`** member function of the ABS3 Solver object, which returns the obtained solution.

## Solving LABS problem using the ABS3 Solver
The following QUBO++ program solves the **Low Autocorrelation Binary Sequence (LABS)** problem using the ABS3 Solver:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

int main() {
  const size_t size = 100;
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

  auto solver = qbpp::abs3::ABS3Solver(f);
  auto params = qbpp::abs3::Params();
  params("time_limit", "10.0");
  params("enable_default_callback", "1");

  auto sol = solver.search(params);
  std::cout << sol.energy() << ": ";
  for (auto val : sol(x)) {
    std::cout << (val == 0 ? "-" : "+");
  }
  std::cout << std::endl;
}
```
In this program, an ABS3 Solver object **`solver`** is first created for the expression `f`.
Next, a parameter object **`params`** is created, and the options `time_limit` and `enable_callback` are set.
The `time_limit` option specifies the maximum search time in seconds, while `enable_default_callback` enables a built-in callback function that prints the energy and TTS of newly found best solutions during the search.
The **`search()`** member function of the `solver` is then invoked with `params` as its argument.
This function returns the best solution found within the given time limit, which is stored in `sol`.

The program prints the energy of the solution and the corresponding binary sequence, where "+" represents 1 and "-" represents 0.

This program produces the following output:
```
TTS = 0.002s Energy = 1218
TTS = 0.002s Energy = 1170
TTS = 0.002s Energy = 994
TTS = 0.015s Energy = 958
TTS = 0.018s Energy = 922
TTS = 0.034s Energy = 874
TTS = 4.364s Energy = 834
834: -+--+---++-++-+---++-++--+++--+-+-+++++----+++-+-+---++-+--+-----+--+----++----+-+--++++++---+------
```

## ABS3 Solver object
An ABS3 Solver (or `qbpp::abs3::ABS3Solver`) object is created for a given expression.
When the solver object is constructed, the expression is converted into an internal data format and loaded into GPU memory.
An optional second argument `gpu` controls GPU usage:
- **`qbpp::abs3::ABS3Solver(expression)`**: Automatically uses all available GPUs. If no GPU is available, falls back to CPU-only mode.
- **`qbpp::abs3::ABS3Solver(expression, 0)`**: Forces CPU-only mode (no GPU is used).
- **`qbpp::abs3::ABS3Solver(expression, n)`**: Uses `n` GPUs.

A parameter object (or `qbpp::abs3::Params`) stores the options used by the ABS3 Solver during solution search.
Each option is specified as a key–value pair of strings.
In the example above:
- **`params("time_limit", "10.0")`**: Sets the time limit to 10.0 seconds.
- **`params("enable_default_callback", "1")`**: Enables the built-in callback function, which prints the energy of newly obtained solutions.

## ABS3 Parameters
A parameter (or `qbpp::abs3::Params()`) object stores parameter options used when the ABS3 Solver searches a solution.
Options consist of the key and the value as strings.
In the program above, `params("time_limit", "10.0")` sets the time limit to 10.0 seconds
and `params("enable_default_callback", "1")` enables the built-in callback function, which prints the energy of newly obtained solutions.

### Basic Options

| Key | Value | Description |
|----|----|----|
| **`time_limit`** | time limit in seconds | Terminates the search when the time limit is reached |
| **`target_energy`** | target energy value | Terminates the search when the target energy is achieved |

### Advanced Options

| Key | Value | Description |
|----|----|----|
| **`enable_default_callback`** | "1" | Enables the built-in callback that prints energy and TTS |
| **`cpu_enable`** | "0" or "1" | Enables/disables the CPU solver running alongside the GPU (default: "1") |
| **`cpu_thread_count`** | number of CPU threads | Number of CPU solver threads (default: auto) |
| **`block_count`** | CUDA block count per GPU | Number of CUDA blocks launched by the solver kernel |
| **`thread_count`** | thread count per CUDA block | Number of threads per CUDA block |
| **`topk_sols`** | number of solutions | Returns the top-K solutions with the best energies |
| **`best_energy_sols`** | max count ("0" = unlimited) | Returns all solutions with the best energy found |

## Collecting Multiple Solutions

The ABS3 Solver can collect multiple solutions during the search.
Two modes are available:

### Top-K Solutions (`topk_sols`)

The `topk_sols` parameter collects the top-K solutions sorted by energy in ascending order.

```cpp
params.add("topk_sols", "10");  // collect up to 10 best solutions
```

### Best Energy Solutions (`best_energy_sols`)

The `best_energy_sols` parameter collects all solutions that share the best energy found.
Whenever a better energy is discovered, the pool is cleared and only solutions with the new best energy are kept.

```cpp
solver.enable_best_energy_sols();  // collect all best-energy solutions (unlimited)
```

Alternatively, `best_energy_sols` can be set as a parameter with a maximum count ("0" means unlimited):
```cpp
params.add("best_energy_sols", "0");  // unlimited
params.add("best_energy_sols", "100");  // collect up to 100
```

Note that `topk_sols` and `best_energy_sols` share the same internal pool.
If both are specified, the last one takes effect.

### Accessing Collected Solutions

The `search()` method returns an `ABS3Sols` object, which provides access to the collected solutions:

```cpp
auto result = solver.search(params);

std::cout << "Best energy: " << result.energy() << std::endl;
std::cout << "Number of solutions: " << result.size() << std::endl;

for (const auto& sol : result.best_sols()) {
  std::cout << "Energy = " << sol.energy() << " TTS = " << sol.tts() << "s" << std::endl;
}
```

The `ABS3Sols` object supports:
- **`size()`** — number of collected solutions
- **`best_sols()`** / **`sols()`** — access the solution vector
- **`operator[](i)`** — access the i-th solution
- Range-based for loop iteration

## Custom Callback

The built-in callback (enabled by `enable_default_callback`) simply prints the energy and TTS whenever a new best solution is found.
For more control, you can subclass `ABS3Solver` and override the `callback()` virtual method.

The callback is invoked with one of the following events:

| Event | Description |
|-------|-------------|
| `CallbackEvent::Start` | Called once at the beginning of `search()` |
| `CallbackEvent::BestUpdated` | Called whenever a new best solution is found |
| `CallbackEvent::Timer` | Called periodically at a configurable interval |

Inside the callback, the following methods are available:
- **`best_sol()`** — returns `const qbpp::Sol&` to the current best solution. Use `.energy()`, `.tts()`, `.get(var)`, etc.
- **`event()`** — returns the event that triggered this callback
- **`hint(sol)`** — provides a hint solution to the solver during the search (see [Solution Hint](#solution-hint))

### Timer Control

The `Timer` event is not enabled by default.
To enable periodic timer callbacks, call `timer(seconds)` inside the `callback()` method:
- **`timer(1.0)`** — fire `Timer` callbacks every 1 second
- **`timer(0)`** — disable the timer
- If `timer()` is not called, the timer interval remains unchanged.

Typically, `timer()` is called once during the `Start` callback to establish the interval.
It can also be called during `BestUpdated` or `Timer` callbacks to adjust or disable the timer dynamically.

### Example: Custom Callback
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

class MySolver : public qbpp::abs3::ABS3Solver {
 public:
  using ABS3Solver::ABS3Solver;

  void callback() const override {
    if (event() == qbpp::abs3::CallbackEvent::Start) {
      timer(1.0);  // enable timer callback every 1 second
    }
    if (event() == qbpp::abs3::CallbackEvent::BestUpdated) {
      std::cout << "New best: energy=" << best_sol().energy()
                << " TTS=" << best_sol().tts() << "s" << std::endl;
    }
  }
};

int main() {
  auto x = qbpp::var("x", 8);
  auto f = qbpp::sum(x) == 4;
  f.simplify_as_binary();

  auto solver = MySolver(f);
  auto params = qbpp::abs3::Params();
  params.add("time_limit", "5");
  params.add("target_energy", "0");
  auto sol = solver.search(params);
  std::cout << "energy=" << sol.energy() << std::endl;
}
```

## Solution Hint

A hint solution allows warm-starting a search with a previously found solution.

The simplest way is to call **`params.hint(sol)`** before `search()`:
```cpp
params.hint(sol);  // provide a hint solution for the search
auto result = solver.search(params);
```
The solution is written directly to the solver's internal data structure before the search begins.

For advanced use cases such as running an external solver (e.g., Gurobi) concurrently, you can also call **`hint(sol)`** during a callback to feed solutions dynamically.
In this scenario, setting up a periodic timer (e.g., `timer(1.0)`) is recommended so that the callback is invoked regularly to check for new external solutions.

### Example: Providing a Hint Solution

The following example solves a factorization problem twice.
The first run finds the optimal solution normally.
The second run provides the first solution as a hint via `params.hint(sol)`, causing the solver to converge much faster.

```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 1000;
  auto q = 2 <= qbpp::var_int("q") <= 1000;
  auto f = p * q == 899 * 997;
  f.simplify_as_binary();

  auto solver = qbpp::abs3::ABS3Solver(f);

  // Run 1: normal search
  auto params1 = qbpp::abs3::Params();
  params1.add("target_energy", "0");
  params1.add("time_limit", "10");
  params1.add("enable_default_callback", "1");
  const auto sol1 = solver.search(params1);
  std::cout << "Run 1: p=" << sol1(p) << " q=" << sol1(q)
            << " energy=" << sol1.energy() << std::endl;

  // Run 2: provide previous solution as a hint
  auto params2 = qbpp::abs3::Params();
  params2.add("target_energy", "0");
  params2.add("time_limit", "10");
  params2.add("enable_default_callback", "1");
  params2.hint(sol1);
  const auto sol2 = solver.search(params2);
  std::cout << "Run 2: p=" << sol2(p) << " q=" << sol2(q)
            << " energy=" << sol2.energy()
            << " TTS=" << sol2.tts() << "s" << std::endl;
}
```

The hint solution is written directly to the solver's internal data structure before the search begins.
The solver evaluates its energy and uses it as the initial state, then continues searching for better solutions.
</div>

<div class="lang-ja" markdown="1">
# ABS3 Solverの使い方
ABS3 Solverを使用して式 `f` を解くには、以下の3つのステップで行います：
1. 式 `f` に対してABS3 Solver（**`qbpp::abs3::ABS3Solver`**）オブジェクトを作成します。
2. パラメータオブジェクト（**`qbpp::abs3::Params`**）を作成し、解探索のオプションを設定します。
3. ABS3 Solverオブジェクトの**`search()`**メンバ関数を呼び出します。得られた解が返されます。

## ABS3 SolverによるLABS問題の求解
以下のQUBO++プログラムは、ABS3 Solverを使用して**Low Autocorrelation Binary Sequence (LABS)**問題を解きます：
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

int main() {
  const size_t size = 100;
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

  auto solver = qbpp::abs3::ABS3Solver(f);
  auto params = qbpp::abs3::Params();
  params("time_limit", "10.0");
  params("enable_default_callback", "1");

  auto sol = solver.search(params);
  std::cout << sol.energy() << ": ";
  for (auto val : sol(x)) {
    std::cout << (val == 0 ? "-" : "+");
  }
  std::cout << std::endl;
}
```
このプログラムでは、まず式 `f` に対してABS3 Solverオブジェクト**`solver`**を作成します。
次に、パラメータオブジェクト**`params`**を作成し、`time_limit` と `enable_callback` のオプションを設定します。
`time_limit` オプションは最大探索時間を秒単位で指定し、`enable_default_callback` は探索中に新たに見つかった最良解のエネルギーとTTSを出力する組み込みコールバック関数を有効にします。
`solver` の**`search()`**メンバ関数が `params` を引数として呼び出されます。
この関数は指定された制限時間内に見つかった最良解を返し、`sol` に格納されます。

プログラムは解のエネルギーと対応するバイナリ列を出力します。"+"は1を、"-"は0を表します。

このプログラムは以下の出力を生成します：
```
TTS = 0.002s Energy = 1218
TTS = 0.002s Energy = 1170
TTS = 0.002s Energy = 994
TTS = 0.015s Energy = 958
TTS = 0.018s Energy = 922
TTS = 0.034s Energy = 874
TTS = 4.364s Energy = 834
834: -+--+---++-++-+---++-++--+++--+-+-+++++----+++-+-+---++-+--+-----+--+----++----+-+--++++++---+------
```

## ABS3 Solverオブジェクト
ABS3 Solver（`qbpp::abs3::ABS3Solver`）オブジェクトは与えられた式に対して作成されます。
ソルバーオブジェクトが構築されると、式は内部データフォーマットに変換され、GPUメモリにロードされます。
オプションの第2引数 `gpu` でGPUの使用を制御します：
- **`qbpp::abs3::ABS3Solver(expression)`**: 利用可能なすべてのGPUを自動的に使用します。GPUが利用できない場合はCPUのみモードにフォールバックします。
- **`qbpp::abs3::ABS3Solver(expression, 0)`**: CPUのみモードを強制します（GPUは使用されません）。
- **`qbpp::abs3::ABS3Solver(expression, n)`**: `n` 個のGPUを使用します。

パラメータオブジェクト（`qbpp::abs3::Params`）は、ABS3 Solverが解探索中に使用するオプションを格納します。
各オプションは文字列のキー・バリューペアとして指定されます。
上記の例では：
- **`params("time_limit", "10.0")`**: 制限時間を10.0秒に設定します。
- **`params("enable_default_callback", "1")`**: 新たに得られた解のエネルギーを出力する組み込みコールバック関数を有効にします。

## ABS3パラメータ
パラメータ（`qbpp::abs3::Params()`）オブジェクトは、ABS3 Solverが解を探索する際に使用するパラメータオプションを格納します。
オプションは文字列のキーとバリューで構成されます。
上記のプログラムでは、`params("time_limit", "10.0")` で制限時間を10.0秒に設定し、`params("enable_default_callback", "1")` で新たに得られた解のエネルギーを出力する組み込みコールバック関数を有効にしています。

### 基本オプション

| キー | 値 | 説明 |
|----|----|----|
| **`time_limit`** | 制限時間（秒） | 制限時間に達すると探索を終了します |
| **`target_energy`** | ターゲットエネルギー値 | ターゲットエネルギーが達成されると探索を終了します |

### 詳細オプション

| キー | 値 | 説明 |
|----|----|----|
| **`enable_default_callback`** | "1" | エネルギーとTTSを出力する組み込みコールバックを有効にします |
| **`cpu_enable`** | "0" または "1" | GPUと並行して動作するCPUソルバーの有効/無効（デフォルト: "1"） |
| **`cpu_thread_count`** | CPUスレッド数 | CPUソルバーのスレッド数（デフォルト: 自動） |
| **`block_count`** | GPU当たりのCUDAブロック数 | ソルバーカーネルが起動するCUDAブロック数 |
| **`thread_count`** | CUDAブロック当たりのスレッド数 | CUDAブロック当たりのスレッド数 |
| **`topk_sols`** | 解の数 | 最良エネルギーのtop-K解を返します |
| **`best_energy_sols`** | 最大数（"0" = 無制限） | 見つかった最良エネルギーを持つすべての解を返します |

## 複数解の収集

ABS3 Solverは探索中に複数の解を収集できます。
2つのモードが利用可能です：

### Top-K解 (`topk_sols`)

`topk_sols` パラメータはエネルギーの昇順にソートされたtop-K解を収集します。

```cpp
params.add("topk_sols", "10");  // 最大10個の最良解を収集
```

### 最良エネルギー解 (`best_energy_sols`)

`best_energy_sols` パラメータは見つかった最良エネルギーを共有するすべての解を収集します。
より良いエネルギーが発見されると、プールがクリアされ、新しい最良エネルギーの解のみが保持されます。

```cpp
solver.enable_best_energy_sols();  // すべての最良エネルギー解を収集（無制限）
```

または、`best_energy_sols` を最大数付きのパラメータとして設定することもできます（"0"は無制限を意味します）：
```cpp
params.add("best_energy_sols", "0");  // 無制限
params.add("best_energy_sols", "100");  // 最大100個を収集
```

`topk_sols` と `best_energy_sols` は同じ内部プールを共有することに注意してください。
両方が指定された場合、最後に指定されたものが有効になります。

### 収集された解へのアクセス

`search()` メソッドは `ABS3Sols` オブジェクトを返し、収集された解へのアクセスを提供します：

```cpp
auto result = solver.search(params);

std::cout << "Best energy: " << result.energy() << std::endl;
std::cout << "Number of solutions: " << result.size() << std::endl;

for (const auto& sol : result.best_sols()) {
  std::cout << "Energy = " << sol.energy() << " TTS = " << sol.tts() << "s" << std::endl;
}
```

`ABS3Sols` オブジェクトは以下をサポートします：
- **`size()`** — 収集された解の数
- **`best_sols()`** / **`sols()`** — 解ベクトルへのアクセス
- **`operator[](i)`** — i番目の解へのアクセス
- 範囲ベースforループによるイテレーション

## カスタムコールバック

組み込みコールバック（`enable_default_callback` で有効化）は、新しい最良解が見つかるたびにエネルギーとTTSを出力するだけです。
より細かい制御が必要な場合は、`ABS3Solver` をサブクラス化して `callback()` 仮想メソッドをオーバーライドできます。

コールバックは以下のイベントのいずれかで呼び出されます：

| イベント | 説明 |
|-------|-------------|
| `CallbackEvent::Start` | `search()` の開始時に1回呼び出されます |
| `CallbackEvent::BestUpdated` | 新しい最良解が見つかるたびに呼び出されます |
| `CallbackEvent::Timer` | 設定可能な間隔で定期的に呼び出されます |

コールバック内では、以下のメソッドが利用可能です：
- **`best_sol()`** — 現在の最良解への `const qbpp::Sol&` を返します。`.energy()`、`.tts()`、`.get(var)` などが使用できます。
- **`event()`** — このコールバックをトリガーしたイベントを返します
- **`hint(sol)`** — 探索中にソルバーにヒント解を提供します（[Solution Hint](#solution-hint)を参照）

### タイマー制御

`Timer` イベントはデフォルトでは有効になっていません。
定期的なタイマーコールバックを有効にするには、`callback()` メソッド内で `timer(seconds)` を呼び出します：
- **`timer(1.0)`** — 1秒ごとに `Timer` コールバックを発火します
- **`timer(0)`** — タイマーを無効にします
- `timer()` が呼ばれない場合、タイマー間隔は変更されません。

通常、`timer()` は `Start` コールバック中に1回呼び出されて間隔を設定します。
`BestUpdated` や `Timer` コールバック中にも呼び出して、タイマーを動的に調整または無効にすることができます。

### 例: カスタムコールバック
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

class MySolver : public qbpp::abs3::ABS3Solver {
 public:
  using ABS3Solver::ABS3Solver;

  void callback() const override {
    if (event() == qbpp::abs3::CallbackEvent::Start) {
      timer(1.0);  // 1秒ごとのタイマーコールバックを有効化
    }
    if (event() == qbpp::abs3::CallbackEvent::BestUpdated) {
      std::cout << "New best: energy=" << best_sol().energy()
                << " TTS=" << best_sol().tts() << "s" << std::endl;
    }
  }
};

int main() {
  auto x = qbpp::var("x", 8);
  auto f = qbpp::sum(x) == 4;
  f.simplify_as_binary();

  auto solver = MySolver(f);
  auto params = qbpp::abs3::Params();
  params.add("time_limit", "5");
  params.add("target_energy", "0");
  auto sol = solver.search(params);
  std::cout << "energy=" << sol.energy() << std::endl;
}
```

## Solution Hint

ヒント解を使用すると、以前に見つかった解で探索をウォームスタートできます。

最も簡単な方法は、`search()` の前に**`params.hint(sol)`**を呼び出すことです：
```cpp
params.hint(sol);  // 探索にヒント解を提供
auto result = solver.search(params);
```
解は探索開始前にソルバーの内部データ構造に直接書き込まれます。

外部ソルバー（例：Gurobi）を並行して実行するような高度なユースケースでは、コールバック中に**`hint(sol)`**を呼び出して動的に解を供給することもできます。
このシナリオでは、コールバックが定期的に呼び出されて新しい外部解をチェックできるように、定期的なタイマー（例：`timer(1.0)`）を設定することを推奨します。

### 例: ヒント解の提供

以下の例は素因数分解問題を2回解きます。
1回目の実行では通常通り最適解を見つけます。
2回目の実行では `params.hint(sol)` を介して最初の解をヒントとして提供し、ソルバーの収束を大幅に高速化します。

```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 1000;
  auto q = 2 <= qbpp::var_int("q") <= 1000;
  auto f = p * q == 899 * 997;
  f.simplify_as_binary();

  auto solver = qbpp::abs3::ABS3Solver(f);

  // 実行1: 通常の探索
  auto params1 = qbpp::abs3::Params();
  params1.add("target_energy", "0");
  params1.add("time_limit", "10");
  params1.add("enable_default_callback", "1");
  const auto sol1 = solver.search(params1);
  std::cout << "Run 1: p=" << sol1(p) << " q=" << sol1(q)
            << " energy=" << sol1.energy() << std::endl;

  // 実行2: 前回の解をヒントとして提供
  auto params2 = qbpp::abs3::Params();
  params2.add("target_energy", "0");
  params2.add("time_limit", "10");
  params2.add("enable_default_callback", "1");
  params2.hint(sol1);
  const auto sol2 = solver.search(params2);
  std::cout << "Run 2: p=" << sol2(p) << " q=" << sol2(q)
            << " energy=" << sol2.energy()
            << " TTS=" << sol2.tts() << "s" << std::endl;
}
```

ヒント解は探索開始前にソルバーの内部データ構造に直接書き込まれます。
ソルバーはそのエネルギーを評価し、初期状態として使用した上で、より良い解の探索を続けます。
</div>
