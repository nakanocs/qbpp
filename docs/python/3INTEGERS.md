---
layout: default
nav_exclude: true
title: "Find Three Integers"
nav_order: 44
---
# Math Problem: Find Three Integers

The following math problem can be solved using PyQBPP.

### Problem
Find integers $x$, $y$, $z$ that satisfy:

$$
\begin{aligned}
\frac{1}{x}+\frac{1}{y}+\frac{1}{z} = 1\\
1 < x < y < z
\end{aligned}
$$



### PyQBPP program

Since PyQBPP can handle polynomial expressions, we first rewrite the constraints.
Multiplying both sides of the first constraint by $xyz$ yields:

$$
xy+yz+zx - xyz = 0
$$

The strict inequalities $x<y<z$ can be encoded as

$$
\begin{aligned}
1 &\leq y-x \\
1 &\leq z-y
\end{aligned}
$$

The following PyQBPP program formulates these constraints as a HUBO expression and solves it using the Exhaustive Solver:

```python
from pyqbpp import var_int, between, ExhaustiveSolver

x = between(var_int("x"), 1, 10)
y = between(var_int("y"), 1, 10)
z = between(var_int("z"), 1, 10)

c1 = x * y + y * z + z * x - x * y * z == 0
c2 = between(y - x, 1, 9)
c3 = between(z - y, 1, 9)

f = c1 + c2 + c3
f.simplify_as_binary()
solver = ExhaustiveSolver(f)
sols = solver.search_optimal_solutions()

seen = set()
for sol in sols:
    key = (sol.eval(x), sol.eval(y), sol.eval(z))
    if key not in seen:
        seen.add(key)
        xv, yv, zv = key
        print(f"(x,y,z) = ({xv}, {yv}, {zv})")
```
The three constraints are encoded as `c1`, `c2`, and `c3`, and combined into a single objective `f`.
The Exhaustive Solver searches for optimal solutions of f and prints the resulting
$(x,y,z)$ tuples.

Because `f` introduces auxiliary variables during binary simplification, the same
$(x,y,z)$ assignment may appear multiple times in the returned solution set.
Therefore, we use a `set` to remove duplicates before printing.

This program produces the following output:
```
(x,y,z) = (2, 3, 6)
```
This indicates that the problem has exactly one solution in the searched range, namely $(x,y,z)=(2,3,6)$.
