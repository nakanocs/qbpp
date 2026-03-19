---
layout: default
nav_exclude: true
title: "Variables and Expressions"
nav_order: 1
---

<div class="lang-en" markdown="1">

# Defining Variables and Expressions

## Header file and namespace
To use QUBO++, you need to include the header file **`qbpp/qbpp.hpp`** and use the **`qbpp`** namespace.

## Maximum degree of terms (`MAXDEG`)
Before including `qbpp/qbpp.hpp`, you must define the macro **`MAXDEG`** to specify the maximum degree (number of variables) allowed in each term.
For QUBO problems (degree 2), set `#define MAXDEG 2`.
For higher-order problems (HUBO), use a larger value such as `#define MAXDEG 4`.

This macro controls the internal storage of terms and enables compile-time optimization.
For more details on `MAXDEG` and the alternative `BASEDEG` macro, see **[Reference: Variables](QR_VARIABLE)**.

## Defining variables and expressions
You can define a variable using **`qbpp::var("name")`** with auto type deduction.
The specified `name` is used when the variable is printed with `std::cout`.

Expressions are constructed using standard arithmetic operators such as **`+`**, **`-`**, and **`*`**.

The following sample program defines three variables `a`, `b`, and `c`, and an expression `f`, which is printed using `std::cout`:
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (a + b - 1) * (b + c - 1);
  std::cout << "f = " << f << std::endl;
}
```
The expression `(a + b - 1) * (b + c - 1)` is automatically expanded and stored in `f`.

In this QUBO++ program, the variables `a`, `b`, and `c` are objects of class **`qbpp::Var`**, and the expression `f` is an object of class **`qbpp::Expr`**.

Assuming the header and library paths are properly set up, this program (saved as **`test.cpp`**) can be compiled with `g++` as follows:
```bash
g++ test.cpp -o test -std=c++17 -lqbpp -ltbb
```
Running the executable prints the expanded expression:
```bash
./test
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

> **NOTE**
> The variable name in `qbpp::var()` may be omitted.
> If omitted, a default name such as `{0}`, `{1}`,... is automatically assigned.

> **WARNING**
> Most QUBO++ class instances, such as `qbpp::Expr`, can be printed as text using `std::cout`.
> However, this textual output is not guaranteed to be stable and should not be used as input for subsequent computations, since its format may change in future releases.
> In addition, the output shown in the QUBO++ documentation may have been generated with an older version of QUBO++, so the output produced by the latest version may differ.

## Simplifying expression
The expression stored in a **`qbpp::Expr`** object can be simplified by calling the **`simplify()`** member function:
```cpp
  std::cout << "f = " << f.simplify() << std::endl;
```
With this change, the output of the program becomes:
```
f.simplify() = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
The member function call **`f.simplify()`** simplifies the expression `f` and returns the resulting value,
which is then printed by `std::cout`.

Assuming that all variables take **binary values (0 or 1)**, we can use the identity
**$b^2=b$** to further simplify the expression.
For this purpose, we use **`simplify_as_binary()`** instead:
```cpp
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
```
Then the output becomes:
```
f = 1 -a -b -c +a*b +a*c +b*c
```

The simplify functions reorder the variables within each term and the terms within the expression so that lower-degree terms appear first, and terms of the same degree are sorted in the lexicographical order of their variables.
The variables themselves are ordered according to the order in which they were defined.

## Simplifying expressions with spin variables
If variables are assumed to take **spin values $-1$/$+1$**, the identity **$b^2 = 1$** can be used to further simplify the expression.
In this case, the expression can be simplified using the **`simplify_as_spin()`** member function:
```cpp
  std::cout << "f = " << f.simplify_as_spin() << std::endl;
```
Then the output becomes:
```
f = 2 -a -2*b -c +a*b +a*c +b*c
```

## Global functions for simplification
Member functions update the expression stored in `f`.
If you do not want to modify `f`, you can instead use the global functions
**`qbpp::simplify(f)`**, **`qbpp::simplify_as_binary(f)`**, and **`qbpp::simplify_as_spin(f)`**, which return the simplified expressions without changing `f`.

> **NOTE**
> In QUBO++, most **member functions** update the object in place when possible, whereas **global functions** return a new value without modifying the original object.

## Negated literals
QUBO++ natively supports **negated literals** using the `~` operator.
For a binary variable `x`, the expression `~x` represents $1 - x$.

```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = ~a * ~b * ~c * ~d + a * b;
  std::cout << "f = " << f << std::endl;
  std::cout << "f = " << qbpp::simplify_as_binary(f) << std::endl;
}
```
Output:
```
f = ~a*~b*~c*~d +a*b
f = a*b +~a*~b*~c*~d
```

The negated literal `~x` is stored internally as a single variable with a negation flag, **not** expanded as `1 - x`.
This is important for performance: if `~x` were naively expanded, a product of $k$ negated literals such as `~x1 * ~x2 * ... * ~xk` would produce up to $2^k$ terms after expanding $(1-x_1)(1-x_2)\cdots(1-x_k)$.
For example, the term `~a*~b*~c*~d` above is stored as a single quartic term, whereas its expanded form $(1-a)(1-b)(1-c)(1-d)$ produces 16 terms:
```
1 -a -b -c -d +a*b +a*c +a*d +b*c +b*d +c*d -a*b*c -a*b*d -a*c*d -b*c*d +a*b*c*d
```

All solvers bundled with QUBO++ (EasySolver, ExhaustiveSolver, ABS3 GPU Solver) handle negated literals natively, so it is not necessary to expand them before solving.

</div>

<div class="lang-ja" markdown="1">

# 変数と式の定義

## ヘッダファイルと名前空間
QUBO++を使用するには、ヘッダファイル **`qbpp/qbpp.hpp`** をインクルードし、**`qbpp`** 名前空間を使用します。

## 項の最大次数（`MAXDEG`）
`qbpp/qbpp.hpp` をインクルードする前に、マクロ **`MAXDEG`** を定義して各項に許される最大次数（変数の数）を指定する必要があります。
QUBO問題（次数2）の場合は `#define MAXDEG 2` と設定します。
高次の問題（HUBO）の場合は `#define MAXDEG 4` のようにより大きな値を使用します。

このマクロは項の内部ストレージを制御し、コンパイル時の最適化を可能にします。
`MAXDEG` および代替の `BASEDEG` マクロの詳細については**[リファレンス: 変数](QR_VARIABLE)**を参照してください。

## 変数と式の定義
変数は **`qbpp::var("name")`** を `auto` 型推論とともに使用して定義できます。
指定した `name` は、変数を `std::cout` で出力する際に使用されます。

式は **`+`**、**`-`**、**`*`** などの標準的な算術演算子を使って構築します。

以下のサンプルプログラムでは、3つの変数 `a`、`b`、`c` と式 `f` を定義し、`std::cout` を使って出力しています。
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (a + b - 1) * (b + c - 1);
  std::cout << "f = " << f << std::endl;
}
```
式 `(a + b - 1) * (b + c - 1)` は自動的に展開され、`f` に格納されます。

このQUBO++プログラムでは、変数 `a`、`b`、`c` は **`qbpp::Var`** クラスのオブジェクトであり、式 `f` は **`qbpp::Expr`** クラスのオブジェクトです。

ヘッダとライブラリのパスが適切に設定されていれば、このプログラム（**`test.cpp`** として保存）は `g++` で以下のようにコンパイルできます。
```bash
g++ test.cpp -o test -std=c++17 -lqbpp -ltbb
```
実行すると、展開された式が出力されます。
```bash
./test
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

> **NOTE**
> `qbpp::var()` の変数名は省略可能です。
> 省略した場合、`{0}`、`{1}` などのデフォルト名が自動的に割り当てられます。

> **WARNING**
> `qbpp::Expr` をはじめとするQUBO++のほとんどのクラスインスタンスは、`std::cout` を使ってテキストとして出力できます。
> ただし、このテキスト出力は安定性が保証されておらず、フォーマットが将来のリリースで変更される可能性があるため、後続の計算の入力として使用すべきではありません。
> また、QUBO++ドキュメントに示されている出力は古いバージョンで生成されたものである可能性があり、最新バージョンの出力とは異なる場合があります。

## 式の簡約化
**`qbpp::Expr`** オブジェクトに格納された式は、**`simplify()`** メンバ関数を呼び出すことで簡約化できます。
```cpp
  std::cout << "f = " << f.simplify() << std::endl;
```
この変更により、プログラムの出力は以下のようになります。
```
f.simplify() = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
メンバ関数 **`f.simplify()`** は式 `f` を簡約化し、結果の値を返します。その値が `std::cout` によって出力されます。

すべての変数が **2値（0または1）** をとると仮定すると、恒等式 **$b^2=b$** を使って式をさらに簡約化できます。
この目的には、代わりに **`simplify_as_binary()`** を使用します。
```cpp
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
```
出力は以下のようになります。
```
f = 1 -a -b -c +a*b +a*c +b*c
```

簡約化関数は、各項内の変数と式内の項を並べ替え、低次の項が先に表示されるようにし、同じ次数の項は変数の辞書順でソートします。
変数自体は定義された順序で並べられます。

## スピン変数による式の簡約化
変数が **スピン値 $-1$/$+1$** をとると仮定する場合、恒等式 **$b^2 = 1$** を使って式をさらに簡約化できます。
この場合、**`simplify_as_spin()`** メンバ関数を使用して式を簡約化します。
```cpp
  std::cout << "f = " << f.simplify_as_spin() << std::endl;
```
出力は以下のようになります。
```
f = 2 -a -2*b -c +a*b +a*c +b*c
```

## 簡約化のためのグローバル関数
メンバ関数は `f` に格納された式を更新します。
`f` を変更したくない場合は、代わりにグローバル関数 **`qbpp::simplify(f)`**、**`qbpp::simplify_as_binary(f)`**、**`qbpp::simplify_as_spin(f)`** を使用できます。これらは `f` を変更せずに簡約化された式を返します。

> **NOTE**
> QUBO++では、ほとんどの **メンバ関数** はオブジェクトをその場で更新しますが、**グローバル関数** は元のオブジェクトを変更せずに新しい値を返します。

## 否定リテラル
QUBO++は `~` 演算子を使った **否定リテラル** をネイティブにサポートしています。
2値変数 `x` に対して、式 `~x` は $1 - x$ を表します。

```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = ~a * ~b * ~c * ~d + a * b;
  std::cout << "f = " << f << std::endl;
  std::cout << "f = " << qbpp::simplify_as_binary(f) << std::endl;
}
```
出力:
```
f = ~a*~b*~c*~d +a*b
f = a*b +~a*~b*~c*~d
```

否定リテラル `~x` は、`1 - x` として展開されるのではなく、否定フラグを持つ単一の変数として内部的に格納されます。
これはパフォーマンスにとって重要です。`~x` を単純に展開すると、`~x1 * ~x2 * ... * ~xk` のような $k$ 個の否定リテラルの積は、$(1-x_1)(1-x_2)\cdots(1-x_k)$ を展開した後に最大 $2^k$ 個の項を生成します。
例えば、上記の項 `~a*~b*~c*~d` は単一の4次項として格納されますが、展開形 $(1-a)(1-b)(1-c)(1-d)$ は16個の項を生成します。
```
1 -a -b -c -d +a*b +a*c +a*d +b*c +b*d +c*d -a*b*c -a*b*d -a*c*d -b*c*d +a*b*c*d
```

QUBO++に同梱されているすべてのソルバー（EasySolver、ExhaustiveSolver、ABS3 GPU Solver）は否定リテラルをネイティブに処理するため、求解前に展開する必要はありません。

</div>
