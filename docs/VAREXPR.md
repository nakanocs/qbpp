---
layout: default
nav_exclude: true
title: "Variable and Expression Classes"
nav_order: 10
---

<div class="lang-en" markdown="1">

# Variable and Expression Classes

## qbpp::Var, qbpp::Term, and qbpp::Expr classes

QUBO++ provides the following fundamental classes:
- **`qbpp::Var`**: Represents a variable symbolically and is associated with a string used for display.
Internally, a 32-bit unsigned integer is used as its identifier.
- **`qbpp::Term`**: Represents a product term consisting of an integer coefficient and one or more `qbpp::Var` objects.
The data type of the integer coefficient is defined by the `COEFF_TYPE` macro, whose default value is `int32_t`.
- **`qbpp::Expr`**: Represents an expanded expression consisting of an integer constant term and zero or more `qbpp::Term` objects.
The data type of the integer constant term is defined by the `ENERGY_TYPE` macro, whose default value is `int64_t`.

In the following program, **`x`** and **`y`** are `qbpp::Var` objects, **`t`** is a `qbpp::Term` object, and **`f`** is a `qbpp::Expr` object:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto t = 2 * x * y;
  auto f = t - x + 1;

  std::cout << "x = " << x << std::endl;
  std::cout << "y = " << y << std::endl;
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
x = x
y = y
t = 2*x*y
f = 1 -x +2*x*y
```
If the data types are to be explicitly specified, the program can be rewritten as follows:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  qbpp::Var x = qbpp::var("x");
  qbpp::Var y = qbpp::var("y");
  qbpp::Term t = 2 * x * y;
  qbpp::Expr f = t - x + 1;

  std::cout << "x = " << x << std::endl;
  std::cout << "y = " << y << std::endl;
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
`qbpp::Var` objects are **immutable** and cannot be updated after creation.
In contrast, `qbpp::Term` and `qbpp::Expr` objects are **mutable** and can be updated via assignment.

For example, as shown in the following program, compound assignment operators can be used to update `qbpp::Term` and `qbpp::Expr` objects:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  qbpp::Var x = qbpp::var("x");
  qbpp::Var y = qbpp::var("y");
  qbpp::Term t = 2 * x * y;
  qbpp::Expr f = t - x + 1;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;

  t *= 3 * x;
  f += 2 * y;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
This program prints the following output:
```
t = 2*x*y
f = 1 -x +2*x*y
t = 6*x*y*x
f = 1 -x +2*x*y +2*y
```
In most cases, there is no need to explicitly use `qbpp::Term` objects.
They should only be used when maximum performance optimization is required.

However, note that `auto` type deduction may create a `qbpp::Term` object, which cannot store general expressions.
For example, the following program results in a compilation error because an expression is assigned to a `qbpp::Term` object:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");

  auto t = 2 * x * y;
  t = x + 1;
}
```
If a `qbpp::Expr` object is intended, **`qbpp::toExpr()`** can be used to explicitly construct one, as shown below:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto t = qbpp::toExpr(2 * x * y);
  auto f = qbpp::toExpr(1);

  t += x + 1;
  f += t;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
In this program, both **`t`** and **`f`** are `qbpp::Expr` objects and can store general expressions.
In particular, `f` is created as a `qbpp::Expr` object containing only a constant term with value `1` and no product terms.

## COEFF_TYPE and ENERGY_TYPE
The macros **`COEFF_TYPE`** and **`ENERGY_TYPE`** define the data types used for coefficients and energy values in expressions.
The `ENERGY_TYPE` macro is also used as the data type for the integer constant term of a `qbpp::Expr` object.
By default, `COEFF_TYPE` and `ENERGY_TYPE` are defined as **`int32_t`** and **`int64_t`**, respectively.
They can be changed either by compiler options or by using `#define` directives in the source code.

The following data types are supported:
- **Standard integer types**:
**`int8_t`**, **`int16_t`**, **`int32_t`**, and **`int64_t`**

- **Boost.Multiprecision integer types**:
**`qbpp::int128_t`**, **`qbpp::int256_t`**, **`qbpp::int512_t`**, **`qbpp::int1024_t`**, and **`qbpp::cpp_int`**

The type **`qbpp::cpp_int`** represents an integer with an arbitrary number of digits.
Constant values of this type can be specified using string literals.

For example, the following program creates a `qbpp::Expr` object with very large coefficient and constant terms:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto f = qbpp::cpp_int("123456789012345678901234567890") * x +
           qbpp::cpp_int("987654321098765432109876543210");
  std::cout << "f = " << f << std::endl;
}
```
This program produces the following output:
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```

</div>

<div class="lang-ja" markdown="1">

# 変数クラスと式クラス

## qbpp::Var、qbpp::Term、qbpp::Expr クラス

QUBO++は以下の基本クラスを提供します。
- **`qbpp::Var`**: 変数をシンボリックに表現し、表示用の文字列が関連付けられます。内部的には32ビット符号なし整数が識別子として使用されます。
- **`qbpp::Term`**: 整数係数と1つ以上の `qbpp::Var` オブジェクトからなる積の項を表現します。整数係数のデータ型は `COEFF_TYPE` マクロで定義され、デフォルト値は `int32_t` です。
- **`qbpp::Expr`**: 整数定数項と0個以上の `qbpp::Term` オブジェクトからなる展開された式を表現します。整数定数項のデータ型は `ENERGY_TYPE` マクロで定義され、デフォルト値は `int64_t` です。

以下のプログラムでは、**`x`** と **`y`** は `qbpp::Var` オブジェクト、**`t`** は `qbpp::Term` オブジェクト、**`f`** は `qbpp::Expr` オブジェクトです。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto t = 2 * x * y;
  auto f = t - x + 1;

  std::cout << "x = " << x << std::endl;
  std::cout << "y = " << y << std::endl;
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
このプログラムは以下の出力を生成します。
```
x = x
y = y
t = 2*x*y
f = 1 -x +2*x*y
```
データ型を明示的に指定する場合、プログラムは以下のように書き直せます。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  qbpp::Var x = qbpp::var("x");
  qbpp::Var y = qbpp::var("y");
  qbpp::Term t = 2 * x * y;
  qbpp::Expr f = t - x + 1;

  std::cout << "x = " << x << std::endl;
  std::cout << "y = " << y << std::endl;
  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
`qbpp::Var` オブジェクトは **不変（immutable）** であり、作成後に更新できません。
一方、`qbpp::Term` と `qbpp::Expr` オブジェクトは **可変（mutable）** であり、代入によって更新できます。

例えば、以下のプログラムに示すように、複合代入演算子を使用して `qbpp::Term` と `qbpp::Expr` オブジェクトを更新できます。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  qbpp::Var x = qbpp::var("x");
  qbpp::Var y = qbpp::var("y");
  qbpp::Term t = 2 * x * y;
  qbpp::Expr f = t - x + 1;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;

  t *= 3 * x;
  f += 2 * y;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
このプログラムは以下の出力を生成します。
```
t = 2*x*y
f = 1 -x +2*x*y
t = 6*x*y*x
f = 1 -x +2*x*y +2*y
```
ほとんどの場合、`qbpp::Term` オブジェクトを明示的に使用する必要はありません。
最大限のパフォーマンス最適化が必要な場合にのみ使用すべきです。

ただし、`auto` 型推論により `qbpp::Term` オブジェクトが作成される場合があり、一般的な式を格納できないことに注意してください。
例えば、以下のプログラムは、式が `qbpp::Term` オブジェクトに代入されるため、コンパイルエラーになります。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");

  auto t = 2 * x * y;
  t = x + 1;
}
```
`qbpp::Expr` オブジェクトを意図する場合、以下に示すように **`qbpp::toExpr()`** を使用して明示的に構築できます。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto t = qbpp::toExpr(2 * x * y);
  auto f = qbpp::toExpr(1);

  t += x + 1;
  f += t;

  std::cout << "t = " << t << std::endl;
  std::cout << "f = " << f << std::endl;
}
```
このプログラムでは、**`t`** と **`f`** の両方が `qbpp::Expr` オブジェクトであり、一般的な式を格納できます。
特に、`f` は値 `1` の定数項のみを持ち、積の項を持たない `qbpp::Expr` オブジェクトとして作成されます。

## COEFF_TYPE と ENERGY_TYPE
マクロ **`COEFF_TYPE`** と **`ENERGY_TYPE`** は、式内の係数とエネルギー値に使用されるデータ型を定義します。
`ENERGY_TYPE` マクロは、`qbpp::Expr` オブジェクトの整数定数項のデータ型としても使用されます。
デフォルトでは、`COEFF_TYPE` と `ENERGY_TYPE` はそれぞれ **`int32_t`** と **`int64_t`** として定義されています。
これらはコンパイラオプションまたはソースコード内の `#define` ディレクティブで変更できます。

以下のデータ型がサポートされています。
- **標準整数型**:
**`int8_t`**、**`int16_t`**、**`int32_t`**、**`int64_t`**

- **Boost.Multiprecision 整数型**:
**`qbpp::int128_t`**、**`qbpp::int256_t`**、**`qbpp::int512_t`**、**`qbpp::int1024_t`**、**`qbpp::cpp_int`**

型 **`qbpp::cpp_int`** は任意桁数の整数を表します。
この型の定数値は文字列リテラルを使用して指定できます。

例えば、以下のプログラムは非常に大きな係数と定数項を持つ `qbpp::Expr` オブジェクトを作成します。
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto x = qbpp::var("x");
  auto f = qbpp::cpp_int("123456789012345678901234567890") * x +
           qbpp::cpp_int("987654321098765432109876543210");
  std::cout << "f = " << f << std::endl;
}
```
このプログラムは以下の出力を生成します。
```
f = 987654321098765432109876543210 +123456789012345678901234567890*x
```

</div>
