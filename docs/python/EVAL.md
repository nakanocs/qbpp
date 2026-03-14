---
layout: default
title: "Evaluating Expressions"
nav_order: 16
parent: "PyQBPP"
---
# Evaluating Expressions

## Evaluation using MapList
The value of an expression can be computed by providing an assignment of values to all variables
as a list of pairs.
A list can be defined as a **`MapList`** object.

The following program computes the function $f(x,y,z)$ for $(x,y,z)=(0,1,1)$:
```python
from pyqbpp import var, sqr, MapList

x = var("x")
y = var("y")
z = var("z")
f = sqr(x + 2 * y + 3 * z - 3)

ml = MapList([(x, 0), (y, 1), (z, 1)])
print("f(0,1,1) =", f.eval(ml))
```
In this program, a `MapList` object `ml` is defined with the assignment $x=0$, $y=1$, $z=1$.
Then `f.eval(ml)` returns the value of $f(0,1,1)$.
This program displays the following output:
```
f(0,1,1) = 4
```

## Evaluation using Sol
A solution object (**`Sol`**) can also be used to evaluate the value of an expression.
To do this, we first construct a `Sol` object associated with a given expression.
The newly created `Sol` object is initialized with the all-zero assignment.

Using the **`set()`** method, we can assign values to individual variables.
Then **`sol.eval(f)`** returns the value of the expression `f` under the assignment stored in `sol`.

```python
from pyqbpp import var, sqr, Sol

x = var("x")
y = var("y")
z = var("z")
f = sqr(x + 2 * y + 3 * z - 3)
f.simplify_as_binary()

sol = Sol(f)
sol.set(y, 1)
sol.set(z, 1)

print("f(0,1,1) =", sol.eval(f))
```

The method **`comp_energy()`** computes the energy value and caches it internally.
A solution object returned by a solver already has its energy cached.
To retrieve the cached energy, use the **`energy()`** method:
```python
from pyqbpp import var, sqr, EasySolver

x = var("x")
y = var("y")
z = var("z")
f = sqr(x + 2 * y + 3 * z - 4)
f.simplify_as_binary()

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print(sol)
print("energy =", sol.energy())
```
This program produces the following output:
```
Sol(energy=0, x=1, y=0, z=1)
energy = 0
```

After modifying a solution (e.g., using `flip()`), the cached energy becomes invalid.
You must explicitly recompute it by calling **`comp_energy()`**:
```python
sol.flip(z)
print("comp_energy =", sol.comp_energy())
print("energy =", sol.energy())
```
