---
layout: default
nav_exclude: true
title: "QR: Variables"
nav_order: 30
---
<div class="lang-en" markdown="1">
# Quick Reference: Variables and Expressions
## Data types in PyQBPP
PyQBPP uses Python's native `int` type for coefficients, energy values, and constants.
Since Python integers have unlimited precision, there is no need to specify `coeff_t` or `energy_t` as in the C++ version.

## Printing objects
All PyQBPP objects can be printed using `print()` or converted to strings using `str()`:
```python
print(obj)
s = str(obj)
```

## Variable classes
- **`pyqbpp.Var`**:
  A class that holds a unique 32-bit integer ID.
  The variable name can be retrieved via the `name` property.

> **NOTE**
> A `pyqbpp.Var` object represents a variable symbolically.
> No specific data type is associated with it.
> It can be used to represent binary, spin, or other types of variables.

### Variable creation functions
The following functions are provided to create variables:

- **`pyqbpp.var("name")`**:
  Creates a `pyqbpp.Var` object with the given name `"name"`.

- **`pyqbpp.var("name", s1)`**:
  Creates a one-dimensional array (vector) of `pyqbpp.Var` objects with the base name `"name"`.
  Each element is represented as `name[i]`.
  The resulting type is `pyqbpp.Vector`.

- **`pyqbpp.var("name", s1, s2)`**:
  Creates a two-dimensional array (matrix) of `pyqbpp.Var` objects with the base name `"name"`.
  Each element is represented as `name[i][j]`.
  The resulting type is a nested `pyqbpp.Vector`.

- **`pyqbpp.var("name", s1, s2, ...)`**:
  Creates a higher-dimensional array of `pyqbpp.Var` objects with the base name `"name"`.
  Each element is represented as `name[i][j]...`.
  The resulting type is a nested `pyqbpp.Vector`.

> **NOTE**
> If `"name"` is omitted, numbered names such as `"{0}"`, `"{1}"`, ... are automatically assigned in creation order.

### Examples
```python
import pyqbpp as qbpp

x = qbpp.var("x")          # Single variable named "x"
y = qbpp.var("y", 3)       # Vector: y[0], y[1], y[2]
z = qbpp.var("z", 2, 3)    # 2x3 matrix: z[0][0], ..., z[1][2]
a = qbpp.var()             # Single unnamed variable
b = qbpp.var(5)            # Vector of 5 unnamed variables
```

## `pyqbpp.Var` properties and methods
For a `pyqbpp.Var` instance `x`, the following are available:

- **`x.name`** (property):
  Returns the name of `x` as a string.

- **`x.index`** (property):
  Returns the unique integer ID of `x`.

Usually, there is no need to use these properties explicitly in PyQBPP programs.

## Integer variable class
- **`pyqbpp.VarInt`**:
  A class derived from `pyqbpp.Expr` that represents an integer variable with a specified range.

### Integer variable creation functions
The following functions are provided to create integer variables:

- **`pyqbpp.var_int("name")`**:
  Returns an internally used helper object (`pyqbpp.VarIntCore`) and does not create a `pyqbpp.VarInt` by itself.
  To define a `pyqbpp.VarInt`, the range must be specified using the `between()` function, as shown below.

- **`pyqbpp.between(pyqbpp.var_int("name"), l, u)`**:
  Here, `l` and `u` must be integers.
  This expression creates a `pyqbpp.VarInt` object with the name `"name"`,
  which internally contains a `pyqbpp.Expr` object representing all integers in the range `[l, u]`.
  Internally, this also creates `pyqbpp.Var` objects used in the underlying expression.

- **`pyqbpp.between(pyqbpp.var_int("name", s1), l, u)`**:
  Creates a one-dimensional array (vector) of `pyqbpp.VarInt` objects with the base name `"name"`
  and the same range `[l, u]`.
  Each element is represented as `name[i]`.
  The resulting type is `pyqbpp.Vector`.
  Higher-dimensional arrays of `pyqbpp.VarInt` objects can be created in the same way as `pyqbpp.Var` objects.

### Examples
```python
import pyqbpp as qbpp

x = qbpp.between(qbpp.var_int("x"), 0, 10)       # Integer variable x in [0, 10]
y = qbpp.between(qbpp.var_int("y", 3), -5, 5)    # Vector of 3 integer variables in [-5, 5]
z = qbpp.between(qbpp.var_int("z", 2, 3), 1, 8)  # 2x3 matrix of integer variables in [1, 8]
```

### Integer variable properties
For a `pyqbpp.VarInt` instance `x`, the following are available:

- **`x.min_val`** (property):
  Returns the minimum value `l` of `x`.

- **`x.max_val`** (property):
  Returns the maximum value `u` of `x`.

- **`x.int_vars`** (property):
  Returns the list of `pyqbpp.Var` objects used to represent the integer variable.

- **`x.coeffs`** (property):
  Returns a list of integer coefficients.

The following expression is equivalent to the expression stored in `x`:
```python
x.min_val + qbpp.sum(x.int_vars * x.coeffs)
```

### Comparison with C++ QUBO++

<table>
<thead>
<tr><th>C++ QUBO++</th><th>PyQBPP</th></tr>
</thead>
<tbody>
<tr><td><code>l &lt;= qbpp::var_int("name") &lt;= u</code></td><td><code>between(var_int("name"), l, u)</code></td></tr>
<tr><td><code>l &lt;= qbpp::var_int("name", s1) &lt;= u</code></td><td><code>between(var_int("name", s1), l, u)</code></td></tr>
<tr><td><code>x.name()</code></td><td><code>x.name</code> (property)</td></tr>
<tr><td><code>x.str()</code></td><td><code>str(x)</code></td></tr>
<tr><td><code>x.min_val()</code></td><td><code>x.min_val</code> (property)</td></tr>
<tr><td><code>x.max_val()</code></td><td><code>x.max_val</code> (property)</td></tr>
<tr><td><code>x.vars()</code></td><td><code>x.int_vars</code> (property)</td></tr>
<tr><td><code>x.coeffs()</code></td><td><code>x.coeffs</code> (property)</td></tr>
</tbody>
</table>

</div>

<div class="lang-ja" markdown="1">
# クイックリファレンス: 変数と式
## PyQBPP のデータ型
PyQBPPは係数、エネルギー値、定数にPythonのネイティブな `int` 型を使用します。
Pythonの整数は精度に制限がないため、C++版のように `coeff_t` や `energy_t` を指定する必要はありません。

## オブジェクトの表示
すべてのPyQBPPオブジェクトは `print()` で表示するか、`str()` で文字列に変換できます。
```python
print(obj)
s = str(obj)
```

## 変数クラス
- **`pyqbpp.Var`**:
  一意な32ビット整数IDを保持するクラスです。
  変数名は `name` プロパティで取得できます。

> **NOTE**
> `pyqbpp.Var` オブジェクトは変数をシンボリックに表現します。
> 特定のデータ型は関連付けられていません。
> バイナリ変数、スピン変数、その他の種類の変数を表現するために使用できます。

### 変数作成関数
変数を作成するために以下の関数が提供されています。

- **`pyqbpp.var("name")`**:
  指定された名前 `"name"` を持つ `pyqbpp.Var` オブジェクトを作成します。

- **`pyqbpp.var("name", s1)`**:
  基本名 `"name"` を持つ `pyqbpp.Var` オブジェクトの1次元配列（ベクトル）を作成します。
  各要素は `name[i]` として表されます。
  結果の型は `pyqbpp.Vector` です。

- **`pyqbpp.var("name", s1, s2)`**:
  基本名 `"name"` を持つ `pyqbpp.Var` オブジェクトの2次元配列（行列）を作成します。
  各要素は `name[i][j]` として表されます。
  結果の型はネストされた `pyqbpp.Vector` です。

- **`pyqbpp.var("name", s1, s2, ...)`**:
  基本名 `"name"` を持つ `pyqbpp.Var` オブジェクトの高次元配列を作成します。
  各要素は `name[i][j]...` として表されます。
  結果の型はネストされた `pyqbpp.Vector` です。

> **NOTE**
> `"name"` を省略すると、作成順に `"{0}"`、`"{1}"` などの番号付き名前が自動的に割り当てられます。

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")          # Single variable named "x"
y = qbpp.var("y", 3)       # Vector: y[0], y[1], y[2]
z = qbpp.var("z", 2, 3)    # 2x3 matrix: z[0][0], ..., z[1][2]
a = qbpp.var()             # Single unnamed variable
b = qbpp.var(5)            # Vector of 5 unnamed variables
```

## `pyqbpp.Var` のプロパティとメソッド
`pyqbpp.Var` のインスタンス `x` に対して、以下が利用可能です。

- **`x.name`** (プロパティ):
  `x` の名前を文字列として返します。

- **`x.index`** (プロパティ):
  `x` の一意な整数IDを返します。

通常、PyQBPPプログラムでこれらのプロパティを明示的に使用する必要はありません。

## 整数変数クラス
- **`pyqbpp.VarInt`**:
  `pyqbpp.Expr` から派生したクラスで、指定された範囲を持つ整数変数を表現します。

### 整数変数作成関数
整数変数を作成するために以下の関数が提供されています。

- **`pyqbpp.var_int("name")`**:
  内部的に使用されるヘルパーオブジェクト（`pyqbpp.VarIntCore`）を返し、それ自体では `pyqbpp.VarInt` を作成しません。
  `pyqbpp.VarInt` を定義するには、以下に示すように `between()` 関数を使って範囲を指定する必要があります。

- **`pyqbpp.between(pyqbpp.var_int("name"), l, u)`**:
  ここで `l` と `u` は整数でなければなりません。
  この式は名前 `"name"` を持つ `pyqbpp.VarInt` オブジェクトを作成し、
  内部的に範囲 `[l, u]` のすべての整数を表す `pyqbpp.Expr` オブジェクトを含みます。
  内部的に、基礎となる式で使用される `pyqbpp.Var` オブジェクトも作成します。

- **`pyqbpp.between(pyqbpp.var_int("name", s1), l, u)`**:
  基本名 `"name"` と同じ範囲 `[l, u]` を持つ `pyqbpp.VarInt` オブジェクトの1次元配列（ベクトル）を作成します。
  各要素は `name[i]` として表されます。
  結果の型は `pyqbpp.Vector` です。
  `pyqbpp.VarInt` オブジェクトの高次元配列は、`pyqbpp.Var` オブジェクトと同じ方法で作成できます。

### 例
```python
import pyqbpp as qbpp

x = qbpp.between(qbpp.var_int("x"), 0, 10)       # Integer variable x in [0, 10]
y = qbpp.between(qbpp.var_int("y", 3), -5, 5)    # Vector of 3 integer variables in [-5, 5]
z = qbpp.between(qbpp.var_int("z", 2, 3), 1, 8)  # 2x3 matrix of integer variables in [1, 8]
```

### 整数変数のプロパティ
`pyqbpp.VarInt` のインスタンス `x` に対して、以下が利用可能です。

- **`x.min_val`** (プロパティ):
  `x` の最小値 `l` を返します。

- **`x.max_val`** (プロパティ):
  `x` の最大値 `u` を返します。

- **`x.int_vars`** (プロパティ):
  整数変数を表現するために使用される `pyqbpp.Var` オブジェクトのリストを返します。

- **`x.coeffs`** (プロパティ):
  整数係数のリストを返します。

以下の式は `x` に格納されている式と等価です。
```python
x.min_val + qbpp.sum(x.int_vars * x.coeffs)
```

### C++ QUBO++ との比較

<table>
<thead>
<tr><th>C++ QUBO++</th><th>PyQBPP</th></tr>
</thead>
<tbody>
<tr><td><code>l &lt;= qbpp::var_int("name") &lt;= u</code></td><td><code>between(var_int("name"), l, u)</code></td></tr>
<tr><td><code>l &lt;= qbpp::var_int("name", s1) &lt;= u</code></td><td><code>between(var_int("name", s1), l, u)</code></td></tr>
<tr><td><code>x.name()</code></td><td><code>x.name</code> (property)</td></tr>
<tr><td><code>x.str()</code></td><td><code>str(x)</code></td></tr>
<tr><td><code>x.min_val()</code></td><td><code>x.min_val</code> (property)</td></tr>
<tr><td><code>x.max_val()</code></td><td><code>x.max_val</code> (property)</td></tr>
<tr><td><code>x.vars()</code></td><td><code>x.int_vars</code> (property)</td></tr>
<tr><td><code>x.coeffs()</code></td><td><code>x.coeffs</code> (property)</td></tr>
</tbody>
</table>

</div>
