# README Version: 2025.11.15
* [**Releases**](https://github.com/nakanocs/qbpp/releases)
* [**Documents**](https://github.com/nakanocs/qbpp/blob/main/docs/DOCUMENT.md)
* [**Hands-on**](https://github.com/nakanocs/qbpp/blob/main/docs/handson/HANDSON.md)
# QUBO++
* A C++ library for constructing polynomials of binary variables to solve combinatorial optimization problems  
* Currently built for **x86_64 Linux systems**  
* Bundled with two CPU solvers: Easy Solver (heuristic) and Exhaustive Solver (complete search)  
* Bundled with a GPU solver: ABS3  
* Supports High-order Unconstrained Binary Optimization (HUBO) up to degree 8  
* Multithreading acceleration is applied using oneTBB wherever possible  
* Includes APIs for calling the Gurobi Optimizer to solve Quadratic Unconstrained Binary Optimization (QUBO) problems  
* Author: Koji Nakano  
* Copyright: © 2025 Koji Nakano. All rights reserved.

# QUBO++ Solvers: Easy Solver, Exhaustive Solver, ABS3 Solver
## Easy Solver
* Searches for solutions to QUBO/HUBO formulations on x86_64 multicore processors.
* Multithreaded acceleration: utilizes Intel oneTBB for parallel search.
* Integer coefficients: supports coefficients of arbitrary magnitude.

## Exhaustive Solver
* Enumerates all solutions to QUBO/HUBO formulations on x86_64 multicore processors.
* Optimality guaranteed: the global optimum is found and certifiable.
* Multithreaded acceleration: utilizes Intel oneTBB for parallel enumeration.
* Integer coefficients: supports coefficients of arbitrary magnitude.

## ABS3 Solver
* Searches for solutions to QUBO/HUBO formulations on CUDA-enabled GPUs in x86_64 hosts.

### **ABS3 Supported GPU architectures**
  - **sm_80** : NVIDIA A100  (Ampere) 
  - **sm_86** : NVIDIA RTX A6000, GeForce RTX 3090/3080/3070 (Ampere) 
  - **sm_89** : NVIDIA RTX 6000 Ada, GeForce RTX 4090/4080/4070 (Ada) 
  - **sm_90** : NVIDIA H100 / H200 / GH200 (Hopper) 
  - **sm_100** : NVIDIA B200 / GB200 (Blackwell, data center) 
  - **sm_120** : GeForce RTX 5090/5080/5070(Ti)/5060(Ti)/5050、RTX PRO 6000/5000/4500/4000/2000 Blackwell (workstation)  
  - **Note on verification** : Only a subset of the architectures above has been verified on real hardware.  
###  **Numeric types**
  - 32-bit and 64-bit term coefficients
  - 64-bit energy values

### **Performance note**
  - Arithmetic overflow checks are omitted to maximize performance.


## Build Environment
The following environment was used to build QUBO++.  
To ensure compatibility, please use the same or newer versions of the listed components.
- **Operating System**: Ubuntu 20.04.6 LTS 
- **glibc**: 2.31  
- **C++ Standard**: C++17  
- **Compiler**: g++ 9.4.0  
- **Boost**: 1.81.0
- **CUDA**: 12.8

## oneTBB / TBB dependency
QUBO++ does not bundle TBB. We include TBB headers in public APIs, but the
library itself does not require linking against TBB for the default use cases.
- **Build & run verified on**: Ubuntu 20.04 (classic TBB 2020.1), 22.04 / 24.04 (oneTBB 2021+).

## Directory Structure and Key Files
- `bin/` - Executable
  - `qbpp-license`: QUBO++ License manager

- `include/` — Header files  
  - `qbpp.hpp`: Main QUBO++ BQ Edition toolkit 
  - `qbpp_abs3_solver.hpp`: ABS3 GPU Solver 
  - `qbpp_defs.hpp`: Common definitions and macros  
  - `qbpp_easy_solver.hpp`: Easy Solver implementation  
  - `qbpp_exhaustive_solver.hpp`: Exhaustive Solver implementation  
  - `qbpp_grb.hpp`: Gurobi Optimizer interface
  - `qbpp_misc.hpp`: Miscelleneous library

- `lib/` — QUBO++ shared libraries  
  - `libqbpp.so`: Shared library for the QUBO++ core  
  - `libqbpp_abs3c32.so`: Shared library for the ABS3 GPU solver with 32-bit term coefficients  
  - `libqbpp_abs3c64.so`: Shared library for the ABS3 GPU solver with 64-bit term coefficients

- `samples/` — Sample programs
  - `factorization.cpp`: Integer factorization
  - `graph_color.cpp`: Graph coloring
  - `tsp.cpp`: Solving Traveling Salesman Problem using Easy Solver
  - `qbpp_tsp.hpp`: Creating QUBO model for the TSP
  - `integer_poly.cpp`: Integer polynomial optimization  
  - `labs.cpp`: Low Autocorrelation Binary Sequences problem  
  - `nqueen_easy.cpp`: Solving N-Queens problem using Easy Solver 
  - `qbpp_nqueen.hpp`: Generates QUBO expression for the N-Queens problem
  - `shift_scheduling`: Shift scheduling problem
  - `factorization_abs3.cpp`: Integer factorization using ABS3
  - `labs_abs3.cpp`: Low Autocorrelation Binary Sequences problem using ABS3 GPU solver
  - `ilp_grb.cpp`: Integer Linear Programming example using Gurobi Optimizer

# QUBO++ Licensing

## License Types
QUBO++ uses Cryptlex for license management.
When activating a node-locked license, some information about the execution environment (e.g., hardware identifiers) is securely stored on Cryptlex servers to verify the license.

- **Anonymous Trial**  
  - No license key required  
  - Valid for 7 days  
  - Maximum variable count: **1,000 (CPU) / 1,000 (GPU)**  

- **Registered Trial**  
  - Requires a free license key  
  - Valid for 30 days  
  - Maximum variable count: **10,000 (CPU) / 10,000 (GPU)**  

- **Standard License**  
  - Requires a paid license key  
  - Valid for the duration of the license agreement  
  - Maximum variable count: **2,147,483,647 ($2^{31} - 1$) (CPU) / 1,000 (GPU)**  
  - *Practically, the maximum variable count may be limited by available physical memory.*  

- **Professional License**  
  - Requires a paid license key  
  - Valid for the duration of the license agreement  
  - Maximum variable count: **2,147,483,647 ($2^{31} - 1$) (CPU) / 2,147,483,647 ($2^{31} - 1$) (GPU)**  
  - *Practically, the maximum variable count may be limited by available physical memory.*  

- **Fallback (No license / Expired license)**   
  - Maximum variable count: **100 (CPU) / 100 (GPU)**


## Display/Activate/Deactivate QUBO++ License
- The `qbpp-license` utility (in the `/bin` directory of the QUBO++ installation) displays, activates, and deactivates the license on the current machine.
- A QUBO++ license must be activated on the machine before QUBO++ programs can run.
- Each license key has limits on how many times it can be activated and deactivated. These limits are tracked per license.

### Display License
If the license has already been activated on this machine, running `qbpp-license` will show the license information (expiration date/time, total activations, total deactivations, and so on).
- **Registered Trial**/**Standard License**/**Professional License**
You can specify the license key either by environment variable or by command-line option.
  - Using environment variable:
  ```bash
  $ export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX
  $ qbpp-license
  ```
  - Using command-line option:
  ```bash
  $ qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX
  ```
- **Anonymous Trial**  
  Just run the command without specifying a license key:
  ```bash
  $ qbpp-license
  ```

### Activate License
- To activate the license on the current machine, add the `-a` (or `--activate`) option to the above command:
  ```bash
  $ qbpp-license -a
  ```
- This consumes one activation from the license.

### Deactivate License
- To deactivate the license on the current machine, use the `-d` (or `--deactivate`) option:
  ```bash
  $ qbpp-license -d
  ```
- Anonymous Trial cannot be deactivated.
- Deactivation will increase the deactivation count and free up an activation slot so that the license can be activated again on this or another machine.

## Executing QUBO++ with license key.
- For **Anonymous Trial**, there is no license key to set.
- For **Registered Trial**, **Standard**, and **Professional** licenses, specify the key in one of the following ways before running your program:
  - **Via Code**:
  ```cpp
  qbpp::license_key("XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX");
  ```
  - **Via Environment Variable**:
  ```bash
  export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX
  ```

# Terms and Conditions

By using QUBO++, you agree to the following terms and conditions:

1. **License Scope**  
   QUBO++ is provided under a time-limited license, depending on the license type (Anonymous Trial, Registered Trial, Standard or Professional License).

2. **Restrictions on Trial Licenses**  
   The Anonymous Trial and Registered Trial licenses are provided strictly for evaluation purposes only.  

3. **Standard/Professional Licenses and Payment Requirement**  
   Standard/Professional Licenses require the purchase of a paid license key. These licenses grant full access to QUBO++ for commercial and research use, subject to the terms of the license agreement.

4. **Redistribution**  
   Redistribution of QUBO++ (in whole or in part, including headers and binaries) is prohibited without prior written permission from the copyright holder.

5. **Reverse Engineering**  
   You may not reverse engineer, decompile, or disassemble any part of the software.

6. **Disclaimer**  
   QUBO++ is provided "as is," without any warranties, express or implied. The authors shall not be held liable for any damages resulting from the use of this software.

7. **Ownership**  
   All intellectual property rights related to QUBO++ remain the property of the original author or copyright holder.

8. **Termination**  
   This license will automatically terminate if you fail to comply with these terms. Upon termination, you must cease all use of the software and destroy all copies.

# Third-Party Libraries

- **Gurobi Optimizer**  
  - QUBO++ supporting APIs for integration with the Gurobi Optimizer.  
  - To use these APIs, you must obtain a valid license for the Gurobi Optimizer from Gurobi Optimization, LLC.  
  - The Gurobi Optimizer is not included with QUBO++, and its use is subject to the terms and conditions of its own license agreement.

- **oneTBB (oneAPI Threading Building Blocks)**  
  - Licensed under the Apache License 2.0.  
  - Copyright © Intel Corporation.  
  - See <https://www.apache.org/licenses/LICENSE-2.0> for details.  
  - *Note:* If you redistribute binaries that include oneTBB, include the Apache 2.0 license text and any required NOTICE.

- **Boost C++ Libraries**  
  - Licensed under the Boost Software License, Version 1.0.  
  - See <https://www.boost.org/LICENSE_1_0.txt> for details.

- **xxHash**  
  - Licensed under the BSD 2-Clause License.  
  - Copyright © Yann Collet.  
  - See <https://opensource.org/license/bsd-2-clause/> for details.


If you have any questions regarding licensing or wish to obtain a license key, please contact the distributor or the developer.

