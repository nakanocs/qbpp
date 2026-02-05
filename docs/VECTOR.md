---
layout: default
title: "VECTOR"
---

# Vector of variables and vector functions

QUBO++ supports vector of variables and vector oparations.

## Defining vector of variables
A vector of binary variables can be created using the **`qbpp::var()`** function.
- **`qbpp::var("name", size)`** returns a vector of `size` variables with the given `name`.

The following program defines a vector of 5 variables with the name **`x`**.
By printing `x` with `std::cout`, we can confirm that it contains the 5 variables **`x[0]`**, **`x[1]`**, **`x[2]`**, **`x[3]`**, and **`x[4]`**.
Next, using the **`qbpp::expr()`** function with type deduction, we create a **`qbpp::Expr`** object **`f`** whose initial value is `0`.
In the for-loop from `i = 0` to `4`, each variable `x[i]` is added to `f` using the compound operator **`+=`**.
Finally, `f` is simplified and printed using `std::cout`.
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 5);
  std::cout << x << std::endl;
  auto f = qbpp::expr();
  for (int i = 0; i < 5; ++i) {
    f += x[i];
  }
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
}
```
The output of this program is as follows:
```
{x[0],x[1],x[2],x[3],x[4]}
f = x[0] +x[1] +x[2] +x[3] +x[4]
```

> **NOTE**
> **`qbpp::var(name, size)`** returns a **`qbpp::Vector<qbpp::Var>`** object that contains `size` elements of type `qbpp::Var`.
> The **`qbpp::Vector<T>`** class is almost compatible with **`std::vector<T>`**.
> It provides overloaded operators that support vector operations for elements of type `T`.

## Sum function
Using the vector utility function **`qbpp::sum()`**, you can obtain the sum of a vector of binary variables.
The following program uses `qbpp::sum()` to compute the sum of all variables in the vector `x`:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x", 5);
  std::cout << x << std::endl;
  auto f = qbpp::sum(x);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
}
```
The output of this program is exactly the same as that of the previous program.

## QUBO for one-hot constraint
A vector of binary variables is **one-hot** if it has **exactly one entry equal to 1**, that is, the sum of its elements is equal to 1.
Let $X = (x_0, x_1, \ldots, x_{n-1})$ denote a vector of $n$ binary variables.
The following expression $f(X)$ takes the minimum value of 0 if and only if $X$ is one-hot:

$$
\begin{align}
f(X) &= \left(1 - \sum_{i=0}^{n-1}x_i\right)^2
\end{align}
$$

The following program creates the expression $f$ and finds all optimal solutions:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto x = qbpp::var("x", 5);
  auto f = qbpp::sqr(qbpp::sum(x) - 1);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search_optimal_solutions();
  std::cout << sol << std::endl;
}
```
The function **`qbpp::sum()`** computes the sum of all variables in the vector.
The function **`qbpp::sqr()`** computes the square of its argument.
The Exhaustive Solver finds all optimal solutions with energy value 0, which are printed using `std::cout` as follows:
{% raw %}
```txt
f = 1 -x[0] -x[1] -x[2] -x[3] -x[4] +2*x[0]*x[1] +2*x[0]*x[2] +2*x[0]*x[3] +2*x[0]*x[4] +2*x[1]*x[2] +2*x[1]*x[3] +2*x[1]*x[4] +2*x[2]*x[3] +2*x[2]*x[4] +2*x[3]*x[4]
(0) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],0},{x[4],1}}
(1) 0:{{x[0],0},{x[1],0},{x[2],0},{x[3],1},{x[4],0}}
(2) 0:{{x[0],0},{x[1],0},{x[2],1},{x[3],0},{x[4],0}}
(3) 0:{{x[0],0},{x[1],1},{x[2],0},{x[3],0},{x[4],0}}
(4) 0:{{x[0],1},{x[1],0},{x[2],0},{x[3],0},{x[4],0}}
```
{% endraw %}
All 5 optimal solutions are displayed.