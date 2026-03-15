---
layout: default
nav_exclude: true
title: "Solving Expressions"
nav_order: 3
---
<div class="lang-en" markdown="1">
# Solving Expressions using Easy Solver and Exhaustive Solver

PyQBPP provides the **Easy Solver** and the **Exhaustive Solver** for QUBO/HUBO expressions.
They run in parallel on multicore CPUs using **Intel Threading Building Blocks (oneTBB)**.

- **Easy Solver**
  - Runs a heuristic algorithm based on simulated annealing.
  - Does not guarantee optimality.

- **Exhaustive Solver**
  - Explores all possible solutions.
  - Guarantees optimality of the returned solution.
  - Is computationally feasible only when the number of binary variables is about 30-40 or fewer.
  - If a CUDA GPU is available, GPU acceleration is automatically enabled alongside CPU threads.

Both solvers are used in two steps:
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
from pyqbpp import var, sqr, EasySolver

a = var("a")
b = var("b")
c = var("c")
d = var("d")
f = sqr(a + 2 * b + 3 * c + 4 * d - 5)
print("f =", f.simplify_as_binary())

solver = EasySolver(f)
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
from pyqbpp import var, sqr, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
d = var("d")
f = sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
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
</div>

<div class="lang-ja" markdown="1">
# Easy SolverとExhaustive Solverによる式の求解

PyQBPPはQUBO/HUBO式のための **Easy Solver** と **Exhaustive Solver** を提供しています。
これらは **Intel Threading Building Blocks (oneTBB)** を使用してマルチコアCPU上で並列実行されます。

- **Easy Solver**
  - シミュレーテッドアニーリングに基づくヒューリスティックアルゴリズムを実行します。
  - 最適性は保証しません。

- **Exhaustive Solver**
  - すべての可能な解を探索します。
  - 返される解の最適性を保証します。
  - バイナリ変数の数が約30〜40個以下の場合にのみ計算が現実的です。
  - CUDA GPUが利用可能な場合、CPUスレッドと併せてGPU高速化が自動的に有効になります。

両方のソルバーは2つのステップで使用します：
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
from pyqbpp import var, sqr, EasySolver

a = var("a")
b = var("b")
c = var("c")
d = var("d")
f = sqr(a + 2 * b + 3 * c + 4 * d - 5)
print("f =", f.simplify_as_binary())

solver = EasySolver(f)
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
from pyqbpp import var, sqr, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
d = var("d")
f = sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
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
</div>
