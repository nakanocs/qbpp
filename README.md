# QUBO++

QUBO++ is a C++ library for creating and solving QUBO (Quadratic Unconstrained Binary Optimization)
and HUBO (Higher-order Unconstrained Binary Optimization) problems.

---

## Solvers
### Easy Solver
- Runs on x86_64 multicore CPU.
- Parallel search using Intel Threading Building Blocks (TBB).
- Supports unlimited integer coefficients.

### Exhaustive Solver
- Searches all possible solutions on x86_64 multicore CPUs.
- Guaranteed optimality.
- Parallel search using TBB.
- Supports unlimited integer coefficients.

### ABS3 Solver
- Runs on CUDA-enabled GPUs hosted on x86_64 multicore CPUs.
- Supports GPU Architectures: sm_80, sm_86, sm_89, sm_90, sm_100, sm_120
- Supports QUBO/HUBO with 32-bit and 64-bit term coefficients.

