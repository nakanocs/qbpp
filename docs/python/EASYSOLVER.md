---
layout: default
nav_exclude: true
title: "Easy Solver (PyQBPP)"
nav_order: 19
---
<div class="lang-en" markdown="1">
# Easy Solver Usage
The **Easy Solver** is a heuristic solver for QUBO/HUBO expressions.

Solving a problem with the Easy Solver consists of the following three steps:
1. Create an **`EasySolver`** object.
2. Set search options by calling methods of the solver object.
3. Search for solutions by calling the **`search()`** method, which returns a **`Sol`** object.

## Creating Easy Solver object
To use the Easy Solver, an `EasySolver` object is constructed with an expression as follows:
- **`EasySolver(f)`**

Here, `f` is the expression to be solved.
It must be simplified as a binary expression in advance by calling `simplify_as_binary()`.

## Setting Easy Solver Options
- **`time_limit(time)`**: Specifies the time limit in seconds. The default value is 10.0 seconds.
If the time limit is set to 0, the solver never terminates due to the time limit.
- **`target_energy(energy)`**: Specifies the target energy. The solver terminates when a solution with energy less than or equal to the target is found.
- **`callback(func)`**: Sets a callback function that is called when a new best solution is found. The callback receives two arguments: `energy` (int) and `tts` (float, time to solution in seconds).
- **`thread_count(n)`**: Sets the number of threads.

## Searching Solutions
The Easy Solver searches for solutions by calling the **`search()`** method.

## Program Example
The following program searches for a solution to the
**Low Autocorrelation Binary Sequences (LABS)** problem using the Easy Solver:
```python
import pyqbpp as qbpp

size = 100
x = qbpp.var("x", size)
f = qbpp.expr()
for d in range(1, size):
    temp = qbpp.expr()
    for i in range(size - d):
        temp += (2 * x[i] - 1) * (2 * x[i + d] - 1)
    f += qbpp.sqr(temp)
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.time_limit(5.0)
solver.target_energy(900)
solver.callback(lambda energy, tts: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
bits = "".join("-" if sol(i) == 0 else "+" for i in range(size))
print(f"{sol.energy()}: {bits}")
```
In this example, the following options are set:
- a 5.0-second time limit,
- a target energy of 900, and
- a callback that prints the energy and TTS whenever a new best solution is found.

Therefore, the solver terminates either when the elapsed time reaches 5.0 seconds
or when a solution with energy 900 or less is found.

For example, this program produces the following output:
{% raw %}
```
TTS = 0.000s Energy = 300162
TTS = 0.000s Energy = 273350
...
TTS = 2.691s Energy = 898
898: ++-++-----+--+--++++++---++-+-+--++-------++-++-+-+-+-+-++-++++-++-+++++-+-+--++++++---+++--+++---++
```
{% endraw %}
</div>

<div class="lang-ja" markdown="1">
# Easy Solverの使い方
**Easy Solver**は、QUBO/HUBO式に対するヒューリスティックソルバーです。

Easy Solverで問題を解くには、以下の3つのステップで行います:
1. **`EasySolver`** オブジェクトを作成する。
2. ソルバーオブジェクトのメソッドを呼び出して探索オプションを設定する。
3. **`search()`** メソッドを呼び出して解を探索する。このメソッドは **`Sol`** オブジェクトを返す。

## Easy Solverオブジェクトの作成
Easy Solverを使用するには、以下のように式を引数として `EasySolver` オブジェクトを作成します:
- **`EasySolver(f)`**

ここで、`f` は解くべき式です。
事前に `simplify_as_binary()` を呼び出してバイナリ式として簡約化しておく必要があります。

## Easy Solverオプションの設定
- **`time_limit(time)`**: 制限時間を秒単位で指定します。デフォルト値は10.0秒です。
制限時間を0に設定すると、時間制限による終了は行われません。
- **`target_energy(energy)`**: 目標エネルギーを指定します。目標以下のエネルギーを持つ解が見つかると、ソルバーは終了します。
- **`callback(func)`**: 新しい最良解が見つかったときに呼び出されるコールバック関数を設定します。コールバックは2つの引数を受け取ります: `energy`（int）と `tts`（float、解発見までの時間（秒））。
- **`thread_count(n)`**: スレッド数を設定します。

## 解の探索
Easy Solverは **`search()`** メソッドを呼び出すことで解を探索します。

## プログラム例
以下のプログラムは、Easy Solverを使用して
**Low Autocorrelation Binary Sequences (LABS)** 問題の解を探索します:
```python
import pyqbpp as qbpp

size = 100
x = qbpp.var("x", size)
f = qbpp.expr()
for d in range(1, size):
    temp = qbpp.expr()
    for i in range(size - d):
        temp += (2 * x[i] - 1) * (2 * x[i + d] - 1)
    f += qbpp.sqr(temp)
f.simplify_as_binary()

solver = qbpp.EasySolver(f)
solver.time_limit(5.0)
solver.target_energy(900)
solver.callback(lambda energy, tts: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
bits = "".join("-" if sol(i) == 0 else "+" for i in range(size))
print(f"{sol.energy()}: {bits}")
```
この例では、以下のオプションが設定されています:
- 制限時間5.0秒、
- 目標エネルギー900、
- 新しい最良解が見つかるたびにエネルギーとTTSを表示するコールバック。

したがって、ソルバーは経過時間が5.0秒に達するか、
エネルギーが900以下の解が見つかると終了します。

例えば、このプログラムは以下のような出力を生成します:
{% raw %}
```
TTS = 0.000s Energy = 300162
TTS = 0.000s Energy = 273350
...
TTS = 2.691s Energy = 898
898: ++-++-----+--+--++++++---++-+-+--++-------++-++-+-+-+-+-++-++++-++-+++++-+-+--++++++---+++--+++---++
```
{% endraw %}
</div>
