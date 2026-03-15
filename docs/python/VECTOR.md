---
layout: default
nav_exclude: true
title: "Vectors"
nav_order: 4
---
<div class="lang-en" markdown="1">
# Vector of variables and vector functions

PyQBPP supports vectors of variables and vector operations.

## Defining vector of variables
A vector of binary variables can be created using the **`var()`** function.
- **`var("name", size)`** returns a `Vector` of `size` variables with the given `name`.

The following program defines a vector of 5 variables with the name **`x`**.
By printing `x`, we can confirm that it contains the 5 variables **`x[0]`**, **`x[1]`**, **`x[2]`**, **`x[3]`**, and **`x[4]`**.
Next, using the **`expr()`** function, we create an **`Expr`** object **`f`** whose initial value is `0`.
In the for-loop from `i = 0` to `4`, each variable `x[i]` is added to `f` using the compound operator **`+=`**.
Finally, `f` is simplified and printed.
```python
from pyqbpp import var, expr

x = var("x", 5)
print(x)
f = expr()
for i in range(5):
    f += x[i]
print("f =", f.simplify_as_binary())
```
The output of this program is as follows:
```
[x[0], x[1], x[2], x[3], x[4]]
f = x[0] +x[1] +x[2] +x[3] +x[4]
```

> **NOTE**
> **`var(name, size)`** returns a **`Vector`** object that contains `size` elements of type `Var`.
> The **`Vector`** class provides overloaded operators that support element-wise operations.

## Sum function
Using the utility function **`sum()`**, you can obtain the sum of a vector of binary variables.
The following program uses `sum()` to compute the sum of all variables in the vector `x`:
```python
from pyqbpp import var, sum

x = var("x", 5)
print(x)
f = sum(x)
print("f =", f.simplify_as_binary())
```
The output of this program is exactly the same as that of the previous program.

## QUBO for one-hot constraint
A vector of binary variables is **one-hot** if it has **exactly one entry equal to 1**, that is, the sum of its elements is equal to 1.
Let $X = (x_0, x_1, \ldots, x_{n-1})$ denote a vector of $n$ binary variables.
The following expression $f(X)$ takes the minimum value of 0 if and only if $X$ is one-hot:

$$
\begin{align}
f(X) &= \left(1 - \sum_{i=0}^{n-1}x_i\right)^2
\end{align}
$$

The following program creates the expression $f$ and finds all optimal solutions:
```python
from pyqbpp import var, sum, sqr, ExhaustiveSolver

x = var("x", 5)
f = sqr(sum(x) - 1)
print("f =", f.simplify_as_binary())

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for i, sol in enumerate(sols):
    print(f"({i}) {sol}")
```
The function **`sum()`** computes the sum of all variables in the vector.
The function **`sqr()`** computes the square of its argument.
The Exhaustive Solver finds all optimal solutions with energy value 0 as follows:
```
f = 1 -x[0] -x[1] -x[2] -x[3] -x[4] +2*x[0]*x[1] +2*x[0]*x[2] +2*x[0]*x[3] +2*x[0]*x[4] +2*x[1]*x[2] +2*x[1]*x[3] +2*x[1]*x[4] +2*x[2]*x[3] +2*x[2]*x[4] +2*x[3]*x[4]
(0) Sol(energy=0, x[0]=0, x[1]=0, x[2]=0, x[3]=0, x[4]=1)
(1) Sol(energy=0, x[0]=0, x[1]=0, x[2]=0, x[3]=1, x[4]=0)
(2) Sol(energy=0, x[0]=0, x[1]=0, x[2]=1, x[3]=0, x[4]=0)
(3) Sol(energy=0, x[0]=0, x[1]=1, x[2]=0, x[3]=0, x[4]=0)
(4) Sol(energy=0, x[0]=1, x[1]=0, x[2]=0, x[3]=0, x[4]=0)
```
All 5 optimal solutions are displayed.
</div>

<div class="lang-ja" markdown="1">
# 変数のベクトルとベクトル関数

PyQBPPは変数のベクトルとベクトル演算をサポートしています。

## 変数ベクトルの定義
バイナリ変数のベクトルは **`var()`** 関数を使って作成できます。
- **`var("name", size)`** は、指定された `name` を持つ `size` 個の変数からなる `Vector` を返します。

以下のプログラムは、**`x`** という名前の5個の変数からなるベクトルを定義します。
`x` を表示すると、5つの変数 **`x[0]`**、**`x[1]`**、**`x[2]`**、**`x[3]`**、**`x[4]`** が含まれていることが確認できます。
次に、**`expr()`** 関数を使って、初期値が `0` の **`Expr`** オブジェクト **`f`** を作成します。
`i = 0` から `4` までの for ループで、各変数 `x[i]` を複合演算子 **`+=`** を使って `f` に加えます。
最後に、`f` を簡約化して表示します。
```python
from pyqbpp import var, expr

x = var("x", 5)
print(x)
f = expr()
for i in range(5):
    f += x[i]
print("f =", f.simplify_as_binary())
```
このプログラムの出力は以下の通りです。
```
[x[0], x[1], x[2], x[3], x[4]]
f = x[0] +x[1] +x[2] +x[3] +x[4]
```

> **NOTE**
> **`var(name, size)`** は `size` 個の `Var` 型要素を含む **`Vector`** オブジェクトを返します。
> **`Vector`** クラスは、要素ごとの演算をサポートするオーバーロードされた演算子を提供します。

## Sum 関数
ユーティリティ関数 **`sum()`** を使って、バイナリ変数ベクトルの合計を取得できます。
以下のプログラムは `sum()` を使ってベクトル `x` のすべての変数の合計を計算します。
```python
from pyqbpp import var, sum

x = var("x", 5)
print(x)
f = sum(x)
print("f =", f.simplify_as_binary())
```
このプログラムの出力は、前のプログラムとまったく同じです。

## One-hot 制約の QUBO
バイナリ変数のベクトルが **one-hot** であるとは、**ちょうど1つの要素が1に等しい**こと、すなわち要素の合計が1に等しいことを意味します。
$X = (x_0, x_1, \ldots, x_{n-1})$ を $n$ 個のバイナリ変数のベクトルとします。
以下の式 $f(X)$ は、$X$ が one-hot であるとき、かつそのときに限り最小値 0 をとります。

$$
\begin{align}
f(X) &= \left(1 - \sum_{i=0}^{n-1}x_i\right)^2
\end{align}
$$

以下のプログラムは式 $f$ を作成し、すべての最適解を求めます。
```python
from pyqbpp import var, sum, sqr, ExhaustiveSolver

x = var("x", 5)
f = sqr(sum(x) - 1)
print("f =", f.simplify_as_binary())

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for i, sol in enumerate(sols):
    print(f"({i}) {sol}")
```
関数 **`sum()`** はベクトル内のすべての変数の合計を計算します。
関数 **`sqr()`** は引数の二乗を計算します。
Exhaustive Solver は、エネルギー値 0 のすべての最適解を以下のように求めます。
```
f = 1 -x[0] -x[1] -x[2] -x[3] -x[4] +2*x[0]*x[1] +2*x[0]*x[2] +2*x[0]*x[3] +2*x[0]*x[4] +2*x[1]*x[2] +2*x[1]*x[3] +2*x[1]*x[4] +2*x[2]*x[3] +2*x[2]*x[4] +2*x[3]*x[4]
(0) Sol(energy=0, x[0]=0, x[1]=0, x[2]=0, x[3]=0, x[4]=1)
(1) Sol(energy=0, x[0]=0, x[1]=0, x[2]=0, x[3]=1, x[4]=0)
(2) Sol(energy=0, x[0]=0, x[1]=0, x[2]=1, x[3]=0, x[4]=0)
(3) Sol(energy=0, x[0]=0, x[1]=1, x[2]=0, x[3]=0, x[4]=0)
(4) Sol(energy=0, x[0]=1, x[1]=0, x[2]=0, x[3]=0, x[4]=0)
```
5つの最適解がすべて表示されます。
</div>
