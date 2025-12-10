---
layout: default
title: "QUBO++ Documentation"
---

# Integer Variables and Solving Simultaneous Equations

## Integer variables
QUBO++ supports integer variables, which are internally implemented using multiple binary variables.
A conventional binary encoding is used to represent integer values.
Suppose that we have $n$ binary variables $x_0, x_1, \ldots, x_{n-1}$.
These variables can represent all integers from $0$ to $2^n-1$ using the following linear expression:

$$
\begin{aligned}
2^0x_0+2^1x_1+\cdots 2^{n-1}x_{n-1}
\end{aligned}
$$

We can introduce a constant offset $l$ and replace the coefficient of $x_{n-1}$ with an arbitrary value $d$ as follows:

$$
\begin{aligned}
l+2^0x_0+2^1x_1+\cdots +2^{n-2}x_{n-2}+dx_{n-1}
\end{aligned}
$$

This expression can represent all integers from $l$ to $c+2^{n-1}+d-1$. 
Based on this encoding, a variable whose integer range is $[l,u]$ can be constructed by choosing appropriate values of $n$ and $d$ ($1\leq d\leq 2^{n-1}$) to satisfy 

$$
\begin{aligned}
u &= l+2^0x_0+2^1x_1+\cdots +2^{n-2}x_{n-2}+dx_{n-1}
\end{aligned}
$$

The following QUBO++ program demonstrates how integer variables are defined:
```cpp
#include "qbpp.hpp"

int main() {
  auto x = 1 <= qbpp::var_int("x") <= 8;
  auto y = -10 <= qbpp::var_int("y") <= 10;
  std::cout << "x = " << x << " uses " << x.size() << " variables.\n";
  std::cout << "y = " << y << " uses " << y.size() << " variables.\n";
}
```

An integer variable is defined using the double inequality operator `<= <=`, which specifies the integer range that the variable can take.
The function `qbpp::var_int(name)` creates a `qbpp::VarInt object` object with the given  `name`, representing the linear expression encoded by binary variables.
The program outputs the following expressions:
```txt
x = 1 +x[0] +2*x[1] +4*x[2] uses 3 variables.
y = -10 +y[0] +2*y[1] +4*y[2] +8*y[3] +5*y[4] uses 5 variables.
```

> **WARNING**
> The number of binary variables required for an integer variable grows logarithmically with its range.
> When $u−l$ is large, the QUBO size increases, so wide integer ranges should be avoided whenever possible.

## QUBO formulation for solving simultaneous equations
QUBO++ can solve systems of simultaneous equations by representing the variables as integer variables.
As an example, we construct a QUBO formulation for the following equations, whose solution is $x=4$ and $y=6$:

$$
\begin{aligned}
x + y = 10\\
2x+4y = 28 
\end{aligned}
$$

To solve these equations, we define integer variables $x$ and $y$ in the range $[0,10]$, each encoded by four binary variables:

$$
\begin{aligned}
x = x_0 +2x_1 +4x_2 +3x_3\\
y = y_0 +2y_1 +4y_2 +3y_3
\end{aligned}
$$

Each of the following penalty expressions takes the minimum value 0 if and only if the corresponding equation is satisfied:

$$
\begin{aligned}
f(x,y) &= (x+y-10)^2\\ 
g(x,y) &= (2x+4y -28)^2
\end{aligned}
$$

Thus, the combined expression

$$
\begin{aligned}
h(x,y) &= f(x,y) +g(x,y)
\end{aligned}
$$

achieves its minimum value 0 precisely when both equations are satisfied simultaneously.


## QUBO++ program 
The following QUBO++ program constructs the QUBO expression $h(x,y)$, solves it, and decodes the resulting values of
$x$ and $y$:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = 0 <= qbpp::var_int("x") <= 10;
  auto y = 0 <= qbpp::var_int("y") <= 10;
  auto f = x + y == 10;
  auto g = 2 * x + 4 * y == 28;
  auto h = f + g;
  h.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(h);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "x = " << x << " = " << sol(x) << std::endl;
  std::cout << "y = " << y << " = " << sol(y) << std::endl;
  std::cout << "f = " << f << " = " << sol(f) << std::endl;
  std::cout << "g = " << g << " = " << sol(g) << std::endl;
  std::cout << "*f = " << *f << " = " << sol(*f) << std::endl;
  std::cout << "*g = " << *g << " = " << sol(*g) << std::endl;
}
```
First, qbpp::VarInt objects  `x` and `y` are defined with the range $[0,10]$.
A `qbpp::Expr` object `f` is created to represent the constraint `x + y == 10`.
Internally, this is equivalent to the QUBO expression `qbpp::sqr(x + y -10)`.
Similarly, `g` represents the constraint `2 * x + 4 * y == 28`.
The combined expression `h = f + g` encodes both equations.
An Easy Solver instance is created with `h`, and the target energy is set to `0`, since the optimal solution minimizes all constraint violations.
Calling `search()` returns a `qbpp::Sol` object sol that stores the optimal assignment of all binary variables.
Finally, the program prints the values of `sol`, `sol(x)`, `sol(y)`, `sol(f)`, `sol(g)`, `sol(*f)`, and `sol(*g)`.
Here,
- `f` is the penalty expression enforcing `x + y = 10`. Thus `sol(f) = 0` if and only if the equation is satisfied.
- `*f` is the linear expression `x + y`. Thus `sol(*f)` returns the actual evaluated value of `x + y`

The same applies to `g` and `*g`.

The program outputs the following result:

{% raw %}
```txt
sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{y[0],0},{y[1],0},{y[2],1},{y[3],0}}
x = x[0] +2*x[1] +4*x[2] +3*x[3] = 6
y = y[0] +2*y[1] +4*y[2] +3*y[3] = 4
f = 100 -19*x[0] -36*x[1] -64*x[2] -51*x[3] -19*y[0] -36*y[1] -64*y[2] -51*y[3] +4*x[0]*x[1] +8*x[0]*x[2] +6*x[0]*x[3] +2*x[0]*y[0] +4*x[0]*y[1] +8*x[0]*y[2] +6*x[0]*y[3] +16*x[1]*x[2] +12*x[1]*x[3] +4*x[1]*y[0] +8*x[1]*y[1] +16*x[1]*y[2] +12*x[1]*y[3] +24*x[2]*x[3] +8*x[2]*y[0] +16*x[2]*y[1] +32*x[2]*y[2] +24*x[2]*y[3] +6*x[3]*y[0] +12*x[3]*y[1] +24*x[3]*y[2] +18*x[3]*y[3] +4*y[0]*y[1] +8*y[0]*y[2] +6*y[0]*y[3] +16*y[1]*y[2] +12*y[1]*y[3] +24*y[2]*y[3] = 0
g = 784 -108*x[0] -208*x[1] -384*x[2] -300*x[3] -208*y[0] -384*y[1] -640*y[2] -528*y[3] +16*x[0]*x[1] +32*x[0]*x[2] +24*x[0]*x[3] +16*x[0]*y[0] +32*x[0]*y[1] +64*x[0]*y[2] +48*x[0]*y[3] +64*x[1]*x[2] +48*x[1]*x[3] +32*x[1]*y[0] +64*x[1]*y[1] +128*x[1]*y[2] +96*x[1]*y[3] +96*x[2]*x[3] +64*x[2]*y[0] +128*x[2]*y[1] +256*x[2]*y[2] +192*x[2]*y[3] +48*x[3]*y[0] +96*x[3]*y[1] +192*x[3]*y[2] +144*x[3]*y[3] +64*y[0]*y[1] +128*y[0]*y[2] +96*y[0]*y[3] +256*y[1]*y[2] +192*y[1]*y[3] +384*y[2]*y[3] = 0
*f = x[0] +2*x[1] +4*x[2] +3*x[3] +y[0] +2*y[1] +4*y[2] +3*y[3] = 10
*g = 2*x[0] +4*x[1] +8*x[2] +6*x[3] +4*y[0] +8*y[1] +16*y[2] +12*y[3] = 28
```
{% endraw %}

Thus, we can confirm that the values of `x`, `y`, and the constraint expressions `f`, `g`, `*f`, and `*g` are consistent with the solution.

> **WARNING**
> QUBO++ supports the `==` operator only when the left-hand side is an expression and the right-hand side is an integer.
> Comparisons of the form integer `==` expression or expression `==` expression are not supported.