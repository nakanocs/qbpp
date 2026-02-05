---
layout: default
title: "RANGE"
---

# Square Root

This example demonstrates how to compute the square root of
$c=2$ using large integers represented by `qbpp::cpp_int`.
Let $s = 10 ^{10}$ be a fixed integer.
Since QUBO++ cannot handle real numbers directly, we compute  $\sqrt{cs^2}$ instead of $\sqrt{c}$.
From the following relation,

$$
\begin{aligned}
\sqrt{c} &= \sqrt{cs^2}/s
\end{aligned}
$$

we can obtain an approximation of $\sqrt{c}$ with 10 decimal-digit precision.

## HUBO formulation of the square root computation
We define an integer variable $x$ that takes values in the range $[s, 2s]$.
We then formulate the problem using the following equation:

$$
\begin{aligned}
x ^ 2 &= cs ^ 2
\end{aligned}
$$

In QUBO++, this equality constraint is converted into the following HUBO expression:

$$
(x ^ 2 -cs^2)^2
$$

By finding the value of $x$ that minimizes this expression,
we obtain an approximation of the square root of $c$ with
10 decimal-digit precision.
Since $x$ is internally represented as a linear expression of binary variables, this objective function becomes quartic in those binary variables.

## QUBO++ parogram
The following QUBO++ program constructs a HUBO expression based on the above idea and solves it using the Easy Solver:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  const int c = 2;
  auto s = qbpp::cpp_int("10000000000");
  auto x = s <= qbpp::var_int("x") <= c * s;
  auto f = x * x == c * s * s;
  f.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.time_limit(1.0);
  auto sol = solver.search();
  std::cout << "Energy = " << sol.energy() << std::endl;
  std::cout << "x = " << x << "\n = " << sol(x) << std::endl;
}
```

Since very large coefficients are used, we define the two macros `COEFF_TYPE` and `ENERGY_TYPE` as `qbpp::cpp_int`,
which can represent arbitrarily large integers.
The constant `s`, the integer variable `x`, and the HUBO expression `f` are defined according to the formulation described above.
The Easy Solver is executed with a time limit of 1.0 second.

This program produces the following output:
```
Energy = 57910111919782629376
x = 10000000000 +x[0] +2*x[1] +4*x[2] +8*x[3] +16*x[4] +32*x[5] +64*x[6] +128*x[7] +256*x[8] +512*x[9] +1024*x[10] +2048*x[11] +4096*x[12] +8192*x[13] +16384*x[14] +32768*x[15] +65536*x[16] +131072*x[17] +262144*x[18] +524288*x[19] +1048576*x[20] +2097152*x[21] +4194304*x[22] +8388608*x[23] +16777216*x[24] +33554432*x[25] +67108864*x[26] +134217728*x[27] +268435456*x[28] +536870912*x[29] +1073741824*x[30] +2147483648*x[31] +4294967296*x[32] +1410065409*x[33]
 = 14142135624
```
We can confirm that the Easy Solver outputs the correct approximation:

$$
 \sqrt{2\times 10^{20}}\approx 14142135624
$$

Note that the reported energy value is not zero, and the equality constraint is not satisfied exactly.
This is simply because there is no exact integer solution to the equality.
Instead, the solver finds a solution that minimizes the error of the equality constraint.
The energy value shown in the output corresponds to the square of this error.
Since the error is minimized, the resulting value of $x$ represents an approximation of the square root.