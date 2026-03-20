---
layout: default
nav_exclude: true
title: "QR: Operations (PyQBPP)"
nav_order: 31
---
<div class="lang-en" markdown="1">
# Quick Reference: Operators and Functions for Expressions (PyQBPP)

The table below summarizes the operators and functions available for `pyqbpp.Expr` objects.

| Operators/Functions           | Operator Symbols/Function Names                      | Function Type | Return Type       | Argument Type            |
|-------------------------------|------------------------------------------------------|---------------|-------------------|--------------------------|
| Assignment                    | `=`                                                  | —             | `pyqbpp.Expr`     | `ExprType`               |
| Binary Operators              | `+`, `-`, `*`                                        | Global        | `pyqbpp.Expr`     | `ExprType`-`ExprType`    |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member        | `pyqbpp.Expr`     | `ExprType` or `int`      |
| Division                      | `/`                                                  | Global        | `pyqbpp.Expr`     | `ExprType`-`int`         |
| Compound Division             | `/=`                                                 | Member        | `pyqbpp.Expr`     | `int`                    |
| Unary Operators               | `+`, `-`                                             | Global        | `pyqbpp.Expr`     | `ExprType`               |
| Comparison (Equality)         | `==`                                                 | Member        | `pyqbpp.ExprExpr` | `ExprType`-`int`         |
| Comparison (Range)            | `between()`                                          | Global        | `pyqbpp.ExprExpr` | `ExprType`-`int`-`int`   |
| Square                        | `sqr()`                                              | Global        | `pyqbpp.Expr`     | `ExprType`               |
| Type Conversion               | `int()`, `toInt()`                                   | Built-in/Global | `int` or `list` | `pyqbpp.Expr` (constant) |
| GCD                           | `gcd()`                                              | Global        | `int`             | `ExprType`               |
| Simplify                      | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | Global/Member | `pyqbpp.Expr`     | `ExprType`               |
| Eval                          | `f(ml)`                                              | Member        | `int`             | `pyqbpp.Expr`-`list`     |
| Replace                       | `replace()`                                          | Global/Member | `pyqbpp.Expr`     | `ExprType`-`list`        |
| Reduce                        | `reduce()`                                           | Global/Member | `pyqbpp.Expr`     | `ExprType`               |
| Binary/Spin Conversion        | `binary_to_spin()`, `spin_to_binary()`               | Global/Member | `pyqbpp.Expr`     | `ExprType`               |

## Expression-related type: **`ExprType`**
The term **`ExprType`** denotes a category of types that can be converted to a `pyqbpp.Expr` object.
In PyQBPP, this includes:
- `int` — integer constant
- `pyqbpp.Var` — binary variable
- `pyqbpp.Term` — polynomial term
- `pyqbpp.Expr` — expression

## Global and Member Functions
Operators and functions related to `pyqbpp.Expr` are provided in two forms:
- **Global functions**:
These take at least one `ExprType` argument and typically return a new `pyqbpp.Expr` object without modifying the inputs.
- **Member functions**:
These are methods of the `pyqbpp.Expr` class.
In many cases, they update the calling object in place and also return the resulting `pyqbpp.Expr`.

### Example: `sqr()`
The `sqr()` function computes the square of an expression:
- `sqr(f)` (global): returns the square of `f` without modifying `f`

## Type Conversion: **`int()`** and **`toInt()`**
Python's built-in **`int()`** can be used to convert a single constant `Expr` (containing no variables) to a Python `int`.
If the expression contains variables, a `ValueError` is raised.

The **`toInt()`** function extends this to `Vector` objects: it recursively converts a `Vector` of constant `Expr` objects into a nested Python list of `int` values.

```python
import pyqbpp as qbpp

n = int(qbpp.Expr(42))   # 42
v = qbpp.Vector([qbpp.Expr(10), qbpp.Expr(20), qbpp.Expr(30)])
print(qbpp.toInt(v))     # [10, 20, 30]
```

> **NOTE**
> In PyQBPP, explicit type conversion from integers or variables to `Expr` is never needed.
> Python's dynamic typing handles conversions automatically (e.g., `f = 1; f += x` automatically produces an `Expr`).

## Assignment
In Python, the `=` operator rebinds the variable name to a new object.
To copy an expression, use the `Expr` constructor:
```python
f = qbpp.Expr(g)  # f is a copy of g
```

## Binary Operators: `+`, `-`, `*`
These operators take two `ExprType` operands, compute the result, and return it.
If at least one operand is a `pyqbpp.Expr`, the result is always a `pyqbpp.Expr`.
If neither operand is a `pyqbpp.Expr`, the result may be a `pyqbpp.Term`.

### Example
For a variable `x` of type `pyqbpp.Var`:
- `2 + x`: `pyqbpp.Expr`
- `2 * x`: `pyqbpp.Term`

## Compound Assignment Operators: `+=`, `-=`, `*=`
The left-hand side must be a `pyqbpp.Expr`.
The specified operation is applied using the right-hand side operand.
The left-hand side expression is updated in place.

> **NOTE**
> `*=` only accepts `int` operands in PyQBPP.

## Division `/` and Compound Division `/=`
The division operator `/` takes a `pyqbpp.Expr` as the **dividend** and an integer as the **divisor**, and returns the **quotient** as a new `pyqbpp.Expr`.

The dividend expression must be divisible by the divisor; that is,
both the integer constant term and all integer coefficients in the expression must be divisible by the divisor.

The compound division operator `/=` divides the expression in place.

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 6 * x + 4 * y + 2
g = f / 2          # g = 3*x + 2*y + 1
f = qbpp.Expr(f)
f /= 2             # f = 3*x + 2*y + 1
```

## Comparison (Equality): `==`
The equality comparison operator `==` takes:
- a `pyqbpp.Expr` (or `ExprType` that creates one) on the left-hand side, and
- an integer on the right-hand side.

It returns an expression whose minimum value is 0 when the equality constraint is satisfied.
More specifically, for a `pyqbpp.Expr` object `f` and an integer `n`, the operator returns: `sqr(f - n)`.

For the returned object `g`:
- **`g`** represents the constraint expression `sqr(f - n)`, and
- **`g.body`** returns the underlying expression `f`.

### `pyqbpp.ExprExpr` class
Here, `g` is a **`pyqbpp.ExprExpr`** object, which is a derived class of `pyqbpp.Expr`.
The `body` property returns the associated underlying `pyqbpp.Expr` object.

### Comparison with C++ QUBO++
In C++ QUBO++, `*g` (dereference operator) is used to access the underlying expression.
In PyQBPP, `g.body` property is used instead.

## Comparison (Range): `between()`
In C++ QUBO++, the range comparison is written as `l <= f <= u`.
In PyQBPP, the `between()` function is used instead:
```python
g = qbpp.between(f, l, u)
```
where:
- `f` is a non-integer `ExprType`, and
- `l` and `u` are integers.

This function returns an expression whose minimum value is 0 when the range constraint `l <= f <= u` is satisfied.

More specifically, an auxiliary integer variable `a` with unit gaps, taking values in the range `[l, u-1]`, is implicitly introduced, and the function returns:
```python
(f - a) * (f - (a + 1))
```

For the returned `pyqbpp.ExprExpr` object `g`:
- **`g`** represents the constraint expression `(f - a) * (f - (a + 1))`, and
- **`g.body`** returns the underlying expression `f`.

### Comparison with C++ QUBO++

| C++ QUBO++       | PyQBPP            |
|------------------|---------------------|
| `l <= f <= u`    | `between(f, l, u)`  |
| `*g`             | `g.body`            |

## Square function: `sqr()`
For a `pyqbpp.Expr` object `f`:
- **`pyqbpp.sqr(f)`** (global function): Returns the expression `f * f`.
The argument `f` may be any `ExprType` object.

For a `pyqbpp.Vector` object `v`:
- **`pyqbpp.sqr(v)`**: Returns a new `pyqbpp.Vector` with each element squared.

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = qbpp.sqr(x)       # x * x
```

## Greatest Common Divisor function: `gcd()`
The global function **`pyqbpp.gcd()`** takes a `pyqbpp.Expr` object as its argument and returns the greatest common divisor (GCD) of all integer coefficients and the integer constant term.

Since the given expression is divisible by the resulting GCD, all integer coefficients and the integer constant term can be reduced by dividing by the GCD.

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 6 * x + 4 * y + 2
print(gcd(f))    # 2
g = f / gcd(f)   # 3*x + 2*y + 1
```

## Simplify functions: `simplify()`, `simplify_as_binary()`, `simplify_as_spin()`
For a `pyqbpp.Expr` object `f`, the member function **`f.simplify()`** performs the following operations in place:
- Sort variables within each term according to their unique variable IDs
- Merge duplicated terms
- Sort terms such that:
  - lower-degree terms appear earlier, and
  - terms of the same degree are ordered lexicographically.

The global function **`pyqbpp.simplify(f)`** performs the same operations without modifying `f`.

### Binary and Spin Simplification
Two specialized variants of the simplification function are provided:
- **`simplify_as_binary()`**:
Simplification is performed under the assumption that all variables take binary values
$\lbrace 0,1\rbrace$.
The identity $x^2=x$ is applied to all variables $x$.
- **`simplify_as_spin()`**:
Simplification is performed under the assumption that all variables take spin values
$\lbrace -1,+1\rbrace$.
The identity $x^2=1$ is applied to all variables $x$.

Both variants are available as member functions and global functions:
- Member functions (in-place): `f.simplify_as_binary()`, `f.simplify_as_spin()`
- Global functions (non-destructive): `simplify_as_binary(f)`, `simplify_as_spin(f)`

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = qbpp.Expr(x * x + x)
f.simplify_as_binary()  # 2*x (since x^2 = x)

g = qbpp.Expr(x * x + x)
g.simplify_as_spin()    # 1 + x (since x^2 = 1)
```

## Evaluation function: `f(ml)`
The evaluation function takes a list of `(variable, value)` pairs, where each pair defines a mapping from a variable to an integer value.

For a `pyqbpp.Expr` object `f` and a list of pairs `ml`, the evaluation function `f(ml)` evaluates the value of `f` under the variable assignments specified by `ml` and returns the resulting integer value.

All variables appearing in `f` must have corresponding mappings defined in `ml`.

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 3 * x + 2 * y + 1

print(f([(x, 1), (y, 0)]))  # 4  (= 3*1 + 2*0 + 1)
```

## Replace functions: `replace()`
The `replace()` function accepts a list of `(variable, value)` pairs, where each pair maps a variable to an integer or an expression.

For a `pyqbpp.Expr` object `f` and a list of pairs `ml`:
- **`pyqbpp.replace(f, ml)`** (global function):
Returns a new `pyqbpp.Expr` object obtained by replacing variables in `f` according to the mappings in `ml`, without modifying `f`.
- **`f.replace(ml)`** (member function):
Replaces variables in `f` according to the mappings in `ml` in place and returns the resulting `pyqbpp.Expr` object.

### Creating a list of pairs
```python
import pyqbpp as qbpp

ml = [(x, 0), (y, 1)]                    # List of (variable, value) pairs
ml = [(x, 0), (y, qbpp.Expr(z))]         # Values can be expressions
```

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 2 * x + 3 * y + 1

ml = [(x, 1), (y, 0)]
g = qbpp.replace(f, ml)   # g = 2*1 + 3*0 + 1 = 3 (new Expr)
f.replace(ml)         # f is modified in place
```

### Comparison with C++ QUBO++

| C++ QUBO++                    | PyQBPP                          |
|-------------------------------|-----------------------------------|
| `qbpp::MapList ml;`           | `ml = []`                         |
| `ml.push_back({x, 0});`      | `ml.append((x, 0))`              |
| `qbpp::replace(f, ml)`       | `replace(f, ml)`                  |
| `f.replace(ml)`              | `f.replace(ml)`                   |

## Reduce function: `reduce()`
The **`reduce()`** function converts a `pyqbpp.Expr` object containing higher-degree terms into an equivalent `pyqbpp.Expr` object consisting only of linear and quadratic terms, resulting in a QUBO expression.

For a `pyqbpp.Expr` object `f`:
- **`pyqbpp.reduce(f)`** (global function):
Returns a new `pyqbpp.Expr` object with linear and quadratic terms that is equivalent to `f`.
- **`f.reduce()`** (member function):
Replaces `f` with the reduced expression in place and returns the updated expression.

### Example
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
z = qbpp.var("z")
f = qbpp.Expr(x * y * z)
f.simplify_as_binary()
g = reduce(f)   # Reduced to linear and quadratic terms
```

## Binary/Spin Conversion functions: `spin_to_binary()`, `binary_to_spin()`
Let `x` be a binary variable and `s` be a spin variable.
We assume that `x = 1` if and only if `s = 1`.
Under this assumption, the following relations hold:

$$
\begin{aligned}
 s &= 2x-1 \\
 x &= (s+1)/2
\end{aligned}
$$

The **`spin_to_binary()`** function converts a spin-variable expression to a binary-variable expression
by replacing all spin variables `s` with `2 * s - 1`.

The **`binary_to_spin()`** function converts a binary-variable expression to a spin-variable expression
by replacing all binary variables `x` with `(x + 1) / 2`.
The resulting expression is multiplied by $2^d$ (where $d$ is the maximum degree) so that all coefficients remain integers.

Both functions are available as member functions (in-place) and global functions (non-destructive).

### Example
```python
import pyqbpp as qbpp

s = qbpp.var("s")
f = 3 * s + 1
g = spin_to_binary(f)   # -2 + 6*s  (replaced s with 2*s-1)

b = qbpp.var("b")
h = 2 * b + 1
k = binary_to_spin(h)   # 2 + 2*b  (replaced b with (b+1)/2, multiplied by 2)
```

### Comparison with C++ QUBO++

| C++ QUBO++                      | PyQBPP                       |
|---------------------------------|--------------------------------|
| `qbpp::spin_to_binary(f)`      | `spin_to_binary(f)`            |
| `f.spin_to_binary()`           | `f.spin_to_binary()`           |
| `qbpp::binary_to_spin(f)`      | `binary_to_spin(f)`            |
| `f.binary_to_spin()`           | `f.binary_to_spin()`           |
</div>

<div class="lang-ja" markdown="1">
# クイックリファレンス: 式の演算子と関数 (PyQBPP)

以下の表は、`pyqbpp.Expr` オブジェクトで利用可能な演算子と関数をまとめたものです。

| 演算子/関数                    | 演算子記号/関数名                                     | 関数タイプ    | 戻り値の型         | 引数の型                   |
|-------------------------------|------------------------------------------------------|---------------|-------------------|--------------------------|
| 代入                          | `=`                                                  | —             | `pyqbpp.Expr`     | `ExprType`               |
| 二項演算子                     | `+`, `-`, `*`                                        | グローバル     | `pyqbpp.Expr`     | `ExprType`-`ExprType`    |
| 複合代入演算子                  | `+=`, `-=`, `*=`                                     | メンバー       | `pyqbpp.Expr`     | `ExprType` or `int`      |
| 除算                          | `/`                                                  | グローバル     | `pyqbpp.Expr`     | `ExprType`-`int`         |
| 複合除算                       | `/=`                                                 | メンバー       | `pyqbpp.Expr`     | `int`                    |
| 単項演算子                     | `+`, `-`                                             | グローバル     | `pyqbpp.Expr`     | `ExprType`               |
| 比較（等価）                    | `==`                                                 | メンバー       | `pyqbpp.ExprExpr` | `ExprType`-`int`         |
| 比較（範囲）                    | `between()`                                          | グローバル     | `pyqbpp.ExprExpr` | `ExprType`-`int`-`int`   |
| 二乗                          | `sqr()`                                              | グローバル     | `pyqbpp.Expr`     | `ExprType`               |
| 型変換                         | `int()`, `toInt()`                                   | 組み込み/グローバル | `int` または `list` | `pyqbpp.Expr`（定数）    |
| 最大公約数                      | `gcd()`                                              | グローバル     | `int`             | `ExprType`               |
| 簡約化                         | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | グローバル/メンバー | `pyqbpp.Expr`     | `ExprType`               |
| 評価                           | `f(ml)`                                              | メンバー       | `int`             | `pyqbpp.Expr`-`list`     |
| 置換                           | `replace()`                                          | グローバル/メンバー | `pyqbpp.Expr`     | `ExprType`-`list`        |
| 次数削減                        | `reduce()`                                           | グローバル/メンバー | `pyqbpp.Expr`     | `ExprType`               |
| バイナリ/スピン変換              | `binary_to_spin()`, `spin_to_binary()`               | グローバル/メンバー | `pyqbpp.Expr`     | `ExprType`               |

## 式関連の型: **`ExprType`**
**`ExprType`** とは、`pyqbpp.Expr` オブジェクトに変換可能な型の総称です。
PyQBPPでは以下が含まれます。
- `int` — 整数定数
- `pyqbpp.Var` — バイナリ変数
- `pyqbpp.Term` — 多項式の項
- `pyqbpp.Expr` — 式

## グローバル関数とメンバー関数
`pyqbpp.Expr` に関連する演算子と関数は2つの形式で提供されています。
- **グローバル関数**:
少なくとも1つの `ExprType` 引数を取り、通常は入力を変更せずに新しい `pyqbpp.Expr` オブジェクトを返します。
- **メンバー関数**:
`pyqbpp.Expr` クラスのメソッドです。
多くの場合、呼び出し元のオブジェクトをその場で更新し、結果の `pyqbpp.Expr` も返します。

### 例: `sqr()`
`sqr()` 関数は式の二乗を計算します。
- `sqr(f)` (グローバル): `f` を変更せずに `f` の二乗を返します

## 型変換: **`int()`** と **`toInt()`**
Pythonの組み込み関数 **`int()`** は、変数を含まない定数 `Expr` をPythonの `int` に変換できます。
式に変数が含まれている場合は `ValueError` が発生します。

**`toInt()`** 関数はこれを `Vector` に拡張し、定数 `Expr` の `Vector` をPythonの `int` のネストされたリストに再帰的に変換します。

```python
import pyqbpp as qbpp

n = int(qbpp.Expr(42))   # 42
v = qbpp.Vector([qbpp.Expr(10), qbpp.Expr(20), qbpp.Expr(30)])
print(qbpp.toInt(v))     # [10, 20, 30]
```

> **NOTE**
> PyQBPPでは、整数や変数から `Expr` への明示的な型変換は不要です。
> Pythonの動的型付けが自動的に変換を処理します（例: `f = 1; f += x` で自動的に `Expr` が生成されます）。

## 代入
Pythonでは、`=` 演算子は変数名を新しいオブジェクトに再バインドします。
式をコピーするには、`Expr` コンストラクタを使用します。
```python
f = qbpp.Expr(g)  # f is a copy of g
```

## 二項演算子: `+`, `-`, `*`
これらの演算子は2つの `ExprType` オペランドを取り、結果を計算して返します。
少なくとも1つのオペランドが `pyqbpp.Expr` の場合、結果は常に `pyqbpp.Expr` になります。
どちらのオペランドも `pyqbpp.Expr` でない場合、結果は `pyqbpp.Term` になることがあります。

### 例
`pyqbpp.Var` 型の変数 `x` に対して:
- `2 + x`: `pyqbpp.Expr`
- `2 * x`: `pyqbpp.Term`

## 複合代入演算子: `+=`, `-=`, `*=`
左辺は `pyqbpp.Expr` でなければなりません。
右辺のオペランドを使って指定された演算が適用されます。
左辺の式はその場で更新されます。

> **NOTE**
> PyQBPPでは `*=` は `int` オペランドのみ受け付けます。

## 除算 `/` と複合除算 `/=`
除算演算子 `/` は **被除数** として `pyqbpp.Expr` を、**除数** として整数を取り、**商** を新しい `pyqbpp.Expr` として返します。

被除数の式は除数で割り切れなければなりません。すなわち、
式の整数定数項とすべての整数係数が除数で割り切れる必要があります。

複合除算演算子 `/=` は式をその場で除算します。

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 6 * x + 4 * y + 2
g = f / 2          # g = 3*x + 2*y + 1
f = qbpp.Expr(f)
f /= 2             # f = 3*x + 2*y + 1
```

## 比較（等価）: `==`
等価比較演算子 `==` は以下を取ります。
- 左辺に `pyqbpp.Expr`（またはそれを作成する `ExprType`）
- 右辺に整数

等価制約が満たされたときに最小値 0 となる式を返します。
より具体的には、`pyqbpp.Expr` オブジェクト `f` と整数 `n` に対して、演算子は `sqr(f - n)` を返します。

返されたオブジェクト `g` に対して:
- **`g`** は制約式 `sqr(f - n)` を表し、
- **`g.body`** は基礎となる式 `f` を返します。

### `pyqbpp.ExprExpr` クラス
ここで `g` は **`pyqbpp.ExprExpr`** オブジェクトで、`pyqbpp.Expr` の派生クラスです。
`body` プロパティは関連する基礎的な `pyqbpp.Expr` オブジェクトを返します。

### C++ QUBO++ との比較
C++ QUBO++では、`*g`（間接参照演算子）を使って基礎となる式にアクセスします。
PyQBPPでは、代わりに `g.body` プロパティを使用します。

## 比較（範囲）: `between()`
C++ QUBO++では、範囲比較は `l <= f <= u` と記述します。
PyQBPPでは、代わりに `between()` 関数を使用します。
```python
g = qbpp.between(f, l, u)
```
ここで:
- `f` は非整数の `ExprType`
- `l` と `u` は整数

この関数は、範囲制約 `l <= f <= u` が満たされたときに最小値 0 となる式を返します。

より具体的には、範囲 `[l, u-1]` の値を取る単位間隔の補助整数変数 `a` が暗黙的に導入され、関数は以下を返します。
```python
(f - a) * (f - (a + 1))
```

返された `pyqbpp.ExprExpr` オブジェクト `g` に対して:
- **`g`** は制約式 `(f - a) * (f - (a + 1))` を表し、
- **`g.body`** は基礎となる式 `f` を返します。

### C++ QUBO++ との比較

| C++ QUBO++       | PyQBPP            |
|------------------|---------------------|
| `l <= f <= u`    | `between(f, l, u)`  |
| `*g`             | `g.body`            |

## 二乗関数: `sqr()`
`pyqbpp.Expr` オブジェクト `f` に対して:
- **`pyqbpp.sqr(f)`** (グローバル関数): 式 `f * f` を返します。
引数 `f` は任意の `ExprType` オブジェクトです。

`pyqbpp.Vector` オブジェクト `v` に対して:
- **`pyqbpp.sqr(v)`**: 各要素を二乗した新しい `pyqbpp.Vector` を返します。

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = qbpp.sqr(x)       # x * x
```

## 最大公約数関数: `gcd()`
グローバル関数 **`pyqbpp.gcd()`** は `pyqbpp.Expr` オブジェクトを引数に取り、すべての整数係数と整数定数項の最大公約数（GCD）を返します。

与えられた式は結果のGCDで割り切れるため、GCDで割ることですべての整数係数と整数定数項を約分できます。

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 6 * x + 4 * y + 2
print(gcd(f))    # 2
g = f / gcd(f)   # 3*x + 2*y + 1
```

## 簡約化関数: `simplify()`, `simplify_as_binary()`, `simplify_as_spin()`
`pyqbpp.Expr` オブジェクト `f` に対して、メンバー関数 **`f.simplify()`** は以下の操作をその場で行います。
- 各項内の変数を一意な変数IDに従ってソート
- 重複する項をマージ
- 項を以下のようにソート:
  - 低次の項が先に配置される
  - 同次の項は辞書順で並べられる

グローバル関数 **`pyqbpp.simplify(f)`** は `f` を変更せずに同じ操作を行います。

### バイナリとスピンの簡約化
簡約化関数の2つの特殊なバリアントが提供されています。
- **`simplify_as_binary()`**:
すべての変数がバイナリ値 $\lbrace 0,1\rbrace$ を取ることを前提として簡約化を行います。
すべての変数 $x$ に対して恒等式 $x^2=x$ が適用されます。
- **`simplify_as_spin()`**:
すべての変数がスピン値 $\lbrace -1,+1\rbrace$ を取ることを前提として簡約化を行います。
すべての変数 $x$ に対して恒等式 $x^2=1$ が適用されます。

両方のバリアントはメンバー関数とグローバル関数として利用可能です。
- メンバー関数（その場で更新）: `f.simplify_as_binary()`, `f.simplify_as_spin()`
- グローバル関数（非破壊的）: `simplify_as_binary(f)`, `simplify_as_spin(f)`

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
f = qbpp.Expr(x * x + x)
f.simplify_as_binary()  # 2*x (since x^2 = x)

g = qbpp.Expr(x * x + x)
g.simplify_as_spin()    # 1 + x (since x^2 = 1)
```

## 評価関数: `f(ml)`
評価関数は `(変数, 値)` のペアのリストを受け取ります。各ペアは変数から整数値へのマッピングを定義します。

`pyqbpp.Expr` オブジェクト `f` とペアのリスト `ml` に対して、評価関数 `f(ml)` は `ml` で指定された変数の割り当ての下で `f` の値を評価し、結果の整数値を返します。

`f` に出現するすべての変数は、`ml` に対応するマッピングが定義されていなければなりません。

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 3 * x + 2 * y + 1

print(f([(x, 1), (y, 0)]))  # 4  (= 3*1 + 2*0 + 1)
```

## 置換関数: `replace()`
`replace()` 関数は `(変数, 値)` のペアのリストを受け取ります。各ペアは変数から整数または式へのマッピングを定義します。

`pyqbpp.Expr` オブジェクト `f` とペアのリスト `ml` に対して:
- **`pyqbpp.replace(f, ml)`** (グローバル関数):
`f` を変更せずに、`ml` のマッピングに従って `f` の変数を置換した新しい `pyqbpp.Expr` オブジェクトを返します。
- **`f.replace(ml)`** (メンバー関数):
`ml` のマッピングに従って `f` の変数をその場で置換し、結果の `pyqbpp.Expr` オブジェクトを返します。

### ペアのリストの作成
```python
import pyqbpp as qbpp

ml = [(x, 0), (y, 1)]                    # (変数, 値) のペアのリスト
ml = [(x, 0), (y, qbpp.Expr(z))]         # 値は式でもよい
```

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
f = 2 * x + 3 * y + 1

ml = [(x, 1), (y, 0)]
g = qbpp.replace(f, ml)   # g = 2*1 + 3*0 + 1 = 3 (new Expr)
f.replace(ml)         # f is modified in place
```

### C++ QUBO++ との比較

| C++ QUBO++                    | PyQBPP                          |
|-------------------------------|-----------------------------------|
| `qbpp::MapList ml;`           | `ml = []`                         |
| `ml.push_back({x, 0});`      | `ml.append((x, 0))`              |
| `qbpp::replace(f, ml)`       | `replace(f, ml)`                  |
| `f.replace(ml)`              | `f.replace(ml)`                   |

## 次数削減関数: `reduce()`
**`reduce()`** 関数は、高次の項を含む `pyqbpp.Expr` オブジェクトを、線形項と二次項のみからなる等価な `pyqbpp.Expr` オブジェクトに変換し、QUBO式を生成します。

`pyqbpp.Expr` オブジェクト `f` に対して:
- **`pyqbpp.reduce(f)`** (グローバル関数):
`f` と等価な線形項と二次項からなる新しい `pyqbpp.Expr` オブジェクトを返します。
- **`f.reduce()`** (メンバー関数):
`f` を削減された式でその場で置き換え、更新された式を返します。

### 例
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
z = qbpp.var("z")
f = qbpp.Expr(x * y * z)
f.simplify_as_binary()
g = reduce(f)   # Reduced to linear and quadratic terms
```

## バイナリ/スピン変換関数: `spin_to_binary()`, `binary_to_spin()`
`x` をバイナリ変数、`s` をスピン変数とします。
`x = 1` と `s = 1` が同値であると仮定します。
この仮定の下で、以下の関係が成り立ちます。

$$
\begin{aligned}
 s &= 2x-1 \\
 x &= (s+1)/2
\end{aligned}
$$

**`spin_to_binary()`** 関数は、すべてのスピン変数 `s` を `2 * s - 1` で置換することにより、スピン変数の式をバイナリ変数の式に変換します。

**`binary_to_spin()`** 関数は、すべてのバイナリ変数 `x` を `(x + 1) / 2` で置換することにより、バイナリ変数の式をスピン変数の式に変換します。
すべての係数が整数のままになるように、結果の式は $2^d$（$d$ は最大次数）で乗算されます。

両方の関数はメンバー関数（その場で更新）とグローバル関数（非破壊的）として利用可能です。

### 例
```python
import pyqbpp as qbpp

s = qbpp.var("s")
f = 3 * s + 1
g = spin_to_binary(f)   # -2 + 6*s  (replaced s with 2*s-1)

b = qbpp.var("b")
h = 2 * b + 1
k = binary_to_spin(h)   # 2 + 2*b  (replaced b with (b+1)/2, multiplied by 2)
```

### C++ QUBO++ との比較

| C++ QUBO++                      | PyQBPP                       |
|---------------------------------|--------------------------------|
| `qbpp::spin_to_binary(f)`      | `spin_to_binary(f)`            |
| `f.spin_to_binary()`           | `f.spin_to_binary()`           |
| `qbpp::binary_to_spin(f)`      | `binary_to_spin(f)`            |
| `f.binary_to_spin()`           | `f.binary_to_spin()`           |
</div>
