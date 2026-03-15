---
layout: default
nav_exclude: true
title: "QR: Operations"
nav_order: 31
---
<div class="lang-en" markdown="1">
# Quick Reference: Operators and Functions for Expressions

The table below summarizes the operators and functions available for `qbpp::Expr` objects.

| Operators/Functions           | Operator Symbols/Function Names                      | Function Type | Return Type    | Argument Type          |
|-------------------------------|------------------------------------------------------|---------------|----------------|------------------------|
| Type Conversion               | `toExpr()`                                             | Global        | `qbpp::Expr`     | `ExprType`               |
| Type Conversion               | `toInt()`                                              | Global        | `Int`            | `qbpp::Expr`                   |
| Assignment                    | `=`                                                  | Member        | `qbpp::Expr`     | `ExprType`               |
| Binary Operators              | `+`, `-`, `*`                                        | Global        | `qbpp::Expr`     | `ExprType`-`ExprType`      |
| Compound Assignment Operators | `+=`, `-=`, `*=`                                     | Member        | `qbpp::Expr`     | `ExprType`               |
| Division                      | `/`                                                  | Global        | `qbpp::Expr`     | `ExprType`-`Int`           |
| Compound Division             | `/=`                                                 | Member        | `qbpp::Expr`     | `Int`                    |
| Unary Operators               | `+`, `-`                                             | Global        | `qbpp::Expr`     | `ExprType`               |
| Comparison (Equality)         | `==`                                                 | Global        | `qbpp::ExprExpr` | `ExprType`-`Int`           |
| Comparison (Range Comparison) | `<= <=`                                              | Global        | `qbpp::ExprExpr` | `IntInf`-`ExprType`-`IntInf` |
| Square                        | `sqr()`                                                | Global        | `qbpp::Expr`     | `ExprType`               |
| Square                        | `sqr()`                                                | Member        | `qbpp::Expr`     | -                      |
| GCD                           | `gcd()`                                                | Global        | `Int`            | `ExprType`               |
| Simplify                      | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | Global        | `qbpp::Expr`     | `ExprType`               |
| Simplify                      | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | Member        | `qbpp::Expr`     | -                      |
| Eval                          | `operator()`                                           | Member        | `Int`            | `ExprType`-`qbpp::MapList`       |
| Replace                       | `replace()`                                            | Global        | `qbpp::Expr`     | `ExprType`-`qbpp::MapList`       |
| Replace                       | `replace()`                                            | Member        | `qbpp::Expr`     | `qbpp::MapList`                |
| Reduce                        | `reduce()`                                             | Global        | `qbpp::Expr`     | `ExprType`               |
| Reduce                        | `reduce()`                                             | Member        | `qbpp::Expr`     | `qbpp::MapList`                |
| Binary/Spin Conversion        | `binary_to_spin()`, `spin_to_binary()`                   | Global        | `qbpp::Expr`     | `ExprType`               |
| Binary/Spin Conversion        | `binary_to_spin()`, `spin_to_binary()`                   | Member        | `qbpp::Expr`     | -                      |

## Type Conversion: **`qbpp::toExpr()`** and **`qbpp::toInt()`**
The global function **`qbpp::toExpr()`** converts its argument into a `qbpp::Expr` instance and returns it.
The argument may be:
- an integer
- a variable (`qbpp::Var`)
- a product term (`qbpp::Term`)
- an expression (`qbpp::Expr`) — in this case, no conversion is performed

We refer to these argument types collectively as `ExprType`.

The global function **`qbpp::toInt()`** extracts and returns the integer constant term of a `qbpp::Expr` object.
If the expression contains any product terms (`qbpp::Term` objects), an error is thrown.


## Expression-related type: **`ExprType`**
The term **`ExprType`** denotes a category of types that can be converted to a `qbpp::Expr` object.

## Integer-Related Types: **`Int`** and **`IntInf`**
- **`Int`**: ordinary integers
- **`IntInf`**: either an integer, `-qbpp::inf`, or `+qbpp::inf`, representing infinite bounds.

## Global and Member Functions
Operators and functions related to `qbpp::Expr` are provided in two forms:
- **Global functions**:
These take at least one ExprType argument and typically return a new `qbpp::Expr` object without modifying the inputs.
- **Member functions**:
These are member functions of the `qbpp::Expr` class.
In many cases, they update the calling object and also return the resulting `qbpp::Expr`.

### Example: `sqr()`
The `sqr()` function computes the square of an expression and is available in both forms:
- `sqr(f)` (global): returns the square of f without modifying f
- `f.sqr()` (member): updates f to its square and returns the updated expression

## Assignment Operator: `=`
The left-hand side must be a `qbpp::Expr` object.
The right-hand side must be an `ExprType`, which is first converted to a `qbpp::Expr`.
The converted expression is then assigned to the left-hand side.

## Binary Operators: `+`, `-`, `*`
These operators are defined as global functions.
They take two `ExprType` operands, compute the result, and return it.
If at least one operand is a `qbpp::Expr`, the result is always a `qbpp::Expr`.
If neither operand is a `qbpp::Expr`, the result may be a `qbpp::Term`.

### Example
For a variable `x` of type `qbpp::Var`:
- `2 + x`: `qbpp::Expr`
- `2 * x`: `qbpp::Term`

## Compound Assignment Operators: `+=`, `-=`, `*=`
These operators are defined as member functions.
The left-hand side must be a `qbpp::Expr`.
The specified operation is applied using the right-hand side operand.
The left-hand side expression is updated in place.

## Division `/` and Compound Division `/=`
The division operator `/` is defined as a global function.

It takes a non-integer `ExprType` operand as the **dividend** and an integer operand as the **divisor**, and returns the **quotient** as a `qbpp::Expr`.

The dividend expression must be divisible by the divisor; that is,
both the integer constant term and all integer coefficients in the expression must be divisible by the divisor.

The compound division operator `/=` is defined as a member function.
- The left-hand side must be a `qbpp::Expr`.
- The right-hand side must be an integer.

The same divisibility condition applies, and the division is performed in place, updating the left-hand side expression.

## Comparison (Equality): `==`
The equality comparison operator `==` takes:
- a non-integer `ExprType` on the left-hand side, and
- an integer on the right-hand side.

It returns an expression whose minimum value is 0 when the equality constraint is satisfied.
More specifically, for a non-integer `ExprType` object `f` and an integer `n`, the operator returns: `qbpp::sqr(f-n)`.

For the returned object `g`:
- **`g`** represents the constraint expression `qbpp::sqr(f - n)`, and
- **`*g`** returns the underlying expression `f`.

### `qbpp::ExprExpr` class

Here, `g` is a **`qbpp::ExprExpr`** object, which is a derived class of `qbpp::Expr`.
Dereferencing `g` using the `*` operator returns the associated underlying qbpp::Expr object.

## Comparison (Range Comparison): `<= <=`
The **range comparison operator** is written in the form:
```cpp
l <= f <= u
```
where:
- `f` is a non-integer ExprType, and
- `l` and `u` are integers.

This operator returns an expression whose minimum value is 0 when the range constraint is satisfied.

More specifically, an auxiliary integer variable `a` with unit gaps, taking values in the range `[l,u−1]`, is implicitly introduced, and the operator returns:
```cpp
(f - a)(f - (a + 1))
```

For the returned `qbpp::ExprExpr` object `g`:
- **`g`** represents the constraint expression `(f - a)(f - (a + 1))`, and
- **`*g`** returns the underlying expression `f`.

## Square functions: `sqr()`
For a qbpp::Expr object `f`:
- **`qbpp::sqr(f)`** (global function): Returns the expression `f * f`.
The argument `f` may be a non-integer `ExprType` object.
- **`f.sqr()`** (member function):
Updates `f` in place by replacing it with `f * f`, and returns the updated expression.

## Greatest Common Divisor function `gcd()`
The global function **`gcd()`** takes a `qbpp::Expr` object as its argument and returns the greatest common divisor (GCD) of all integer coefficients and the integer constant term.

Since the given `qbpp::Expr` object is divisible by the resulting GCD, all integer coefficients and the integer constant term can be reduced by dividing by the GCD without changing the structure of the expression or its optimal solutions.


## Simplify functions: `simplify()`, `simplify_as_binary()`, `simplify_as_spin()`
For a `qbpp::Expr` object `f`, the member function **`f.simplify()`** performs the following operations in place:
- Sort variables within each term according to their unique variable IDs
- Merge duplicated terms
- Sort terms such that:
  - lower-degree terms appear earlier, and
  - terms of the same degree are ordered lexicographically.

The global function **`qbpp::simplify(f)`** performs the same operations without modifying `f`.

### Binary and Spin Simplification
Two specialized variants of the simplification function are provided:
- **`simplify_as_binary()`**:
Simplification is performed under the assumption that all variables take binary values
$\lbrace 0,1\rbrace$.
The identity $x^2=x$ is applied to all variables $x$.
- **`simplify_as_spin()`**
Simplification is performed under the assumption that all variables take spin values
$\lbrace -1,+1\rbrace$.
The identity $x^2=1$ is applied to all variables $x$.

Both variants are available as member functions and global functions:
- Member functions: These perform simplification in place and update `f`.
  - `f.simplify_as_binary()`
  - `f.simplify_as_spin()`
- Global functions: These return a simplified expression without modifying f.
  - `qbpp::simplify_as_binary(f)`
  - `qbpp::simplify_as_spin(f)`

## Evaluation function
A **`qbpp::MapList`** object stores a list of pairs consisting of a `qbpp::Var` object and an integer.
Each pair defines a mapping from a variable to an integer value.

For a `qbpp::Expr` object `f` and a `qbpp::MapList` object `ml`, the evaluation function `f(ml)` evaluates the value of `f` under the variable assignments specified by `ml` and returns the resulting integer value.

All variables appearing in `f` must have corresponding mappings defined in `ml`.

## Replace Functions: `replace()`
A **`qbpp::MapList`** object may also contain pairs consisting of a `qbpp::Var` object and an `ExprType` object.
Such pairs define mappings from variables to expressions.

For a `qbpp::Expr` object `f` and a `qbpp::MapList` object `ml`:
- **`qbpp::replace(f, ml)`**:
Returns a new `qbpp::Expr` object obtained by replacing variables in `f` according to the mappings in `ml`, without modifying `f`.
- **`f.replace(ml)`**:
Replaces variables in `f` according to the mappings in `ml` in place and returns the resulting `qbpp::Expr` object.

## Reduce Functions: `reduce()`
The **`reduce()`** function converts a `qbpp::Expr` object containing higher-degree terms into an equivalent `qbpp::Expr` object consisting only of linear and quadratic terms, resulting in a QUBO expression.

For a `qbpp::Expr` object `f`:
- **`qbpp::reduce(f)`**:
Returns a new `qbpp::Expr` object with linear and quadratic terms that is equivalent to `f`.
- **`f.reduce()`**:
Replaces `f` with the result of `qbpp::reduce(f)` and returns the updated expression.

## Binary/Spin Conversion functions: `spin_to_binary()`, `binary_to_spin()`
Let `x` be a binary variable and s be a spin variable.
We assume that `x = 1` if and only if `s = 1`.
Under this assumption, the following relations hold:

$$
\begin{aligned}
 s &= 2x-1 \\
 x &= (s+1)/2
\end{aligned}
$$

Let $f(s)$ be a function of a spin variable $s$.
Then the function $g(x)=f(2x−1)$ is a function of the binary variable $x$ that yields the same value under the above relation.

The **`spin_to_binary()`** function uses this relation to convert a `qbpp::Expr` object representing a function of spin variables into an equivalent `qbpp::Expr` object representing a function of binary variables.
More specifically, it replaces all spin variables `s` in `f` by `2 * s - 1`.
- **`qbpp::spin_to_binary(f)`**:
Produces and returns a new `qbpp::Expr` object by replacing all spin variables `s` in `f` with `2 * s - 1`.
- **`f.spin_to_binary()`**:
Updates `f` in place using `qbpp::spin_to_binary(f)` and returns the updated expression.

Similarly, the **`binary_to_spin()`** function replaces all binary variables `x` in `f` by `(x + 1) / 2`.
The resulting expression may contain non-integer coefficients.
Therefore, the entire expression is multiplied by
$2^d$ where $d$ is the maximum degree of all terms, so that all coefficients become integers.

As with `spin_to_binary()`, both global and member function variants of `binary_to_spin()` are provided.
</div>

<div class="lang-ja" markdown="1">
# クイックリファレンス: 式の演算子と関数

以下の表は、`qbpp::Expr`オブジェクトで利用可能な演算子と関数をまとめたものです。

| 演算子/関数                    | 演算子記号/関数名                                      | 関数タイプ     | 戻り値の型       | 引数の型                 |
|-------------------------------|------------------------------------------------------|---------------|----------------|------------------------|
| 型変換                         | `toExpr()`                                             | グローバル     | `qbpp::Expr`     | `ExprType`               |
| 型変換                         | `toInt()`                                              | グローバル     | `Int`            | `qbpp::Expr`                   |
| 代入                          | `=`                                                  | メンバ         | `qbpp::Expr`     | `ExprType`               |
| 二項演算子                     | `+`, `-`, `*`                                        | グローバル     | `qbpp::Expr`     | `ExprType`-`ExprType`      |
| 複合代入演算子                  | `+=`, `-=`, `*=`                                     | メンバ         | `qbpp::Expr`     | `ExprType`               |
| 除算                          | `/`                                                  | グローバル     | `qbpp::Expr`     | `ExprType`-`Int`           |
| 複合除算                       | `/=`                                                 | メンバ         | `qbpp::Expr`     | `Int`                    |
| 単項演算子                     | `+`, `-`                                             | グローバル     | `qbpp::Expr`     | `ExprType`               |
| 比較（等値）                    | `==`                                                 | グローバル     | `qbpp::ExprExpr` | `ExprType`-`Int`           |
| 比較（範囲比較）                | `<= <=`                                              | グローバル     | `qbpp::ExprExpr` | `IntInf`-`ExprType`-`IntInf` |
| 二乗                          | `sqr()`                                                | グローバル     | `qbpp::Expr`     | `ExprType`               |
| 二乗                          | `sqr()`                                                | メンバ         | `qbpp::Expr`     | -                      |
| 最大公約数                     | `gcd()`                                                | グローバル     | `Int`            | `ExprType`               |
| 簡約化                         | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | グローバル     | `qbpp::Expr`     | `ExprType`               |
| 簡約化                         | `simplify()`, `simplify_as_binary()`, `simplify_as_spin()` | メンバ         | `qbpp::Expr`     | -                      |
| 評価                          | `operator()`                                           | メンバ         | `Int`            | `ExprType`-`qbpp::MapList`       |
| 置換                          | `replace()`                                            | グローバル     | `qbpp::Expr`     | `ExprType`-`qbpp::MapList`       |
| 置換                          | `replace()`                                            | メンバ         | `qbpp::Expr`     | `qbpp::MapList`                |
| 次数削減                       | `reduce()`                                             | グローバル     | `qbpp::Expr`     | `ExprType`               |
| 次数削減                       | `reduce()`                                             | メンバ         | `qbpp::Expr`     | `qbpp::MapList`                |
| バイナリ/スピン変換              | `binary_to_spin()`, `spin_to_binary()`                   | グローバル     | `qbpp::Expr`     | `ExprType`               |
| バイナリ/スピン変換              | `binary_to_spin()`, `spin_to_binary()`                   | メンバ         | `qbpp::Expr`     | -                      |

## 型変換: **`qbpp::toExpr()`**と**`qbpp::toInt()`**
グローバル関数**`qbpp::toExpr()`**は引数を`qbpp::Expr`インスタンスに変換して返します。
引数は以下のいずれかです:
- 整数
- 変数（`qbpp::Var`）
- 積項（`qbpp::Term`）
- 式（`qbpp::Expr`）-- この場合、変換は行われません

これらの引数の型を総称して`ExprType`と呼びます。

グローバル関数**`qbpp::toInt()`**は`qbpp::Expr`オブジェクトの整数定数項を抽出して返します。
式に積項（`qbpp::Term`オブジェクト）が含まれている場合、エラーがスローされます。


## 式関連の型: **`ExprType`**
**`ExprType`**という用語は、`qbpp::Expr`オブジェクトに変換可能な型のカテゴリを示します。

## 整数関連の型: **`Int`**と**`IntInf`**
- **`Int`**: 通常の整数
- **`IntInf`**: 整数、`-qbpp::inf`、または`+qbpp::inf`のいずれかで、無限の境界を表します。

## グローバル関数とメンバ関数
`qbpp::Expr`に関連する演算子と関数は2つの形式で提供されます:
- **グローバル関数**:
少なくとも1つのExprType引数を取り、通常は入力を変更せずに新しい`qbpp::Expr`オブジェクトを返します。
- **メンバ関数**:
`qbpp::Expr`クラスのメンバ関数です。
多くの場合、呼び出し元のオブジェクトを更新し、結果の`qbpp::Expr`も返します。

### 例: `sqr()`
`sqr()`関数は式の二乗を計算し、両方の形式で利用できます:
- `sqr(f)`（グローバル）: fを変更せずにfの二乗を返します
- `f.sqr()`（メンバ）: fをその二乗に更新し、更新された式を返します

## 代入演算子: `=`
左辺は`qbpp::Expr`オブジェクトでなければなりません。
右辺は`ExprType`でなければならず、まず`qbpp::Expr`に変換されます。
変換された式が左辺に代入されます。

## 二項演算子: `+`, `-`, `*`
これらの演算子はグローバル関数として定義されています。
2つの`ExprType`オペランドを取り、結果を計算して返します。
少なくとも1つのオペランドが`qbpp::Expr`の場合、結果は常に`qbpp::Expr`になります。
どちらのオペランドも`qbpp::Expr`でない場合、結果は`qbpp::Term`になることがあります。

### 例
`qbpp::Var`型の変数`x`の場合:
- `2 + x`: `qbpp::Expr`
- `2 * x`: `qbpp::Term`

## 複合代入演算子: `+=`, `-=`, `*=`
これらの演算子はメンバ関数として定義されています。
左辺は`qbpp::Expr`でなければなりません。
右辺のオペランドを使用して指定された演算が適用されます。
左辺の式がその場で更新されます。

## 除算`/`と複合除算`/=`
除算演算子`/`はグローバル関数として定義されています。

非整数の`ExprType`オペランドを**被除数**として、整数オペランドを**除数**として取り、**商**を`qbpp::Expr`として返します。

被除数の式は除数で割り切れなければなりません。つまり、
式内の整数定数項とすべての整数係数が除数で割り切れる必要があります。

複合除算演算子`/=`はメンバ関数として定義されています。
- 左辺は`qbpp::Expr`でなければなりません。
- 右辺は整数でなければなりません。

同じ割り切れ条件が適用され、除算はその場で実行され、左辺の式が更新されます。

## 比較（等値）: `==`
等値比較演算子`==`は以下を取ります:
- 左辺に非整数の`ExprType`
- 右辺に整数

等値制約が満たされたときに最小値0となる式を返します。
より具体的には、非整数の`ExprType`オブジェクト`f`と整数`n`に対して、演算子は`qbpp::sqr(f-n)`を返します。

返されたオブジェクト`g`について:
- **`g`**は制約式`qbpp::sqr(f - n)`を表し、
- **`*g`**は基礎となる式`f`を返します。

### `qbpp::ExprExpr`クラス

ここで`g`は**`qbpp::ExprExpr`**オブジェクトであり、`qbpp::Expr`の派生クラスです。
`*`演算子を使用して`g`を間接参照すると、関連付けられた基礎となるqbpp::Exprオブジェクトが返されます。

## 比較（範囲比較）: `<= <=`
**範囲比較演算子**は次の形式で記述されます:
```cpp
l <= f <= u
```
ここで:
- `f`は非整数のExprType、
- `l`と`u`は整数です。

この演算子は、範囲制約が満たされたときに最小値0となる式を返します。

より具体的には、単位間隔を持つ補助整数変数`a`が範囲`[l,u-1]`の値を取るように暗黙的に導入され、演算子は以下を返します:
```cpp
(f - a)(f - (a + 1))
```

返された`qbpp::ExprExpr`オブジェクト`g`について:
- **`g`**は制約式`(f - a)(f - (a + 1))`を表し、
- **`*g`**は基礎となる式`f`を返します。

## 二乗関数: `sqr()`
qbpp::Exprオブジェクト`f`に対して:
- **`qbpp::sqr(f)`**（グローバル関数）: 式`f * f`を返します。
引数`f`は非整数の`ExprType`オブジェクトでもかまいません。
- **`f.sqr()`**（メンバ関数）:
`f`をその場で`f * f`に置き換え、更新された式を返します。

## 最大公約数関数`gcd()`
グローバル関数**`gcd()`**は`qbpp::Expr`オブジェクトを引数として取り、すべての整数係数と整数定数項の最大公約数（GCD）を返します。

与えられた`qbpp::Expr`オブジェクトは結果のGCDで割り切れるため、すべての整数係数と整数定数項をGCDで除算しても、式の構造や最適解は変わりません。


## 簡約化関数: `simplify()`, `simplify_as_binary()`, `simplify_as_spin()`
`qbpp::Expr`オブジェクト`f`に対して、メンバ関数**`f.simplify()`**は以下の操作をその場で実行します:
- 各項内の変数を一意な変数IDに従ってソート
- 重複する項をマージ
- 項を以下の規則でソート:
  - 低次の項が先に現れる
  - 同じ次数の項は辞書順に並べる

グローバル関数**`qbpp::simplify(f)`**は`f`を変更せずに同じ操作を実行します。

### バイナリとスピンの簡約化
簡約化関数の2つの特殊なバリアントが提供されています:
- **`simplify_as_binary()`**:
すべての変数がバイナリ値$\lbrace 0,1\rbrace$を取ることを仮定して簡約化が実行されます。
恒等式$x^2=x$がすべての変数$x$に適用されます。
- **`simplify_as_spin()`**
すべての変数がスピン値$\lbrace -1,+1\rbrace$を取ることを仮定して簡約化が実行されます。
恒等式$x^2=1$がすべての変数$x$に適用されます。

両方のバリアントはメンバ関数とグローバル関数として利用できます:
- メンバ関数: その場で簡約化を実行し、`f`を更新します。
  - `f.simplify_as_binary()`
  - `f.simplify_as_spin()`
- グローバル関数: fを変更せずに簡約化された式を返します。
  - `qbpp::simplify_as_binary(f)`
  - `qbpp::simplify_as_spin(f)`

## 評価関数
**`qbpp::MapList`**オブジェクトは、`qbpp::Var`オブジェクトと整数のペアのリストを格納します。
各ペアは変数から整数値へのマッピングを定義します。

`qbpp::Expr`オブジェクト`f`と`qbpp::MapList`オブジェクト`ml`に対して、評価関数`f(ml)`は`ml`で指定された変数割り当ての下で`f`の値を評価し、結果の整数値を返します。

`f`に現れるすべての変数は、`ml`に対応するマッピングが定義されていなければなりません。

## 置換関数: `replace()`
**`qbpp::MapList`**オブジェクトには、`qbpp::Var`オブジェクトと`ExprType`オブジェクトのペアも含めることができます。
このようなペアは変数から式へのマッピングを定義します。

`qbpp::Expr`オブジェクト`f`と`qbpp::MapList`オブジェクト`ml`に対して:
- **`qbpp::replace(f, ml)`**:
`f`を変更せずに、`ml`のマッピングに従って`f`内の変数を置換した新しい`qbpp::Expr`オブジェクトを返します。
- **`f.replace(ml)`**:
`ml`のマッピングに従って`f`内の変数をその場で置換し、結果の`qbpp::Expr`オブジェクトを返します。

## 次数削減関数: `reduce()`
**`reduce()`**関数は、高次の項を含む`qbpp::Expr`オブジェクトを、線形項と二次項のみで構成される等価な`qbpp::Expr`オブジェクトに変換し、QUBO式を生成します。

`qbpp::Expr`オブジェクト`f`に対して:
- **`qbpp::reduce(f)`**:
`f`と等価な線形項と二次項のみの新しい`qbpp::Expr`オブジェクトを返します。
- **`f.reduce()`**:
`f`を`qbpp::reduce(f)`の結果で置き換え、更新された式を返します。

## バイナリ/スピン変換関数: `spin_to_binary()`, `binary_to_spin()`
`x`をバイナリ変数、sをスピン変数とします。
`x = 1`であるとき、かつそのときに限り`s = 1`であると仮定します。
この仮定の下で、以下の関係が成り立ちます:

$$
\begin{aligned}
 s &= 2x-1 \\
 x &= (s+1)/2
\end{aligned}
$$

$f(s)$をスピン変数$s$の関数とします。
このとき、関数$g(x)=f(2x-1)$は上記の関係の下で同じ値を与えるバイナリ変数$x$の関数です。

**`spin_to_binary()`**関数はこの関係を使用して、スピン変数の関数を表す`qbpp::Expr`オブジェクトをバイナリ変数の関数を表す等価な`qbpp::Expr`オブジェクトに変換します。
具体的には、`f`内のすべてのスピン変数`s`を`2 * s - 1`に置換します。
- **`qbpp::spin_to_binary(f)`**:
`f`内のすべてのスピン変数`s`を`2 * s - 1`に置換した新しい`qbpp::Expr`オブジェクトを生成して返します。
- **`f.spin_to_binary()`**:
`qbpp::spin_to_binary(f)`を使用して`f`をその場で更新し、更新された式を返します。

同様に、**`binary_to_spin()`**関数は`f`内のすべてのバイナリ変数`x`を`(x + 1) / 2`に置換します。
結果の式には非整数の係数が含まれる場合があります。
そのため、すべての係数が整数になるように、式全体が$2^d$（$d$はすべての項の最大次数）で乗算されます。

`spin_to_binary()`と同様に、`binary_to_spin()`にもグローバル関数とメンバ関数の両方のバリアントが提供されています。
</div>
