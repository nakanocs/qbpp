---
section: python
layout: default
title: "ABS3 Solver (Python)"
---

# ABS3 Solver Usage
Solving an expression `f` using the ABS3 Solver involves the following three steps:
1. Create an **`ABS3Solver`** object for the expression `f`.
2. Set search options by calling methods of the solver object.
3. Call the **`search()`** method, which returns the obtained solution.

## Solving LABS problem using the ABS3 Solver
The following program solves the **Low Autocorrelation Binary Sequence (LABS)** problem using the ABS3 Solver:
```python
from pyqbpp import var, expr, sqr, ABS3Solver

size = 100
x = var("x", size)
f = expr()
for d in range(1, size):
    temp = expr()
    for i in range(size - d):
        temp += (2 * x[i] - 1) * (2 * x[i + d] - 1)
    f += sqr(temp)
f.simplify_as_binary()

solver = ABS3Solver(f)
solver.time_limit(10.0)
solver.callback(lambda energy, tts, event: print(f"TTS = {tts:.3f}s Energy = {energy}"))
sol = solver.search()
bits = "".join("-" if sol.get(i) == 0 else "+" for i in range(size))
print(f"{sol.energy()}: {bits}")
```
In this program, an `ABS3Solver` object is created for the expression `f`.
The `time_limit()` method sets the maximum search time, and the `callback()` sets a function that prints the energy and TTS of newly found best solutions.

This program produces output similar to the following:
{% raw %}
```txt
TTS = 0.002s Energy = 1218
TTS = 0.002s Energy = 1170
TTS = 0.002s Energy = 994
TTS = 0.015s Energy = 958
TTS = 0.018s Energy = 922
TTS = 0.034s Energy = 874
TTS = 4.364s Energy = 834
834: -+--+---++-++-+---++-++--+++--+-+-+++++----+++-+-+---++-+--+-----+--+----++----+-+--++++++---+------
```
{% endraw %}

## ABS3 Solver object
An `ABS3Solver` object is created for a given expression.
An optional second argument `gpu` controls GPU usage:
- **`ABS3Solver(f)`**: Automatically uses all available GPUs. If no GPU is available, falls back to CPU-only mode.
- **`ABS3Solver(f, 0)`**: Forces CPU-only mode (no GPU is used).
- **`ABS3Solver(f, n)`**: Uses `n` GPUs.

## Setting ABS3 Solver Options
- **`time_limit(time)`**: Sets the time limit in seconds.
- **`target_energy(energy)`**: Sets the target energy for early termination.
- **`callback(func)`**: Sets a callback function called when a new best solution is found. The callback receives three arguments: `energy` (int), `tts` (float), and `event` (string).
- **`set_param(key, val)`**: Sets an advanced parameter as a key-value pair of strings.

### Advanced Parameters

| Key | Value | Description |
|----|----|----|
| **`cpu_enable`** | "0" or "1" | Enables/disables the CPU solver alongside the GPU (default: "1") |
| **`cpu_thread_count`** | number | Number of CPU solver threads (default: auto) |
| **`block_count`** | number | Number of CUDA blocks per GPU |
| **`thread_count`** | number | Number of threads per CUDA block |
| **`topk_sols`** | number | Returns the top-K solutions with the best energies |

## Properties
- **`is_gpu`**: Returns `True` if the solver is using GPU acceleration.

## Program Example: CPU-only mode
To use the ABS3 Solver without a GPU, pass `0` as the second argument:
```python
solver = ABS3Solver(f, 0)
solver.time_limit(5.0)
solver.target_energy(0)
sol = solver.search()
print(sol)
```
