---
layout: default
title: "PyQBPP"
nav_order: 50
has_children: true
---
# PyQBPP: Python Interface for QUBO++

PyQBPP is a Python wrapper for the QUBO++ library,
allowing you to model and solve combinatorial optimization problems
using QUBO/HUBO formulations directly from Python.

> **Note:** PyQBPP is currently in alpha.
> The API may change without notice, and there may be bugs.
> Please report any issues to the author.

## Features
- Symbolic construction of QUBO/HUBO expressions in Python
- Access to QUBO++ solvers (Easy Solver, Exhaustive Solver, ABS3)
- Familiar Python syntax with the full power of the QUBO++ engine

For installation instructions, see [**Installation**](INSTALL).

## Quick Example

The following program finds an 8×8 binary matrix where each row and each column contains exactly one 1 (a one-hot constraint).

```python
from pyqbpp import EasySolver, var, sum, vector_sum

n = 8
x = var("x", n, n)

# Each row and column has exactly one 1
f = sum(vector_sum(x, 0) == 1) + sum(vector_sum(x, 1) == 1)

f.simplify_as_binary()

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

for i in range(n):
    print([sol(x[i][j]) for j in range(n)])
```

Example output:
```
[0, 0, 0, 0, 0, 1, 0, 0]
[1, 0, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 1, 0, 0, 0, 0]
[0, 0, 1, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 1, 0]
[0, 1, 0, 0, 0, 0, 0, 0]
[0, 0, 0, 0, 1, 0, 0, 0]
[0, 0, 0, 0, 0, 0, 0, 1]
```

