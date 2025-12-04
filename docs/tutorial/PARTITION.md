# Solving Partitioning Problem Using Vector of variables

## Partitioning problem
Let $w=(w_0, w_1, \ldots, w_{n-1})$ be $n$ positive numbers.
The partitioning problem is to partition these numbers into two sets $P$ and $Q$ ($Q=\overline{P}$) such that the sums of the elements in the two sets are as close as possible.
More specifically, the problem is to find a subset $L \subseteq \{0,1,\ldots, n-1\}$ that minimizes:

$$
\begin{aligned}
P(L) &= \sum_{i\in L}w_i \\
Q(L) &= \sum_{i\not\in L}w_i \\
f(L) &= \left| P(L)-Q(L) \right|
\end{aligned}
$$

This problem can be formulated as a QUBO problem.
Let $x=(x_0, x_1, \ldots, x_{n-1})$ be binary variables representing the set $L$,
that is, $i\in L$ if and only if $x_i=1$.
We can rewrite $P(L)$, $Q(L)$ and $f(L)$ using $x$ as follows:

$$
\begin{aligned}
P(x) &= \sum_{i=0}^{n-1} w_ix_i \\
Q(x) &= \sum_{i=0}^{n-1} w_i (1-x_i) \\
f(x)    &= \left( P(x)-Q(x) \right)^2
\end{aligned}
$$

Clealy, $f(x)=f(L)^2$ holds.
The function $f(x)$ is a quadratic expression of $x$, and an optimal solution that minimizes $f(x)$ also gives an optimal solution to the original partitioning problem.

## QUBO++ program for the partitioning problem
The following QUBO++ program creates the QUBO formulation of the partitioning problem for a fixed set of 8 numbers and finds a solution using the Exhaustive Solver.


```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  std::vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
  auto x = qbpp::var("x", w.size());
  auto p = qbpp::expr();
  auto q = qbpp::expr();
  for (size_t i = 0; i < w.size(); ++i) {
    p += w[i] * x[i];
    q += w[i] * (1 - x[i]);
  }
  auto f = qbpp::sqr(p - q);
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sol = solver.search();
  std::cout << "Solution: " << sol << std::endl;
  std::cout << "f(sol) = " << f(sol) << std::endl;
  std::cout << "p(sol) = " << p(sol) << std::endl;
  std::cout << "q(sol) = " << q(sol) << std::endl;
  std::cout << "L  :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](sol) == 1) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
  std::cout << "~L :";
  for (size_t i = 0; i < w.size(); ++i) {
    if (x[i](sol) == 0) {
      std::cout << " " << w[i];
    }
  }
  std::cout << std::endl;
}
```

In this program `w` is defined as a `std::vector` object with 8 numbers.
A vector `x` of `w.size()=8` binary variables is defined.
Two `qbpp::Expr` objects `p` and `q`  are defined, and the expressions for $P(x)$ and $Q(x)$
are constructed in the for-loop.
A `qbpp::Expr` object `f` stores the expression for $f(x)$.

An Exhaustive Solver object `solver` for `f` is created 
and the solution `sol` (a `qbpp::Sol` object) is obtained by calling its `search()` member function.

The values of $f(x)$, $P(x)$, and $Q(x)$ are evaluated by calling `f(sol)`, `p(sol)` and `q(sol)`, respectively.
The numbers in the sets $L$ and $\overline{L}$ are displayed using the for loops.
In these loops, `x[i](sol)` returns the value of `x[i]` in `sol`.

This program outputs:
```txt
f = 168100 -88576*x[0] -41364*x[1] -68244*x[2] -99456*x[3] -19104*x[4] -108564*x[5] -87444*x[6] -59200*x[7] +13824*x[0]*x[1] +24064*x[0]*x[2] +37888*x[0]*x[3] +6144*x[0]*x[4] +42496*x[0]*x[5] +32256*x[0]*x[6] +20480*x[0]*x[7] +10152*x[1]*x[2] +15984*x[1]*x[3] +2592*x[1]*x[4] +17928*x[1]*x[5] +13608*x[1]*x[6] +8640*x[1]*x[7] +27824*x[2]*x[3] +4512*x[2]*x[4] +31208*x[2]*x[5] +23688*x[2]*x[6] +15040*x[2]*x[7] +7104*x[3]*x[4] +49136*x[3]*x[5] +37296*x[3]*x[6] +23680*x[3]*x[7] +7968*x[4]*x[5] +6048*x[4]*x[6] +3840*x[4]*x[7] +41832*x[5]*x[6] +26560*x[5]*x[7] +20160*x[6]*x[7]
Solution: 0:{{x[0],0},{x[1],0},{x[2],1},{x[3],0},{x[4],1},{x[5],1},{x[6],1},{x[7],0}}
f(sol) = 0
p(sol) = 205
q(sol) = 205
L  : 47 12 83 63
~L : 64 27 74 40
```

> [!NOTE]
> For a `qbpp::Expr` object `f` and a `qbpp::Sol` object `sol`, both `f(sol)` and `sol(f)` return the resuting value of `f` for `sol`.
> Similarly, a `qbpp::Var` object `a` and a `qbpp::Sol` object `sol`, both `a(sol)` and `sol(a)` return the resuting value of `a` in the solution `sol`.
> Users can use either form according to their preference.

## QUBO++ program using vector operations
QUBO++ has rich vector operations that can simplify the code.
For this purpose,  `qbpp::Vector` class, which is similar to `std::vector` class shoud be used.
In the following code, `w` is defined as an object of `qbpp::Vector<uint32_t>` class.
Also, `x` is defined as an object of `qbpp::Vector<qbpp::Var>` class.
Since the overloaded operator `*` for `qbpp::Vector` class returns the element-wise product of two
`qbpp::Vector` objects, `qbpp::sum(w*x)` returns the `qbpp::Expr`object representing $P(L)$.
Furthermore, the overloaded operator `-` for a scalar and a `qbpp::Vector` object returns
a `qbpp::Vector` object whose components are the scalar minus each element of the vector.
Thus, `qbpp::sum(w * (1 - x))` returns a `qbpp::Expr` object storing $Q(L)$.

```cpp
  qbpp::Vector<uint32_t> w = {64, 27, 47, 74, 12, 83, 63, 40};
  auto x = qbpp::var("x", w.size());
  auto p = qbpp::sum(w * x);
  auto q = qbpp::sum(w * (1 - x));
  auto f = qbpp::sqr(p - q);
```

QUBO++ programs can be simplified by using these vector operations.
In addition, since vector operations for large vectors are parallelized by multithreading, they can accelerate the process of creating QUBO models.

> [!NOTE]
> The operators `+`, `-`, and `*` are overloaded both for two `qbpp::Vector` objects and for a scalar and a `qbpp::Vector` object.
> For two `qbpp::Vector` objects, the overloaded operators perform element-wise operations.
> For a scalar and a `qbpp::Vector` object, the overloaded operators apply the scalar operation to each element of the vector.