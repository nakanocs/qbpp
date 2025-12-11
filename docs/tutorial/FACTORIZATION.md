---
layout: default
title: "QUBO++ Documentation"
---

# Factorization Through HUBO expression

## HUBO for factorizing the product of two prime numbers
We consider the factorization of integers that are products of two prime numbers.
For example, when the product $pq = 35$ is given, the goal is to recover the two prime factors  $p=5$ and $q=7$.

Since $\sqrt{15}=5.91$ and $35/2=17.5$, we can restrict the search ranges $p$ and $q$ as follows:

$$
\begin{aligned}
  2 \leq p \leq 5 \\
  6 \leq q \leq 17  
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

## QUBO++ program for factorization
The following QUBO++ program constructs the HUBO expression $f(p,q)$, and solves the optimization problem using the Easy Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 5;
  auto q = 6 <= qbpp::var_int("q") <= 17;
  auto f = p * q == 35;
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "p = " << sol(p) << std::endl;
  std::cout << "q = " << sol(q) << std::endl;
}
```

this program, the expression `p * q == 35` is automatically converted into `qbpp::sqr(p * q - 35)`, which achieves an energy value of 0 when the equality is satisfied.
The output of this program is as follows:

{% raw %}
```cpp
f = 529 -240*p[0] -408*p[1] -88*q[0] -168*q[1] -304*q[2] -304*q[3] +144*p[0]*p[1] -5*p[0]*q[0] +40*p[0]*q[2] +40*p[0]*q[3] +16*p[1]*q[0] +56*p[1]*q[1] +208*p[1]*q[2] +208*p[1]*q[3] +16*q[0]*q[1] +32*q[0]*q[2] +32*q[0]*q[3] +64*q[1]*q[2] +64*q[1]*q[3] +128*q[2]*q[3] +52*p[0]*p[1]*q[0] +112*p[0]*p[1]*q[1] +256*p[0]*p[1]*q[2] +256*p[0]*p[1]*q[3] +20*p[0]*q[0]*q[1] +40*p[0]*q[0]*q[2] +40*p[0]*q[0]*q[3] +80*p[0]*q[1]*q[2] +80*p[0]*q[1]*q[3] +160*p[0]*q[2]*q[3] +48*p[1]*q[0]*q[1] +96*p[1]*q[0]*q[2] +96*p[1]*q[0]*q[3] +192*p[1]*q[1]*q[2] +192*p[1]*q[1]*q[3] +384*p[1]*q[2]*q[3] +16*p[0]*p[1]*q[0]*q[1] +32*p[0]*p[1]*q[0]*q[2] +32*p[0]*p[1]*q[0]*q[3] +64*p[0]*p[1]*q[1]*q[2] +64*p[0]*p[1]*q[1]*q[3] +128*p[0]*p[1]*q[2]*q[3]
sol = 0:{{p[0],1},{p[1],1},{q[0],1},{q[1],0},{q[2],0},{q[3],0}}
p = 5
q = 7
```
{% endraw %}
From the output, we can observe that the expression `f` contains quartic terms, confirming that it is a HUBO expression.
The solver correctly finds the prime factors $p=5$ and $q=7$.

## Unlimited large coefficients for prime factorization of large numbers
By default, the data types of expression coefficients and energy values in QUBO++ are `int32_t` and `int64_t`, respectively.
These types can be changed by defining the macros `COEFF_TYPE` and `ENERGY_TYPE`.

Furthermore, QUBO++ supports expressions with arbitrarily large coefficients and energy values.
To enable this option, both macros can be set to `qbpp::cpp_int`.
The following QUBO++ program factorizes the product of two large prime numbers:
```cpp
#define COEFF_TYPE qbpp::cpp_int
#define ENERGY_TYPE qbpp::cpp_int

#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto p = 2 <= qbpp::var_int("p") <= qbpp::cpp_int("2000000");
  auto q = 2 <= qbpp::var_int("q") <= qbpp::cpp_int("2000000");
  auto f = p * q == qbpp::cpp_int("1000039") * qbpp::cpp_int("1000079");
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "p = " << sol(p) << std::endl;
  std::cout << "q = " << sol(q) << std::endl;
}
```

Before including `qbpp.hpp`, the macros `COEFF_TYPE` and `ENERGY_TYPE` are set to `qbpp::cpp_int`.
Constant integers are specified using `qbpp::cpp_int()` with a string literal as its argument.

This program outputs the following result:
{% raw %}
```txt
f = 1000236020078726181467929 -4000472012304*p[0] -8000944024600*p[1] -16001888049168*p[2] -32003776098208*p[3] -64007552195904*p[4] -128015104389760*p[5] -256030208771328*p[6] -512060417509888*p[7] -1024120834888704*p[8] -2048241669253120*p[9] -4096483336409088*p[10] -8192966664429568*p[11] -16385933295304704*p[12] -32771866456391680*p[13] -65543732375912448*p[14] -131087462604341248*p[15] -262174916618747904*p[16] -524349798877757440*p[17] -1048699460316561408*p[18] -2097398370877308928*p[19] -3806137462543214568*p[20] -4000472012304*q[0] -8000944024600*q[1] -16001888049168*q[2] -32003776098208*q[3] -64007552195904*q[4] -128015104389760*q[5] -256030208771328*q[6] -512060417509888*q[7] -1024120834888704*q[8] -2048241669253120*q[9] -4096483336409088*q[10] -8192966664429568*q[11] -16385933295304704*q[12] -32771866456391680*q[13] -65543732375912448*q[14] -131087462604341248*q[15] -262174916618747904*q[16] -524349798877757440*q[17] -1048699460316561408*q[18] -2097398370877308928*q[19] -3806137462543214568*q[20] 

[omitted]

+17139313073086877663232*p[19]*p[20]*q[14]*q[19] +31102631877776215375872*p[19]*p[20]*q[14]*q[20] +4284828268271719415808*p[19]*p[20]*q[15]*q[16] +8569656536543438831616*p[19]*p[20]*q[15]*q[17] +17139313073086877663232*p[19]*p[20]*q[15]*q[18] +34278626146173755326464*p[19]*p[20]*q[15]*q[19] +62205263755552430751744*p[19]*p[20]*q[15]*q[20] +17139313073086877663232*p[19]*p[20]*q[16]*q[17] +34278626146173755326464*p[19]*p[20]*q[16]*q[18] +68557252292347510652928*p[19]*p[20]*q[16]*q[19] +124410527511104861503488*p[19]*p[20]*q[16]*q[20] +68557252292347510652928*p[19]*p[20]*q[17]*q[18] +137114504584695021305856*p[19]*p[20]*q[17]*q[19] +248821055022209723006976*p[19]*p[20]*q[17]*q[20] +274229009169390042611712*p[19]*p[20]*q[18]*q[19] +497642110044419446013952*p[19]*p[20]*q[18]*q[20] +995284220088838892027904*p[19]*p[20]*q[19]*q[20]
sol = 0:{{p[0],0},{p[1],1},{p[2],1},{p[3],1},{p[4],0},{p[5],0},{p[6],0},{p[7],0},{p[8],0},{p[9],1},{p[10],1},{p[11],1},{p[12],1},{p[13],1},{p[14],0},{p[15],1},{p[16],0},{p[17],0},{p[18],0},{p[19],0},{p[20],1},{q[0],0},{q[1],1},{q[2],1},{q[3],0},{q[4],0},{q[5],1},{q[6],1},{q[7],1},{q[8],1},{q[9],0},{q[10],1},{q[11],1},{q[12],1},{q[13],1},{q[14],0},{q[15],1},{q[16],0},{q[17],0},{q[18],0},{q[19],0},{q[20],1}}
p = 1000079
q = 1000039
```
{% endraw %}
We can see that the expression `f` contains very large coefficients, and the factorization of the large composite number is correctly obtained.

>**TIP**
> `COEFF_TYPE` and `ENERGY_TYPE` can be set to `int8_t`, `int16_t`, `int32_t`, `int64_t`, `qbpp::int128_t`, `qbpp::int256_t`, `qbpp::int512_t`, `qbpp::int1024_t`, or `qbpp::cpp_int`.