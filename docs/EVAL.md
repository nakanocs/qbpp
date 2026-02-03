---
layout: default
title: "Evaluating Expressions"
---

# Evaluating Expressions

## Evaluation using qbpp::Maplist
The value of expressions can be simply done by providing an assignment of values to all elements
as a list of pairs of a variable and its value.
A list can be defined as a **`qbpp::MapList`** object.
For example, the following program computes the function $f(x,y,z)$ for $(x,y,z)=(0,1,1)$.
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto z = qbpp::var("z");
  auto f = qbpp::sqr(x + 2 * y + 3 * z - 3);

  qbpp::MapList ml;
  ml.push_back({x, 0});
  ml.push_back({y, 1});
  ml.push_back({z, 1});
  
  std::cout << ml << std::endl;
  std::cout << "f(0,1,1) = " << f(ml) << std::endl;
}
```

In this program, `qbpp::MapList` object `ml` is defined,
and an assignment `{x, 0}`, `{y, 1}` and `{z, 1}` is appended to `ml`.
Then `f(ml)` returns the value of $f(0,1,1)$.
This program displays the following output:

{% raw %}
```
{{x,0},{y,1},{z,1}}
f(0,1,1) = 4
```
{% endraw %}

Alternatively, we can provide an assignment directly as follows:
{% raw %}
```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto z = qbpp::var("z");
  auto f = qbpp::sqr(x + 2 * y + 3 * z - 3);

  std::cout << "f(0,1,1) = " << f({{x, 0}, {y, 1}, {z, 1}}) << std::endl;
}
```
{% endraw %}


## Evaluation using qbpp::Sol
A solution object (**`qbpp::Sol`**) can be used to evaluate the value of an expression (**`qbpp::Expr`**).
To do this, we first construct a `qbpp::Sol` object sol associated with a given expression `f`.
The newly created **`qbpp::Sol`** object is initialized with the all-zero assignment.

Using the **`set()`** member function of `qbpp::Sol`, we can assign values to individual variables.
Then, both **`f(sol)`** and **`sol(f)`** return the value of the expression f under the assignment stored in sol.
Furthermore, the **`comp_energy()`** member function computes and returns the same value.


```cpp
#include "qbpp.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto z = qbpp::var("z");
  auto f = qbpp::sqr(x + 2 * y + 3 * z - 3);

  qbpp::Sol sol(f);
  sol.set(y, 1);
  sol.set(z, 1);
  
  std::cout << "f(0,1,1) = " << f(sol) << std::endl;
  std::cout << "f(0,1,1) = " << sol(f) << std::endl;
  std::cout << "f(0,1,1) = " << sol.comp_energy() << std::endl;
}
```

Note that the member function **`comp_energy()`** of a solution object `sol` computes the energy value and caches it internally.
In addition, a solution object returned by a solver already has its energy value computed and cached.
To retrieve the energy without recomputing it, you can use the member function **`energy()`**, as shown below:
```cpp
#include "qbpp.hpp"
#include "qbpp_easy_solver.hpp"

int main() {
  auto x = qbpp::var("x");
  auto y = qbpp::var("y");
  auto z = qbpp::var("z");
  auto f = qbpp::sqr(x + 2 * y + 3 * z - 4);

  f.simplify_as_binary();
  auto solver = qbpp::easy_solver::EasySolver(f);
  solver.target_energy(0);
  auto sol = solver.search();
  
  std::cout << "sol = " << sol << std::endl;
  std::cout << "energy = " << sol.energy() << std::endl;
  
  sol.flip(z);
  std::cout << "flipped sol = " << sol << std::endl;
  std::cout << "flipped energy = " << sol.energy() << std::endl;
}
```
In this program, `sol.energy()` correctly returns 0.
However, after flipping the variable `z`, the cached energy value becomes invalid.
Calling `sol.energy()` without recomputing the energy therefore results in **a runtime error**, as shown below:
{% raw %}
```
sol = 0:{{x,1},{y,0},{z,1}}
energy = 0
terminate called after throwing an instance of 'std::runtime_error'
```
{% endraw %}
To resolve this issue, you must explicitly recompute the energy by calling **`sol.comp_energy()`** after modifying the solution, as follows:
```cpp
  std::cout << "sol = " << sol << std::endl;
  std::cout << "energy = " << sol.energy() << std::endl;
  
  sol.flip(z);
  std::cout << "sol.comp_energy() = " << sol.comp_energy() << std::endl;
  std::cout << "flipped sol = " << sol << std::endl;
  std::cout << "flipped energy = " << sol.energy() << std::endl;
```
This program produces the following output:
{% raw %}
```
sol = 0:{{x,1},{y,0},{z,1}}
energy = 0
sol.comp_energy() = 9
flipped sol = 9:{{x,1},{y,0},{z,0}}
flipped energy = 9
```
{% endraw %}