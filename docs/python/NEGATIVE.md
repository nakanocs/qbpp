---
layout: default
nav_exclude: true
title: "Negated Literals"
nav_order: 18
---
<div class="lang-en" markdown="1">

# Negated Literals
Unlike other QUBO/HUBO tools, PyQBPP natively supports negated literals of binary variables.
Conventionally, the negated literal $\bar{x}$ of a binary variable $x$ is expressed as $1-x$.
This causes an explosion of terms when expanding a term with many negated literals.
For example, a term with four negated literals expands to
16 terms including a constant term:

$$
\begin{aligned}
\bar{x}_0\cdot \bar{x}_1\cdot\bar{x}_2\cdot \bar{x}_3
&= (1-x_0)(1-x_1)(1-x_2)(1-x_3)\\
&= 1-x_0-x_1-x_2-x_3+x_0x_1+x_0x_2+x_0x_3+x_1x_2+x_1x_3+x_2x_3\\
&\quad -x_0x_1x_2-x_0x_1x_3-x_0x_2x_3-x_1x_2x_3+x_0x_1x_2x_3
\end{aligned}
$$

PyQBPP can handle such terms without expanding negated literals.

## Using negated literals in PyQBPP

PyQBPP uses the `~` operator to represent negated literals,
so `~x` denotes the negated literal of `x`.
The following program demonstrates how PyQBPP handles negated literals:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)
f = 1
for i in range(len(x)):
    f *= ~x[i]

ml = [(~x[i], 1 - x[i]) for i in range(len(x))]
g = qbpp.replace(f, ml)

f.simplify_as_binary()
g.simplify_as_binary()
print("f =", f)
print("g =", g)
```
In this program, the expression `f` stores:

$$
\begin{aligned}
f = \bar{x}_0\cdot \bar{x}_1\cdot\bar{x}_2\cdot \bar{x}_3
\end{aligned}
$$

The expression `g` is obtained by replacing each `~x[i]` with `1 - x[i]`,
as conventional tools would do.
This program produces the following output:

```
f = ~x[0]*~x[1]*~x[2]*~x[3]
g = 1 -x[0] -x[1] -x[2] -x[3] +x[0]*x[1] +x[0]*x[2] +x[0]*x[3] +x[1]*x[2] +x[1]*x[3] +x[2]*x[3] -x[0]*x[1]*x[2] -x[0]*x[1]*x[3] -x[0]*x[2]*x[3] -x[1]*x[2]*x[3] +x[0]*x[1]*x[2]*x[3]
```

The solvers bundled with PyQBPP accept HUBO formulas with negated literals directly,
without expanding them into positive literals.
Conventional tools, on the other hand, require using $1-x$ instead of $\bar{x}$.
Thus, for HUBO models containing terms with many negated literals,
PyQBPP can outperform conventional tools.

## `simplify_as_binary()` and negated literals
Since terms with one or two variables do not cause term explosion,
and replacing $\bar{x}$ with $1-x$ may reduce the number of terms,
`simplify_as_binary()` expands negated literals only in terms of degree one or two.
Terms of degree three or higher retain their negated literals.

The following examples show how such replacement can reduce the size of expressions:

$$
\begin{aligned}
x+ \bar{x} & = x + (1-x) = 1\\
x\cdot \bar{x} & = x \cdot (1-x) = x-x^2 = 0\\
-x\cdot y+\bar{x}\cdot\bar{y} &= -x\cdot y+(1-x)(1-y) = 1-x-y
\end{aligned}
$$

The above program demonstrates this behavior at different variable counts.
With 1 variable, `f` and `g` produce the same output:
```
f = 1 -x
g = 1 -x
```
With 2 variables:
```
f = 1 -x[0] -x[1] +x[0]*x[1]
g = 1 -x[0] -x[1] +x[0]*x[1]
```

With 3 or more variables, `f` retains the negated literals while `g` is fully expanded:
```
f = ~x[0]*~x[1]*~x[2]
g = 1 -x[0] -x[1] -x[2] +x[0]*x[1] +x[0]*x[2] +x[1]*x[2] -x[0]*x[1]*x[2]
```

## `simplify_as_spin()` and negated literals
For spin variables ($s \in \lbrace -1, +1\rbrace$), the negated literal $\bar{s}$ corresponds to $-s$.
The `simplify_as_spin()` function replaces all negated literals $\bar{s}$ with $-s$
by negating the coefficient for each negated variable.

## `simplify()` and negated literals
The `simplify()` function makes no assumption on the domain of variables,
so it never replaces negated literals.

## `replace()` and negated literals
The `replace()` function treats `x` and `~x` as independent keys.
Therefore, to fix a variable value, both the positive and negated literals
should be specified consistently.

The following program first fixes `x` to 1, then fixes `~x` to 0:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
z = qbpp.var("z")
f = x * y * z + ~x * ~y * ~z
print("f =", f)
f.replace([(x, 1)])
print("f =", f)
f.replace([(~x, 0)])
print("f =", f)
```
This program produces the following output:
```
f = x*y*z +~x*~y*~z
f = y*z +~x*~y*~z
f = y*z
```

## Evaluating expressions with negated literals
When evaluating an expression with negated literals,
it is sufficient to specify either the positive or negated literal for each variable.
If both are specified, their values must be consistent
(e.g., `(x, 0)` and `(~x, 1)`).

The following program evaluates the expression at $x=0, y=0, z=0$:
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
z = qbpp.var("z")
f = x * y * z + ~x * ~y * ~z
print("f =", f)
print("f(0, 0, 0) =", f([(x, 0), (~y, 1), (~z, 1)]))
```
This program produces the following output:
```
f = x*y*z +~x*~y*~z
f(0, 0, 0) = 1
```
Note that `x` is specified as a positive literal with value 0,
while `~y` and `~z` are specified as negated literals with value 1.
All three variables are effectively set to 0, and $\bar{x}\cdot\bar{y}\cdot\bar{z} = 1$.

### Comparison with C++ QUBO++

| C++ QUBO++                              | PyQBPP                                     |
|-----------------------------------------|--------------------------------------------|
| `qbpp::Expr(1)`                         | `1`                                        |
| `f *= ~x[i]`                            | `f *= ~x[i]`                               |
| `qbpp::MapList ml;`<br>`ml.push_back({~x[i], 1 - x[i]});` | `ml = [(~x[i], 1 - x[i]) for i in range(len(x))]` |
| `f({% raw %}{{x, 0}, {~y, 1}}{% endraw %})` | `f([(x, 0), (~y, 1)])`               |

</div>

<div class="lang-ja" markdown="1">

# 否定リテラル
他のQUBO/HUBOツールとは異なり、PyQBPPはバイナリ変数の否定リテラルをネイティブにサポートしています。
従来、バイナリ変数 $x$ の否定リテラル $\bar{x}$ は $1-x$ と表現されます。
これにより、多くの否定リテラルを含む項を展開する際に項数が爆発的に増加します。
例えば、4つの否定リテラルを含む項は、定数項を含む16項に展開されます：

$$
\begin{aligned}
\bar{x}_0\cdot \bar{x}_1\cdot\bar{x}_2\cdot \bar{x}_3
&= (1-x_0)(1-x_1)(1-x_2)(1-x_3)\\
&= 1-x_0-x_1-x_2-x_3+x_0x_1+x_0x_2+x_0x_3+x_1x_2+x_1x_3+x_2x_3\\
&\quad -x_0x_1x_2-x_0x_1x_3-x_0x_2x_3-x_1x_2x_3+x_0x_1x_2x_3
\end{aligned}
$$

PyQBPPはこのような項を否定リテラルを展開せずに扱うことができます。

## PyQBPPでの否定リテラルの使用

PyQBPPでは `~` 演算子で否定リテラルを表現します。
`~x` は `x` の否定リテラルを表します。
以下のプログラムは、PyQBPPが否定リテラルをどのように扱うかを示しています：
```python
import pyqbpp as qbpp

x = qbpp.var("x", 4)
f = 1
for i in range(len(x)):
    f *= ~x[i]

ml = [(~x[i], 1 - x[i]) for i in range(len(x))]
g = qbpp.replace(f, ml)

f.simplify_as_binary()
g.simplify_as_binary()
print("f =", f)
print("g =", g)
```
このプログラムでは、式 `f` に以下が格納されます：

$$
\begin{aligned}
f = \bar{x}_0\cdot \bar{x}_1\cdot\bar{x}_2\cdot \bar{x}_3
\end{aligned}
$$

式 `g` は、従来のツールと同様に各 `~x[i]` を `1 - x[i]` で置換して得られた式です。
このプログラムは以下の出力を生成します：

```
f = ~x[0]*~x[1]*~x[2]*~x[3]
g = 1 -x[0] -x[1] -x[2] -x[3] +x[0]*x[1] +x[0]*x[2] +x[0]*x[3] +x[1]*x[2] +x[1]*x[3] +x[2]*x[3] -x[0]*x[1]*x[2] -x[0]*x[1]*x[3] -x[0]*x[2]*x[3] -x[1]*x[2]*x[3] +x[0]*x[1]*x[2]*x[3]
```

PyQBPPに付属するソルバーは、否定リテラルを含むHUBO式を正リテラルに展開せずに直接受け付けます。
一方、従来のツールは $\bar{x}$ の代わりに $1-x$ を使用する必要があります。
そのため、多くの否定リテラルを含む項を持つHUBOモデルでは、PyQBPPは従来のツールより高い性能を発揮できます。

## `simplify_as_binary()` と否定リテラル
1変数または2変数の項は項数爆発を起こさず、
$\bar{x}$ を $1-x$ で置き換えることで項数が減少する場合があるため、
`simplify_as_binary()` は1次または2次の項でのみ否定リテラルを展開します。
3次以上の項では否定リテラルがそのまま保持されます。

以下の例は、このような置換が式のサイズを縮小できることを示しています：

$$
\begin{aligned}
x+ \bar{x} & = x + (1-x) = 1\\
x\cdot \bar{x} & = x \cdot (1-x) = x-x^2 = 0\\
-x\cdot y+\bar{x}\cdot\bar{y} &= -x\cdot y+(1-x)(1-y) = 1-x-y
\end{aligned}
$$

上記のプログラムは、変数の数に応じて以下のように動作します。
1変数の場合、`f` と `g` は同じ出力になります：
```
f = 1 -x
g = 1 -x
```
2変数の場合：
```
f = 1 -x[0] -x[1] +x[0]*x[1]
g = 1 -x[0] -x[1] +x[0]*x[1]
```

3変数以上の場合、`f` は否定リテラルを保持し、`g` は完全に展開されます：
```
f = ~x[0]*~x[1]*~x[2]
g = 1 -x[0] -x[1] -x[2] +x[0]*x[1] +x[0]*x[2] +x[1]*x[2] -x[0]*x[1]*x[2]
```

## `simplify_as_spin()` と否定リテラル
スピン変数（$s \in \lbrace -1, +1\rbrace$）の場合、否定リテラル $\bar{s}$ は $-s$ に対応します。
`simplify_as_spin()` 関数は、否定変数ごとに係数を反転することで、すべての否定リテラル $\bar{s}$ を $-s$ に置き換えます。

## `simplify()` と否定リテラル
`simplify()` 関数は変数の取る値について仮定を置かないため、否定リテラルの置換は行いません。

## `replace()` と否定リテラル
`replace()` 関数は `x` と `~x` を独立したキーとして扱います。
したがって、変数の値を固定するには、正リテラルと否定リテラルの両方を整合的に指定する必要があります。

以下のプログラムは、まず `x` を 1 に固定し、次に `~x` を 0 に固定します：
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
z = qbpp.var("z")
f = x * y * z + ~x * ~y * ~z
print("f =", f)
f.replace([(x, 1)])
print("f =", f)
f.replace([(~x, 0)])
print("f =", f)
```
このプログラムは以下の出力を生成します：
```
f = x*y*z +~x*~y*~z
f = y*z +~x*~y*~z
f = y*z
```

## 否定リテラルを含む式の評価
否定リテラルを含む式を評価するには、各変数について正リテラルまたは否定リテラルのいずれか一方の値を指定すれば十分です。
両方を指定する場合、値は整合的でなければなりません（例：`(x, 0)` と `(~x, 1)`）。

以下のプログラムは、$x=0, y=0, z=0$ で式を評価します：
```python
import pyqbpp as qbpp

x = qbpp.var("x")
y = qbpp.var("y")
z = qbpp.var("z")
f = x * y * z + ~x * ~y * ~z
print("f =", f)
print("f(0, 0, 0) =", f([(x, 0), (~y, 1), (~z, 1)]))
```
このプログラムは以下の出力を生成します：
```
f = x*y*z +~x*~y*~z
f(0, 0, 0) = 1
```
`x` は正リテラルとして値 0 で指定し、`~y` と `~z` は否定リテラルとして値 1 で指定しています。
3つの変数はすべて実質的に 0 に設定され、$\bar{x}\cdot\bar{y}\cdot\bar{z} = 1$ となります。

### C++ QUBO++との比較

| C++ QUBO++                              | PyQBPP                                     |
|-----------------------------------------|--------------------------------------------|
| `qbpp::Expr(1)`                         | `1`                                        |
| `f *= ~x[i]`                            | `f *= ~x[i]`                               |
| `qbpp::MapList ml;`<br>`ml.push_back({~x[i], 1 - x[i]});` | `ml = [(~x[i], 1 - x[i]) for i in range(len(x))]` |
| `f({% raw %}{{x, 0}, {~y, 1}}{% endraw %})` | `f([(x, 0), (~y, 1)])`               |

</div>
