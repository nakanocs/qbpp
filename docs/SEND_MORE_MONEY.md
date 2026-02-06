---
layout: default
title: "SEND+MORE=MONEY"
---

# Math Puzzle: SEND MORE MONEY

**SEND + MORE = MONEY** is a famous alphametic puzzle: assign a decimal digit to each letter so that
$$
\text{SEND}+\text{MORE}=\text{MONEY}
$$

The constraints are:
- The digits assigned to letters are all distinct.
- `S` and `M` must not be 0.

## QUBO++ formulation

We assign a unique index to each letter as follows:

| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| letter | S | E | N | D | M | O | R | Y |

Let $I(\alpha)$ denote the index of letter $\alpha$ ($\in \lbrace S,E,N,D,M,O,R,Y\rbrace$).
We use an $8\times 10$ binary matrix $X=(x_{i,j})$ $(0\leq i\leq 7, 0\leq j\leq 9)$ to represent the digit assigned to each letter:  $x_{I(\alpha),j}=1$ if and only if letter $\alpha$ is assigned digit $j$.

### One-hot constraints (each letter takes exactly one digit)
Each row of $X$ must be one-hot:

$$
\begin{aligned}
\text{onehot} &=\sum_{i=0}^{7}\Bigl(\sum_{j=0}^{9}x_{i,j}=1\Bigr) \\
              &=\sum_{i=0}^{7}\Bigl(1-\sum_{j=0}^{9}x_{i,j}\Bigr)^2
\end{aligned}
$$

The value of $\text{onehot}$ is minimized to 0 if and only if every row is one-hot.

### All-different constraints (no two letters share the same digit)
Digits must be distinct across letters, i.e., no two rows choose the same column:
$$
\begin{aligned}
\text{different} &=\sum_{0\leq i<j\leq 7}\sum_{k=0}^9x_{i,k}x_{j,k}
\end{aligned}
$$

### Encoding the words as linear expressions
The values of $\text{SEND}$, $\text{MORE}$, and $\text{MONEY}$ are represented
by:

$$
\begin{aligned}
\text{SEND} &= 1000\sum_{k=0}^9 kx_{I(S),k}+ 100\sum_{k=0}^9 kx_{I(E),k}+ 10\sum_{k=0}^9 kx_{I(N),k}+\sum_{k=0}^9 kx_{I(D),k}\\ 
       &= \sum_{k=0}^9k(1000x_{I(S),k}+100x_{I(E),k}+10x_{I(N),k}+x_{I(D),k})\\
\text{MORE} &= 1000\sum_{k=0}^9 kx_{I(M),k}+ 100\sum_{k=0}^9 kx_{I(O),k}+ 10\sum_{k=0}^9 kx_{I(R),k}+\sum_{k=0}^9 kx_{I(E),k}\\ 
       &= \sum_{k=0}^9k(1000x_{I(M),k}+100x_{I(O),k}+10x_{I(R),k}+x_{I(E),k})\\
\text{MONEY} &= 10000\sum_{k=0}^9 kx_{I(M),k}+1000\sum_{k=0}^9 kx_{I(O),k}+ 100\sum_{k=0}^9 kx_{I(N),k}+ 10\sum_{k=0}^9 kx_{I(E),k}+\sum_{k=0}^9 kx_{I(Y),k}\\ 
       &= \sum_{k=0}^9k(10000x_{I(M),k}+ 1000x_{I(O),k}+100x_{I(N),k}+10x_{I(E),k}+x_{I(Y),k})
\end{aligned}
$$

### Equality constraint
We enforce the equation by penalizing the residual:

$$
\begin{aligned}
\text{equal} &= \Bigl(\text{SEND}+\text{MORE} = \text{MONEY}\Bigr) \\
             &= \Bigl(\text{SEND}+\text{MORE} - \text{MONEY}\Bigr)^2
\end{aligned}
$$

### Combined objective
All constraints are combined into a single HUBO objective:

$$
\begin{aligned}
f & = P\cdot (\text{onehot}+\text{different})+\text{equal}
\end{aligned}
$$
where
`P` is a sufficiently large constant to prioritize feasibility (`onehot` and `different`).
In principle, if all terms are nonnegative and each becomes 0 exactly when its constraint holds, then any solution with $f=0$ satisfies all constraints.
In practice, choosing a larger `P` often helps heuristic solvers.

In this case, there is no need to prioritize them and we can set $P=1$,
because $\text{equal}\geq 0$ always holds and $f$ takes a minimum value of 0
only if $\text{onehot}=\text{different}=\text{equal}=0$ holds.
However, a large constant $P$ helps solvers to find the optimal solution.

Finally, since $\text{S}$ and $\text{M}$ must not be 0, we fix
the binary variables as follows:
$$
x_{I(S),0} = x_{I(M),0}= 0
$$

## QUBO++ program for SEND+MORE=MONEY
The following QUBO++ program implements the HUBO formulation above and finds a solution using EasySolver:
{% raw %}
```cpp
#define COEFF_TYPE qbpp::int128_t
#define ENERGY_TYPE qbpp::int128_t

#include <string_view>

#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

constexpr std::string_view LETTERS = "SENDMORY";
constexpr size_t L = LETTERS.size();

constexpr size_t I(char c) {
  for (size_t i = 0; i < LETTERS.size(); ++i) {
    if (LETTERS[i] == c) return i;
  }
  return L;
}

const qbpp::Vector<int> K = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

int main() {
  auto x = qbpp::var("x", L, 10);


  auto onehot = qbpp::sum(qbpp::vector_sum(x) == 1);

  auto different = qbpp::toExpr(0);
  for (size_t i = 0; i < L - 1; ++i) {
    for (size_t j = i + 1; j < L; ++j) {
      different += qbpp::sum(x[i] * x[j]);
    }
  }

  auto send = qbpp::sum((x[I('S')] * 1000 + x[I('E')] * 100 + x[I('N')] * 10 + x[I('D')]) * K);
  auto more = qbpp::sum((x[I('M')] * 1000 + x[I('O')] * 100 + x[I('R')] * 10 + x[I('E')]) * K);
  auto money = qbpp::sum((x[I('M')] * 10000 + x[I('O')] * 1000 + x[I('N')] * 100 + x[I('E')] * 10 + x[I('Y')]) * K);


  auto equal = send + more - money == 0;

  qbpp::coeff_t P = 10000;
  auto f = P * (onehot + different) + equal;

  f.simplify_as_binary();

  qbpp::MapList ml = {{x[I('S')][0], 0}, {x[I('M')][0], 0}};
  auto g = qbpp::replace(f, ml);

  g.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(g);
  solver.target_energy(0);

  auto sol = solver.search();

  auto full_sol = qbpp::Sol(f).set(sol).set(ml);

  std::cout << "onehot = " << full_sol(onehot) << std::endl;
  std::cout << "different = " << full_sol(different) << std::endl;
  std::cout << "equal = " << full_sol(equal) << std::endl;

  auto val = qbpp::onehot_to_int(full_sol(x));

  auto str = [](int d) -> std::string {
    return (d < 0) ? "*" : std::to_string(d);
  };

  std::cout << "SEND + MORE = MONEY" << std::endl;
  std::cout << str(val[I('S')]) << str(val[I('E')]) << str(val[I('N')])
            << str(val[I('D')]) << " + " << str(val[I('M')]) << str(val[I('O')])
            << str(val[I('R')]) << str(val[I('E')]) << " = " << str(val[I('M')])
            << str(val[I('O')]) << str(val[I('N')]) << str(val[I('E')])
            << str(val[I('Y')]) << std::endl;
}
```
{% endraw %}
In this program, `LETTERS` assigns an integer index to each letter in `"SENDMORY"`, which is used to implement $I(\alpha)$.
We define an `L`$\times$`10` matrix `x` of binary variables (here $L=8$).
The expressions `onehot`, `different`, and `equal` are computed according to the formulation and combined into a single objective `f` with a penalty weight `P`.

We use a `qbpp::MapList` object `ml` to fix `x[I('S')][0]` and `x[I('M')][0]` to 0, and create a reduced expression g by applying this replacement.
The solver is run on `g`, and the resulting assignment `sol` is merged with the fixed mapping `ml` to produce `full_sol` for the original objective f.

Finally, `qbpp::onehot_to_int(full_sol(x))` converts the one-hot rows into digits, and the program prints the obtained solution.
This program produces the following output:
```
onehot = 0
different = 0
equal = 0
SEND + MORE = MONEY
9567 + 1085 = 10652
```
This confirms that all constraints are satisfied and the correct solution is obtained.