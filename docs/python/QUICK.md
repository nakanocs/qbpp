---
layout: default
title: "PyQBPP Quick Start"
nav_order: 4
---
# Quick Start

This page provides an overview of the Quick Start procedure for PyQBPP.

## Installation

Install PyQBPP by following the instructions in [**Installation**](INSTALL).

## Create and run a sample program

### Create a PyQBPP sample program
Create a PyQBPP sample program below and save as file **`test.py`**:
```python
from pyqbpp import var_int, ExhaustiveSolver

x = var_int("x", 0, 10)
y = var_int("y", 0, 10)

f = (x + y == 10) + (2 * x + 4 * y == 28)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"x = {sol(x)}, y = {sol(y)}")
```

### Run the program
Run `test.py` as follows:
```bash
python3 test.py
x = 6, y = 4
```

## Next steps
1. Activate your license. See [**Installation**](INSTALL) for details.
2. Learn the basics of PyQBPP. Start with **Basics** in [**PyQBPP (Python)**](./).
