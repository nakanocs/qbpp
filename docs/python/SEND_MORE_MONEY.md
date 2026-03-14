---
layout: default
nav_exclude: true
title: "SEND+MORE=MONEY"
nav_order: 82
---
# Math Puzzle: SEND MORE MONEY

**SEND + MORE = MONEY** is a famous alphametic puzzle: assign a decimal digit to each letter so that
$$
\text{SEND}+\text{MORE}=\text{MONEY}
$$

The constraints are:
- The digits assigned to letters are all distinct.
- `S` and `M` must not be 0.

## QUBO formulation

We assign a unique index to each letter as follows:

| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| letter | S | E | N | D | M | O | R | Y |

Let $I(\alpha)$ denote the index of letter $\alpha$ ($\in \lbrace S,E,N,D,M,O,R,Y\rbrace$).
We use an $8\times 10$ binary matrix $X=(x_{i,j})$ $(0\leq i\leq 7, 0\leq j\leq 9)$ to represent the digit assigned to each letter:  $x_{I(\alpha),j}=1$ if and only if letter $\alpha$ is assigned digit $j$.

### One-hot constraints (each letter takes exactly one digit)
Each row of $X$ must be one-hot:

$$
\begin{aligned}
\text{onehot} &=\sum_{i=0}^{7}\Bigl(\sum_{j=0}^{9}x_{i,j}=1\Bigr) \\
              &=\sum_{i=0}^{7}\Bigl(1-\sum_{j=0}^{9}x_{i,j}\Bigr)^2
\end{aligned}
$$

The value of $\text{onehot}$ is minimized to 0 if and only if every row is one-hot.

### All-different constraints (no two letters share the same digit)
Digits must be distinct across letters, i.e., no two rows choose the same column:
$$
\begin{aligned}
\text{different} &=\sum_{0\leq i<j\leq 7}\sum_{k=0}^9x_{i,k}x_{j,k}
\end{aligned}
$$

### Encoding the words as linear expressions
The values of $\text{SEND}$, $\text{MORE}$, and $\text{MONEY}$ are represented
by:

$$
\begin{aligned}
\text{SEND} &= \sum_{k=0}^9k(1000x_{I(S),k}+100x_{I(E),k}+10x_{I(N),k}+x_{I(D),k})\\
\text{MORE} &= \sum_{k=0}^9k(1000x_{I(M),k}+100x_{I(O),k}+10x_{I(R),k}+x_{I(E),k})\\
\text{MONEY} &= \sum_{k=0}^9k(10000x_{I(M),k}+ 1000x_{I(O),k}+100x_{I(N),k}+10x_{I(E),k}+x_{I(Y),k})
\end{aligned}
$$

### Equality constraint
We enforce the equation by penalizing the residual:

$$
\begin{aligned}
\text{equal} &= \Bigl(\text{SEND}+\text{MORE} = \text{MONEY}\Bigr) \\
             &= \Bigl(\text{SEND}+\text{MORE} - \text{MONEY}\Bigr)^2
\end{aligned}
$$

### Combined objective
All constraints are combined into a single objective:

$$
\begin{aligned}
f & = P\cdot (\text{onehot}+\text{different})+\text{equal}
\end{aligned}
$$

where
`P` is a sufficiently large constant to prioritize feasibility (`onehot` and `different`).

Finally, since $\text{S}$ and $\text{M}$ must not be 0, we fix
the binary variables as follows:
$$
x_{I(S),0} = x_{I(M),0}= 0
$$

## PyQBPP program for SEND+MORE=MONEY
The following PyQBPP program implements the QUBO formulation above and finds a solution using EasySolver:
```python
from pyqbpp import var, sum, vector_sum, toExpr, replace, MapList, Sol, EasySolver

LETTERS = "SENDMORY"
L = len(LETTERS)

def I(c):
    return LETTERS.index(c)

x = var("x", L, 10)

onehot = sum(vector_sum(x) == 1)

different = toExpr(0)
for i in range(L - 1):
    for j in range(i + 1, L):
        different += sum(x[i] * x[j])

send = toExpr(0)
more = toExpr(0)
money = toExpr(0)
for k in range(10):
    send += k * (1000 * x[I('S')][k] + 100 * x[I('E')][k] + 10 * x[I('N')][k] + x[I('D')][k])
    more += k * (1000 * x[I('M')][k] + 100 * x[I('O')][k] + 10 * x[I('R')][k] + x[I('E')][k])
    money += k * (10000 * x[I('M')][k] + 1000 * x[I('O')][k] + 100 * x[I('N')][k] + 10 * x[I('E')][k] + x[I('Y')][k])

equal = send + more - money == 0

P = 10000
f = P * (onehot + different) + equal
f.simplify_as_binary()

ml = MapList()
ml.add(x[I('S')][0], 0)
ml.add(x[I('M')][0], 0)
g = replace(f, ml)
g.simplify_as_binary()

solver = EasySolver(g)
solver.target_energy(0)
sol = solver.search()

full_sol = Sol(f)
full_sol.set(sol)
full_sol.set(ml)

print(f"onehot = {full_sol.eval(onehot)}")
print(f"different = {full_sol.eval(different)}")
print(f"equal = {full_sol.eval(equal)}")

val = [next((k for k in range(10) if full_sol.get(x[i][k]) == 1), -1) for i in range(L)]

def digit_str(d):
    return "*" if d < 0 else str(d)

print("SEND + MORE = MONEY")
print(f"{digit_str(val[I('S')])}{digit_str(val[I('E')])}{digit_str(val[I('N')])}{digit_str(val[I('D')])} + "
      f"{digit_str(val[I('M')])}{digit_str(val[I('O')])}{digit_str(val[I('R')])}{digit_str(val[I('E')])} = "
      f"{digit_str(val[I('M')])}{digit_str(val[I('O')])}{digit_str(val[I('N')])}{digit_str(val[I('E')])}{digit_str(val[I('Y')])}")
```
In this program, `LETTERS` assigns an integer index to each letter in `"SENDMORY"`, which is used to implement $I(\alpha)$.
We define an `L`$\times$`10` matrix `x` of binary variables (here $L=8$).
The expressions `onehot`, `different`, and `equal` are computed according to the formulation and combined into a single objective `f` with a penalty weight `P`.

We use a `MapList` object `ml` to fix `x[I('S')][0]` and `x[I('M')][0]` to 0, and create a reduced expression `g` by applying this replacement.
The solver is run on `g`, and the resulting assignment `sol` is merged with the fixed mapping `ml` to produce `full_sol` for the original objective `f`.

Finally, the one-hot rows are decoded into digits, and the program prints the obtained solution.

> **Note:** Unlike the C++ version, Python has unlimited precision integers, so there is no need for `COEFF_TYPE=qbpp::int128_t`.

This program produces the following output:
```
onehot = 0
different = 0
equal = 0
SEND + MORE = MONEY
9567 + 1085 = 10652
```
This confirms that all constraints are satisfied and the correct solution is obtained.
