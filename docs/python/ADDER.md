---
layout: default
nav_exclude: true
title: "Adder Simulation"
nav_order: 90
---
# Adder Simulation

## Full adder and ripple carry adder
A full adder has three input bits: $a$, $b$, and $i$ (carry-in) and
$o$ (carry-out) and $s$ (sum).
The sum of the three input bits is represented using these two output bits.

A ripple-carry adder computes the sum of two multi-bit integers by cascading multiple full adders, as illustrated below:
<p align="center">
 <img src="../images/adder.svg" alt="4-bit ripple carry adder" width="50%">
</p>

This ripple-carry adder computes the sum of two 4-bit integers $x_3x_2x_1x_0$ and $y_3y_2y_1y_0$
and outputs the 4-bit sum$z_3z_2z_1z_0$ using four full adders.
The corresponding 5-bit carry signals $c_4c_3c_2c_1c_0$ are also shown.

## QUBO formulation for full adder
A full adder can be formulated using the following expression:

$$
\begin{aligned}
fa(a,b,i,c,s) &=((a+b+i)-(2o+s))^2
\end{aligned}
$$

This expression attains its minimum value of 0 if and only if the five variables take values consistent with a valid full-adder operation.
The following PyQBPP program verifies this formulation using the exhaustive solver:
```python
from pyqbpp import var, replace, MapList, ExhaustiveSolver

a = var("a")
b = var("b")
i = var("i")
o = var("o")
s = var("s")
fa = (a + b + i) - (2 * o + s) == 0
fa.simplify_as_binary()
solver = ExhaustiveSolver(fa)
sols = solver.search_optimal_solutions()
for idx, sol in enumerate(sols):
    vals = {v: sol.get(v) for v in [a, b, i, o, s]}
    print(f"({idx}) {sol.energy()}: a={vals[a]}, b={vals[b]}, i={vals[i]}, o={vals[o]}, s={vals[s]}")
```
In this program, the constraint $fa(a,b,i,c,s)$ is implemented using the equality operator `==`, which intuitively represents the constraint $a+b+i=2o+s$.
The program produces the following output, confirming that the expression correctly models a full adder:
```
(0) 0: a=0, b=0, i=0, o=0, s=0
(1) 0: a=0, b=0, i=1, o=0, s=1
(2) 0: a=0, b=1, i=0, o=0, s=1
(3) 0: a=0, b=1, i=1, o=1, s=0
(4) 0: a=1, b=0, i=0, o=0, s=1
(5) 0: a=1, b=0, i=1, o=1, s=0
(6) 0: a=1, b=1, i=0, o=1, s=0
(7) 0: a=1, b=1, i=1, o=1, s=1
```

If some bits are fixed, the valid values of the remaining bits can be derived.
For example, the three input bits can be fixed using the `replace()` function:
```python
ml = MapList()
ml.add(a, 1)
ml.add(b, 1)
ml.add(i, 0)
fa2 = replace(fa, ml)
fa2.simplify_as_binary()
solver2 = ExhaustiveSolver(fa2)
sols2 = solver2.search_optimal_solutions()
for idx, sol in enumerate(sols2):
    print(f"({idx}) {sol.energy()}: o={sol.get(o)}, s={sol.get(s)}")
```

The program then produces the following output:
```
(0) 0: o=1, s=0
```

## Simulating a ripple carry adder using multiple full adders
Using the QUBO expression for a full adder, we can construct a QUBO expression that simulates a ripple-carry adder.
The following PyQBPP program creates a QUBO expression for simulating a 4-bit adder by combining four full adders:
```python
from pyqbpp import var, replace, MapList, ExhaustiveSolver

a = var("a")
b = var("b")
i = var("i")
o = var("o")
s = var("s")
fa = (a + b + i) - (2 * o + s) == 0

x = var("x", 4)
y = var("y", 4)
c = var("c", 5)
z = var("z", 4)

ml0 = MapList()
ml0.add(a, x[0]); ml0.add(b, y[0]); ml0.add(i, c[0]); ml0.add(o, c[1]); ml0.add(s, z[0])
fa0 = replace(fa, ml0)

ml1 = MapList()
ml1.add(a, x[1]); ml1.add(b, y[1]); ml1.add(i, c[1]); ml1.add(o, c[2]); ml1.add(s, z[1])
fa1 = replace(fa, ml1)

ml2 = MapList()
ml2.add(a, x[2]); ml2.add(b, y[2]); ml2.add(i, c[2]); ml2.add(o, c[3]); ml2.add(s, z[2])
fa2 = replace(fa, ml2)

ml3 = MapList()
ml3.add(a, x[3]); ml3.add(b, y[3]); ml3.add(i, c[3]); ml3.add(o, c[4]); ml3.add(s, z[3])
fa3 = replace(fa, ml3)

adder = fa0 + fa1 + fa2 + fa3
adder.simplify_as_binary()

solver = ExhaustiveSolver(adder)
sols = solver.search_optimal_solutions()
print(f"Number of valid solutions: {len(sols)}")
for idx in [0, 1, len(sols)-2, len(sols)-1]:
    sol = sols[idx]
    xv = "".join(str(sol.get(x[j])) for j in range(4))
    yv = "".join(str(sol.get(y[j])) for j in range(4))
    cv = "".join(str(sol.get(c[j])) for j in range(5))
    zv = "".join(str(sol.get(z[j])) for j in range(4))
    print(f"({idx}) x={xv}, y={yv}, c={cv}, z={zv}")
```
In this program, four full-adder expressions are created using the `replace()` function and combined into a single expression, `adder`.
The Exhaustive Solver is then used to enumerate all optimal solutions.

This program produces 512 valid solutions, corresponding to all possible input combinations of a 4-bit adder:
```
Number of valid solutions: 512
(0) x=0000, y=0000, c=00000, z=0000
(1) x=0000, y=0000, c=10000, z=1000
(510) x=1111, y=1111, c=01111, z=0111
(511) x=1111, y=1111, c=11111, z=1111
```

Alternatively, we can define a Python function `fa` to construct full-adder constraints in a more concise manner:
```python
from pyqbpp import var, toExpr, replace, MapList, ExhaustiveSolver

def fa(a, b, i, o, s):
    return (a + b + i) - (2 * o + s) == 0

x = var("x", 4)
y = var("y", 4)
c = var("c", 5)
z = var("z", 4)

adder = toExpr(0)
for j in range(4):
    adder += fa(x[j], y[j], c[j], c[j + 1], z[j])

ml = MapList()
ml.add(c[0], 0)
ml.add(c[4], 0)
adder = replace(adder, ml)
adder.simplify_as_binary()

solver = ExhaustiveSolver(adder)
sols = solver.search_optimal_solutions()
print(f"Number of valid solutions: {len(sols)}")
```
This program produces 136 valid solutions (carry-in and carry-out are both fixed to 0, so only pairs with $x + y \leq 15$ are valid).
