---
layout: default
title: "Home"
nav_order: 1
---
# QUBO++: A model-and-solve framework for combinatorial optimization via QUBO/HUBO

* A **C++ library** for constructing polynomials of binary variables to solve combinatorial optimization problems.
* Currently available for **amd64** (**x86_64**) and **arm64** Linux systems.
* Bundled with two CPU solvers: **Easy Solver** (heuristic) and **Exhaustive Solver** (complete search).
* Bundled with a GPU solver: **ABS3**. The Exhaustive Solver also supports GPU acceleration when a CUDA GPU is available.
* Supports **High-Order Unconstrained Binary Optimization (HUBO)** with unlimited degree.
* Natively supports variable complements ($\overline{x}$ or `~x`) without replacing them with $1-x$,
  which avoids term explosion in high-order terms.
* Applies **multithreading acceleration** using oneTBB wherever possible.
* Includes APIs for calling the **Gurobi Optimizer** to solve Quadratic Unconstrained Binary Optimization (QUBO) problems.
* Copyright: © 2026 Koji Nakano. All rights reserved.

# QUBO++ Solvers: Easy Solver, Exhaustive Solver, ABS3 Solver
## Easy Solver
* **Heuristic solver optimized for QUBO/HUBO**: Searches for solutions to QUBO/HUBO models on multicore CPUs.
* **Multithreaded acceleration**: Uses Intel oneTBB for parallel search.
* **Unlimited integer coefficients**: Supports integer coefficients of arbitrary magnitude.

## Exhaustive Solver
* **Enumerates all solutions** to QUBO/HUBO formulations on multicore CPUs and CUDA GPUs.
* **Optimality guaranteed**: the global optimum is found and certifiable.
* **Multithreaded acceleration**: Uses Intel oneTBB for parallel search.
* **Unlimited integer coefficients**: Supports integer coefficients of arbitrary magnitude.
* **GPU acceleration**: If a CUDA GPU is available, GPU workers automatically join the search alongside CPU threads. GPU acceleration is available for coefficients up to 128-bit integers; larger coefficients fall back to CPU-only search.


## ABS3 Solver
* **Heuristic solver on multicore CPUs and CUDA GPUs**: Searches for solutions to QUBO/HUBO instances using both CPU threads and CUDA GPUs.
* **Unlimited integer coefficients**: Supports integer coefficients of arbitrary magnitude.
* **Multi-GPU scaling**: Uses all detected GPUs on a Linux host. GPU acceleration is available for coefficients up to 128-bit integers; larger coefficients fall back to CPU-only search.


### **ABS3 Supported GPU architectures**
  - **sm_80** : NVIDIA A100  (Ampere) 
  - **sm_86** : NVIDIA RTX A6000, GeForce RTX 3090/3080/3070 (Ampere) 
  - **sm_89** : NVIDIA RTX 6000 Ada, GeForce RTX 4090/4080/4070 (Ada) 
  - **sm_90** : NVIDIA H100 / H200 / GH200 (Hopper) 
  - **sm_100** : NVIDIA B200 / GB200 (Blackwell, data center) 
  - **sm_120** : GeForce RTX 5090/5080/5070(Ti)/5060(Ti)/5050、RTX PRO 6000/5000/4500/4000/2000 Blackwell (workstation)  
  - **Note on verification** : Only a subset of the architectures above has been verified on real hardware. 


### **Performance note**
  - Arithmetic overflow checks are omitted to maximize performance.


## Build Environment
The following environment was used to build QUBO++.
**QUBO++ is not limited to Ubuntu 20.04**; it has also been tested on Ubuntu 22.04/24.04 and other Linux distributions (including CentOS/RHEL-based systems).
To ensure compatibility, please use the same or newer versions of the listed components.
- **Operating System**: Ubuntu 20.04.6 LTS 
- **C++ Standard**: C++17  
- **glibc**: 2.31  
- **Compiler**: g++ 9.4.0  
- **Boost**: 1.81.0
- **CUDA**: 12.8

## oneTBB / TBB dependency
QUBO++ does not bundle TBB. We include TBB headers in public APIs, but the
library itself does not require linking against TBB for the default use cases.
- **Build & run verified on**: Ubuntu 20.04 (classic TBB 2020.1), 22.04 / 24.04 (oneTBB 2021+).

# QUBO++ Licensing

To start using QUBO++, activate the license after installation:
```bash
$ qbpp-license -a
```

If no license key is set, an **Anonymous Trial** (7 days, 1,000 variables) is activated.

| License Type | Key Required | Validity | CPU Variables | GPU Variables |
|---|---|---|---|---|
| **Anonymous Trial** | No | 7 days | 1,000 | 1,000 |
| **Registered Trial** | Yes  | 30 days | 10,000 | 10,000 |
| **Standard** | Yes  | Agreement term | $2^{31} - 1$ | 1,000 |
| **Professional** | Yes | Agreement term | $2^{31} - 1$ | $2^{31} - 1$ |
| **Fallback** | N/A | Always | 100 | 100 |

For details on license activation, deactivation, key configuration, troubleshooting, and terms and conditions, see **[License Management](LICENSE_MANAGEMENT)**.

# Third-Party Libraries

- **oneTBB (oneAPI Threading Building Blocks)**
  - Licensed under the Apache License 2.0.
  - Copyright © Intel Corporation.
  - See <https://www.apache.org/licenses/LICENSE-2.0> for details.

- **Boost C++ Libraries**
  - Licensed under the Boost Software License, Version 1.0.
  - See <https://www.boost.org/LICENSE_1_0.txt> for details.

- **xxHash**
  - Licensed under the BSD 2-Clause License.
  - Copyright © Yann Collet.
  - See <https://opensource.org/license/bsd-2-clause/> for details.

- **Gurobi Optimizer**
  - QUBO++ supporting APIs for integration with the Gurobi Optimizer.
  - To use these APIs, you must obtain a valid license for the Gurobi Optimizer from Gurobi Optimization, LLC.
  - The Gurobi Optimizer is not included with QUBO++, and its use is subject to the terms and conditions of its own license agreement.

