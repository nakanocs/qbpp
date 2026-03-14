---
layout: default
title: "Exhaustive Solver"
nav_order: 20
parent: "PyQBPP (Python)"
---
# Exhaustive Solver Usage
The **Exhaustive Solver** is a complete-search solver for QUBO/HUBO expressions.
Since all possible assignments are examined, the optimality of the solutions is guaranteed.
The search is parallelized using CPU threads, and if a CUDA GPU is available, GPU acceleration is automatically enabled to further speed up the search.

Solving a problem with the Exhaustive Solver consists of the following three steps:
1. Create an **`ExhaustiveSolver`** object.
2. Set search options by calling methods of the solver object.
3. Search for solutions by calling one of the search methods.


## Creating Exhaustive Solver object
To use the Exhaustive Solver, an **`ExhaustiveSolver`** object is constructed with an expression
(`Expr`) object as follows:
- **`ExhaustiveSolver(f)`**:
Here, `f` is the expression to be solved.
It must be simplified as a binary expression in advance by calling the
`simplify_as_binary()` method.

## Setting Exhaustive Solver Options
- **`verbose()`**:
Displays the search progress as a percentage, which is helpful for estimating the total runtime.
- **`callback(func)`**:
Sets a callback function that is called when a new best solution is found.
The callback receives two arguments: `energy` (int) and `tts` (float, time to solution in seconds).
- **`target_energy(energy)`**:
Sets a target energy value for early termination.
When the solver finds a solution with energy less than or equal to the target, the search terminates immediately.

## Searching Solutions
The Exhaustive Solver searches for solutions by calling one of the following
methods of the solver object:
- **`search()`**: Returns the best solution found. If a CUDA GPU is available, the search is automatically accelerated using the GPU alongside CPU threads.
- **`search_optimal_solutions()`**: Returns a list of all optimal solutions (i.e., solutions with the minimum energy), sorted by energy.
- **`search_topk_solutions(k)`**: Returns a list of the top-k solutions with the lowest energy, sorted in increasing order of energy.
- **`search_all_solutions()`**: Returns a list of all solutions, sorted in increasing order of energy.

# Program example
The following program searches for a solution to the
**Low Autocorrelation Binary Sequences (LABS)** problem using the Exhaustive
Solver:
```python
from pyqbpp import var, expr, sqr, ExhaustiveSolver

size = 20
x = var("x", size)
f = expr()
for d in range(1, size):
    temp = expr()
    for i in range(size - d):
        temp += (2 * x[i] - 1) * (2 * x[i + d] - 1)
    f += sqr(temp)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
solver.callback(lambda energy, tts: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
bits = "".join("-" if sol.get(i) == 0 else "+" for i in range(size))
print(f"{sol.energy()}: {bits}")
```
The output of this program is as follows:
{% raw %}
```txt
TTS = 0.002s Energy = 1786
TTS = 0.003s Energy = 314
TTS = 0.003s Energy = 206
TTS = 0.003s Energy = 154
TTS = 0.003s Energy = 102
TTS = 0.003s Energy = 94
TTS = 0.003s Energy = 74
TTS = 0.003s Energy = 66
TTS = 0.003s Energy = 50
TTS = 0.006s Energy = 46
TTS = 0.011s Energy = 34
TTS = 0.014s Energy = 26
26: -++---++-+---+-+++++
```
{% endraw %}
All optimal solutions can be obtained by calling the
**`search_optimal_solutions()`** method as follows:
```python
solver = ExhaustiveSolver(f)
opts = solver.search_optimal_solutions()
for s in opts:
    bits = "".join("-" if s.get(i) == 0 else "+" for i in range(size))
    print(f"{s.energy()}: {bits}")
```
The output is as follows:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
```
{% endraw %}
The top-k solutions with the lowest energy can be obtained by calling the
**`search_topk_solutions(k)`** method as follows:
```python
solver = ExhaustiveSolver(f)
topk = solver.search_topk_solutions(10)
for s in topk:
    bits = "".join("-" if s.get(i) == 0 else "+" for i in range(size))
    print(f"{s.energy()}: {bits}")
```
The output is as follows:
{% raw %}
```txt
26: -----+-+++-+--+++--+
26: --++-++----+----+-+-
26: -+-+----+----++-++--
26: -++---++-+---+-+++++
26: +--+++--+-+++-+-----
26: +-+-++++-++++--+--++
26: ++--+--++++-++++-+-+
26: +++++-+---+-++---++-
34: ----+----++-++---+-+
34: +-++-+-+++-+++-----+
```
{% endraw %}
Furthermore, all solutions, including non-optimal ones, can be obtained by calling
the **`search_all_solutions()`** method.
Note that this function stores all $2^n$ solutions in memory, where $n$ is the number of variables.
For example, with $n = 20$, over one million solutions are stored, and memory usage grows exponentially with $n$.
Use this function only when $n$ is small enough.
```python
solver = ExhaustiveSolver(f)
all_sols = solver.search_all_solutions()
for s in all_sols:
    bits = "".join("-" if s.get(i) == 0 else "+" for i in range(size))
    print(f"{s.energy()}: {bits}")
```
This prints all $2^{20}$ solutions in increasing order of energy.
