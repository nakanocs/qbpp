---
layout: default
nav_exclude: true
title: "SAT (PyQBPP)"
nav_order: 44
---
<div class="lang-en" markdown="1">
# Boolean Satisfiability Problem (SAT)

The **Boolean satisfiability problem (SAT)** is to determine whether there exists an assignment of truth values to Boolean variables that makes a given Boolean formula in **conjunctive normal form (CNF)** evaluate to True.
A CNF formula is a conjunction (AND) of **clauses**, where each clause is a disjunction (OR) of **literals**.
A literal is either a variable $x_i$ (positive literal) or its negation $\lnot x_i$ (negative literal).

For example, the following is a 3-SAT instance with 5 variables $x_0,x_1,x_2,x_3,x_4$ and 6 clauses:

$$
(x_0 \lor x_1 \lor x_2) \land (\lnot x_0 \lor x_3 \lor x_4) \land (x_1 \lor \lnot x_2 \lor \lnot x_3) \land (\lnot x_1 \lor \lnot x_3 \lor x_4) \land (\lnot x_0 \lor \lnot x_1 \lor \lnot x_2) \land (x_0 \lor x_1 \lor \lnot x_4)
$$

A satisfying assignment must make every clause True, i.e., at least one literal in each clause must be True.

## HUBO Formulation

We use the convention **True = 0** and **False = 1** for binary variables.
Under this convention:
- A positive literal $x_i$ is **False** when $x_i = 1$.
- A negative literal $\lnot x_i$ is **False** when $x_i = 0$, i.e., when $\tilde{x}_i = 1$ (where $\tilde{x}_i$ denotes the negated literal in PyQBPP).

A clause is violated (all its literals are False) exactly when the product of the "False indicator" for each literal equals 1.
For a clause $C_k$, we define the penalty:

$$
p_k = \prod_{\ell \in C_k} f(\ell)
$$

where $f(\ell) = x_i$ if $\ell$ is the positive literal $x_i$, and $f(\ell) = \tilde{x}_i$ if $\ell$ is the negative literal $\lnot x_i$.
This product equals 1 if and only if the clause is violated.

The total constraint expression is:

$$
\text{constraint} = \sum_{k} p_k
$$

This expression achieves the minimum value 0 if and only if all clauses are satisfied.
Note that the constraint is naturally a **HUBO** (higher-order unconstrained binary optimization) expression, since each clause with $m$ literals produces a term of degree $m$.

## PyQBPP Formulation

The following PyQBPP program solves the 3-SAT instance described above:

```python
import pyqbpp as qbpp

# 5 Boolean variables
x = qbpp.var("x", 5)

# Convention: True=0, False=1
# Positive literal x_i: False when x_i=1 -> contribute x[i]
# Negative literal ~x_j: False when x_j=0 -> contribute ~x[j]
# Product = 1 iff all literals in the clause are False (violated)

# Clause 0: (x0 OR x1 OR x2)
#   violated when x0=False AND x1=False AND x2=False
#   penalty = x[0] * x[1] * x[2]
c0 = qbpp.Expr(x[0] * x[1] * x[2])

# Clause 1: (~x0 OR x3 OR x4)
#   violated when x0=True AND x3=False AND x4=False
#   penalty = ~x[0] * x[3] * x[4]
c1 = qbpp.Expr(~x[0] * x[3] * x[4])

# Clause 2: (x1 OR ~x2 OR ~x3)
#   violated when x1=False AND x2=True AND x3=True
#   penalty = x[1] * ~x[2] * ~x[3]
c2 = qbpp.Expr(x[1] * ~x[2] * ~x[3])

# Clause 3: (~x1 OR ~x3 OR x4)
#   violated when x1=True AND x3=True AND x4=False
#   penalty = ~x[1] * ~x[3] * x[4]
c3 = qbpp.Expr(~x[1] * ~x[3] * x[4])

# Clause 4: (~x0 OR ~x1 OR ~x2)
#   violated when x0=True AND x1=True AND x2=True
#   penalty = ~x[0] * ~x[1] * ~x[2]
c4 = qbpp.Expr(~x[0] * ~x[1] * ~x[2])

# Clause 5: (x0 OR x1 OR ~x4)
#   violated when x0=False AND x1=False AND x4=True
#   penalty = x[0] * x[1] * ~x[4]
c5 = qbpp.Expr(x[0] * x[1] * ~x[4])

# Total constraint: sum of clause penalties
constraint = c0 + c1 + c2 + c3 + c4 + c5

constraint.simplify_as_binary()
solver = qbpp.EasySolver(constraint)
solver.target_energy(0)
sol = solver.search()

# Print result
print(f"Energy = {sol.energy()}")
print("Assignment (True=0, False=1):")
for i in range(5):
    val = sol(x[i])
    print(f"  x[{i}] = {val} ({'True' if val == 0 else 'False'})")

# Verify each clause
print("Clause penalties:")
for i, c in enumerate([c0, c1, c2, c3, c4, c5]):
    print(f"  c{i} = {sol(c)}")
print(f"Violated clauses = {sol(constraint)}")
```

In this program, we define 5 binary variables and construct the penalty expression for each clause.
For a positive literal $x_i$, we use `x[i]`, which equals 1 when the literal is False.
For a negative literal $\lnot x_i$, we use `~x[i]`, which equals 1 when the literal is False (i.e., when $x_i$ is True, meaning $x_i = 0$).
PyQBPP natively supports negated literals `~x[i]`, so there is no need to manually replace them with `1 - x[i]`.

The product of these terms for a clause equals 1 only when all literals in the clause are False, i.e., when the clause is violated.
The total constraint is the sum of all clause penalties, and it achieves 0 if and only if all clauses are satisfied.

We call `simplify_as_binary()` to apply the idempotent rule $x_i^2 = x_i$ and simplify the expression, then solve with EasySolver targeting energy 0.

### Output
```
Energy = 0
Assignment (True=0, False=1):
  x[0] = 0 (True)
  x[1] = 1 (False)
  x[2] = 0 (True)
  x[3] = 1 (False)
  x[4] = 0 (True)
Clause penalties:
  c0 = 0
  c1 = 0
  c2 = 0
  c3 = 0
  c4 = 0
  c5 = 0
Violated clauses = 0
```

The solver finds a satisfying assignment with energy 0, meaning all clauses are satisfied.
Note that the actual assignment may vary across runs, as the solver is stochastic.

</div>

<div class="lang-ja" markdown="1">
# 充足可能性問題 (SAT)

**充足可能性問題 (SAT)** は、**連言標準形 (CNF)** で与えられたブール式を真にするような変数への真偽値の割り当てが存在するかを判定する問題です。
CNF 式は**節** (clause) の連言 (AND) であり、各節は**リテラル**の選言 (OR) です。
リテラルは変数 $x_i$（正リテラル）またはその否定 $\lnot x_i$（負リテラル）です。

例えば、以下は 5 変数 $x_0,x_1,x_2,x_3,x_4$、6 節の 3-SAT インスタンスです：

$$
(x_0 \lor x_1 \lor x_2) \land (\lnot x_0 \lor x_3 \lor x_4) \land (x_1 \lor \lnot x_2 \lor \lnot x_3) \land (\lnot x_1 \lor \lnot x_3 \lor x_4) \land (\lnot x_0 \lor \lnot x_1 \lor \lnot x_2) \land (x_0 \lor x_1 \lor \lnot x_4)
$$

充足する割り当てはすべての節を真にする、つまり各節で少なくとも 1 つのリテラルが真でなければなりません。

## HUBO 定式化

バイナリ変数の規約として **True = 0**、**False = 1** を用います。
この規約の下で：
- 正リテラル $x_i$ は $x_i = 1$ のとき**偽 (False)** です。
- 負リテラル $\lnot x_i$ は $x_i = 0$ のとき**偽 (False)** です。すなわち $\tilde{x}_i = 1$ のとき（$\tilde{x}_i$ は PyQBPP における否定リテラル）です。

節が違反される（すべてのリテラルが偽）のは、各リテラルの「偽指標」の積が 1 に等しいときです。
節 $C_k$ に対して、ペナルティを以下のように定義します：

$$
p_k = \prod_{\ell \in C_k} f(\ell)
$$

ここで、$\ell$ が正リテラル $x_i$ の場合 $f(\ell) = x_i$、$\ell$ が負リテラル $\lnot x_i$ の場合 $f(\ell) = \tilde{x}_i$ です。
この積は節が違反されたときのみ 1 になります。

制約式の合計は以下の通りです：

$$
\text{constraint} = \sum_{k} p_k
$$

この式はすべての節が充足されたときのみ最小値 0 を達成します。
制約は自然に **HUBO**（高次無制約バイナリ最適化）式となります。$m$ リテラルの節は次数 $m$ の項を生成するためです。

## PyQBPP による定式化

以下の PyQBPP プログラムは、上記の 3-SAT インスタンスを解きます：

```python
import pyqbpp as qbpp

# 5つのブール変数
x = qbpp.var("x", 5)

# 規約: True=0, False=1
# 正リテラル x_i: x_i=1 のとき偽 -> x[i] を使用
# 負リテラル ~x_j: x_j=0 のとき偽 -> ~x[j] を使用
# 積 = 1 は節のすべてのリテラルが偽のとき（違反）

# 節 0: (x0 OR x1 OR x2)
#   x0=偽 AND x1=偽 AND x2=偽 のとき違反
#   ペナルティ = x[0] * x[1] * x[2]
c0 = x[0] * x[1] * x[2]

# 節 1: (~x0 OR x3 OR x4)
#   x0=真 AND x3=偽 AND x4=偽 のとき違反
#   ペナルティ = ~x[0] * x[3] * x[4]
c1 = ~x[0] * x[3] * x[4]

# 節 2: (x1 OR ~x2 OR ~x3)
#   x1=偽 AND x2=真 AND x3=真 のとき違反
#   ペナルティ = x[1] * ~x[2] * ~x[3]
c2 = x[1] * ~x[2] * ~x[3]

# 節 3: (~x1 OR ~x3 OR x4)
#   x1=真 AND x3=真 AND x4=偽 のとき違反
#   ペナルティ = ~x[1] * ~x[3] * x[4]
c3 = ~x[1] * ~x[3] * x[4]

# 節 4: (~x0 OR ~x1 OR ~x2)
#   x0=真 AND x1=真 AND x2=真 のとき違反
#   ペナルティ = ~x[0] * ~x[1] * ~x[2]
c4 = ~x[0] * ~x[1] * ~x[2]

# 節 5: (x0 OR x1 OR ~x4)
#   x0=偽 AND x1=偽 AND x4=真 のとき違反
#   ペナルティ = x[0] * x[1] * ~x[4]
c5 = x[0] * x[1] * ~x[4]

# 制約の合計: 節ペナルティの和
constraint = c0 + c1 + c2 + c3 + c4 + c5

constraint.simplify_as_binary()
solver = qbpp.EasySolver(constraint)
solver.target_energy(0)
sol = solver.search()

# 結果の出力
print(f"Energy = {sol.energy()}")
print("Assignment (True=0, False=1):")
for i in range(5):
    val = sol(x[i])
    print(f"  x[{i}] = {val} ({'True' if val == 0 else 'False'})")

# 各節の検証
print("Clause penalties:")
for i, c in enumerate([c0, c1, c2, c3, c4, c5]):
    print(f"  c{i} = {sol(c)}")
print(f"Violated clauses = {sol(constraint)}")
```

このプログラムでは、5 つのバイナリ変数を定義し、各節のペナルティ式を構築します。
正リテラル $x_i$ には `x[i]` を使用し、これはリテラルが偽のとき 1 になります。
負リテラル $\lnot x_i$ には `~x[i]` を使用し、これはリテラルが偽のとき（すなわち $x_i$ が真、つまり $x_i = 0$ のとき）1 になります。
PyQBPP は否定リテラル `~x[i]` をネイティブにサポートするため、`1 - x[i]` に手動で置き換える必要はありません。

節内のこれらの項の積は、節のすべてのリテラルが偽のとき、つまり節が違反されたときのみ 1 になります。
制約の合計はすべての節ペナルティの和であり、すべての節が充足されたときのみ 0 を達成します。

`simplify_as_binary()` を呼び出して冪等律 $x_i^2 = x_i$ を適用し式を簡約化した後、目標エネルギー 0 で EasySolver を用いて解きます。

### 出力結果
```
Energy = 0
Assignment (True=0, False=1):
  x[0] = 0 (True)
  x[1] = 1 (False)
  x[2] = 0 (True)
  x[3] = 1 (False)
  x[4] = 0 (True)
Clause penalties:
  c0 = 0
  c1 = 0
  c2 = 0
  c3 = 0
  c4 = 0
  c5 = 0
Violated clauses = 0
```

ソルバーはエネルギー 0 の充足割り当てを見つけます。これはすべての節が充足されていることを意味します。
ソルバーは確率的であるため、実際の割り当ては実行ごとに異なる場合があります。

</div>
