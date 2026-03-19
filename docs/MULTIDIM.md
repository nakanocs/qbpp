---
layout: default
nav_exclude: true
title: "Multi-dimensional Variables"
nav_order: 13
---
<div class="lang-en" markdown="1">
# Multi-dimensional Variables and Expressions

## Defining multi-dimensional variables
QUBO++ supports **multi-dimensional variables** (or `qbpp::Var` objects) and **multi-dimensional integer variables** (or `qbpp::VarInt` objects) of arbitrary depth using the functions `qbpp::var()` and `qbpp::var_int()`, respectively.
Their basic usage is as follows:
- `qbpp::var("name",s1,s2,...,sd)`: Creates an array of `qbpp::Var` objects with the given `name` and shape $s1\times s2\times \cdots\times sd$.
- `l <= qbpp::var_int("name",s1,s2,...,sd) <= u`: Creates an array of `qbpp::VarInt` objects with the specified range and shape $s1\times s2\times \cdots\times sd$.

The following QUBO++ program creates a binary variable and an integer variable, each with dimension $2\times 3\times 4$.
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3, 4);
  auto y = 1 <= qbpp::var_int("y", 2, 3, 4) <= 8;
  std::cout << "x : " << x << std::endl;
  std::cout << "y : " << y << std::endl;
}
```
This program outputs the following:
{% raw %}
```
x : {{{x[0][0][0],x[0][0][1],x[0][0][2],x[0][0][3]},{x[0][1][0],x[0][1][1],x[0][1][2],x[0][1][3]},{x[0][2][0],x[0][2][1],x[0][2][2],x[0][2][3]}},{{x[1][0][0],x[1][0][1],x[1][0][2],x[1][0][3]},{x[1][1][0],x[1][1][1],x[1][1][2],x[1][1][3]},{x[1][2][0],x[1][2][1],x[1][2][2],x[1][2][3]}}}
y : {{{1 +y[0][0][0][0] +2*y[0][0][0][1] +4*y[0][0][0][2],1 +y[0][0][1][0] +2*y[0][0][1][1] +4*y[0][0][1][2],1 +y[0][0][2][0] +2*y[0][0][2][1] +4*y[0][0][2][2],1 +y[0][0][3][0] +2*y[0][0][3][1] +4*y[0][0][3][2]},{1 +y[0][1][0][0] +2*y[0][1][0][1] +4*y[0][1][0][2],1 +y[0][1][1][0] +2*y[0][1][1][1] +4*y[0][1][1][2],1 +y[0][1][2][0] +2*y[0][1][2][1] +4*y[0][1][2][2],1 +y[0][1][3][0] +2*y[0][1][3][1] +4*y[0][1][3][2]},{1 +y[0][2][0][0] +2*y[0][2][0][1] +4*y[0][2][0][2],1 +y[0][2][1][0] +2*y[0][2][1][1] +4*y[0][2][1][2],1 +y[0][2][2][0] +2*y[0][2][2][1] +4*y[0][2][2][2],1 +y[0][2][3][0] +2*y[0][2][3][1] +4*y[0][2][3][2]}},{{1 +y[1][0][0][0] +2*y[1][0][0][1] +4*y[1][0][0][2],1 +y[1][0][1][0] +2*y[1][0][1][1] +4*y[1][0][1][2],1 +y[1][0][2][0] +2*y[1][0][2][1] +4*y[1][0][2][2],1 +y[1][0][3][0] +2*y[1][0][3][1] +4*y[1][0][3][2]},{1 +y[1][1][0][0] +2*y[1][1][0][1] +4*y[1][1][0][2],1 +y[1][1][1][0] +2*y[1][1][1][1] +4*y[1][1][1][2],1 +y[1][1][2][0] +2*y[1][1][2][1] +4*y[1][1][2][2],1 +y[1][1][3][0] +2*y[1][1][3][1] +4*y[1][1][3][2]},{1 +y[1][2][0][0] +2*y[1][2][0][1] +4*y[1][2][0][2],1 +y[1][2][1][0] +2*y[1][2][1][1] +4*y[1][2][1][2],1 +y[1][2][2][0] +2*y[1][2][2][1] +4*y[1][2][2][2],1 +y[1][2][3][0] +2*y[1][2][3][1] +4*y[1][2][3][2]}}}
```
{% endraw %}
Each `qbpp::Var` object in **`x`** can be accessed as **`x[i][j][k]`**.
Each `qbpp::VarInt` object in **`y`** can be accessed as **`y[i][j][k]`**,
which is internally represented by three binary variables:
- **`y[i][j][k][0]`**
- **`y[i][j][k][1]`**
- **`y[i][j][k][2]`**

corresponding to the binary encoding of integers in the specified range.

## Creating integer variable arrays with individual ranges

When defining a multi-dimensional array of integer variables, all elements created by `l <= qbpp::var_int("name", s1, s2, ...) <= u` share the same range $[l, u]$.
In many practical problems, however, each element may need a different range.

In such cases, you can first create a **placeholder array** using **`qbpp::var_int("name", s1, s2, ...) == 0`**, and then assign individual ranges to each element in a loop:

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  qbpp::Vector<int> max_vals = {3, 7, 15, 5};
  auto x = qbpp::var_int("x", max_vals.size()) == 0;
  for (size_t i = 0; i < max_vals.size(); ++i) {
    x[i] = 0 <= qbpp::var_int() <= max_vals[i];
  }
  for (size_t i = 0; i < max_vals.size(); ++i) {
    std::cout << "x[" << i << "] = " << x[i] << std::endl;
  }
}
```
In this program, `qbpp::var_int("x", 4) == 0` creates an array of 4 constant-zero `VarInt` objects as placeholders.
Each element is then reassigned with its own range using `0 <= qbpp::var_int() <= max_vals[i]`.

This technique is commonly used in problems such as the **[Cutting Stock Problem](BAR_CUTTING)**, where the upper bound of each variable differs.

> **NOTE**
> The `== 0` syntax creates a `VarInt` with `min_val = max_val = 0` (i.e., a constant zero).
> It does **not** create an equality constraint.
> Any integer constant can be used, e.g., `qbpp::var_int("x", 4) == 5` creates constant-five placeholders.

## Defining multi-dimensional expressions
QUBO++ allows you to define **multi-dimensional expressions** (or `qbpp::Expr` objects) with arbitrary depth using the function `qbpp::expr()` as follows:
- **`qbpp::expr(s1,s2,...,sd)`**: Creates a multi-dimensional array of `qbpp::Expr` objects with shape $s1\times s2\times \cdots\times sd$.

The following program defines a 3-dimensional array **`x`** of `qbpp::Var` objects with shape $2\times 3\times 4$ and
a 2-dimensional array `f` of  size $2\times 3$.
Then, using a triple for-loop, each `f[i][j]` accumulates the sum of `x[i][j][0]`, `x[i][j][1]`, `x[i][j][2]`, and `x[i][j][3]`:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3, 4);
  auto f = qbpp::expr(2, 3);
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      for (size_t k = 0; k < 4; ++k) {
        f[i][j] += x[i][j][k];
      }
    }
  }
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
Note that the `simplify_as_binary()` member function can be applied to a multi-dimensional array of `qbpp::Expr` objects.
When called on such an array, it applies `simplify_as_binary()` to each element individually (element-wise).

This program produces the following output:
{% raw %}
```
f = {{x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3],x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3],x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]},{x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3],x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3],x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]}}
f[0][0] = x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3]
f[0][1] = x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3]
f[0][2] = x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]
f[1][0] = x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3]
f[1][1] = x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3]
f[1][2] = x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]
```
{% endraw %}
## Creating an array of expressions by auto type deduction
An array of **`qbpp::Expr`** objects can be created without explicitly calling `qbpp::expr()`.
When a function call or an arithmetic operation yields an array-shaped result, an array of `qbpp::Expr` objects with the same shape can be defined using auto type deduction.

The following QUBO++ program creates an array **`f`** of `qbpp::Expr` objects with the same dimensions as an array **`x`** of `qbpp::Var` objects:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3);
  auto f = x + 1;
  f += x - 2;
  f.simplify_as_binary();
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
In this program, `x` is defined as a $2 \times 3$ array of `qbpp::Var` objects.
The expression `x + 1` produces a $2 \times 3$ array of `qbpp::Expr` objects, which is used to initialize `f` via auto type deduction.
After that, the expression `x - 2` is added element-wise to `f`.

This program outputs:
```
f[0][0] = -1 +2*x[0][0]
f[0][1] = -1 +2*x[0][1]
f[0][2] = -1 +2*x[0][2]
f[1][0] = -1 +2*x[1][0]
f[1][1] = -1 +2*x[1][1]
f[1][2] = -1 +2*x[1][2]
```

## Implementation of vecvtors and arrays
QUBO++ implements vectors (i.e., one-dimensional arrays) as **`qbpp::Vector<T>`** objects, which are largely compatible with `std::vector<T>`.
The template parameter `T` can be `qbpp::Expr`, `qbpp::Var`, or an integer type.

The class `qbpp::Vector<T>` provides the following member functions for compatibility with `std::vector<T>`:
- **`size()`**: Returns the number of elements in the vector.
- **`resize()`**: Changes the number of elements in the vector.
- **`reserve()`**: Reserves memory space for the vector.
- **`push_back()`**: Appends an element to the end of the vector.
- **`emplace_back()`**: Constructs and appends an element to the end of the vector.
- **`empty()`**: Returns true if the vector contains no elements.
- **`operator[]`**: Returns the element at the specified index.
- **`begin()`**, **`end()`**: Iterators for accessing and manipulating elements.

In addition, unlike `std::vector<T>`, **`qbpp::Vector<T>`** supports the following operators for element-wise operations:
- **`+`**: Element-wise addition of two vectors, or a vector and a scalar.
- **`-`**: Element-wise subtraction of two vectors, or a vector and a scalar.
- **`*`**: Element-wise multiplication of two vectors, or a vector and a scalar.
- unary **`-`**: Negates all elements in the vector.

Furthermore, multi-dimensional arrays are implemented as nested instances of `qbpp::Vector<T>`.
For example, the data type of `x` in the following code is **`qbpp::Vector<qbpp::Vector<qbpp::Var>>`**:
```cpp
  auto x = qbpp::var("x", 2, 3);
```
The element-wise operations described above are supported for multi-dimensional arrays only when the operands have the same shape.

Since `qbpp::Vector<T>` supports iterators, range-based `for` loops with `auto` type deduction can be used as follows:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3);
  auto f = x + 1;
  f += x - 2;
  f.simplify_as_binary();
  for (const auto& vec : f) {
    for (const auto& element : vec) {
      std::cout << "(" << element << ")";
    }
    std::cout << std::endl;
  }
}
```
In the outer `for` loop, each `qbpp::Vector<qbpp::Expr>` object contained in the `qbpp::Vector<qbpp::Vector<qbpp::Expr>>` object `f` is referenced in turn by `vec`.
In the inner `for` loop, each `qbpp::Expr` object contained in `vec` is referenced in turn by `element`.

This program outputs:
```
(-1 +2*x[0][0])(-1 +2*x[0][1])(-1 +2*x[0][2])
(-1 +2*x[1][0])(-1 +2*x[1][1])(-1 +2*x[1][2])
```
</div>

<div class="lang-ja" markdown="1">
# 多次元変数と式

## 多次元変数の定義
QUBO++は、関数`qbpp::var()`および`qbpp::var_int()`を使用して、任意の深さの**多次元変数**（`qbpp::Var`オブジェクト）および**多次元整数変数**（`qbpp::VarInt`オブジェクト）をサポートしています。
基本的な使い方は次のとおりです:
- `qbpp::var("name",s1,s2,...,sd)`: 指定された`name`と形状$s1\times s2\times \cdots\times sd$を持つ`qbpp::Var`オブジェクトの配列を作成します。
- `l <= qbpp::var_int("name",s1,s2,...,sd) <= u`: 指定された範囲と形状$s1\times s2\times \cdots\times sd$を持つ`qbpp::VarInt`オブジェクトの配列を作成します。

以下のQUBO++プログラムは、それぞれ次元$2\times 3\times 4$のバイナリ変数と整数変数を作成します。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3, 4);
  auto y = 1 <= qbpp::var_int("y", 2, 3, 4) <= 8;
  std::cout << "x : " << x << std::endl;
  std::cout << "y : " << y << std::endl;
}
```
このプログラムの出力は次のとおりです:
{% raw %}
```
x : {{{x[0][0][0],x[0][0][1],x[0][0][2],x[0][0][3]},{x[0][1][0],x[0][1][1],x[0][1][2],x[0][1][3]},{x[0][2][0],x[0][2][1],x[0][2][2],x[0][2][3]}},{{x[1][0][0],x[1][0][1],x[1][0][2],x[1][0][3]},{x[1][1][0],x[1][1][1],x[1][1][2],x[1][1][3]},{x[1][2][0],x[1][2][1],x[1][2][2],x[1][2][3]}}}
y : {{{1 +y[0][0][0][0] +2*y[0][0][0][1] +4*y[0][0][0][2],1 +y[0][0][1][0] +2*y[0][0][1][1] +4*y[0][0][1][2],1 +y[0][0][2][0] +2*y[0][0][2][1] +4*y[0][0][2][2],1 +y[0][0][3][0] +2*y[0][0][3][1] +4*y[0][0][3][2]},{1 +y[0][1][0][0] +2*y[0][1][0][1] +4*y[0][1][0][2],1 +y[0][1][1][0] +2*y[0][1][1][1] +4*y[0][1][1][2],1 +y[0][1][2][0] +2*y[0][1][2][1] +4*y[0][1][2][2],1 +y[0][1][3][0] +2*y[0][1][3][1] +4*y[0][1][3][2]},{1 +y[0][2][0][0] +2*y[0][2][0][1] +4*y[0][2][0][2],1 +y[0][2][1][0] +2*y[0][2][1][1] +4*y[0][2][1][2],1 +y[0][2][2][0] +2*y[0][2][2][1] +4*y[0][2][2][2],1 +y[0][2][3][0] +2*y[0][2][3][1] +4*y[0][2][3][2]}},{{1 +y[1][0][0][0] +2*y[1][0][0][1] +4*y[1][0][0][2],1 +y[1][0][1][0] +2*y[1][0][1][1] +4*y[1][0][1][2],1 +y[1][0][2][0] +2*y[1][0][2][1] +4*y[1][0][2][2],1 +y[1][0][3][0] +2*y[1][0][3][1] +4*y[1][0][3][2]},{1 +y[1][1][0][0] +2*y[1][1][0][1] +4*y[1][1][0][2],1 +y[1][1][1][0] +2*y[1][1][1][1] +4*y[1][1][1][2],1 +y[1][1][2][0] +2*y[1][1][2][1] +4*y[1][1][2][2],1 +y[1][1][3][0] +2*y[1][1][3][1] +4*y[1][1][3][2]},{1 +y[1][2][0][0] +2*y[1][2][0][1] +4*y[1][2][0][2],1 +y[1][2][1][0] +2*y[1][2][1][1] +4*y[1][2][1][2],1 +y[1][2][2][0] +2*y[1][2][2][1] +4*y[1][2][2][2],1 +y[1][2][3][0] +2*y[1][2][3][1] +4*y[1][2][3][2]}}}
```
{% endraw %}
**`x`**内の各`qbpp::Var`オブジェクトは**`x[i][j][k]`**としてアクセスできます。
**`y`**内の各`qbpp::VarInt`オブジェクトは**`y[i][j][k]`**としてアクセスでき、
内部的には以下の3つのバイナリ変数で表現されます:
- **`y[i][j][k][0]`**
- **`y[i][j][k][1]`**
- **`y[i][j][k][2]`**

これらは指定された範囲の整数のバイナリエンコーディングに対応しています。

## 個別の範囲を持つ整数変数配列の作成

多次元整数変数配列を定義する場合、`l <= qbpp::var_int("name", s1, s2, ...) <= u` で作成された全要素は同じ範囲 $[l, u]$ を共有します。
しかし実際の問題では、各要素に異なる範囲が必要な場合が多くあります。

このような場合、まず **`qbpp::var_int("name", s1, s2, ...) == 0`** で**プレースホルダ配列**を作成し、ループ内で各要素に個別の範囲を割り当てます:

```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  qbpp::Vector<int> max_vals = {3, 7, 15, 5};
  auto x = qbpp::var_int("x", max_vals.size()) == 0;
  for (size_t i = 0; i < max_vals.size(); ++i) {
    x[i] = 0 <= qbpp::var_int() <= max_vals[i];
  }
  for (size_t i = 0; i < max_vals.size(); ++i) {
    std::cout << "x[" << i << "] = " << x[i] << std::endl;
  }
}
```
このプログラムでは、`qbpp::var_int("x", 4) == 0` がプレースホルダとして定数ゼロの `VarInt` オブジェクト4個の配列を作成します。
各要素は `0 <= qbpp::var_int() <= max_vals[i]` で個別の範囲に再代入されます。

このテクニックは、各変数の上限が異なる**[切り出し問題](BAR_CUTTING)**などでよく使われます。

> **NOTE**
> `== 0` の構文は `min_val = max_val = 0`（定数ゼロ）の `VarInt` を作成するものであり、等号制約を作成するものでは**ありません**。
> 任意の整数定数を使用でき、例えば `qbpp::var_int("x", 4) == 5` は定数5のプレースホルダを作成します。

## 多次元式の定義
QUBO++では、関数`qbpp::expr()`を使用して任意の深さの**多次元式**（`qbpp::Expr`オブジェクト）を定義できます:
- **`qbpp::expr(s1,s2,...,sd)`**: 形状$s1\times s2\times \cdots\times sd$の`qbpp::Expr`オブジェクトの多次元配列を作成します。

以下のプログラムは、形状$2\times 3\times 4$の`qbpp::Var`オブジェクトの3次元配列**`x`**と、サイズ$2\times 3$の2次元配列`f`を定義します。
次に、3重forループを使用して、各`f[i][j]`に`x[i][j][0]`、`x[i][j][1]`、`x[i][j][2]`、`x[i][j][3]`の和を蓄積します:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3, 4);
  auto f = qbpp::expr(2, 3);
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      for (size_t k = 0; k < 4; ++k) {
        f[i][j] += x[i][j][k];
      }
    }
  }
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
`simplify_as_binary()`メンバ関数は`qbpp::Expr`オブジェクトの多次元配列に対しても適用できます。
このような配列に対して呼び出された場合、各要素に対して個別に（要素ごとに）`simplify_as_binary()`が適用されます。

このプログラムの出力は次のとおりです:
{% raw %}
```
f = {{x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3],x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3],x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]},{x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3],x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3],x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]}}
f[0][0] = x[0][0][0] +x[0][0][1] +x[0][0][2] +x[0][0][3]
f[0][1] = x[0][1][0] +x[0][1][1] +x[0][1][2] +x[0][1][3]
f[0][2] = x[0][2][0] +x[0][2][1] +x[0][2][2] +x[0][2][3]
f[1][0] = x[1][0][0] +x[1][0][1] +x[1][0][2] +x[1][0][3]
f[1][1] = x[1][1][0] +x[1][1][1] +x[1][1][2] +x[1][1][3]
f[1][2] = x[1][2][0] +x[1][2][1] +x[1][2][2] +x[1][2][3]
```
{% endraw %}
## auto型推論による式配列の作成
**`qbpp::Expr`**オブジェクトの配列は、`qbpp::expr()`を明示的に呼び出さずに作成できます。
関数呼び出しや算術演算が配列形状の結果を返す場合、auto型推論を使用して同じ形状の`qbpp::Expr`オブジェクトの配列を定義できます。

以下のQUBO++プログラムは、`qbpp::Var`オブジェクトの配列**`x`**と同じ次元の`qbpp::Expr`オブジェクトの配列**`f`**を作成します:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3);
  auto f = x + 1;
  f += x - 2;
  f.simplify_as_binary();
  for (size_t i = 0; i < 2; ++i) {
    for (size_t j = 0; j < 3; ++j) {
      std::cout << "f[" << i << "][" << j << "] = " << f[i][j] << std::endl;
    }
  }
}
```
このプログラムでは、`x`は$2 \times 3$の`qbpp::Var`オブジェクトの配列として定義されています。
式`x + 1`は$2 \times 3$の`qbpp::Expr`オブジェクトの配列を生成し、auto型推論を通じて`f`の初期化に使用されます。
その後、式`x - 2`が`f`に要素ごとに加算されます。

このプログラムの出力は次のとおりです:
```
f[0][0] = -1 +2*x[0][0]
f[0][1] = -1 +2*x[0][1]
f[0][2] = -1 +2*x[0][2]
f[1][0] = -1 +2*x[1][0]
f[1][1] = -1 +2*x[1][1]
f[1][2] = -1 +2*x[1][2]
```

## ベクトルと配列の実装
QUBO++はベクトル（すなわち1次元配列）を**`qbpp::Vector<T>`**オブジェクトとして実装しており、`std::vector<T>`と高い互換性を持っています。
テンプレートパラメータ`T`には`qbpp::Expr`、`qbpp::Var`、または整数型を指定できます。

`qbpp::Vector<T>`クラスは、`std::vector<T>`との互換性のために以下のメンバ関数を提供しています:
- **`size()`**: ベクトル内の要素数を返します。
- **`resize()`**: ベクトル内の要素数を変更します。
- **`reserve()`**: ベクトルのメモリ領域を予約します。
- **`push_back()`**: ベクトルの末尾に要素を追加します。
- **`emplace_back()`**: ベクトルの末尾に要素を構築して追加します。
- **`empty()`**: ベクトルが要素を含まない場合にtrueを返します。
- **`operator[]`**: 指定されたインデックスの要素を返します。
- **`begin()`**、**`end()`**: 要素のアクセスと操作のためのイテレータ。

さらに、`std::vector<T>`とは異なり、**`qbpp::Vector<T>`**は要素ごとの演算のために以下の演算子をサポートしています:
- **`+`**: 2つのベクトル、またはベクトルとスカラーの要素ごとの加算。
- **`-`**: 2つのベクトル、またはベクトルとスカラーの要素ごとの減算。
- **`*`**: 2つのベクトル、またはベクトルとスカラーの要素ごとの乗算。
- 単項**`-`**: ベクトル内のすべての要素を否定します。

さらに、多次元配列は`qbpp::Vector<T>`のネストされたインスタンスとして実装されています。
例えば、以下のコードにおける`x`のデータ型は**`qbpp::Vector<qbpp::Vector<qbpp::Var>>`**です:
```cpp
  auto x = qbpp::var("x", 2, 3);
```
上記の要素ごとの演算は、オペランドが同じ形状を持つ場合にのみ多次元配列でサポートされます。

`qbpp::Vector<T>`はイテレータをサポートしているため、auto型推論を用いた範囲ベースの`for`ループを次のように使用できます:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x", 2, 3);
  auto f = x + 1;
  f += x - 2;
  f.simplify_as_binary();
  for (const auto& vec : f) {
    for (const auto& element : vec) {
      std::cout << "(" << element << ")";
    }
    std::cout << std::endl;
  }
}
```
外側の`for`ループでは、`qbpp::Vector<qbpp::Vector<qbpp::Expr>>`オブジェクト`f`に含まれる各`qbpp::Vector<qbpp::Expr>`オブジェクトが順に`vec`で参照されます。
内側の`for`ループでは、`vec`に含まれる各`qbpp::Expr`オブジェクトが順に`element`で参照されます。

このプログラムの出力は次のとおりです:
```
(-1 +2*x[0][0])(-1 +2*x[0][1])(-1 +2*x[0][2])
(-1 +2*x[1][0])(-1 +2*x[1][1])(-1 +2*x[1][2])
```
</div>
