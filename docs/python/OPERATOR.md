---
layout: default
nav_exclude: true
title: "Operators and Functions (PyQBPP)"
nav_order: 11
---
<div class="lang-en" markdown="1">
# Basic Operators and Functions

## Unary and Binary Operators
PyQBPP supports the following basic operators for constructing expressions:
- **`+`**: Returns the sum of the operands.
- **`-`**: Returns the difference of the operands.
- **`*`**: Returns the product of the operands.
- **`/`**: Returns the quotient of the operands (integer division; all coefficients must be divisible).
- unary **`-`**: Returns the negation of the operand.

The following program demonstrates how to construct expressions using these operators:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 6 * -(x + 1) * (y - 1)
g = f / 3

print("f =", f)
print("g =", g)
```
This program produces the following output:
```
f = 6 -6*x*y +6*x -6*y
g = 2 -2*x*y +2*x -2*y
```

## Compound operators
The following compound operators are supported for updating `Expr` objects:
- **`+=`**: Adds the right-hand side operand to the left-hand side.
- **`-=`**: Subtracts the right-hand side operand from the left-hand side.
- **`*=`**: Multiplies the left-hand side by the right-hand side.
- **`/=`**: Divides the left-hand side by the right-hand side (integer division).

The following program demonstrates these compound operators:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = toExpr(6 * x + 4)

f += 3 * y
print("f =", f)

f -= 12
print("f =", f)

f *= 2 * y
print("f =", f)

f /= 2
print("f =", f)
```
This program produces the following output:
```
f = 4 +6*x +3*y
f = -8 +6*x +3*y
f = 12*x*y +6*y*y -16*y
f = 6*x*y +3*y*y -8*y
```

## Square function
PyQBPP provides both a global function **`sqr()`** and a member function **`sqr()`** of the `Expr` class to compute the square of an expression.

The global function **`sqr(f)`** returns a new `Expr` object representing the square of `f` without modifying `f`,
whereas the member function **`f.sqr()`** updates `f` in place.

```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = x + 1

print("qbpp.sqr(f) =", qbpp.sqr(f))
print("f =", f)

f.sqr()
print("f =", f)
```
This program produces the following output:
```
sqr(f) = 1 +x*x +x +x
f = 1 +x
f = 1 +x*x +x +x
```

## Simplify functions
After operators or functions are applied, expressions are automatically expanded but not simplified.
To simplify the resulting expressions, simplify functions must be explicitly called.

PyQBPP provides the following three **global simplify functions**:
- **`simplify(f)`**: Returns a simplified expression by merging coefficients of identical terms.
- **`simplify_as_binary(f)`**: Returns a simplified expression under the assumption that all variables take binary values $0/1$ (i.e., $x^2=x$).
- **`simplify_as_spin(f)`**: Returns a simplified expression under the assumption that all variables take spin values $-1/+1$ (i.e., $x^2=1$).

**Member function** versions are also available. They update the object in place.

```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = qbpp.sqr(x - 1)

print("f =", f)
print("qbpp.simplify(f) =", qbpp.simplify(f))
print("qbpp.simplify_as_binary(f) =", qbpp.simplify_as_binary(f))
print("qbpp.simplify_as_spin(f) =", qbpp.simplify_as_spin(f))
```
This program produces the following output:
```
f = 1 +x*x -x -x
simplify(f) = 1 -2*x +x*x
simplify_as_binary(f) = 1 -x
simplify_as_spin(f) = 2 -2*x
```

> **NOTE**
> In PyQBPP, **member functions** update the object in place, whereas **global functions** return a new value without modifying the original object.
</div>

<div class="lang-ja" markdown="1">
# 基本演算子と関数

## 単項演算子と二項演算子
PyQBPPは式を構築するための以下の基本演算子をサポートしています：
- **`+`**: オペランドの和を返します。
- **`-`**: オペランドの差を返します。
- **`*`**: オペランドの積を返します。
- **`/`**: オペランドの商を返します（整数除算。すべての係数が割り切れる必要があります）。
- 単項 **`-`**: オペランドの符号を反転します。

以下のプログラムは、これらの演算子を使って式を構築する方法を示しています：
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 6 * -(x + 1) * (y - 1)
g = f / 3

print("f =", f)
print("g =", g)
```
このプログラムの出力は以下の通りです：
```
f = 6 -6*x*y +6*x -6*y
g = 2 -2*x*y +2*x -2*y
```

## 複合代入演算子
`Expr`オブジェクトの更新には、以下の複合代入演算子がサポートされています：
- **`+=`**: 右辺のオペランドを左辺に加算します。
- **`-=`**: 右辺のオペランドを左辺から減算します。
- **`*=`**: 左辺に右辺を乗算します。
- **`/=`**: 左辺を右辺で除算します（整数除算）。

以下のプログラムは、これらの複合代入演算子を示しています：
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = toExpr(6 * x + 4)

f += 3 * y
print("f =", f)

f -= 12
print("f =", f)

f *= 2 * y
print("f =", f)

f /= 2
print("f =", f)
```
このプログラムの出力は以下の通りです：
```
f = 4 +6*x +3*y
f = -8 +6*x +3*y
f = 12*x*y +6*y*y -16*y
f = 6*x*y +3*y*y -8*y
```

## 二乗関数
PyQBPPは、式の二乗を計算するためのグローバル関数 **`sqr()`** と `Expr` クラスのメンバ関数 **`sqr()`** の両方を提供しています。

グローバル関数 **`sqr(f)`** は `f` を変更せずに `f` の二乗を表す新しい `Expr` オブジェクトを返します。
一方、メンバ関数 **`f.sqr()`** は `f` をその場で更新します。

```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = x + 1

print("qbpp.sqr(f) =", qbpp.sqr(f))
print("f =", f)

f.sqr()
print("f =", f)
```
このプログラムの出力は以下の通りです：
```
sqr(f) = 1 +x*x +x +x
f = 1 +x
f = 1 +x*x +x +x
```

## 簡約化関数
演算子や関数が適用された後、式は自動的に展開されますが簡約化はされません。
結果の式を簡約化するには、簡約化関数を明示的に呼び出す必要があります。

PyQBPPは以下の3つの**グローバル簡約化関数**を提供しています：
- **`simplify(f)`**: 同一の項の係数をまとめることで簡約化された式を返します。
- **`simplify_as_binary(f)`**: すべての変数がバイナリ値 $0/1$ を取る（すなわち $x^2=x$）という仮定のもとで簡約化された式を返します。
- **`simplify_as_spin(f)`**: すべての変数がスピン値 $-1/+1$ を取る（すなわち $x^2=1$）という仮定のもとで簡約化された式を返します。

**メンバ関数**版も利用可能です。こちらはオブジェクトをその場で更新します。

```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = qbpp.sqr(x - 1)

print("f =", f)
print("qbpp.simplify(f) =", qbpp.simplify(f))
print("qbpp.simplify_as_binary(f) =", qbpp.simplify_as_binary(f))
print("qbpp.simplify_as_spin(f) =", qbpp.simplify_as_spin(f))
```
このプログラムの出力は以下の通りです：
```
f = 1 +x*x -x -x
simplify(f) = 1 -2*x +x*x
simplify_as_binary(f) = 1 -x
simplify_as_spin(f) = 2 -2*x
```

> **注意**
> PyQBPPでは、**メンバ関数**はオブジェクトをその場で更新しますが、**グローバル関数**は元のオブジェクトを変更せずに新しい値を返します。
</div>
