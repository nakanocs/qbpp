---
section: python
layout: default
title: "Factorization Through HUBO Expression (Python)"
---

# Factorization Through HUBO Expression

## HUBO for factorizing the product of two prime numbers
We consider the **factorization of integers** that are products of two prime numbers.
For example, when the product $pq = 35$ is given, the goal is to recover the two prime factors $p=5$ and $q=7$.

Since $\sqrt{35}=5.91$ and $35/2=17.5$, we can restrict the search ranges of $p$ and $q$ as follows:

$$
\begin{aligned}
  2 \leq &p \leq 5 \\
  6 \leq &q \leq 17
\end{aligned}
$$

For such integer variables, the factorization problem for $35$ can be formulated using the penalty expression:

$$
\begin{aligned}
f(p,q) &= (pq-35)^2
\end{aligned}
$$

Because the integer variables $p$ and $q$ are implemented as linear expressions of binary variables, their product
$pq$ becomes a quadratic expression, and therefore
$f(p,q)$ becomes quartic.
Clearly, $f(p,q)$ attains the minimum value 0 exactly when $p$ and $q$ are the correct factors of 35.

## PyQBPP program for factorization
The following program constructs the HUBO expression $f(p,q)$, and solves the optimization problem using the Easy Solver:
```python
from pyqbpp import var_int, between, EasySolver

p = between(var_int("p"), 2, 5)
q = between(var_int("q"), 6, 17)

f = (p * q) == 35
print("f =", f.simplify_as_binary())

solver = EasySolver(f)
solver.target_energy(0)
sol = solver.search()

print("sol =", sol)
print("p =", sol.eval(p))
print("q =", sol.eval(q))
```

In this program, the expression `(p * q) == 35` is automatically converted into `sqr(p * q - 35)`, which achieves an energy value of 0 when the equality is satisfied.
The output of this program is as follows:
```
f = 529 -240*p[0] -408*p[1] -88*q[0] -168*q[1] -304*q[2] -304*q[3] +144*p[0]*p[1] ...
sol = Sol(energy=0, p[0]=1, p[1]=1, q[0]=1, q[1]=0, q[2]=0, q[3]=0)
p = 5
q = 7
```
From the output, we can observe that the expression `f` contains quartic terms, confirming that it is a HUBO expression.
The solver correctly finds the prime factors $p=5$ and $q=7$.

> **NOTE**
> PyQBPP uses arbitrarily large integer arithmetic (Python's native `int`) for all coefficient and energy computations.
> Unlike the C++ version, there is no need to specify `COEFF_TYPE` or `ENERGY_TYPE` macros.
> Factorization of large composite numbers works out of the box without any special configuration.
