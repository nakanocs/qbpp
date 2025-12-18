---
layout: default
title: "SOLVE"
---


# Solving Expressions using Easy Solver and Exhaustive Solver

QUBO++ provides the **Easy Solver** and the **Exhaustive Solver** for QUBO/HUBO expressions.  
They run in parallel on multicore CPUs using Intel Threading Building Blocks (oneTBB).

- **Easy Solver**
  - Runs a heuristic algorithm based on simulated annealing.
  - Does not guarantee optimality.

- **Exhaustive Solver**
  - Explores all possible solutions.
  - Guarantees optimality of the returned solution.
  - Is computationally feasible only when the number of binary variables is about 30–40 or fewer.

Both solvers are used in two steps:
1. Create a solver object, `qbpp::easy_solver::EasySolver` or `qbpp::exhaustive_solver::ExhaustiveSolver`.
2. Call the `search()` member function on the solver object. It returns a `qbpp::Sol` object that stores the obtained solution.

## Easy Solver
To use the **Easy Solver**, include the header file `qbpp_easy_solver.hpp`.
It is defined in the namespace `qbpp::easy_solver`.

We use the following expression $f(a,b,c,d)$ as an example:

$$
\begin{aligned}
f(a,b,c,d,e) &= (a+2b+3c+4d-5)^2
\end{aligned}
$$

Clearly, this expression attains its minimum value $f=0$
when $a+2b+3c+4d=5$.
Therefore, it has two optimal solutions, $(a,b,c,d)=(0,1,1,0)$ and $(1,0,0,1)$.

In this program, expression `f` is created using the symbolic computation.
Note that the function `qbpp::sqr()` returns the square of the argument.
We then construct an instance of the class `qbpp::easy_solver::EasySolver`
by passing `f` to its constructor.
Before doing so, `f` must be simplified for binary variables by calling `simplify_as_binary()`.
The constructor returns an `EasySolver` object named `solver`.
Since we know that the optimal value is $f=0$, we set the target energy to $0$ by calling the `target_energy()` member function.
Calling the `search()` member function on `solver` returns a solution instance `sol`  of
class `qbpp::Sol`, which is printed using `std::cout`.

```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << sol << std::endl;
}
```

The output of this program is as follows:
{% raw %}
```txt
f = 25 -9*a -16*b -21*c -24*d +4*a*b +6*a*c +8*a*d +12*b*c +16*b*d +24*c*d
0:{{a,1},{b,0},{c,0},{d,1}}
```
{% endraw %}
One of the optimal solutions is correctly output.

## Exhaustive Solver
To use the **Exhaustive Solver**, include the header file `qbpp_exhaustive_solver.hpp`.  
It is defined in the namespace `qbpp::exhaustive_solver`.

We construct an instance `solver` of the class `qbpp::exhaustive_solver::ExhaustiveSolver`
by passing `f` to its constructor.  
Calling the `search()` member function on `solver` returns a solution instance `sol` of
class `qbpp::Sol`, which is printed using `std::cout`.  
Since the Exhaustive Solver explores all possible assignments, it is guaranteed that `sol`
stores an optimal solution.

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto d = qbpp::var("d");
  auto f = qbpp::sqr(a + 2 * b + 3 * c + 4 * d - 5);
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search();
  std::cout << sol << std::endl;
}
```
The output of this program is as follows:
{% raw %}
```txt
0:{{a,0},{b,1},{c,1},{d,0}}
```
{% endraw %}
All optimal solutions can be obtained by the `search_optimal_solutions()` member function as follows:
```cpp
  auto sol = solver.search_optimal_solutions();
```
The output is as follows:
{% raw %}
```txt
(0) 0:{{a,0},{b,1},{c,1},{d,0}}
(1) 0:{{a,1},{b,0},{c,0},{d,1}}
```
{% endraw %}
Furthermore, all solutions including non-optimal ones can be obtained by the
`search_all_solutions()` member function as follows:
```cpp
 auto sol = solver.search_all_solutions();
```
The output is as follows:
{% raw %}
```txt
(0) 0:{{a,0},{b,1},{c,1},{d,0}}
(1) 0:{{a,1},{b,0},{c,0},{d,1}}
(2) 1:{{a,0},{b,0},{c,0},{d,1}}
(3) 1:{{a,0},{b,1},{c,0},{d,1}}
(4) 1:{{a,1},{b,0},{c,1},{d,0}}
(5) 1:{{a,1},{b,1},{c,1},{d,0}}
(6) 4:{{a,0},{b,0},{c,1},{d,0}}
(7) 4:{{a,0},{b,0},{c,1},{d,1}}
(8) 4:{{a,1},{b,1},{c,0},{d,0}}
(9) 4:{{a,1},{b,1},{c,0},{d,1}}
(10) 9:{{a,0},{b,1},{c,0},{d,0}}
(11) 9:{{a,1},{b,0},{c,1},{d,1}}
(12) 16:{{a,0},{b,1},{c,1},{d,1}}
(13) 16:{{a,1},{b,0},{c,0},{d,0}}
(14) 25:{{a,0},{b,0},{c,0},{d,0}}
(15) 25:{{a,1},{b,1},{c,1},{d,1}}
```
{% endraw %}
The Exhaustive Solver is very useful for analyzing small expressions and for debugging.