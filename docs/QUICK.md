---
layout: default
title: "Quick Start"
---


# Quick Start

## Download and Installation
Download the latest `tar.gz` from <a href="https://github.com/nakanocs/qbpp/releases"><strong>Releases</strong></a> and extract it:
```bash
$ tar xf qbpp_<arch>_<version>.tar.gz
```
QUBO++ will be extracted into a directory such as `qbpp_<arch>_<version>`.

## Set Environment Variable
Set the environment variables as follows:
```bash
$ export QBPP_PATH=[QUBO++ install directory]
$ export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
$ export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
$ export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
```

## Compile and execute a sample program
### Create a QUBO++ sample program below and save as file `test.cpp`:

```cpp
#include "qbpp.hpp"
#include "qbpp_exhaustive_solver.hpp"

int main() {
  auto x = 0 <= qbpp::var_int("x") <= 10;
  auto y = 0 <= qbpp::var_int("y") <= 10;
  auto f = x + y == 10;
  auto g = 2 * x + 4 * y == 28;
  auto h = f + g;
  h.simplify_as_binary();
  auto solver = qbpp::exhaustive_solver::ExhaustiveSolver(h);
  auto sol = solver.search();
  std::cout << "sol = " << sol << std::endl;
  std::cout << "x = " << sol(x) << ", y = " << sol(y) << std::endl;
}
```

### Compile the program:
Compile the program:
```bash
$ g++ test.cpp -o test -std=c++17 -lqbpp -ltbb
```

### Execute the program:
Run the program:
{% raw %}
```
$ ./test
sol = 0:{{x[0],0},{x[1],1},{x[2],1},{x[3],0},{y[0],0},{y[1],0},{y[2],1},{y[3],0}}
x = 6, y = 4
```
{% endraw %}

## Next to do
1. AActivate the Anonymous Trial license or your license key. See Installation in **Installation** in [QUBO++ Document](DOCUMENT).
2. Learn the basics of QUBO++. Start from **Basics** in [QUBO++ Document](DOCUMENT).

