---
layout: default
title: "ABS3"
---

# ABS3 Solver Usage
Solving an expression `f` using the ABS3 Solver involves the following three steps:
1. Create an ABS3 Solver (or **`qbpp::abs3::ABS3Solver`**) object for the expression `f`.
2. Create a parameter object (or **`qbpp::abs3::Params`**) and set the options for solution search.
3. Call the **`search()`** member function of the ABS3 Solver object, which returns the obtained solution.

## Solving LABS problem using the ABS3 Solver
The following QUBO++ program solves the **Low Autocorrelation Binary Sequence (LABS)** problem using the ABS3 Solver:
```cpp
#include "qbpp.hpp"
#include "qbpp_abs3_solver.hpp"

int main() {
  const size_t size = 100;
  auto x = qbpp::var("x", size);
  auto f = qbpp::expr();
  for (size_t d = 1; d < size; ++d) {
    auto temp = qbpp::expr();
    for (size_t i = 0; i < size - d; ++i) {
      temp += (2 * x[i] - 1) * (2 * x[i + d] - 1);
    }
    f += qbpp::sqr(temp);
  }
  f.simplify_as_binary();

  auto solver = qbpp::abs3::ABS3Solver(f);
  auto params = qbpp::abs3::Params();
  params("time_limit", "10.0");
  params("enable_callback", "1");

  auto sol = solver.search(params);
  std::cout << sol.energy() << ": ";
  for (auto val : sol(x)) {
    std::cout << (val == 0 ? "-" : "+");
  }
  std::cout << std::endl;
}
```
In this program, an ABS3 Solver object **`solver`** is first created for the expression `f`.
Next, a parameter object **`params`** is created, and the options `time_limit` and `enable_callback` are set.
The `time_limit` option specifies the maximum search time in seconds, while `enable_callback` enables a callback function that reports newly found best solutions during the search.
The **`search()`** member function of the `solver` is then invoked with `params` as its argument.
This function returns the best solution found within the given time limit, which is stored in `sol`.

The program prints the energy of the solution and the corresponding binary sequence, where "+" represents 1 and "-" represents 0.

This program produces the following output:
```
TTS = 0.002s Energy = 1218
TTS = 0.002s Energy = 1170
TTS = 0.002s Energy = 994
TTS = 0.015s Energy = 958
TTS = 0.018s Energy = 922
TTS = 0.034s Energy = 874
TTS = 4.364s Energy = 834
834: -+--+---++-++-+---++-++--+++--+-+-+++++----+++-+-+---++-+--+-----+--+----++----+-+--++++++---+------
```

## ABS3 Solver object
An ABS3 Solver (or `qbpp::abs3::ABS3Solver`) object is created for a given expression.
When the solver object is constructed, the expression is converted into an internal data format and loaded into GPU memory.
Therefore, the number of GPU devices to be used can be specified as a second argument:
- **`qbpp::abs3::ABS3Solver(expression, device_count)`**:
If `device_count` is omitted, all available GPUs are used by default.

A parameter object (or `qbpp::abs3::Params`) stores the options used by the ABS3 Solver during solution search.
Each option is specified as a key–value pair of strings.
In the example above:
- **`params("time_limit", "10.0")`**: Sets the time limit to 10.0 seconds.
- **`params("enable_callback", "1")`**: Enables the callback function, which reports the energy of newly obtained solutions.

## ABS3 Parameters
A parameter (or `qbpp::abs3::Param()`) object stores parameter options used when the ABS3 Solver searches a solution.
Options consists of the key and the value as strings.
In the program above, `params("time_limit", "10.0")` set the time limit to 10.0 seconds
and `params("enable_callback", "1")` enables the callback function, which displays the energy of newly obtained solution.

### Basic Options

| Key | Value | Description |
|----|----|----|
| **`time_limit`** | time limit in seconds | Terminates the search when the time limit is reached |
| **`target_energy`** | target enagy value | Terminates the search when the target energy is achieved |

### Advanced Options

| Key | Value | Description |
|----|----|----|
| **`enable_callback`** | "1" | Enables the callback function |
| **`block_count`** | CUDA block count per GPU | Number of CUDA blocks launched by the solver kernel |
| **`thread_count`** | thread count per CUDA block | Number of threads per CUDA block|
