---
layout: default
nav_exclude: true
title: "Pythagorean Triples"
nav_order: 40
---
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

| C++ QUBO++                   | PyQBPP                              |
|------------------------------|---------------------------------------|
| `1 <= qbpp::var_int("x") <= 16` | `between(var_int("x"), 1, 16)` |
| `1 <= y - x <= +qbpp::inf`  | `between(y - x, 1, 15)`              |
| `sol(x)`                     | `sol.eval(x)`                         |
| `sol(*f)`                    | `sol.eval(f.body)`                    |
