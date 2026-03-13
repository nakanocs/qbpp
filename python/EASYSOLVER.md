---
section: python
layout: default
title: "Easy Solver (Python)"
---

# Easy Solver Usage
The **Easy Solver** is a heuristic solver for QUBO/HUBO expressions.

Solving a problem with the Easy Solver consists of the following three steps:
1. Create an **`EasySolver`** object.
2. Set search options by calling methods of the solver object.
3. Search for solutions by calling the **`search()`** method, which returns a **`Sol`** object.

## Creating Easy Solver object
To use the Easy Solver, an `EasySolver` object is constructed with an expression as follows:
- **`EasySolver(f)`**

Here, `f` is the expression to be solved.
It must be simplified as a binary expression in advance by calling `simplify_as_binary()`.

## Setting Easy Solver Options
- **`time_limit(time)`**: Specifies the time limit in seconds. The default value is 10.0 seconds.
If the time limit is set to 0, the solver never terminates due to the time limit.
- **`target_energy(energy)`**: Specifies the target energy. The solver terminates when a solution with energy less than or equal to the target is found.
- **`callback(func)`**: Sets a callback function that is called when a new best solution is found. The callback receives two arguments: `energy` (int) and `tts` (float, time to solution in seconds).
- **`thread_count(n)`**: Sets the number of threads.

## Searching Solutions
The Easy Solver searches for solutions by calling the **`search()`** method.

## Program Example
The following program searches for a solution to the
**Low Autocorrelation Binary Sequences (LABS)** problem using the Easy Solver:
```python
from pyqbpp import var, expr, sqr, EasySolver

size = 100
x = var("x", size)
f = expr()
for d in range(1, size):
    temp = expr()
    for i in range(size - d):
        temp += (2 * x[i] - 1) * (2 * x[i + d] - 1)
    f += sqr(temp)
f.simplify_as_binary()

solver = EasySolver(f)
solver.time_limit(5.0)
solver.target_energy(900)
solver.callback(lambda energy, tts: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
bits = "".join("-" if sol.get(i) == 0 else "+" for i in range(size))
print(f"{sol.energy()}: {bits}")
```
In this example, the following options are set:
- a 5.0-second time limit,
- a target energy of 900, and
- a callback that prints the energy and TTS whenever a new best solution is found.

Therefore, the solver terminates either when the elapsed time reaches 5.0 seconds
or when a solution with energy 900 or less is found.

For example, this program produces the following output:
{% raw %}
```txt
TTS = 0.000s Energy = 300162
TTS = 0.000s Energy = 273350
...
TTS = 2.691s Energy = 898
898: ++-++-----+--+--++++++---++-+-+--++-------++-++-+-+-+-+-++-++++-++-+++++-+-+--++++++---+++--+++---++
```
{% endraw %}
