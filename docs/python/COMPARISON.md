---
layout: default
nav_exclude: true
title: "Comparison Operators (PyQBPP)"
nav_order: 14
---
<div class="lang-en" markdown="1">
# Comparison Operators
PyQBPP supports two types of operators for creating constraints:

- **Equality operator**: `f == n`, where `f` is an expression and `n` is an integer.
- **Range operator**: `between(f, l, u)`, where `f` is an expression and `l` and `u` ($l\leq u$) are integers.

These operators return an `ExprExpr` object that attains **the minimum value of 0 if and only if the corresponding constraints are satisfied**.

## Equality operator
The equality operator `f == n` creates the following expression:

$$
(f-n)^2
$$

This expression attains the minimum value of 0 if and only if the equality $f=n$ is satisfied.

The following program searches for all solutions satisfying
$a+2b+3c=3$ using the Exhaustive Solver:
```python
from pyqbpp import var, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
f = a + 2 * b + 3 * c == 3
f.simplify_as_binary()
print("f =", f)
print("body =", f.body)

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for sol in sols:
    print(f"a={sol.get(a)}, b={sol.get(b)}, c={sol.get(c)}, "
          f"f={sol.eval(f)}, body={sol.eval(f.body)}")
```
In this program, `f` internally holds two expressions:
- **`f`**: $(a+2b+3c-3)^2$, which attains the minimum value of 0 if the equality is satisfied.
- **`f.body`**: the left-hand side of the equality, $a+2b+3c$.

This program produces the following output:
```
f = 9 -5*a -8*b -9*c +4*a*b +6*a*c +12*b*c
body = a +2*b +3*c
a=0, b=0, c=1, f=0, body=3
a=1, b=1, c=0, f=0, body=3
```

## Notes on Supported Equality Forms
PyQBPP supports the equality operator only in the following form:
- **`expression == integer`**

The following forms are not supported:
- **`integer == expression`**
- **`expression1 == expression2`**

Instead of `expression1 == expression2`, you can rewrite the constraint as:
- **`expression1 - expression2 == 0`**

## Range operator
The range operator `between(f, l, u)` creates an expression that attains the minimum value of 0 if and only if the constraint $l\leq f\leq u$ is satisfied.

> **NOTE**
> Unlike the C++ version which uses the syntax `l <= f <= u`, PyQBPP uses the function `between(f, l, u)` for range constraints on expressions.

The following program demonstrates the use of the range operator:
```python
from pyqbpp import var, between, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
f = between(4 * a + 9 * b + 15 * c, 5, 14)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for sol in sols:
    print(f"a={sol.get(a)}, b={sol.get(b)}, c={sol.get(c)}, "
          f"f={sol.eval(f)}, body={sol.eval(f.body)}")
```
This program searches for solutions satisfying the constraint $5\leq 4a+9b+15c \leq 14$ and produces the following output:
```
a=0, b=1, c=0, f=0, body=9
a=1, b=1, c=0, f=0, body=13
```
</div>

<div class="lang-ja" markdown="1">
# 比較演算子
PyQBPPは制約を作成するための2種類の演算子をサポートしています：

- **等価演算子**: `f == n`。`f` は式、`n` は整数です。
- **範囲演算子**: `between(f, l, u)`。`f` は式、`l` と `u`（$l\leq u$）は整数です。

これらの演算子は、**対応する制約が満たされるときかつそのときに限り最小値0を取る** `ExprExpr` オブジェクトを返します。

## 等価演算子
等価演算子 `f == n` は以下の式を作成します：

$$
(f-n)^2
$$

この式は、等式 $f=n$ が満たされるときかつそのときに限り最小値0を取ります。

以下のプログラムは、Exhaustive Solverを使って $a+2b+3c=3$ を満たすすべての解を探索します：
```python
from pyqbpp import var, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
f = a + 2 * b + 3 * c == 3
f.simplify_as_binary()
print("f =", f)
print("body =", f.body)

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for sol in sols:
    print(f"a={sol.get(a)}, b={sol.get(b)}, c={sol.get(c)}, "
          f"f={sol.eval(f)}, body={sol.eval(f.body)}")
```
このプログラムでは、`f` は内部的に2つの式を保持しています：
- **`f`**: $(a+2b+3c-3)^2$。等式が満たされるとき最小値0を取ります。
- **`f.body`**: 等式の左辺、$a+2b+3c$。

このプログラムの出力は以下の通りです：
```
f = 9 -5*a -8*b -9*c +4*a*b +6*a*c +12*b*c
body = a +2*b +3*c
a=0, b=0, c=1, f=0, body=3
a=1, b=1, c=0, f=0, body=3
```

## サポートされる等価演算子の形式に関する注意
PyQBPPは等価演算子を以下の形式でのみサポートしています：
- **`expression == integer`**

以下の形式はサポートされていません：
- **`integer == expression`**
- **`expression1 == expression2`**

`expression1 == expression2` の代わりに、制約を以下のように書き換えることができます：
- **`expression1 - expression2 == 0`**

## 範囲演算子
範囲演算子 `between(f, l, u)` は、制約 $l\leq f\leq u$ が満たされるときかつそのときに限り最小値0を取る式を作成します。

> **注意**
> C++版では `l <= f <= u` という構文を使いますが、PyQBPPでは式に対する範囲制約に `between(f, l, u)` 関数を使います。

以下のプログラムは範囲演算子の使用方法を示しています：
```python
from pyqbpp import var, between, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
f = between(4 * a + 9 * b + 15 * c, 5, 14)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()
for sol in sols:
    print(f"a={sol.get(a)}, b={sol.get(b)}, c={sol.get(c)}, "
          f"f={sol.eval(f)}, body={sol.eval(f.body)}")
```
このプログラムは制約 $5\leq 4a+9b+15c \leq 14$ を満たす解を探索し、以下の出力を生成します：
```
a=0, b=1, c=0, f=0, body=9
a=1, b=1, c=0, f=0, body=13
```
</div>
