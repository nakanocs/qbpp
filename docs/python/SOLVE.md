---
section: python
layout: default
title: "SOLVE (Python)"
---


# Solving Expressions using Easy Solver and Exhaustive Solver

PyQBPP provides the **Easy Solver** and the **Exhaustive Solver** for QUBO/HUBO expressions.
They run in parallel on multicore CPUs using **Intel Threading Building Blocks (oneTBB)**.

- **Easy Solver**
  - Runs a heuristic algorithm based on simulated annealing.
  - Does not guarantee optimality.

- **Exhaustive Solver**
  - Explores all possible solutions.
  - Guarantees optimality of the returned solution.
  - Is computationally feasible only when the number of binary variables is about 30-40 or fewer.
  - If a CUDA GPU is available, GPU acceleration is automatically enabled alongside CPU threads.

Both solvers are used in two steps:
1. Create a solver object, **`EasySolver`** or **`ExhaustiveSolver`**.
2. Call the **`search()`** method on the solver object. It returns a **`Sol`** object that stores the obtained solution.

## Easy Solver
We use the following expression $f(a,b,c,d)$ as an example:

$$
\begin{aligned}
f(a,b,c,d) &= (a+2b+3c+4d-5)^2
\end{aligned}
$$

Clearly, this expression attains its minimum value $f=0$
when $a+2b+3c+4d=5$.
Therefore, it has two optimal solutions, $(a,b,c,d)=(0,1,1,0)$ and $(1,0,0,1)$.

In the following program, expression `f` is created using symbolic computation.
The function **`sqr()`** returns the square of the argument.
We then construct an `EasySolver` instance by passing `f` to its constructor.
Before doing so, `f` must be simplified for binary variables by calling **`simplify_as_binary()`**.
Since we know that the optimal value is $f=0$, we set the target energy to $0$ by calling the **`target_energy()`** method.
Calling the **`search()`** method on `solver` returns a solution instance **`sol`** of class **`Sol`**.

```python
from pyqbpp import var, sqr, EasySolver

a = var("a")
b = var("b")
c = var("c")
d = var("d")
f = sqr(a + 2 * b + 3 * c + 4 * d - 5)
print("f =", f.simplify_as_binary())

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()
print(sol)
```

The output of this program is as follows:
```
f = 25 -9*a -16*b -21*c -24*d +4*a*b +6*a*c +8*a*d +12*b*c +16*b*d +24*c*d
Sol(energy=0, a=1, b=0, c=0, d=1)
```
One of the optimal solutions is correctly output.

## Exhaustive Solver
We construct an **`ExhaustiveSolver`** instance by passing `f` to its constructor.
Calling the **`search()`** method on `solver` returns a solution instance **`sol`** of
class **`Sol`**.
Since the Exhaustive Solver explores all possible assignments, it is guaranteed that `sol`
stores an optimal solution.

```python
from pyqbpp import var, sqr, ExhaustiveSolver

a = var("a")
b = var("b")
c = var("c")
d = var("d")
f = sqr(a + 2 * b + 3 * c + 4 * d - 5)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()
print(sol)
```
The output of this program is as follows:
```
Sol(energy=0, a=0, b=1, c=1, d=0)
```
All optimal solutions can be obtained by the **`search_optimal_solutions()`** method as follows:
```python
sols = solver.search_optimal_solutions()
for i, sol in enumerate(sols):
    print(f"({i}) {sol}")
```
The output is as follows:
```
(0) Sol(energy=0, a=0, b=1, c=1, d=0)
(1) Sol(energy=0, a=1, b=0, c=0, d=1)
```

The Exhaustive Solver is very useful for analyzing small expressions and for debugging.
