---
layout: default
nav_exclude: true
title: "Data Types (PyQBPP)"
nav_order: 10
---
<div class="lang-en" markdown="1">
# Variable and Expression Classes

## Var, Term, and Expr classes

PyQBPP provides the following fundamental classes:
- **`Var`**: Represents a variable symbolically and is associated with a string used for display.
- **`Term`**: Represents a product term consisting of an integer coefficient and one or more `Var` objects.
- **`Expr`**: Represents an expanded expression consisting of an integer constant term and zero or more `Term` objects.

In the following program, **`x`** and **`y`** are `Var` objects, **`t`** is a `Term` object, and **`f`** is an `Expr` object:
```python
from pyqbpp import var

x = var("x")
y = var("y")
t = 2 * x * y
f = t - x + 1

print("x =", x)
print("y =", y)
print("t =", t)
print("f =", f)
```
This program produces the following output:
```
x = x
y = y
t = 2*x*y
f = 1 -x +2*x*y
```

`Var` objects are **immutable** and cannot be updated after creation.
In contrast, `Term` and `Expr` objects are **mutable** and can be updated via assignment.

For example, compound assignment operators can be used to update `Expr` objects:
```python
from pyqbpp import var, toExpr

x = var("x")
y = var("y")
f = toExpr(2 * x * y)
f += 3 * y

print("f =", f)
```
This program prints:
```
f = 2*x*y +3*y
```

## Differences from C++

In Python, there is no risk of accidentally creating a `Term` when you meant to create an `Expr`,
because Python's dynamic typing handles conversions automatically.
However, to explicitly create an `Expr` from an integer, a `Var`, or a `Term`, the **`toExpr()`** function is available:
```python
from pyqbpp import var, toExpr

x = var("x")
f = toExpr(0)    # Expr with constant 0
g = toExpr(x)    # Expr containing variable x
h = toExpr(3 * x)  # Expr from Term

f += 2 * x + 1
print("f =", f)
print("g =", g)
print("h =", h)
```
This program prints:
```
f = 1 +2*x
g = x
h = 3*x
```

## Coefficient and Energy Types
In PyQBPP, coefficients and energy values use **arbitrary-precision integers** by default.
Unlike the C++ version, there is no need to specify `COEFF_TYPE` or `ENERGY_TYPE` macros.

For example, the following program creates an expression with very large coefficients:
```python
from pyqbpp import var

x = var("x")
f = 123456789012345678901234567890 * x + 987654321098765432109876543210
print("f =", f)
```
This program produces the following output:
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```
</div>

<div class="lang-ja" markdown="1">
# 変数と式のクラス

## Var、Term、Exprクラス

PyQBPPは以下の基本クラスを提供します:
- **`Var`**: 変数をシンボリックに表現し、表示用の文字列と関連付けられます。
- **`Term`**: 整数係数と1つ以上の`Var`オブジェクトからなる積の項を表現します。
- **`Expr`**: 整数の定数項と0個以上の`Term`オブジェクトからなる展開された式を表現します。

以下のプログラムでは、**`x`**と**`y`**は`Var`オブジェクト、**`t`**は`Term`オブジェクト、**`f`**は`Expr`オブジェクトです:
```python
from pyqbpp import var

x = var("x")
y = var("y")
t = 2 * x * y
f = t - x + 1

print("x =", x)
print("y =", y)
print("t =", t)
print("f =", f)
```
このプログラムは以下の出力を生成します:
```
x = x
y = y
t = 2*x*y
f = 1 -x +2*x*y
```

`Var`オブジェクトは**イミュータブル（不変）**であり、作成後に更新することはできません。
一方、`Term`と`Expr`オブジェクトは**ミュータブル（可変）**であり、代入によって更新できます。

例えば、複合代入演算子を使って`Expr`オブジェクトを更新できます:
```python
from pyqbpp import var, toExpr

x = var("x")
y = var("y")
f = toExpr(2 * x * y)
f += 3 * y

print("f =", f)
```
このプログラムは以下を出力します:
```
f = 2*x*y +3*y
```

## C++との違い

Pythonでは、`Expr`を作成するつもりで誤って`Term`を作成してしまうリスクはありません。
Pythonの動的型付けが変換を自動的に処理するためです。
ただし、整数、`Var`、または`Term`から明示的に`Expr`を作成するには、**`toExpr()`**関数を使用できます:
```python
from pyqbpp import var, toExpr

x = var("x")
f = toExpr(0)    # 定数0のExpr
g = toExpr(x)    # 変数xを含むExpr
h = toExpr(3 * x)  # TermからのExpr

f += 2 * x + 1
print("f =", f)
print("g =", g)
print("h =", h)
```
このプログラムは以下を出力します:
```
f = 1 +2*x
g = x
h = 3*x
```

## 係数型とエネルギー型
PyQBPPでは、係数とエネルギー値はデフォルトで**任意精度整数**を使用します。
C++版とは異なり、`COEFF_TYPE`や`ENERGY_TYPE`マクロを指定する必要はありません。

例えば、以下のプログラムは非常に大きな係数を持つ式を作成します:
```python
from pyqbpp import var

x = var("x")
f = 123456789012345678901234567890 * x + 987654321098765432109876543210
print("f =", f)
```
このプログラムは以下の出力を生成します:
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```
</div>
