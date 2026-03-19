---
layout: default
nav_exclude: true
title: "Pythagorean Triples (PyQBPP)"
nav_order: 40
---
<div class="lang-en" markdown="1">
# Pythagorean Triples

Three integers $x$, $y$, and $z$ are **Pythagorean triples** if they satisfy

$$
\begin{aligned}
x^2+y^2&=z^2
\end{aligned}
$$

To avoid duplicates, we assume $x<y$.

## PyQBPP program for listing Pythagorean Triples
The following program lists Pythagorean triples with $x\leq 16$, $y\leq 16$, and $z\leq 16$:
```python
from pyqbpp import var_int, between, ExhaustiveSolver

x = between(var_int("x"), 1, 16)
y = between(var_int("y"), 1, 16)
z = between(var_int("z"), 1, 16)
f = x * x + y * y - z * z == 0
c = between(y - x, 1, 15)
g = f + c
g.simplify_as_binary()

solver = ExhaustiveSolver(g)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    key = (sol.eval(x), sol.eval(y), sol.eval(z))
    if key not in seen:
        seen.add(key)
        print(f"x={key[0]}, y={key[1]}, z={key[2]}, f={sol.eval(f.body)}, c={sol.eval(c.body)}")
```
In this program, we define integer variables `x`, `y`, and `z` with ranges from 1 to 16.
We then create two constraint expressions:
- `f` for $x^2+y^2-z^2=0$, and
- `c` for $x+1\leq y$.

They are combined into `g`.
The expression `g` attains its minimum value 0 when all constraints are satisfied.

An Exhaustive Solver object `solver` is created for `g`.
The call to `search_optimal_solutions()` returns a list of all optimal solutions.

Because integer variables are encoded by multiple binary variables, the same
$(x,y,z)$ assignment may appear multiple times.
Therefore, we use a `set` to remove duplicates before printing.

This program produces the following output:
```
x=3, y=4, z=5, f=0, c=1
x=5, y=12, z=13, f=0, c=7
x=6, y=8, z=10, f=0, c=2
x=9, y=12, z=15, f=0, c=3
```

### Comparison with C++ QUBO++

<table>
<thead>
<tr><th>C++ QUBO++</th><th>PyQBPP</th></tr>
</thead>
<tbody>
<tr><td><code>1 &lt;= qbpp::var_int("x") &lt;= 16</code></td><td><code>between(var_int("x"), 1, 16)</code></td></tr>
<tr><td><code>1 &lt;= y - x &lt;= +qbpp::inf</code></td><td><code>between(y - x, 1, 15)</code></td></tr>
<tr><td><code>sol(x)</code></td><td><code>sol.eval(x)</code></td></tr>
<tr><td><code>sol(*f)</code></td><td><code>sol.eval(f.body)</code></td></tr>
</tbody>
</table>

</div>

<div class="lang-ja" markdown="1">
# ピタゴラス数

3つの整数 $x$、$y$、$z$ が以下を満たすとき、**ピタゴラス数**と呼ばれます:

$$
\begin{aligned}
x^2+y^2&=z^2
\end{aligned}
$$

重複を避けるため、$x<y$ と仮定します。

## ピタゴラス数を列挙する PyQBPP プログラム
以下のプログラムは、$x\leq 16$、$y\leq 16$、$z\leq 16$ のピタゴラス数を列挙します:
```python
from pyqbpp import var_int, between, ExhaustiveSolver

x = between(var_int("x"), 1, 16)
y = between(var_int("y"), 1, 16)
z = between(var_int("z"), 1, 16)
f = x * x + y * y - z * z == 0
c = between(y - x, 1, 15)
g = f + c
g.simplify_as_binary()

solver = ExhaustiveSolver(g)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    key = (sol.eval(x), sol.eval(y), sol.eval(z))
    if key not in seen:
        seen.add(key)
        print(f"x={key[0]}, y={key[1]}, z={key[2]}, f={sol.eval(f.body)}, c={sol.eval(c.body)}")
```
このプログラムでは、範囲 1 から 16 の整数変数 `x`、`y`、`z` を定義します。
次に、2つの制約式を作成します:
- `f`: $x^2+y^2-z^2=0$ の制約、
- `c`: $x+1\leq y$ の制約。

これらを `g` にまとめます。
式 `g` はすべての制約が満たされたときに最小値 0 を達成します。

`g` に対して全探索ソルバーオブジェクト `solver` を作成します。
`search_optimal_solutions()` の呼び出しは、すべての最適解のリストを返します。

整数変数は複数のバイナリ変数でエンコードされるため、同じ $(x,y,z)$ の割り当てが複数回出現する可能性があります。
そのため、出力前に `set` を使って重複を除去しています。

このプログラムは以下の出力を生成します:
```
x=3, y=4, z=5, f=0, c=1
x=5, y=12, z=13, f=0, c=7
x=6, y=8, z=10, f=0, c=2
x=9, y=12, z=15, f=0, c=3
```

### C++ QUBO++ との比較

<table>
<thead>
<tr><th>C++ QUBO++</th><th>PyQBPP</th></tr>
</thead>
<tbody>
<tr><td><code>1 &lt;= qbpp::var_int("x") &lt;= 16</code></td><td><code>between(var_int("x"), 1, 16)</code></td></tr>
<tr><td><code>1 &lt;= y - x &lt;= +qbpp::inf</code></td><td><code>between(y - x, 1, 15)</code></td></tr>
<tr><td><code>sol(x)</code></td><td><code>sol.eval(x)</code></td></tr>
<tr><td><code>sol(*f)</code></td><td><code>sol.eval(f.body)</code></td></tr>
</tbody>
</table>

</div>
