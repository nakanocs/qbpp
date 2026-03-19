---
layout: default
nav_exclude: true
title: "Data Types (PyQBPP)"
nav_order: 10
---
<div class="lang-en" markdown="1">
# Variable and Expression Classes

## Var and Expr classes

PyQBPP provides two main classes for building QUBO/HUBO expressions:
- **`Var`**: Represents a binary variable, associated with a display name.
- **`Expr`**: Represents an expression consisting of an integer constant and zero or more product terms.

`Var` objects are created using `qbpp.var()` and are **immutable**.
Expressions are built from variables using arithmetic operators (`+`, `-`, `*`) and are **mutable** via compound assignment operators such as `+=`.

The following program demonstrates variable and expression creation:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 2 * x * y - x + 1
f += 3 * y

print("f =", f)
```
This program prints:
```
f = 1 -x +2*x*y +3*y
```

> **NOTE**
> PyQBPP uses the C++ QUBO++ library as its backend, which internally distinguishes between `Term` (a single product term like `2*x*y`) and `Expr` (a sum of terms).
> In PyQBPP, however, you do not need to be aware of this distinction.
> Python's dynamic typing automatically converts between types as needed, so you can simply write expressions naturally.

## Coefficient and Energy Types
In PyQBPP, coefficients and energy values use **arbitrary-precision integers** by default.
Unlike the C++ version, there is no need to specify `COEFF_TYPE` or `ENERGY_TYPE` macros.

For example, the following program creates an expression with very large coefficients:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
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

## VarクラスとExprクラス

PyQBPPはQUBO/HUBO式を構築するための2つの主要なクラスを提供します：
- **`Var`**: バイナリ変数を表し、表示用の名前が関連付けられます。
- **`Expr`**: 整数の定数と0個以上の積の項からなる式を表します。

`Var` オブジェクトは `qbpp.var()` で作成し、**イミュータブル（不変）**です。
式は変数から算術演算子（`+`, `-`, `*`）を使って構築し、`+=` などの複合代入演算子で**更新可能**です。

以下のプログラムは、変数と式の作成を示しています：
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 2 * x * y - x + 1
f += 3 * y

print("f =", f)
```
このプログラムは以下を出力します：
```
f = 1 -x +2*x*y +3*y
```

> **NOTE**
> PyQBPPはバックエンドとしてC++ QUBO++ライブラリを使用しており、内部的には `Term`（`2*x*y` のような単一の積の項）と `Expr`（項の和）を区別しています。
> しかしPyQBPPでは、この区別を意識する必要はありません。
> Pythonの動的型付けが必要に応じて自動的に型変換を行うため、自然に式を記述できます。

## 係数型とエネルギー型
PyQBPPでは、係数とエネルギー値はデフォルトで**任意精度整数**を使用します。
C++版とは異なり、`COEFF_TYPE`や`ENERGY_TYPE`マクロを指定する必要はありません。

例えば、以下のプログラムは非常に大きな係数を持つ式を作成します:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = 123456789012345678901234567890 * x + 987654321098765432109876543210
print("f =", f)
```
このプログラムは以下の出力を生成します:
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```
</div>
