---
layout: default
nav_exclude: true
title: "Solving Expressions (PyQBPP)"
nav_order: 3
---
<div class="lang-en" markdown="1">
# Solving Expressions

PyQBPP provides three solvers for QUBO/HUBO expressions:

- **Easy Solver**
  - Runs a heuristic algorithm based on simulated annealing.
  - Runs in parallel on multicore CPUs using **Intel Threading Building Blocks (oneTBB)**.
  - Does not guarantee optimality.

- **Exhaustive Solver**
  - Explores all possible solutions.
  - Guarantees optimality of the returned solution.
  - Is computationally feasible only when the number of binary variables is about 30-40 or fewer.
  - If a CUDA GPU is available, GPU acceleration is automatically enabled alongside CPU threads.

- **ABS3 Solver**
  - A high-performance solver that uses CUDA GPUs and multicore CPUs.
  - Does not guarantee optimality, but is much more powerful than the Easy Solver.
  - If no GPU is available, falls back to CPU-only mode.

The Easy Solver and Exhaustive Solver are used in two steps:
1. Create a solver object, **`EasySolver`** or **`ExhaustiveSolver`**.
2. Call the **`search()`** method on the solver object. It returns a **`Sol`** object that stores the obtained solution.

## Easy Solver
We use the following expression $f(a,b,c,d)$ as an example:

$$
\begin{aligned}
f(a,b,c,d) &= (a+2b+3c+4d-5)^2
\end{aligned}
$$

Clearly, this expression attains its minimum value $f=0$
when $a+2b+3c+4d=5$.
Therefore, it has two optimal solutions, $(a,b,c,d)=(0,1,1,0)$ and $(1,0,0,1)$.

In the following program, expression `f` is created using symbolic computation.
The function **`sqr()`** returns the square of the argument.
We then construct an `EasySolver` instance by passing `f` to its constructor.
Before doing so, `f` must be simplified for binary variables by calling **`simplify_as_binary()`**.
Since we know that the optimal value is $f=0$, we set the target energy to $0$ by calling the **`target_energy()`** method.
Calling the **`search()`** method on `solver` returns a solution instance **`sol`** of class **`Sol`**.

```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
d = qbpp.var("d")
f = qbpp.sqr(a + 2 * b + 3 * c + 4 * d - 5)
print("f =", f.simplify_as_binary())

solver = qbpp.EasySolver(f)
solver.target_energy(0)
sol = solver.search()
print(sol)
```

The output of this program is as follows:
```
f = 25 -9*a -16*b -21*c -24*d +4*a*b +6*a*c +8*a*d +12*b*c +16*b*d +24*c*d
Sol(energy=0, a=1, b=0, c=0, d=1)
```
One of the optimal solutions is correctly output.

## Exhaustive Solver
We construct an **`ExhaustiveSolver`** instance by passing `f` to its constructor.
Calling the **`search()`** method on `solver` returns a solution instance **`sol`** of
class **`Sol`**.
Since the Exhaustive Solver explores all possible assignments, it is guaranteed that `sol`
stores an optimal solution.

```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
d = qbpp.var("d")
f = qbpp.sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()
print(sol)
```
The output of this program is as follows:
```
Sol(energy=0, a=0, b=1, c=1, d=0)
```
All optimal solutions can be obtained by the **`search_optimal_solutions()`** method as follows:
```python
sols = solver.search_optimal_solutions()
for i, sol in enumerate(sols):
    print(f"({i}) {sol}")
```
The output is as follows:
```
(0) Sol(energy=0, a=0, b=1, c=1, d=0)
(1) Sol(energy=0, a=1, b=0, c=0, d=1)
```

The Exhaustive Solver is very useful for analyzing small expressions and for debugging.

## ABS3 Solver
The **ABS3 Solver** is a high-performance solver that uses CUDA GPUs and multicore CPUs.
If no GPU is available, it automatically falls back to CPU-only mode.

Usage involves three steps:
1. Create an **`ABS3Solver`** object for the expression.
2. Set search options using methods of the solver object.
3. Call the **`search()`** method, which returns the obtained solution.

```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
d = qbpp.var("d")
f = qbpp.sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = qbpp.ABS3Solver(f)
solver.time_limit(5.0)
solver.target_energy(0)
solver.callback(lambda energy, tts, event: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
print(sol)
```
The output of this program is as follows:
```
TTS = 0.000s Energy = 0
Sol(energy=0, a=0, b=1, c=1, d=0)
```

For details on parameters, callbacks, and multiple solution collection, see **[ABS3 Solver](ABS3)**.
</div>

<div class="lang-ja" markdown="1">
# 式の求解

PyQBPPはQUBO/HUBO式を解くための3つのソルバーを提供しています：

- **Easy Solver**
  - シミュレーテッドアニーリングに基づくヒューリスティックアルゴリズムを実行します。
  - **Intel Threading Building Blocks (oneTBB)** を使用してマルチコアCPU上で並列実行されます。
  - 最適性は保証しません。

- **Exhaustive Solver**
  - すべての可能な解を探索します。
  - 返される解の最適性を保証します。
  - バイナリ変数の数が約30〜40個以下の場合にのみ計算が現実的です。
  - CUDA GPUが利用可能な場合、CPUスレッドと併せてGPU高速化が自動的に有効になります。

- **ABS3 Solver**
  - CUDA GPUとマルチコアCPUを活用する高性能ソルバーです。
  - 最適性は保証されませんが、Easy Solverよりはるかに強力です。
  - GPUが利用できない場合はCPUのみモードにフォールバックします。

Easy SolverとExhaustive Solverは2つのステップで使用します：
1. **`EasySolver`** または **`ExhaustiveSolver`** のソルバーオブジェクトを作成します。
2. ソルバーオブジェクトの **`search()`** メソッドを呼び出します。得られた解を格納する **`Sol`** オブジェクトが返されます。

## Easy Solver
以下の式 $f(a,b,c,d)$ を例として使用します：

$$
\begin{aligned}
f(a,b,c,d) &= (a+2b+3c+4d-5)^2
\end{aligned}
$$

この式は $a+2b+3c+4d=5$ のとき明らかに最小値 $f=0$ を取ります。
したがって、2つの最適解 $(a,b,c,d)=(0,1,1,0)$ と $(1,0,0,1)$ があります。

以下のプログラムでは、シンボリック計算を使って式 `f` を作成します。
関数 **`sqr()`** は引数の二乗を返します。
次に、`f` をコンストラクタに渡して `EasySolver` のインスタンスを構築します。
その前に、**`simplify_as_binary()`** を呼び出してバイナリ変数用に `f` を簡約化する必要があります。
最適値が $f=0$ であることがわかっているため、**`target_energy()`** メソッドを呼び出してターゲットエネルギーを $0$ に設定します。
`solver` の **`search()`** メソッドを呼び出すと、**`Sol`** クラスの解インスタンス **`sol`** が返されます。

```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
d = qbpp.var("d")
f = qbpp.sqr(a + 2 * b + 3 * c + 4 * d - 5)
print("f =", f.simplify_as_binary())

solver = qbpp.EasySolver(f)
solver.target_energy(0)
sol = solver.search()
print(sol)
```

このプログラムの出力は以下の通りです：
```
f = 25 -9*a -16*b -21*c -24*d +4*a*b +6*a*c +8*a*d +12*b*c +16*b*d +24*c*d
Sol(energy=0, a=1, b=0, c=0, d=1)
```
最適解の1つが正しく出力されています。

## Exhaustive Solver
`f` をコンストラクタに渡して **`ExhaustiveSolver`** のインスタンスを構築します。
`solver` の **`search()`** メソッドを呼び出すと、**`Sol`** クラスの解インスタンス **`sol`** が返されます。
Exhaustive Solverはすべての可能な割り当てを探索するため、`sol` が最適解を格納していることが保証されます。

```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
d = qbpp.var("d")
f = qbpp.sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(f)
sol = solver.search()
print(sol)
```
このプログラムの出力は以下の通りです：
```
Sol(energy=0, a=0, b=1, c=1, d=0)
```
**`search_optimal_solutions()`** メソッドを使うと、すべての最適解を取得できます：
```python
sols = solver.search_optimal_solutions()
for i, sol in enumerate(sols):
    print(f"({i}) {sol}")
```
出力は以下の通りです：
```
(0) Sol(energy=0, a=0, b=1, c=1, d=0)
(1) Sol(energy=0, a=1, b=0, c=0, d=1)
```

Exhaustive Solverは、小さな式の解析やデバッグに非常に有用です。

## ABS3 Solver
**ABS3 Solver**は、CUDA GPUとマルチコアCPUを活用する高性能ソルバーです。
GPUが利用できない場合は、自動的にCPUのみモードにフォールバックします。

使用方法は以下の3ステップです：
1. 式に対して**`ABS3Solver`**オブジェクトを作成します。
2. ソルバーオブジェクトのメソッドを使って探索オプションを設定します。
3. **`search()`**メソッドを呼び出します。得られた解が返されます。

```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
d = qbpp.var("d")
f = qbpp.sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = qbpp.ABS3Solver(f)
solver.time_limit(5.0)
solver.target_energy(0)
solver.callback(lambda energy, tts, event: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
print(sol)
```
このプログラムの出力は以下の通りです：
```
TTS = 0.000s Energy = 0
Sol(energy=0, a=0, b=1, c=1, d=0)
```

パラメータ、コールバック、複数解の収集の詳細については**[ABS3 Solver](ABS3)**をご覧ください。
</div>
