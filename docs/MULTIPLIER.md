---
layout: default
nav_exclude: true
title: "Multiplier Simulation"
nav_order: 51
---
<div class="lang-en" markdown="1">
# Multiplier Simulation and Factorization
Multiplication of two integers can be performed using additions.
In this section, we design a multiplier for two 4-bit integers using full adders.
The figure below shows how two　$x_3x_2x_1x_0$ and　$y_3y_2y_1y_0$ are multiplied to obtain an 8-bit integer $z_7z_6z_5z_4z_3z_2z_1z_0$.
In this figure, $p_{i,j}=x_iy_j$ ($0\leq i,j\leq 3$) and these partial products are summed to compute the final 8-bit result.

<p align="center">
 <img src="images/multiplication.svg" alt="4-bit multiplication" width="50%">
</p>

We use a 4-bit ripple-carry adder that computes the sum of two 4-bit integers
$a_3a_2a_1a_0$ and $b_3b_2b_1b_0$ producing the 5-bit sum $z_4z_3z_2z_1z_0$.
It consists of four full adders connected by a 5-bit carry wire $c_4c_3c_2c_1c_0$
that propagates carries.

<p align="center">
 <img src="images/adder4.svg" alt="The 4-bit ripple carry adder" width="50%">
</p>

A 4-bit multiplier can be constructed using three 4-bit adders.
They are connected by wires $c_{i,j}$ ($0\leq i\leq 2, 0\leq j\leq 3$) to propagate intermediate sum bits, as shown below:
<p align="center">
 <img src="images/multiplier.svg" alt="The 4-bit multiplier using three 4-bit adders" width="50%">
</p>

## QUBO formulation for multiplier
We will show QUBO formulation for simulating the `N`-bit multiplier.
To do this, we implement functions that construct a full adder, an adder, and a multiplier.

### Full adder
The following QUBO expression simulates a full adder with three input bits `a`, `b`, and `i`, and two output bits: carry-out `o` and sum `s`:
```cpp
qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}
```
The function `fa` returns an expression that enforces consistency between the input and output bits of a full adder.

### Adder
Assume that vectors `a`, `b`, and `s` of `qbpp::Expr` objects represent integers.
We assume that `a` and `b` each have `N` elements representing `N`-bit integers, while `s` has `N + 1` elements representing an `(N + 1)`-bit integer.
The following function adder returns a QUBO expression whose minimum value is 0 if and only if `a + b == s` holds:
{% raw %}
```cpp
qbpp::Expr adder(const qbpp::Vector<qbpp::Expr>& a,
                 const qbpp::Vector<qbpp::Expr>& b,
                 const qbpp::Vector<qbpp::Expr>& s) {
  auto N = a.size();
  auto c = qbpp::var(N + 1);
  auto f = qbpp::toExpr(0);
  for (size_t j = 0; j < N; ++j) {
    f += fa(a[j], b[j], c[j], c[j + 1], s[j]);
  }
  f.replace({{c[0], 0}, {c[N], s[N]}});
  return f;
}
```
{% endraw %}
In this function, `c` is a vector of `N + 1` variables used to connect the carry-out and carry-in signals of the `fa` blocks, forming an `N`-bit ripple-carry adder.

### Multiplier
Assume that vectors `x`, `y`, and `z` of `qbpp::Expr` represent integers.
We assume that `x` and `y` each have `N` elements and that `z` has `2 * N` elements.
The following function multiplier returns a QUBO expression whose minimum value is 0 if and only if `x * y == z` holds.
```cpp
qbpp::Expr multiplier(const qbpp::Vector<qbpp::Expr>& x,
                      const qbpp::Vector<qbpp::Expr>& y,
                      const qbpp::Vector<qbpp::Expr>& z) {
  auto N = x.size();
  auto c = qbpp::var("c", N - 1, N + 1);

  auto f = qbpp::toExpr(0);

  for (size_t i = 0; i < N - 1; ++i) {
    qbpp::Vector<qbpp::Expr> a, b, s;
    for (size_t j = 0; j < N; ++j) {
      b.push_back(x[i + 1] * y[j]);
    }

    if (i == 0) {
      for (size_t j = 0; j < N - 1; ++j) {
        a.push_back(x[0] * y[j + 1]);
      }
      a.push_back(0);
    } else {
      for (size_t j = 0; j < N; ++j) {
        a.push_back(c[i - 1][j + 1]);
      }
    }

    for (size_t j = 0; j < N + 1; ++j) {
      s.push_back(c[i][j]);
    }
    f += adder(a, b, s);
  }
  f += z[0] - x[0] * y[0] == 0;

  qbpp::MapList ml;
  for (size_t i = 0; i < N - 2; ++i) {
    ml.push_back({c[i][0], z[i + 1]});
  }
  for (size_t i = 0; i < N + 1; ++i) {
    ml.push_back({c[N - 2][i], z[N + i - 1]});
  }
  return f.replace(ml).simplify_as_binary();
}
```
This function uses an `(N−1)×(N+1)` matrix `c` of `qbpp::Var` objects to connect the `N−1` adders of `N` bits.
Since each bit of `z` corresponds to one element of `c`, their correspondence is defined in `ml`, and the replacements are performed using `replace()`.

## QUBO++ program for factorization
Using the function `multiplier`, we can factor a composite integer into two factors.
The following program constructs a 4-bit multiplier with
- `x`: 4 binary variables,
- `y`: 4 binary variables,
- `z`: a vector of constants `{1, 1, 1, 1, 0, 0, 0, 1}`, representing the 8-bit integer `10001111` `(143)`, and stores the resulting expression in `f`:
{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}

qbpp::Expr adder(const qbpp::Vector<qbpp::Expr>& a,
                 const qbpp::Vector<qbpp::Expr>& b,
                 const qbpp::Vector<qbpp::Expr>& s) {
  auto N = a.size();
  auto c = qbpp::var(N + 1);
  auto f = qbpp::toExpr(0);
  for (size_t j = 0; j < N; ++j) {
    f += fa(a[j], b[j], c[j], c[j + 1], s[j]);
  }
  return f.replace({{c[0], 0}, {c[N], s[N]}});
}

qbpp::Expr multiplier(const qbpp::Vector<qbpp::Expr>& x,
                      const qbpp::Vector<qbpp::Expr>& y,
                      const qbpp::Vector<qbpp::Expr>& z) {
  auto N = x.size();
  auto c = qbpp::var("c", N - 1, N + 1);

  auto f = qbpp::toExpr(0);

  for (size_t i = 0; i < N - 1; ++i) {
    qbpp::Vector<qbpp::Expr> a, b, s;
    for (size_t j = 0; j < N; ++j) {
      b.push_back(x[i + 1] * y[j]);
    }

    if (i == 0) {
      for (size_t j = 0; j < N - 1; ++j) {
        a.push_back(x[0] * y[j + 1]);
      }
      a.push_back(0);
    } else {
      for (size_t j = 0; j < N; ++j) {
        a.push_back(c[i - 1][j + 1]);
      }
    }

    for (size_t j = 0; j < N + 1; ++j) {
      s.push_back(c[i][j]);
    }
    f += adder(a, b, s);
  }
  f += z[0] - x[0] * y[0] == 0;

  qbpp::MapList ml;
  for (size_t i = 0; i < N - 2; ++i) {
    ml.push_back({c[i][0], z[i + 1]});
  }
  for (size_t i = 0; i < N + 1; ++i) {
    ml.push_back({c[N - 2][i], z[N + i - 1]});
  }
  return f.replace(ml).simplify_as_binary();
}

int main() {
  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  qbpp::Vector<int> z = {1, 1, 1, 1, 0, 0, 0, 1};
  auto f = multiplier(x, y, z).simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  for (auto it = x.rbegin(); it != x.rend(); ++it) {
    std::cout << sol(*it);
  }
  std::cout << " * ";
  for (auto it = y.rbegin(); it != y.rend(); ++it) {
    std::cout << sol(*it);
  }
  std::cout << " = ";
  for (auto it = z.rbegin(); it != z.rend(); ++it) {
    std::cout << *it;
  }
  std::cout << std::endl;
}
```
{% endraw %}
The Easy Solver is executed on `f`, and the obtained solution is stored in `sol`.
The resulting values of `x` and `y` are printed as:
```
1011 * 1101 = 10001111
```
This output indicates $11\times 13 = 143$, demonstrating the factorization result.
</div>

<div class="lang-ja" markdown="1">
# 乗算器シミュレーションと因数分解
2つの整数の乗算は加算を用いて実行できます。
このセクションでは、全加算器を使って2つの4ビット整数の乗算器を設計します。
以下の図は、2つの4ビット整数 $x_3x_2x_1x_0$ と $y_3y_2y_1y_0$ を乗算して8ビット整数 $z_7z_6z_5z_4z_3z_2z_1z_0$ を得る方法を示しています。
この図では、$p_{i,j}=x_iy_j$ ($0\leq i,j\leq 3$) であり、これらの部分積を加算して最終的な8ビットの結果を計算します。

<p align="center">
 <img src="images/multiplication.svg" alt="4-bit multiplication" width="50%">
</p>

2つの4ビット整数 $a_3a_2a_1a_0$ と $b_3b_2b_1b_0$ の和を計算し、5ビットの和 $z_4z_3z_2z_1z_0$ を出力する4ビットリプルキャリー加算器を使用します。
これは、キャリーを伝搬する5ビットのキャリー線 $c_4c_3c_2c_1c_0$ で接続された4つの全加算器で構成されます。

<p align="center">
 <img src="images/adder4.svg" alt="The 4-bit ripple carry adder" width="50%">
</p>

4ビット乗算器は3つの4ビット加算器を使って構築できます。
以下に示すように、中間の和ビットを伝搬するためにワイヤ $c_{i,j}$ ($0\leq i\leq 2, 0\leq j\leq 3$) で接続されています:
<p align="center">
 <img src="images/multiplier.svg" alt="The 4-bit multiplier using three 4-bit adders" width="50%">
</p>

## 乗算器のQUBO定式化
`N`ビット乗算器をシミュレートするためのQUBO定式化を示します。
そのために、全加算器、加算器、乗算器を構築する関数を実装します。

### 全加算器
以下のQUBO式は、3つの入力ビット `a`、`b`、`i` と2つの出力ビット（キャリー出力 `o` と和 `s`）を持つ全加算器をシミュレートします:
```cpp
qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}
```
関数 `fa` は、全加算器の入力ビットと出力ビットの間の整合性を強制する式を返します。

### 加算器
`qbpp::Expr` オブジェクトのベクトル `a`、`b`、`s` が整数を表すとします。
`a` と `b` はそれぞれ `N` 個の要素を持ち `N` ビット整数を表し、`s` は `N + 1` 個の要素を持ち `(N + 1)` ビット整数を表すと仮定します。
以下の関数 adder は、`a + b == s` が成り立つとき、かつそのときに限り最小値0となるQUBO式を返します:
{% raw %}
```cpp
qbpp::Expr adder(const qbpp::Vector<qbpp::Expr>& a,
                 const qbpp::Vector<qbpp::Expr>& b,
                 const qbpp::Vector<qbpp::Expr>& s) {
  auto N = a.size();
  auto c = qbpp::var(N + 1);
  auto f = qbpp::toExpr(0);
  for (size_t j = 0; j < N; ++j) {
    f += fa(a[j], b[j], c[j], c[j + 1], s[j]);
  }
  f.replace({{c[0], 0}, {c[N], s[N]}});
  return f;
}
```
{% endraw %}
この関数では、`c` は `N + 1` 個の変数のベクトルであり、`fa` ブロックのキャリー出力信号とキャリー入力信号を接続して `N` ビットのリプルキャリー加算器を構成するために使用されます。

### 乗算器
`qbpp::Expr` のベクトル `x`、`y`、`z` が整数を表すとします。
`x` と `y` はそれぞれ `N` 個の要素を持ち、`z` は `2 * N` 個の要素を持つと仮定します。
以下の関数 multiplier は、`x * y == z` が成り立つとき、かつそのときに限り最小値0となるQUBO式を返します。
```cpp
qbpp::Expr multiplier(const qbpp::Vector<qbpp::Expr>& x,
                      const qbpp::Vector<qbpp::Expr>& y,
                      const qbpp::Vector<qbpp::Expr>& z) {
  auto N = x.size();
  auto c = qbpp::var("c", N - 1, N + 1);

  auto f = qbpp::toExpr(0);

  for (size_t i = 0; i < N - 1; ++i) {
    qbpp::Vector<qbpp::Expr> a, b, s;
    for (size_t j = 0; j < N; ++j) {
      b.push_back(x[i + 1] * y[j]);
    }

    if (i == 0) {
      for (size_t j = 0; j < N - 1; ++j) {
        a.push_back(x[0] * y[j + 1]);
      }
      a.push_back(0);
    } else {
      for (size_t j = 0; j < N; ++j) {
        a.push_back(c[i - 1][j + 1]);
      }
    }

    for (size_t j = 0; j < N + 1; ++j) {
      s.push_back(c[i][j]);
    }
    f += adder(a, b, s);
  }
  f += z[0] - x[0] * y[0] == 0;

  qbpp::MapList ml;
  for (size_t i = 0; i < N - 2; ++i) {
    ml.push_back({c[i][0], z[i + 1]});
  }
  for (size_t i = 0; i < N + 1; ++i) {
    ml.push_back({c[N - 2][i], z[N + i - 1]});
  }
  return f.replace(ml).simplify_as_binary();
}
```
この関数は、`N-1` 個の `N` ビット加算器を接続するために `(N-1)x(N+1)` の `qbpp::Var` オブジェクトの行列 `c` を使用します。
`z` の各ビットは `c` の1つの要素に対応するため、その対応関係が `ml` に定義され、`replace()` を使って置換が実行されます。

## 因数分解のためのQUBO++プログラム
関数 `multiplier` を使用して、合成整数を2つの因数に分解できます。
以下のプログラムは4ビット乗算器を構築します:
- `x`: 4個のバイナリ変数、
- `y`: 4個のバイナリ変数、
- `z`: 定数ベクトル `{1, 1, 1, 1, 0, 0, 0, 1}`（8ビット整数 `10001111` すなわち `143` を表す）。結果の式を `f` に格納します:
{% raw %}
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/easy_solver.hpp>

qbpp::Expr fa(const qbpp::Expr& a, const qbpp::Expr& b, const qbpp::Expr& i,
              const qbpp::Expr& o, const qbpp::Expr& s) {
  return (a + b + i) - (2 * o + s) == 0;
}

qbpp::Expr adder(const qbpp::Vector<qbpp::Expr>& a,
                 const qbpp::Vector<qbpp::Expr>& b,
                 const qbpp::Vector<qbpp::Expr>& s) {
  auto N = a.size();
  auto c = qbpp::var(N + 1);
  auto f = qbpp::toExpr(0);
  for (size_t j = 0; j < N; ++j) {
    f += fa(a[j], b[j], c[j], c[j + 1], s[j]);
  }
  return f.replace({{c[0], 0}, {c[N], s[N]}});
}

qbpp::Expr multiplier(const qbpp::Vector<qbpp::Expr>& x,
                      const qbpp::Vector<qbpp::Expr>& y,
                      const qbpp::Vector<qbpp::Expr>& z) {
  auto N = x.size();
  auto c = qbpp::var("c", N - 1, N + 1);

  auto f = qbpp::toExpr(0);

  for (size_t i = 0; i < N - 1; ++i) {
    qbpp::Vector<qbpp::Expr> a, b, s;
    for (size_t j = 0; j < N; ++j) {
      b.push_back(x[i + 1] * y[j]);
    }

    if (i == 0) {
      for (size_t j = 0; j < N - 1; ++j) {
        a.push_back(x[0] * y[j + 1]);
      }
      a.push_back(0);
    } else {
      for (size_t j = 0; j < N; ++j) {
        a.push_back(c[i - 1][j + 1]);
      }
    }

    for (size_t j = 0; j < N + 1; ++j) {
      s.push_back(c[i][j]);
    }
    f += adder(a, b, s);
  }
  f += z[0] - x[0] * y[0] == 0;

  qbpp::MapList ml;
  for (size_t i = 0; i < N - 2; ++i) {
    ml.push_back({c[i][0], z[i + 1]});
  }
  for (size_t i = 0; i < N + 1; ++i) {
    ml.push_back({c[N - 2][i], z[N + i - 1]});
  }
  return f.replace(ml).simplify_as_binary();
}

int main() {
  auto x = qbpp::var("x", 4);
  auto y = qbpp::var("y", 4);
  qbpp::Vector<int> z = {1, 1, 1, 1, 0, 0, 0, 1};
  auto f = multiplier(x, y, z).simplify_as_binary();

  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();

  for (auto it = x.rbegin(); it != x.rend(); ++it) {
    std::cout << sol(*it);
  }
  std::cout << " * ";
  for (auto it = y.rbegin(); it != y.rend(); ++it) {
    std::cout << sol(*it);
  }
  std::cout << " = ";
  for (auto it = z.rbegin(); it != z.rend(); ++it) {
    std::cout << *it;
  }
  std::cout << std::endl;
}
```
{% endraw %}
Easy Solverが `f` に対して実行され、得られた解が `sol` に格納されます。
`x` と `y` の結果の値は以下のように出力されます:
```
1011 * 1101 = 10001111
```
この出力は $11\times 13 = 143$ を示しており、因数分解の結果を実証しています。
</div>
