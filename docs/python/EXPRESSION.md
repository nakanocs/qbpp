---
layout: default
nav_exclude: true
title: "Expression Classes (PyQBPP)"
nav_order: 15
---
<div class="lang-en" markdown="1">
# Expression Classes
The most important feature of PyQBPP is its ability to create expressions for solving combinatorial optimization problems.
The following three classes are used for this purpose:

| Class | Contains | Details |
|------|-----|-----|
| `Var` | A variable  |  a 32-bit ID and a string to display |
| `Term` | A product term | Zero or more variables and an integer coefficient |
| `Expr` | An expression | Zero or more terms and an integer constant term |

## `Var` class
An instance of this class represents **a variable symbolically**.
Each `Var` instance consists of a unique ID and a string used for display.

```python
import pyqbpp as qbpp

x = qbpp.var("x")
print(x)
```
This simply prints `x`.

## `Term` class
An instance of this class represents **a product term** involving an integer coefficient and zero or more `Var` objects.

```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
t = 2 * x * y
print(t)
```
This program prints `2*x*y`.

## `Expr` class
An instance of this class represents **an expression** involving an integer constant term and zero or more `Term` objects.

```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 3 + 2 * x * y + 3 * x
print(f)
```
This program prints `3 +2*x*y +3*x`.

Expressions can be written using basic operators such as **`+`**, **`-`**, and **`*`**, as well as parentheses.
Expressions are automatically expanded and stored as an `Expr` object:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = (x + y - 2) * (x - 2 * y + 3)
print("f =", f)
f.simplify()
print("simplified f =", f)
```
This program prints:
```
f = -6 +x*x +y*x -2*x*y -2*y*y +3*x +3*y -2*x +4*y
simplified f = -6 +x +7*y +x*x -x*y -2*y*y
```

## Using `toExpr()` for explicit construction
To explicitly create an `Expr` from an integer, a `Var`, or a `Term`, the **`toExpr()`** function is available.
This is useful when building up an expression incrementally:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)
f = toExpr(-1)
for i in range(len(x)):
    f += x[i]
print(f)
```
This program prints:
```
-1 +x[0] +x[1] +x[2] +x[3]
```
</div>

<div class="lang-ja" markdown="1">
# 式クラス
PyQBPPの最も重要な機能は、組合せ最適化問題を解くための式を作成できることです。
この目的のために、以下の3つのクラスが用意されています。

| クラス | 内容 | 詳細 |
|------|-----|-----|
| `Var` | 変数 | 32ビットのIDと表示用の文字列 |
| `Term` | 積の項 | 0個以上の変数と整数係数 |
| `Expr` | 式 | 0個以上の項と整数定数項 |

## `Var` クラス
このクラスのインスタンスは**変数をシンボリックに**表現します。
各 `Var` インスタンスは、一意のIDと表示用の文字列で構成されます。

```python
import pyqbpp as qbpp

x = qbpp.var("x")
print(x)
```
これは単に `x` と表示します。

## `Term` クラス
このクラスのインスタンスは、整数係数と0個以上の `Var` オブジェクトからなる**積の項**を表現します。

```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
t = 2 * x * y
print(t)
```
このプログラムは `2*x*y` と表示します。

## `Expr` クラス
このクラスのインスタンスは、整数定数項と0個以上の `Term` オブジェクトからなる**式**を表現します。

```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 3 + 2 * x * y + 3 * x
print(f)
```
このプログラムは `3 +2*x*y +3*x` と表示します。

式は **`+`**、**`-`**、**`*`** などの基本演算子や括弧を使って記述できます。
式は自動的に展開され、`Expr` オブジェクトとして格納されます。
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = (x + y - 2) * (x - 2 * y + 3)
print("f =", f)
f.simplify()
print("simplified f =", f)
```
このプログラムの出力は以下の通りです。
```
f = -6 +x*x +y*x -2*x*y -2*y*y +3*x +3*y -2*x +4*y
simplified f = -6 +x +7*y +x*x -x*y -2*y*y
```

## `toExpr()` による明示的な構築
整数、`Var`、または `Term` から明示的に `Expr` を作成するには、**`toExpr()`** 関数を使用します。
これは式をインクリメンタルに構築する場合に便利です。
```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)
f = toExpr(-1)
for i in range(len(x)):
    f += x[i]
print(f)
```
このプログラムの出力は以下の通りです。
```
-1 +x[0] +x[1] +x[2] +x[3]
```
</div>
