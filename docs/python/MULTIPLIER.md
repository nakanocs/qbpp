---
layout: default
nav_exclude: true
title: "Multiplier Simulation"
nav_order: 91
---
# Multiplier Simulation and Factorization
Multiplication of two integers can be performed using additions.
In this section, we design a multiplier for two 4-bit integers using full adders.
The figure below shows how two $x_3x_2x_1x_0$ and $y_3y_2y_1y_0$ are multiplied to obtain an 8-bit integer $z_7z_6z_5z_4z_3z_2z_1z_0$.
In this figure, $p_{i,j}=x_iy_j$ ($0\leq i,j\leq 3$) and these partial products are summed to compute the final 8-bit result.

<p align="center">
 <img src="../images/multiplication.svg" alt="4-bit multiplication" width="50%">
</p>

We use a 4-bit ripple-carry adder that computes the sum of two 4-bit integers
$a_3a_2a_1a_0$ and $b_3b_2b_1b_0$ producing the 5-bit sum $z_4z_3z_2z_1z_0$.
It consists of four full adders connected by a 5-bit carry wire $c_4c_3c_2c_1c_0$
that propagates carries.

<p align="center">
 <img src="../images/adder4.svg" alt="The 4-bit ripple carry adder" width="50%">
</p>

A 4-bit multiplier can be constructed using three 4-bit adders.
They are connected by wires $c_{i,j}$ ($0\leq i\leq 2, 0\leq j\leq 3$) to propagate intermediate sum bits, as shown below:
<p align="center">
 <img src="../images/multiplier.svg" alt="The 4-bit multiplier using three 4-bit adders" width="50%">
</p>

## QUBO formulation for multiplier
We will show QUBO formulation for simulating the `N`-bit multiplier.
To do this, we implement functions that construct a full adder, an adder, and a multiplier.

### Full adder
The following QUBO expression simulates a full adder with three input bits `a`, `b`, and `i`, and two output bits: carry-out `o` and sum `s`:
```python
def fa(a, b, i, o, s):
    return (a + b + i) - (2 * o + s) == 0
```
The function `fa` returns an expression that enforces consistency between the input and output bits of a full adder.

### Adder
Assume that lists `a`, `b`, and `s` represent integers.
We assume that `a` and `b` each have `N` elements representing `N`-bit integers, while `s` has `N + 1` elements representing an `(N + 1)`-bit integer.
The following function `adder` returns a QUBO expression whose minimum value is 0 if and only if `a + b == s` holds:
```python
def adder(a, b, s):
    N = len(a)
    c = var(N + 1)
    f = toExpr(0)
    for j in range(N):
        f += fa(a[j], b[j], c[j], c[j + 1], s[j])
    ml = MapList()
    ml.add(c[0], 0)
    ml.add(c[N], s[N])
    return replace(f, ml)
```
In this function, `c` is a vector of `N + 1` variables used to connect the carry-out and carry-in signals of the `fa` blocks, forming an `N`-bit ripple-carry adder.

### Multiplier
Assume that lists `x`, `y`, and `z` represent integers.
We assume that `x` and `y` each have `N` elements and that `z` has `2 * N` elements.
The following function `multiplier` returns a QUBO expression whose minimum value is 0 if and only if `x * y == z` holds.
```python
def multiplier(x, y, z):
    N = len(x)
    c = var("c", N - 1, N + 1)

    f = toExpr(0)

    for i in range(N - 1):
        b_vec = [x[i + 1] * y[j] for j in range(N)]

        if i == 0:
            a_vec = [x[0] * y[j + 1] for j in range(N - 1)] + [toExpr(0)]
        else:
            a_vec = [c[i - 1][j + 1] for j in range(N)]

        s_vec = [c[i][j] for j in range(N + 1)]
        f += adder(a_vec, b_vec, s_vec)

    f += z[0] - x[0] * y[0] == 0

    ml = MapList()
    for i in range(N - 2):
        ml.add(c[i][0], z[i + 1])
    for i in range(N + 1):
        ml.add(c[N - 2][i], z[N + i - 1])
    f = replace(f, ml)
    f.simplify_as_binary()
    return f
```
This function uses an `(N−1)×(N+1)` matrix `c` of variables to connect the `N−1` adders of `N` bits.
Since each bit of `z` corresponds to one element of `c`, their correspondence is defined in `ml`, and the replacements are performed using `replace()`.

## PyQBPP program for factorization
Using the function `multiplier`, we can factor a composite integer into two factors.
The following program constructs a 4-bit multiplier with
- `x`: 4 binary variables,
- `y`: 4 binary variables,
- `z`: a list of constants `[1, 1, 1, 1, 0, 0, 0, 1]`, representing the 8-bit integer `10001111` `(143)`, and stores the resulting expression in `f`:
```python
from pyqbpp import var, toExpr, replace, MapList, EasySolver

def fa(a, b, i, o, s):
    return (a + b + i) - (2 * o + s) == 0

def adder(a, b, s):
    N = len(a)
    c = var(N + 1)
    f = toExpr(0)
    for j in range(N):
        f += fa(a[j], b[j], c[j], c[j + 1], s[j])
    ml = MapList()
    ml.add(c[0], 0)
    ml.add(c[N], s[N])
    return replace(f, ml)

def multiplier(x, y, z):
    N = len(x)
    c = var("c", N - 1, N + 1)

    f = toExpr(0)

    for i in range(N - 1):
        b_vec = [x[i + 1] * y[j] for j in range(N)]

        if i == 0:
            a_vec = [x[0] * y[j + 1] for j in range(N - 1)] + [toExpr(0)]
        else:
            a_vec = [c[i - 1][j + 1] for j in range(N)]

        s_vec = [c[i][j] for j in range(N + 1)]
        f += adder(a_vec, b_vec, s_vec)

    f += z[0] - x[0] * y[0] == 0

    ml = MapList()
    for i in range(N - 2):
        ml.add(c[i][0], z[i + 1])
    for i in range(N + 1):
        ml.add(c[N - 2][i], z[N + i - 1])
    f = replace(f, ml)
    f.simplify_as_binary()
    return f

x = var("x", 4)
y = var("y", 4)
z = [1, 1, 1, 1, 0, 0, 0, 1]
f = multiplier(x, y, z)
f.simplify_as_binary()

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

x_bits = "".join(str(sol.get(x[j])) for j in reversed(range(4)))
y_bits = "".join(str(sol.get(y[j])) for j in reversed(range(4)))
z_bits = "".join(str(z[j]) for j in reversed(range(8)))
print(f"{x_bits} * {y_bits} = {z_bits}")
```
The Easy Solver is executed on `f`, and the obtained solution is stored in `sol`.
The resulting values of `x` and `y` are printed as:
```
1011 * 1101 = 10001111
```
This output indicates $11\times 13 = 143$, demonstrating the factorization result.
