---
layout: default
nav_exclude: true
title: "Variables and Expressions (PyQBPP)"
nav_order: 2
---
<div class="lang-en" markdown="1">
# Defining Variables and Creating Expressions

## Installing PyQBPP
To use PyQBPP, install it via pip:
```bash
pip install pyqbpp
```

## Importing the library
To use PyQBPP, import the necessary functions from the **`pyqbpp`** module:
```python
import pyqbpp as qbpp
```

## Defining variables and expressions
You can define a variable using **`var("name")`**.
The specified `name` is used when the variable is printed.

Expressions are constructed using standard arithmetic operators such as **`+`**, **`-`**, and **`*`**.

The following program defines three variables `a`, `b`, and `c`, and an expression `f`, which is printed:
```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
f = (a + b - 1) * (b + c - 1)
print("f =", f)
```
The expression `(a + b - 1) * (b + c - 1)` is automatically expanded and stored in `f`.

In this program, the variables `a`, `b`, and `c` are objects of class **`Var`**, and the expression `f` is an object of class **`Expr`**.

Running the program prints the expanded expression:
```
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

> **NOTE**
> The variable name in `var()` may be omitted.
> If omitted, a default name such as `{0}`, `{1}`,... is automatically assigned.

> **WARNING**
> The textual output of expressions is not guaranteed to be stable and should not be used as input for subsequent computations, since its format may change in future releases.

## Simplifying expression
The expression stored in an **`Expr`** object can be simplified by calling the **`simplify()`** member function:
```python
print("f =", f.simplify())
```
With this change, the output of the program becomes:
```
f = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
The member function call **`f.simplify()`** simplifies the expression `f` in place and returns itself.

Assuming that all variables take **binary values (0 or 1)**, we can use the identity
**$b^2=b$** to further simplify the expression.
For this purpose, we use **`simplify_as_binary()`** instead:
```python
print("f =", f.simplify_as_binary())
```
Then the output becomes:
```
f = 1 -a -b -c +a*b +a*c +b*c
```

## Simplifying expressions with spin variables
If variables are assumed to take **spin values -1/+1**, the identity **$b^2 = 1$** can be used to further simplify the expression.
In this case, the expression can be simplified using the **`simplify_as_spin()`** member function:
```python
print("f =", f.simplify_as_spin())
```
Then the output becomes:
```
f = 2 -a -2*b -c +a*b +a*c +b*c
```

## Global functions for simplification
Member functions update the expression in place.
If you do not want to modify `f`, you can instead use the global functions
**`simplify(f)`**, **`simplify_as_binary(f)`**, and **`simplify_as_spin(f)`**, which return the simplified expressions without changing `f`.

```python
import pyqbpp as qbpp
g = qbpp.simplify_as_binary(f)  # f is not modified, g is a new simplified expression
```

> **NOTE**
> In PyQBPP, most **member functions** update the object in place when possible, whereas **global functions** return a new value without modifying the original object.
</div>

<div class="lang-ja" markdown="1">
# 変数の定義と式の作成

## PyQBPPのインストール
PyQBPPを使用するには、pipでインストールしてください:
```bash
pip install pyqbpp
```

## ライブラリのインポート
PyQBPPを使用するには、**`pyqbpp`**モジュールから必要な関数をインポートします:
```python
import pyqbpp as qbpp
```

## 変数と式の定義
変数は**`var("name")`**を使って定義できます。
指定した`name`は変数を表示する際に使用されます。

式は**`+`**、**`-`**、**`*`**などの標準的な算術演算子を使って構築します。

以下のプログラムは、3つの変数`a`、`b`、`c`と式`f`を定義し、表示します:
```python
import pyqbpp as qbpp

a = qbpp.var("a")
b = qbpp.var("b")
c = qbpp.var("c")
f = (a + b - 1) * (b + c - 1)
print("f =", f)
```
式`(a + b - 1) * (b + c - 1)`は自動的に展開され、`f`に格納されます。

このプログラムでは、変数`a`、`b`、`c`はクラス**`Var`**のオブジェクトであり、式`f`はクラス**`Expr`**のオブジェクトです。

プログラムを実行すると、展開された式が表示されます:
```
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

> **NOTE**
> `var()`の変数名は省略可能です。
> 省略した場合、`{0}`、`{1}`、...のようなデフォルト名が自動的に割り当てられます。

> **WARNING**
> 式のテキスト出力は安定性が保証されておらず、今後のリリースで形式が変更される可能性があるため、後続の計算への入力として使用しないでください。

## 式の簡約化
**`Expr`**オブジェクトに格納された式は、**`simplify()`**メンバ関数を呼び出すことで簡約化できます:
```python
print("f =", f.simplify())
```
この変更により、プログラムの出力は以下のようになります:
```
f = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
メンバ関数呼び出し**`f.simplify()`**は式`f`をその場で簡約化し、自身を返します。

すべての変数が**バイナリ値（0または1）**を取ると仮定すると、恒等式
**$b^2=b$**を使って式をさらに簡約化できます。
この目的には、代わりに**`simplify_as_binary()`**を使用します:
```python
print("f =", f.simplify_as_binary())
```
すると出力は以下のようになります:
```
f = 1 -a -b -c +a*b +a*c +b*c
```

## スピン変数での式の簡約化
変数が**スピン値 -1/+1**を取ると仮定する場合、恒等式**$b^2 = 1$**を使って式をさらに簡約化できます。
この場合、**`simplify_as_spin()`**メンバ関数を使って式を簡約化できます:
```python
print("f =", f.simplify_as_spin())
```
すると出力は以下のようになります:
```
f = 2 -a -2*b -c +a*b +a*c +b*c
```

## 簡約化のグローバル関数
メンバ関数は式をその場で更新します。
`f`を変更したくない場合は、代わりにグローバル関数
**`simplify(f)`**、**`simplify_as_binary(f)`**、**`simplify_as_spin(f)`**を使用できます。これらは`f`を変更せずに簡約化された式を返します。

```python
import pyqbpp as qbpp
g = qbpp.simplify_as_binary(f)  # fは変更されず、gは新しい簡約化された式
```

> **NOTE**
> PyQBPPでは、ほとんどの**メンバ関数**は可能な場合にオブジェクトをその場で更新しますが、**グローバル関数**は元のオブジェクトを変更せずに新しい値を返します。
</div>
