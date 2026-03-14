---
layout: default
title: "ABS3 Solver"
nav_order: 21
parent: "C++ Document"
---
# ABS3 Solver Usage
Solving an expression `f` using the ABS3 Solver involves the following three steps:
1. Create an ABS3 Solver (or **`qbpp::abs3::ABS3Solver`**) object for the expression `f`.
2. Create a parameter object (or **`qbpp::abs3::Params`**) and set the options for solution search.
3. Call the **`search()`** member function of the ABS3 Solver object, which returns the obtained solution.

## Solving LABS problem using the ABS3 Solver
The following QUBO++ program solves the **Low Autocorrelation Binary Sequence (LABS)** problem using the ABS3 Solver:
```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

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
  params("enable_default_callback", "1");

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
The `time_limit` option specifies the maximum search time in seconds, while `enable_default_callback` enables a built-in callback function that prints the energy and TTS of newly found best solutions during the search.
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
An optional second argument `gpu` controls GPU usage:
- **`qbpp::abs3::ABS3Solver(expression)`**: Automatically uses all available GPUs. If no GPU is available, falls back to CPU-only mode.
- **`qbpp::abs3::ABS3Solver(expression, 0)`**: Forces CPU-only mode (no GPU is used).
- **`qbpp::abs3::ABS3Solver(expression, n)`**: Uses `n` GPUs.

A parameter object (or `qbpp::abs3::Params`) stores the options used by the ABS3 Solver during solution search.
Each option is specified as a key–value pair of strings.
In the example above:
- **`params("time_limit", "10.0")`**: Sets the time limit to 10.0 seconds.
- **`params("enable_default_callback", "1")`**: Enables the built-in callback function, which prints the energy of newly obtained solutions.

## ABS3 Parameters
A parameter (or `qbpp::abs3::Params()`) object stores parameter options used when the ABS3 Solver searches a solution.
Options consist of the key and the value as strings.
In the program above, `params("time_limit", "10.0")` sets the time limit to 10.0 seconds
and `params("enable_default_callback", "1")` enables the built-in callback function, which prints the energy of newly obtained solutions.

### Basic Options

| Key | Value | Description |
|----|----|----|
| **`time_limit`** | time limit in seconds | Terminates the search when the time limit is reached |
| **`target_energy`** | target energy value | Terminates the search when the target energy is achieved |

### Advanced Options

| Key | Value | Description |
|----|----|----|
| **`enable_default_callback`** | "1" | Enables the built-in callback that prints energy and TTS |
| **`cpu_enable`** | "0" or "1" | Enables/disables the CPU solver running alongside the GPU (default: "1") |
| **`cpu_thread_count`** | number of CPU threads | Number of CPU solver threads (default: auto) |
| **`block_count`** | CUDA block count per GPU | Number of CUDA blocks launched by the solver kernel |
| **`thread_count`** | thread count per CUDA block | Number of threads per CUDA block |
| **`topk_sols`** | number of solutions | Returns the top-K solutions with the best energies |

## Custom Callback

The built-in callback (enabled by `enable_default_callback`) simply prints the energy and TTS whenever a new best solution is found.
For more control, you can subclass `ABS3Solver` and override the `callback()` virtual method.

The callback is invoked with one of the following events:

| Event | Description |
|-------|-------------|
| `CallbackEvent::Start` | Called once at the beginning of `search()` |
| `CallbackEvent::BestUpdated` | Called whenever a new best solution is found |
| `CallbackEvent::Timer` | Called periodically at a configurable interval |

Inside the callback, the following accessor methods are available:
- **`best_sol()`** — returns `const qbpp::Sol&` to the current best solution. Use `.energy()`, `.tts()`, `.get(var)`, etc.
- **`current_event()`** — the event that triggered this callback

### Timer Control

The `Timer` event is not enabled by default.
To enable periodic timer callbacks, call `timer(seconds)` inside the `callback()` method:
- **`timer(1.0)`** — fire `Timer` callbacks every 1 second
- **`timer(0)`** — disable the timer
- If `timer()` is not called, the timer interval remains unchanged.

Typically, `timer()` is called once during the `Start` callback to establish the interval.
It can also be called during `BestUpdated` or `Timer` callbacks to adjust or disable the timer dynamically.

### Example: Custom Callback
```cpp
#define MAXDEG 2
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>

class MySolver : public qbpp::abs3::ABS3Solver {
 public:
  using ABS3Solver::ABS3Solver;

  void callback(qbpp::abs3::CallbackEvent event) const override {
    if (event == qbpp::abs3::CallbackEvent::Start) {
      timer(1.0);  // enable timer callback every 1 second
    }
    if (event == qbpp::abs3::CallbackEvent::BestUpdated) {
      std::cout << "New best: energy=" << best_sol().energy()
                << " TTS=" << best_sol().tts() << "s" << std::endl;
    }
  }
};

int main() {
  auto x = qbpp::var("x", 8);
  auto f = qbpp::sum(x) == 4;
  f.simplify_as_binary();

  auto solver = MySolver(f);
  auto params = qbpp::abs3::Params();
  params.add("time_limit", "5");
  params.add("target_energy", "0");
  auto sol = solver.search(params);
  std::cout << "energy=" << sol.energy() << std::endl;
}
```

## Solution Injection

The `inject()` method allows you to inject an external solution into the solver during a callback.
This is useful for warm-starting a search with a previously found solution.
The injected bit array is evaluated internally by the solver, and if its energy is better than the current best, it replaces it.

Another important use case is running an external solver (e.g., Gurobi) concurrently with the ABS3 solver.
The external solver runs in a separate thread, and whenever it finds a good solution, the solution is injected into the ABS3 solver via the `Timer` callback.
In this scenario, setting up a periodic timer (e.g., `timer(1.0)`) is recommended so that the callback is invoked regularly to check for new external solutions.

Two overloads are available:
- **`inject(const uint64_t* bitarray)`** — inject a raw bit array
- **`inject(const qbpp::Sol& sol)`** — inject a `Sol` object

The `inject()` method can be called during any callback event, but the most common usage is to inject at `CallbackEvent::Start` to warm-start the search, or during `CallbackEvent::Timer` to periodically feed solutions from an external solver.

### Example: Injecting a Previous Solution

The following example solves a factorization problem twice.
The first run finds the optimal solution normally.
The second run injects the first solution at the start of the search, causing the solver to converge much faster.

```cpp
#define MAXDEG 4
#include <qbpp/qbpp.hpp>
#include <qbpp/abs3_solver.hpp>
#include <vector>

class InjectSolver : public qbpp::abs3::ABS3Solver {
 public:
  using ABS3Solver::ABS3Solver;

  void set_inject_solution(const std::vector<uint64_t>& bitarray) {
    inject_bitarray_ = bitarray;
    has_inject_ = true;
  }

  void callback(qbpp::abs3::CallbackEvent event) const override {
    if (event == qbpp::abs3::CallbackEvent::Start) {
      timer(1.0);
      if (has_inject_) {
        inject(inject_bitarray_.data());
        has_inject_ = false;
      }
    }
    if (event == qbpp::abs3::CallbackEvent::BestUpdated) {
      std::cout << "  energy=" << best_sol().energy()
                << " TTS=" << best_sol().tts() << "s" << std::endl;
    }
  }

 private:
  mutable std::vector<uint64_t> inject_bitarray_;
  mutable bool has_inject_ = false;
};

int main() {
  auto p = 2 <= qbpp::var_int("p") <= 1000;
  auto q = 2 <= qbpp::var_int("q") <= 1000;
  auto f = p * q == 899 * 997;
  f.simplify_as_binary();

  auto solver = InjectSolver(f);

  // Run 1: normal search
  auto params1 = qbpp::abs3::Params();
  params1.add("target_energy", "0");
  params1.add("time_limit", "10");
  const auto sol1 = solver.search(params1);
  std::cout << "Run 1: p=" << sol1(p) << " q=" << sol1(q)
            << " energy=" << sol1.energy() << std::endl;

  // Save solution for injection
  size_t size64 = (static_cast<size_t>(qbpp::all_var_count()) + 63) / 64;
  std::vector<uint64_t> bits(sol1.bit_vector().bits_ptr(),
                             sol1.bit_vector().bits_ptr() + size64);
  solver.set_inject_solution(bits);

  // Run 2: inject previous solution at Start
  auto params2 = qbpp::abs3::Params();
  params2.add("target_energy", "0");
  params2.add("time_limit", "10");
  const auto sol2 = solver.search(params2);
  std::cout << "Run 2: p=" << sol2(p) << " q=" << sol2(q)
            << " energy=" << sol2.energy()
            << " TTS=" << sol2.tts() << "s" << std::endl;
}
```

Note that the `inject()` method only sets the initial state of the solver.
The solver continues searching from the injected solution and may find better solutions.
