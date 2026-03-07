---
layout: default
title: "PyQUBO++ Introduction"
section: python
---

# PyQUBO++: Python Interface for QUBO++

PyQUBO++ is a Python wrapper for the QUBO++ library,
allowing you to model and solve combinatorial optimization problems
using QUBO/HUBO formulations directly from Python.

> **Note:** PyQUBO++ is currently in alpha.
> The API may change without notice, and there may be bugs.
> Please report any issues to the author.

## Features
- Symbolic construction of QUBO/HUBO expressions in Python
- Access to QUBO++ solvers (Easy Solver, Exhaustive Solver, ABS3)
- Familiar Python syntax with the full power of the QUBO++ engine

## Supported Environment
- Linux (Ubuntu 20.04 or later)
- x86_64 or arm64 (aarch64) CPUs
- Python 3.8 or later

## Installation

PyQUBO++ is available on [PyPI](https://pypi.org/project/pyqbpp/).
We recommend using a Python virtual environment (venv) to install PyQUBO++.
No sudo privileges are required.

```bash
$ python3 -m venv ~/qbpp-env
$ source ~/qbpp-env/bin/activate
$ pip install pyqbpp
```

After installation, activate your QUBO++ license.
Set the `QBPP_LICENSE_KEY` environment variable to your license key and run `qbpp-license -a`.
If `QBPP_LICENSE_KEY` is not set, an anonymous trial license will be activated.

```bash
$ export QBPP_LICENSE_KEY=[Your QUBO++ license key]
$ qbpp-license -a
```

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

# PyQUBO++ Documentation
- [Quick Reference: Variables and Expressions (PyQUBO++)](QR_VARIABLE)
- [Quick Reference: Operators and Functions (PyQUBO++)](QR_OPERATION)

More detailed documentation will be added soon.


