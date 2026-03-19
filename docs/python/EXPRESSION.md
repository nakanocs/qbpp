---
layout: default
nav_exclude: true
title: "Expression Classes (PyQBPP)"
nav_order: 15
---
<div class="lang-en" markdown="1">
# Expression Classes

PyQBPP uses three internal classes to represent expressions:

| Class | Contains | Details |
|------|-----|-----|
| `Var` | A variable  |  a 32-bit ID and a string to display |
| `Term` | A product term | Zero or more variables and an integer coefficient |
| `Expr` | An expression | Zero or more terms and an integer constant term |

Unlike C++ QUBO++, **you do not need to be aware of these class distinctions** in PyQBPP.
Python's dynamic typing automatically converts between types as needed.
For example, `2 * x * y` creates a `Term` internally, but you can use `+=` or other operators on it and it will be converted to an `Expr` automatically.

The following program demonstrates how expressions are built and simplified:
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

Expressions can also be built incrementally starting from a plain integer:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)
f = -1
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

PyQBPPは式を表現するために内部的に3つのクラスを使用しています：

| クラス | 内容 | 詳細 |
|------|-----|-----|
| `Var` | 変数 | 32ビットのIDと表示用の文字列 |
| `Term` | 積の項 | 0個以上の変数と整数係数 |
| `Expr` | 式 | 0個以上の項と整数定数項 |

C++ QUBO++とは異なり、PyQBPPでは**これらのクラスの違いを意識する必要はありません**。
Pythonの動的型付けが必要に応じて自動的に型変換を行います。
例えば、`2 * x * y` は内部的に `Term` を生成しますが、`+=` 等の演算子を使えば自動的に `Expr` に変換されます。

以下のプログラムは式の構築と簡約化を示しています：
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = (x + y - 2) * (x - 2 * y + 3)
print("f =", f)
f.simplify()
print("simplified f =", f)
```
このプログラムの出力は以下の通りです：
```
f = -6 +x*x +y*x -2*x*y -2*y*y +3*x +3*y -2*x +4*y
simplified f = -6 +x +7*y +x*x -x*y -2*y*y
```

整数から始めてインクリメンタルに式を構築することもできます：
```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)
f = -1
for i in range(len(x)):
    f += x[i]
print(f)
```
このプログラムの出力は以下の通りです：
```
-1 +x[0] +x[1] +x[2] +x[3]
```
</div>
