---
layout: default
title: "Comparison Operators"
---

# Comparison Operators
QUBO++ supports two types of operators for creating constraints:

- **Equality operator**: $f=n$, where $f$ is an expression and $n$ is an integer.
- **Range operator**: $l\leq f\leq u$, where $f$ is an expression and 
$l$ and $u$ ($l\leq u$) are integers.

These operators return an expression that attains **the minimum value of 0 if and only if the corresponding constraints are satisfied**.

## Equality operator
The equality operator $f=n$ creates the following expression:

$$
(f−n)^2
$$

This expression attains the minimum value of 0 if and only if the equality $f=n$ is satisfied.

The following QUBO++ program searches for all solutions satisfying
$a+2b+3c=3$ using the Exhaustive Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = a + 2 * b + 3 * c == 3;
  f.simplify_as_binary();
  std::cout << "f = " << f << std::endl;
  std::cout << "*f = " << *f << std::endl;

  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << "a = " << a(sol) << ", b = " << b(sol) << ", c = " << c(sol)
              << ", f = " << f(sol) << ", *f = " << (*f)(sol) << std::endl;
  }
}
```
In this program, `f` internally holds two qbpp::Expr objects:
- **`f`**: $(a+2b+3c−3)^2$, which attains the minimum value of 0 if the equality $a+2b+3c=3$ is satisfied.
- **`*f`**: the left-hand side of the equality, $a+2b+3c$.

Using the Exhaustive Solver object created for `f`, all optimal solutions are stored in **`sols`**.
By iterating over `sols`, all solutions and the values of `f` and `*f` are printed as follows:
```
f = 9 -5*a -8*b -9*c +4*a*b +6*a*c +12*b*c
*f = a +2*b +3*c
a = 0, b = 0, c = 1, f = 0, *f = 3
a = 1, b = 1, c = 0, f = 0, *f = 3
```
These results confirm that two optimal solutions attain `f = 0` and satisfy `*f = 3`.

## Notes on Supported Equality Forms
QUBO++ supports the equality operator only in the following form:
- **`expression == integer`**.

The following forms are not supported:
- **`integer == expression`**
- **`expression1 == expression2`**

Instead of `expression1 == expression2`, you can rewrite the constraint as:
- **`expression1 - expression2 == 0`**

which is fully supported.


## Range operator
The range operator of the form $l\leq f \leq u$ ($l\leq u$) creates an expression that attains the minimum value of 0 if and only if the constraint is satisfied.

We consider the following cases depending on the values of $l$ and $u$.
- **Case 1**: **$u=l$**
- **Case 2**: **$u=l+1$**
- **Case 3**: **$u=l+2$**
- **Case 4**: **$u\geq l+3$**.

### Case 1: $u=l$
If $u=l$, the range constraint reduces to the equality constraint $f=l$,
which can be implemented directly using the equality operator.

### Case 2: $u=l+1$
If $u=l+1$, the following expression is created:

$$
 (f-l)(f-u)
$$

Since there is no integer strictly between $l$ and $u$, this expression attains the minimum value of 0 if and only if 
$f=l$ or $f=u$.

### Case 3: $u=l+2$
We introduce an **auxiliary binary variable** $a \in \lbrace 0,1\rbrace$
and use the following expression:

$$
\begin{aligned}
(f-l-a)(f-l-(a+1))
\end{aligned}
$$

This expression evaluates as follows for $f=l$, $l+1$, and $l+2$

$$
\begin{aligned}
(f-l-a)(f-l-(a+1)) &= (-a)(-(a+1)) && \text{if } f=l \\
                   &= (1-a)(-a) && \text{if } f=l+1 \\
                   &=(2-a)(1-a)  && \text{if } f=l+2
\end{aligned}
$$

In all cases, the minimum value 0 is attainable by an appropriate choice of $a$.
Therefore, the expression takes the minimum value of 0 if $l\leq f\leq u$ is satisfied.

Let $g = f-l-a$.
Then we have

$$
\begin{aligned}
(f-l-a)(f-l-(a+1)) &= g(g-1)
\end{aligned}
$$

which is always positive if $g\leq -1$ or $g\geq 2$.
Hence, the expression attains the minimum value of 0 if and only if  $l\leq f\leq u$ is satisfied.

### Case 4: $u\geq l+3$
We introduce an auxiliary integer variable $a$ that takes integer values in the range $[l,u−1]$.
Such an integer variable can be defined using multiple binary variables, as described in [Integer Variables and Solving Simultaneous Equations](INTEGER).

The expression for this case is:

$$
\begin{aligned}
(f-a)(f-(a+1))
\end{aligned}
$$

Similarly to Case 3, we can show that this expression is always positive if $f$ is not in $[l,u]$.

Suppose that $f$ takes an integer value in the range  $[l,u]$.
If we choose $a=f$, then

$$
\begin{aligned}
f-a &= 0 & {\rm if\,\,} f\in [l,u-1]\\
f-(a+1) &= 0& {\rm if\,\,} f\in [l+1,u]
\end{aligned}
$$

Thus, either $f−a=0$ or $f−(a+1)=0$ holds for any $f\in[l,u]$.
Therefore, $(f−a)(f−(a+1))$ attains the minimum value of 0
if and only if $l\leq f\leq u$.


### Reducing the number of binary variables
In [Integer Variables and Solving Simultaneous Equations](INTEGER),
an integer variable $a\in [l,u]$ is represented using 
$n$ binary variables $x_0, x_1, \ldots, x_{n-1}$ as follows:

$$
\begin{aligned}
a & = l+2^0x_0+2^1x_1+\cdots +2^{n-2}x_{n-2}+dx_{n-1}
\end{aligned}
$$

This expression can represent all integers from $l$ to $l+2^{n-1}+d-1$.
Thus, we can choose $n$ and $d$ such that

$$
\begin{aligned}
u-1&=l+2^{n-1}+d-1.
\end{aligned}
$$


For Case 4, QUBO++ instead uses the following linear expression with $n-1$ binary variables $x_1, \ldots, x_{n-1}$:

$$
\begin{aligned}
a &= l+2^1x_1+\cdots +2^{n-2}x_{n-2}+dx_{n-1}
\end{aligned}
$$

This expression represents integers from $l$ to $l+2^{n-1}+d-2$.
Accordingly, we select $n$ and $d$ so that 

$$
\begin{aligned}
u-1&=l+2^{n-1}+d-2.
\end{aligned}
$$

We call such an integer variable $a$ a **unit-gap integer variable**.
Although some values in $[l,u]$ cannot be taken by $a$,
for any $k\in[l,u]$ that cannot be represented, 
$k−1$ can be represented.
Therefore, either $a$ or $a+1$ can take any value in the range 
$[l,u]$, which is sufficient for enforcing the range constraint.

### QUBO++ program for the four cases
The following program demonstrates how the four cases are implemented in QUBO++:
```cpp
#include "qbpp.hpp"

int main() {
  auto f = qbpp::toExpr(qbpp::var("f"));
  auto f1 = 1 <= f <= 1;
  auto f2 = 1 <= f <= 2;
  auto f3 = 1 <= f <= 3;
  auto f4 = 1 <= f <= 5;
  std::cout << "f1 = " << f1.simplify() << std::endl;
  std::cout << "f2 = " << f2.simplify() << std::endl;
  std::cout << "f3 = " << f3.simplify() << std::endl;
  std::cout << "f4 = " << f4.simplify() << std::endl;
}
```
This program produces the following output:
```
f1 = 1 -2*f +f*f
f2 = 2 -3*f +f*f
f3 = 2 -3*f +3*{0} +f*f -2*f*{0} +{0}*{0}
f4 = 2 -3*f +6*{1}[0] +3*{1}[1] +f*f -4*f*{1}[0] -2*f*{1}[1] +4*{1}[0]*{1}[0] +4*{1}[0]*{1}[1] +{1}[1]*{1}[1]
```
These outputs correspond to the following expressions:

$$
\begin{aligned}
f_1 &= (f-1)^2\\
f_2 &= (f-1)(f-2)\\
f_3 &= (f-x_0)(f-(x_0+1))\\
f_4 &= (f-(2x_{1,0}+x_{1,1}+1))(f-(2x_{1,0}+x_{1,1}+2))
\end{aligned}
$$

### QUBO++ program using the range operator
The following program demonstrates the use of the range operator in QUBO++:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = 5 <= 4 * a + 9 * b + 15 * c <= 14;
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << "a = " << a(sol) << ", b = " << b(sol) << ", c = " << c(sol)
              << ", f = " << f(sol) << ", *f = " << (*f)(sol)
              << ", sol = " << sol << std::endl;
  }
}
```
For three binary variables $a$, $b$, and $c$,
this program searches for solutions satisfying the constraint

$$
\begin{aligned}
5\leq 4a+9b+15c \leq 14
\end{aligned}
$$

This program produces the following output:
{% raw %}
```
a = 0, b = 1, c = 0, f = 0, *f = 9, sol = 0:{{a,0},{b,1},{c,0},{{0}[0],0},{{0}[1],1},{{0}[2],0}}
a = 0, b = 1, c = 0, f = 0, *f = 9, sol = 0:{{a,0},{b,1},{c,0},{{0}[0],1},{{0}[1],0},{{0}[2],1}}
a = 1, b = 1, c = 0, f = 0, *f = 13, sol = 0:{{a,1},{b,1},{c,0},{{0}[0],1},{{0}[1],1},{{0}[2],1}}
```
{% endraw %}

## Lower and upper bound operators
QUBO++ does not directly support the following **one-sided bound operators**:
- **Lower-bound operator**: $l\leq f$
- **Upper-bound operator**: $f\leq u$

Instead, QUBO++ provides a symbolic representation of **infinity** ($\infty$)
and these constraints are implemented using **the range operator** as follows:
- **Lower-bound operator**: $l\leq f\leq +\infty$
- **Upper-bound operator**: $-\infty \leq f\leq u$

Since the range operator internally introduces auxiliary variables,
true infinite values cannot be represented explicitly.
Therefore, QUBO++ estimates **finite maximum and minimum values** of the expression 
$f$ and substitutes them for $+\infty$ and $-\infty$, respectively.

For example, consider the expression 

$$
\begin{aligned}
f=4a + 9 b + 11 c
\end{aligned}
$$

where $a$, $b$, and $c$ are binary variables.
The minimum and maximum possible values of $f$ are 0 and 24, respectively.
Thus, QUBO++ uses 0 and 24 as substitutes for $-\infty$ and $+\infty$
when constructing the corresponding range constraints.

> **NOTE**
> QUBO++ intentionally requires both lower and upper bounds to be specified in inequality constraints.
> This avoids ambiguity between **MIP-style interpretations** (e.g., 
> $f\leq u$ meaning $0\leq f\leq u$) and **QUBO-style interpretations** (e.g., $f\leq u$ meaning $-\infty\leq f\leq u$),
> which could otherwise lead to subtle modeling errors.

### QUBO++ program for lower and upper bound operators
In QUBO++, an infinite value is represented by **`qbpp::inf`**.

The following program demonstrates **the lower-bound operator**:
```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = 14 <= 4 * a + 9 * b + 11 * c <= +qbpp::inf;
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << "a = " << a(sol) << ", b = " << b(sol) << ", c = " << c(sol)
              << ", f = " << f(sol) << ", *f = " << (*f)(sol)
              << ", sol = " << sol << std::endl;
  }
}
```
In this program, **`+qbpp::inf`** represents a positive infinite value,
which is automatically replaced by 24.

This program produces the following output:
{% raw %}
```
a = 0, b = 1, c = 1, f = 0, *f = 20, sol = 0:{{a,0},{b,1},{c,1},{{0}[0],1},{{0}[1],0},{{0}[2],1}}
a = 0, b = 1, c = 1, f = 0, *f = 20, sol = 0:{{a,0},{b,1},{c,1},{{0}[0],1},{{0}[1],1},{{0}[2],0}}
a = 1, b = 0, c = 1, f = 0, *f = 15, sol = 0:{{a,1},{b,0},{c,1},{{0}[0],0},{{0}[1],0},{{0}[2],0}}
a = 1, b = 1, c = 1, f = 0, *f = 24, sol = 0:{{a,1},{b,1},{c,1},{{0}[0],1},{{0}[1],1},{{0}[2],1}}
```
{% endraw %}

The following program demonstrates **the upper-bound operator**:
```cpp
int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = -qbpp::inf <= 4 * a + 9 * b + 11 * c <= 14;
  f.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(f);
  auto sols = solver.search_optimal_solutions();
  for (const auto& sol : sols) {
    std::cout << "a = " << a(sol) << ", b = " << b(sol) << ", c = " << c(sol)
              << ", f = " << f(sol) << ", *f = " << (*f)(sol)
              << ", sol = " << sol << std::endl;
  }
}
```
In this program, **`-qbpp::inf`** represents a negative infinite value,
which is automatically replaced by 0.

This program produces the following output:
{% raw %}
```
a = 0, b = 0, c = 0, f = 0, *f = 0, sol = 0:{{a,0},{b,0},{c,0},{{0}[0],0},{{0}[1],0},{{0}[2],0}}
a = 0, b = 0, c = 1, f = 0, *f = 11, sol = 0:{{a,0},{b,0},{c,1},{{0}[0],0},{{0}[1],1},{{0}[2],1}}
a = 0, b = 1, c = 0, f = 0, *f = 9, sol = 0:{{a,0},{b,1},{c,0},{{0}[0],1},{{0}[1],0},{{0}[2],1}}
a = 1, b = 0, c = 0, f = 0, *f = 4, sol = 0:{{a,1},{b,0},{c,0},{{0}[0],0},{{0}[1],1},{{0}[2],0}}
a = 1, b = 1, c = 0, f = 0, *f = 13, sol = 0:{{a,1},{b,1},{c,0},{{0}[0],1},{{0}[1],1},{{0}[2],1}}
```
{% endraw %}