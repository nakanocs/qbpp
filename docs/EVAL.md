---
layout: default
title: "Evaluating Expressions"
---

# Evaluating Expressions

## Evaluation using qbpp::Maplist
The value of expressions can be simply done by providing an assignment of values to all elements
as a list of pairs of a variable and its value.
A list can be defined as a `qbpp::MapList` object.
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
This program displays the follwing output:

{% raw %}
```
{{x,0},{y,1},{z,1}}
f(0,1,1) = 4
```
{% endraw %}

Alternratively, we can provide an assignemt directly as follows:
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
A solution (or qbpp::Sol) object can be used to evaluate the value of expression (or qbpp::Expr).
For this purpose, we first create a qbpp::Sol object `sol` with respect to qbpp::Expr object `f`.
A created qbpp::Sol object `f` is initialized by all zero solution.
By `set()` member function for the qbpp::Sol object, we can assign a new value to a variable.
Then `f(sol)` and `sol(f)` returns the resulting values of `f` for `sol`.
Furthermore, `comp_energy()` member function computes this value and returns it.


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
