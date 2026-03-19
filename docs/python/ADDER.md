---
layout: default
nav_exclude: true
title: "Adder Simulation (PyQBPP)"
nav_order: 90
---
<div class="lang-en" markdown="1">
# Adder Simulation

## Full adder and ripple carry adder
A full adder has three input bits: $a$, $b$, and $i$ (carry-in) and
$o$ (carry-out) and $s$ (sum).
The sum of the three input bits is represented using these two output bits.

A ripple-carry adder computes the sum of two multi-bit integers by cascading multiple full adders, as illustrated below:
<p align="center">
 <img src="../images/adder.svg" alt="4-bit ripple carry adder" width="50%">
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
The following PyQBPP program verifies this formulation using the exhaustive solver:
```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
i = qbpp.var("i")
o = qbpp.var("o")
s = qbpp.var("s")
fa = (a + b + i) - (2 * o + s) == 0
fa.simplify_as_binary()
solver = qbpp.ExhaustiveSolver(fa)
sols = solver.search_optimal_solutions()
for idx, sol in enumerate(sols):
    vals = {v: sol(v) for v in [a, b, i, o, s]}
    print(f"({idx}) {sol.energy()}: a={vals[a]}, b={vals[b]}, i={vals[i]}, o={vals[o]}, s={vals[s]}")
```
In this program, the constraint $fa(a,b,i,c,s)$ is implemented using the equality operator `==`, which intuitively represents the constraint $a+b+i=2o+s$.
The program produces the following output, confirming that the expression correctly models a full adder:
```
(0) 0: a=0, b=0, i=0, o=0, s=0
(1) 0: a=0, b=0, i=1, o=0, s=1
(2) 0: a=0, b=1, i=0, o=0, s=1
(3) 0: a=0, b=1, i=1, o=1, s=0
(4) 0: a=1, b=0, i=0, o=0, s=1
(5) 0: a=1, b=0, i=1, o=1, s=0
(6) 0: a=1, b=1, i=0, o=1, s=0
(7) 0: a=1, b=1, i=1, o=1, s=1
```

If some bits are fixed, the valid values of the remaining bits can be derived.
For example, the three input bits can be fixed using the `replace()` function:
```python
ml = [(a, 1), (b, 1), (i, 0)]
fa2 = qbpp.replace(fa, ml)
fa2.simplify_as_binary()
solver2 = qbpp.ExhaustiveSolver(fa2)
sols2 = solver2.search_optimal_solutions()
for idx, sol in enumerate(sols2):
    print(f"({idx}) {sol.energy()}: o={sol(o)}, s={sol(s)}")
```

The program then produces the following output:
```
(0) 0: o=1, s=0
```

## Simulating a ripple carry adder using multiple full adders
Using the QUBO expression for a full adder, we can construct a QUBO expression that simulates a ripple-carry adder.
The following PyQBPP program creates a QUBO expression for simulating a 4-bit adder by combining four full adders:
```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
i = qbpp.var("i")
o = qbpp.var("o")
s = qbpp.var("s")
fa = (a + b + i) - (2 * o + s) == 0

x = qbpp.var("x", 4)
y = qbpp.var("y", 4)
c = qbpp.var("c", 5)
z = qbpp.var("z", 4)

fa0 = qbpp.replace(fa, [(a, x[0]), (b, y[0]), (i, c[0]), (o, c[1]), (s, z[0])])
fa1 = qbpp.replace(fa, [(a, x[1]), (b, y[1]), (i, c[1]), (o, c[2]), (s, z[1])])
fa2 = qbpp.replace(fa, [(a, x[2]), (b, y[2]), (i, c[2]), (o, c[3]), (s, z[2])])
fa3 = qbpp.replace(fa, [(a, x[3]), (b, y[3]), (i, c[3]), (o, c[4]), (s, z[3])])

adder = fa0 + fa1 + fa2 + fa3
adder.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(adder)
sols = solver.search_optimal_solutions()
print(f"Number of valid solutions: {len(sols)}")
for idx in [0, 1, len(sols)-2, len(sols)-1]:
    sol = sols[idx]
    xv = "".join(str(sol(x[j])) for j in range(4))
    yv = "".join(str(sol(y[j])) for j in range(4))
    cv = "".join(str(sol(c[j])) for j in range(5))
    zv = "".join(str(sol(z[j])) for j in range(4))
    print(f"({idx}) x={xv}, y={yv}, c={cv}, z={zv}")
```
In this program, four full-adder expressions are created using the `replace()` function with lists of pairs, and combined into a single expression, `adder`.
The Exhaustive Solver is then used to enumerate all optimal solutions.

This program produces 512 valid solutions, corresponding to all possible input combinations of a 4-bit adder:
```
Number of valid solutions: 512
(0) x=0000, y=0000, c=00000, z=0000
(1) x=0000, y=0000, c=10000, z=1000
(510) x=1111, y=1111, c=01111, z=0111
(511) x=1111, y=1111, c=11111, z=1111
```

Alternatively, we can define a Python function `fa` to construct full-adder constraints in a more concise manner:
```python
import pyqbpp as qbpp

def fa(a, b, i, o, s):
    return (a + b + i) - (2 * o + s) == 0

x = qbpp.var("x", 4)
y = qbpp.var("y", 4)
c = qbpp.var("c", 5)
z = qbpp.var("z", 4)

adder = toExpr(0)
for j in range(4):
    adder += fa(x[j], y[j], c[j], c[j + 1], z[j])

adder = qbpp.replace(adder, [(c[0], 0), (c[4], 0)])
adder.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(adder)
sols = solver.search_optimal_solutions()
print(f"Number of valid solutions: {len(sols)}")
```
This program produces 136 valid solutions (carry-in and carry-out are both fixed to 0, so only pairs with $x + y \leq 15$ are valid).
</div>

<div class="lang-ja" markdown="1">
# 加算器シミュレーション

## 全加算器とリプルキャリー加算器
全加算器は3つの入力ビット $a$、$b$、$i$（キャリーイン）と、
$o$（キャリーアウト）および $s$（和）を持ちます。
3つの入力ビットの和は、これら2つの出力ビットで表現されます。

リプルキャリー加算器は、以下に示すように複数の全加算器をカスケード接続して、2つの多ビット整数の和を計算します:
<p align="center">
 <img src="../images/adder.svg" alt="4-bit ripple carry adder" width="50%">
</p>

このリプルキャリー加算器は、2つの4ビット整数 $x_3x_2x_1x_0$ と $y_3y_2y_1y_0$ の和を計算し、4つの全加算器を使って4ビットの和 $z_3z_2z_1z_0$ を出力します。
対応する5ビットのキャリー信号 $c_4c_3c_2c_1c_0$ も示されています。

## 全加算器の QUBO 定式化
全加算器は以下の式を用いて定式化できます:

$$
\begin{aligned}
fa(a,b,i,c,s) &=((a+b+i)-(2o+s))^2
\end{aligned}
$$

この式は、5つの変数が有効な全加算器の動作と矛盾しない値を取る場合に限り、最小値 0 を達成します。
以下の PyQBPP プログラムは、全探索ソルバーを用いてこの定式化を検証します:
```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
i = qbpp.var("i")
o = qbpp.var("o")
s = qbpp.var("s")
fa = (a + b + i) - (2 * o + s) == 0
fa.simplify_as_binary()
solver = qbpp.ExhaustiveSolver(fa)
sols = solver.search_optimal_solutions()
for idx, sol in enumerate(sols):
    vals = {v: sol(v) for v in [a, b, i, o, s]}
    print(f"({idx}) {sol.energy()}: a={vals[a]}, b={vals[b]}, i={vals[i]}, o={vals[o]}, s={vals[s]}")
```
このプログラムでは、制約 $fa(a,b,i,c,s)$ は等価演算子 `==` を使って実装されており、直感的に制約 $a+b+i=2o+s$ を表現しています。
プログラムは以下の出力を生成し、この式が全加算器を正しくモデル化していることを確認できます:
```
(0) 0: a=0, b=0, i=0, o=0, s=0
(1) 0: a=0, b=0, i=1, o=0, s=1
(2) 0: a=0, b=1, i=0, o=0, s=1
(3) 0: a=0, b=1, i=1, o=1, s=0
(4) 0: a=1, b=0, i=0, o=0, s=1
(5) 0: a=1, b=0, i=1, o=1, s=0
(6) 0: a=1, b=1, i=0, o=1, s=0
(7) 0: a=1, b=1, i=1, o=1, s=1
```

一部のビットを固定すると、残りのビットの有効な値を導出できます。
例えば、3つの入力ビットは `replace()` 関数を使って固定できます:
```python
ml = [(a, 1), (b, 1), (i, 0)]
fa2 = qbpp.replace(fa, ml)
fa2.simplify_as_binary()
solver2 = qbpp.ExhaustiveSolver(fa2)
sols2 = solver2.search_optimal_solutions()
for idx, sol in enumerate(sols2):
    print(f"({idx}) {sol.energy()}: o={sol(o)}, s={sol(s)}")
```

プログラムは以下の出力を生成します:
```
(0) 0: o=1, s=0
```

## 複数の全加算器を用いたリプルキャリー加算器のシミュレーション
全加算器の QUBO 式を用いて、リプルキャリー加算器をシミュレートする QUBO 式を構築できます。
以下の PyQBPP プログラムは、4つの全加算器を組み合わせて4ビット加算器をシミュレートする QUBO 式を作成します:
```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
i = qbpp.var("i")
o = qbpp.var("o")
s = qbpp.var("s")
fa = (a + b + i) - (2 * o + s) == 0

x = qbpp.var("x", 4)
y = qbpp.var("y", 4)
c = qbpp.var("c", 5)
z = qbpp.var("z", 4)

fa0 = qbpp.replace(fa, [(a, x[0]), (b, y[0]), (i, c[0]), (o, c[1]), (s, z[0])])
fa1 = qbpp.replace(fa, [(a, x[1]), (b, y[1]), (i, c[1]), (o, c[2]), (s, z[1])])
fa2 = qbpp.replace(fa, [(a, x[2]), (b, y[2]), (i, c[2]), (o, c[3]), (s, z[2])])
fa3 = qbpp.replace(fa, [(a, x[3]), (b, y[3]), (i, c[3]), (o, c[4]), (s, z[3])])

adder = fa0 + fa1 + fa2 + fa3
adder.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(adder)
sols = solver.search_optimal_solutions()
print(f"Number of valid solutions: {len(sols)}")
for idx in [0, 1, len(sols)-2, len(sols)-1]:
    sol = sols[idx]
    xv = "".join(str(sol(x[j])) for j in range(4))
    yv = "".join(str(sol(y[j])) for j in range(4))
    cv = "".join(str(sol(c[j])) for j in range(5))
    zv = "".join(str(sol(z[j])) for j in range(4))
    print(f"({idx}) x={xv}, y={yv}, c={cv}, z={zv}")
```
このプログラムでは、`replace()` 関数にペアのリストを渡して4つの全加算器の式を作成し、1つの式 `adder` にまとめています。
次に全探索ソルバーを使ってすべての最適解を列挙します。

このプログラムは512個の有効な解を生成し、4ビット加算器のすべての入力の組み合わせに対応しています:
```
Number of valid solutions: 512
(0) x=0000, y=0000, c=00000, z=0000
(1) x=0000, y=0000, c=10000, z=1000
(510) x=1111, y=1111, c=01111, z=0111
(511) x=1111, y=1111, c=11111, z=1111
```

あるいは、Python 関数 `fa` を定義して、より簡潔に全加算器の制約を構築することもできます:
```python
import pyqbpp as qbpp

def fa(a, b, i, o, s):
    return (a + b + i) - (2 * o + s) == 0

x = qbpp.var("x", 4)
y = qbpp.var("y", 4)
c = qbpp.var("c", 5)
z = qbpp.var("z", 4)

adder = toExpr(0)
for j in range(4):
    adder += fa(x[j], y[j], c[j], c[j + 1], z[j])

adder = qbpp.replace(adder, [(c[0], 0), (c[4], 0)])
adder.simplify_as_binary()

solver = qbpp.ExhaustiveSolver(adder)
sols = solver.search_optimal_solutions()
print(f"Number of valid solutions: {len(sols)}")
```
このプログラムは136個の有効な解を生成します（キャリーインとキャリーアウトの両方が0に固定されているため、$x + y \leq 15$ を満たす組のみが有効です）。
</div>
