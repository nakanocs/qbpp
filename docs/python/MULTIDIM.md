---
layout: default
nav_exclude: true
title: "Multi-dimensional Variables"
nav_order: 13
---
<div class="lang-en" markdown="1">
# Multi-dimensional Variables and Expressions

## Defining multi-dimensional variables
PyQBPP supports **multi-dimensional variables** and **multi-dimensional integer variables** of arbitrary depth using the functions `var()` and `var_int()`, respectively.
Their basic usage is as follows:
- `var("name", s1, s2, ..., sd)`: Creates an array of `Var` objects with the given `name` and shape $s_1\times s_2\times \cdots\times s_d$.
- `between(var_int("name", s1, s2, ..., sd), l, u)`: Creates an array of `VarInt` objects with the specified range and shape.

The following program creates a binary variable with dimension $2\times 3\times 4$:
```python
from pyqbpp import var

x = var("x", 2, 3, 4)
print("x =", x)
```
{% raw %}
Each `Var` object in **`x`** can be accessed as **`x[i][j][k]`**.
{% endraw %}

## Defining multi-dimensional expressions
PyQBPP allows you to define **multi-dimensional expressions** with arbitrary depth using the function `expr()`:
- **`expr(s1, s2, ..., sd)`**: Creates a multi-dimensional array of `Expr` objects with shape $s_1\times s_2\times \cdots\times s_d$.

The following program defines a 3-dimensional array **`x`** of `Var` objects with shape $2\times 3\times 4$ and
a 2-dimensional array `f` of size $2\times 3$.
Then, using nested loops, each `f[i][j]` accumulates the sum of `x[i][j][0]` through `x[i][j][3]`:
```python
from pyqbpp import var, expr

x = var("x", 2, 3, 4)
f = expr(2, 3)
for i in range(2):
    for j in range(3):
        for k in range(4):
            f[i][j] += x[i][j][k]
f.simplify_as_binary()

for i in range(2):
    for j in range(3):
        print(f"f[{i}][{j}] =", f[i][j])
```
This program produces the following output:
```
f[0][0] = x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3]
f[0][1] = x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3]
f[0][2] = x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]
f[1][0] = x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3]
f[1][1] = x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3]
f[1][2] = x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]
```

## Creating an array of expressions by operations
An array of `Expr` objects can be created without explicitly calling `expr()`.
When an arithmetic operation yields an array-shaped result, an array of `Expr` objects with the same shape is created automatically.

```python
from pyqbpp import var

x = var("x", 2, 3)
f = x + 1
f += x - 2
f.simplify_as_binary()
for i in range(2):
    for j in range(3):
        print(f"f[{i}][{j}] =", f[i][j])
```
This program outputs:
```
f[0][0] = -1 +2*x[0][0]
f[0][1] = -1 +2*x[0][1]
f[0][2] = -1 +2*x[0][2]
f[1][0] = -1 +2*x[1][0]
f[1][1] = -1 +2*x[1][1]
f[1][2] = -1 +2*x[1][2]
```

## Iterating over multi-dimensional arrays
Since PyQBPP vectors support Python iteration, nested for loops can be used:
```python
from pyqbpp import var

x = var("x", 2, 3)
f = x + 1
f += x - 2
f.simplify_as_binary()
for row in f:
    for element in row:
        print(f"({element})", end="")
    print()
```
This program outputs:
```
(-1 +2*x[0][0])(-1 +2*x[0][1])(-1 +2*x[0][2])
(-1 +2*x[1][0])(-1 +2*x[1][1])(-1 +2*x[1][2])
```
</div>

<div class="lang-ja" markdown="1">
# 多次元変数と式

## 多次元変数の定義
PyQBPPは、関数 `var()` および `var_int()` を使って、任意の深さの**多次元変数**および**多次元整数変数**をサポートしています。
基本的な使い方は以下の通りです。
- `var("name", s1, s2, ..., sd)`: 指定された `name` と形状 $s_1\times s_2\times \cdots\times s_d$ を持つ `Var` オブジェクトの配列を作成します。
- `between(var_int("name", s1, s2, ..., sd), l, u)`: 指定された範囲と形状を持つ `VarInt` オブジェクトの配列を作成します。

以下のプログラムは $2\times 3\times 4$ の次元を持つバイナリ変数を作成します。
```python
from pyqbpp import var

x = var("x", 2, 3, 4)
print("x =", x)
```
{% raw %}
**`x`** 内の各 `Var` オブジェクトは **`x[i][j][k]`** としてアクセスできます。
{% endraw %}

## 多次元式の定義
PyQBPPでは、関数 `expr()` を使って任意の深さの**多次元式**を定義できます。
- **`expr(s1, s2, ..., sd)`**: 形状 $s_1\times s_2\times \cdots\times s_d$ を持つ `Expr` オブジェクトの多次元配列を作成します。

以下のプログラムは、形状 $2\times 3\times 4$ の `Var` オブジェクトの3次元配列 **`x`** と、
サイズ $2\times 3$ の2次元配列 `f` を定義します。
次に、ネストされたループを使って、各 `f[i][j]` に `x[i][j][0]` から `x[i][j][3]` までの合計を蓄積します。
```python
from pyqbpp import var, expr

x = var("x", 2, 3, 4)
f = expr(2, 3)
for i in range(2):
    for j in range(3):
        for k in range(4):
            f[i][j] += x[i][j][k]
f.simplify_as_binary()

for i in range(2):
    for j in range(3):
        print(f"f[{i}][{j}] =", f[i][j])
```
このプログラムの出力は以下の通りです。
```
f[0][0] = x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3]
f[0][1] = x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3]
f[0][2] = x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]
f[1][0] = x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3]
f[1][1] = x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3]
f[1][2] = x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]
```

## 演算による式配列の作成
`Expr` オブジェクトの配列は、`expr()` を明示的に呼び出さなくても作成できます。
算術演算が配列形状の結果を生成する場合、同じ形状の `Expr` オブジェクトの配列が自動的に作成されます。

```python
from pyqbpp import var

x = var("x", 2, 3)
f = x + 1
f += x - 2
f.simplify_as_binary()
for i in range(2):
    for j in range(3):
        print(f"f[{i}][{j}] =", f[i][j])
```
このプログラムの出力は以下の通りです。
```
f[0][0] = -1 +2*x[0][0]
f[0][1] = -1 +2*x[0][1]
f[0][2] = -1 +2*x[0][2]
f[1][0] = -1 +2*x[1][0]
f[1][1] = -1 +2*x[1][1]
f[1][2] = -1 +2*x[1][2]
```

## 多次元配列のイテレーション
PyQBPPのベクトルはPythonのイテレーションをサポートしているため、ネストされた for ループが使用できます。
```python
from pyqbpp import var

x = var("x", 2, 3)
f = x + 1
f += x - 2
f.simplify_as_binary()
for row in f:
    for element in row:
        print(f"({element})", end="")
    print()
```
このプログラムの出力は以下の通りです。
```
(-1 +2*x[0][0])(-1 +2*x[0][1])(-1 +2*x[0][2])
(-1 +2*x[1][0])(-1 +2*x[1][1])(-1 +2*x[1][2])
```
</div>
