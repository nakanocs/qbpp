---
section: python
layout: default
title: "Replace Functions (Python)"
---

# Replace Functions

PyQBPP provides the following replace function, which can be used to fix variable values in an expression:
- **`replace(f, ml)`**: Returns a new expression in which variables are replaced according to the mapping `ml`.
- **`f.replace(ml)`**: Replaces variables in expression `f` in place.

## Using the replace function to fix variable values
We explain the `replace()` function using the
[partitioning problem](PARTITION).
This program finds a partition of the numbers in a list into two subsets $P$ and $Q$ such that the difference between their sums is minimized.

We modify this problem so that 64 must belong to $P$ and 27 must belong to $Q$:

```python
from pyqbpp import var, sum, sqr, replace, MapList, Sol, ExhaustiveSolver

w = [64, 27, 47, 74, 12, 83, 63, 40]
x = var("x", len(w))
p = sum([w[i] * x[i] for i in range(len(w))])
q = sum([w[i] * (1 - x[i]) for i in range(len(w))])
f = sqr(p - q)
f.simplify_as_binary()

ml = MapList([(x[0], 1), (x[1], 0)])
g = replace(f, ml)
g.simplify_as_binary()

solver = ExhaustiveSolver(g)
sol = solver.search()

full_sol = Sol(f)
full_sol.set(sol)
full_sol.set(ml)

print("energy =", full_sol.comp_energy())
P = [w[i] for i in range(len(w)) if full_sol.get(x[i]) == 1]
Q = [w[i] for i in range(len(w)) if full_sol.get(x[i]) == 0]
print("P:", P)
print("Q:", Q)
```
In this program, a `MapList` object **`ml`** fixes `x[0]=1` (64 in $P$) and `x[1]=0` (27 in $Q$).
The `replace()` function substitutes these values into `f`, and the Exhaustive Solver finds the optimal partition for the remaining variables.

This program produces the following output:
```
energy = 4
P: [64, 47, 12, 83]
Q: [27, 74, 63, 40]
```

## Using the replace function to replace variables with expressions
The `replace()` function can also replace a variable with an expression.

For example, to ensure that 64 and 27 are placed in distinct subsets,
we replace `x[0]` with `1 - x[1]` so they always take opposite values:

```python
ml = MapList([(x[0], 1 - x[1])])
g = replace(f, ml)
g.simplify_as_binary()

solver = ExhaustiveSolver(g)
sol = solver.search()

full_sol = Sol(f)
full_sol.set(sol, ml)

print("energy =", full_sol.comp_energy())
P = [w[i] for i in range(len(w)) if full_sol.get(x[i]) == 1]
Q = [w[i] for i in range(len(w)) if full_sol.get(x[i]) == 0]
print("P:", P)
print("Q:", Q)
```
This program produces the following output:
```
energy = 4
P: [64, 47, 12, 83]
Q: [27, 74, 63, 40]
```

## Replace functions for integer variables
Integer variables can be replaced with fixed integer values using the `replace()` function.

Here, we demonstrate this feature using a simple **multiplication/factorization** example.
Let $p$, $q$, and $r$ be integer variables with the constraint $p\times q - r = 0$.

### Multiplication
Fix $p=5$ and $q=7$ to find $r=35$:
```python
from pyqbpp import var_int, between, replace, MapList, Sol, EasySolver

p = between(var_int("p"), 2, 8)
q = between(var_int("q"), 2, 8)
r = between(var_int("r"), 2, 40)
f = p * q - r == 0
f.simplify_as_binary()

ml = MapList([(p, 5), (q, 7)])
g = replace(f, ml)
g.simplify_as_binary()

solver = EasySolver(g)
solver.target_energy(0)
sol = solver.search()

full_sol = Sol(f)
full_sol.set(sol)
full_sol.set(ml)
print(f"p={full_sol.eval(p)}, q={full_sol.eval(q)}, r={full_sol.eval(r)}")
```
This program produces the following output:
```
p=5, q=7, r=35
```

### Factorization
Fix $r=35$ to find $p$ and $q$:
```python
ml = MapList([(r, 35)])
g = replace(f, ml)
g.simplify_as_binary()
# ... same solver setup ...
```

### Division
Fix $p=5$ and $r=35$ to find $q=7$:
```python
ml = MapList([(p, 5), (r, 35)])
g = replace(f, ml)
g.simplify_as_binary()
# ... same solver setup ...
```

> **NOTE**
> - **`f.replace(ml)`** updates the expression `f` in place.
> - **`replace(f, ml)`** returns a new expression without modifying the original.
