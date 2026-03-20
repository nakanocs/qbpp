---
layout: default
title: "Home"
nav_order: 1
---
<div class="lang-en" markdown="1">

# QUBO++: A model-and-solve framework for combinatorial optimization via QUBO/HUBO

**QUBO++** is a framework for formulating and solving combinatorial optimization problems
as polynomials of binary variables (QUBO/HUBO).

- **C++ and Python** — Use QUBO++ from C++ ([QUBO++](DOCUMENT)) or Python ([PyQBPP](python/)). `pip install pyqbpp` to get started with Python.
- **Symbolic DSL** — Write optimization models as mathematical expressions, not matrix indices. Use natural for-loops to build constraints, or leverage vector operations for loop-free formulations.
- **Easy installation** — `sudo apt install qbpp` for C++, `pip install pyqbpp` for Python. No build from source required.
- **Unlimited-degree HUBO** — Supports high-order terms of any degree, not just quadratic. Native support for negated literals (`~x`) avoids the term explosion caused by replacing $\overline{x}$ with $1-x$.
- **GPU-accelerated solving** — The built-in ABS3 solver fully utilizes GPU resources for parallel search, with multi-GPU scaling. The Exhaustive Solver also automatically uses CUDA GPUs when available.
- **CPU parallel acceleration** — All solvers and expression-building operations are multithreaded via oneTBB.
- **Arbitrary-precision integer coefficients** — Handles integer coefficients of unlimited bit width via Boost.Multiprecision. No overflow worries, from 16-bit to thousands of digits.
- **Three built-in solvers** — Easy Solver (fast heuristic), Exhaustive Solver (complete search with optimality guarantee), and ABS3 (GPU+CPU heuristic).
- **Run anywhere** — From a laptop to GPU servers and supercomputers. Available for amd64 (x86_64) and arm64 Linux.

Copyright: © 2026 Koji Nakano. All rights reserved.

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

QUBO++ can be used without a license key.
If no license key is set, an **Anonymous Trial** (7 days, 1,000 variables) is automatically activated, allowing you to try QUBO++ immediately.

For details on license activation, license types, and terms, see **[License Management](LICENSE_MANAGEMENT)**.

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

</div>

<div class="lang-ja" markdown="1">

# QUBO++: QUBO/HUBOによる組合せ最適化のためのモデリング・求解フレームワーク

**QUBO++** は、組合せ最適化問題をバイナリ変数の多項式（QUBO/HUBO）として定式化・求解するフレームワークです。

- **C++ と Python** — C++（[QUBO++](DOCUMENT)）でもPython（[PyQBPP](python/)）でも使えます。`pip install pyqbpp` ですぐにPythonで始められます。
- **シンボリックDSL** — 行列のインデックスではなく、数式を書くように最適化モデルを構築。自然なforループで制約を記述することも、ベクトル演算でループなしに記述することもできます。
- **簡単インストール** — C++は `sudo apt install qbpp`、Pythonは `pip install pyqbpp`。ソースからのビルドは不要です。
- **次数無制限のHUBO** — 二次だけでなく任意の次数の高次項をサポート。否定リテラル（`~x`）をネイティブにサポートし、$\overline{x}$ を $1-x$ に置き換えることによる項数爆発を回避します。
- **GPU加速** — 内蔵のABS3ソルバーがGPUリソースをフル活用して並列探索を実行。マルチGPUにも対応。Exhaustive SolverもCUDA GPUを自動的に使用します。
- **CPU並列加速** — すべてのソルバーおよび式構築操作がoneTBBによるマルチスレッドで高速化されています。
- **任意精度の整数係数** — Boost.Multiprecisionにより、ビット数に上限のない整数係数を扱えます。16ビットから数千桁まで、オーバーフローの心配なく計算可能。
- **3つの内蔵ソルバー** — Easy Solver（高速ヒューリスティック）、Exhaustive Solver（最適性保証付き完全探索）、ABS3（GPU+CPUヒューリスティック）。
- **どこでも実行** — ノートPCからGPUサーバー、スーパーコンピュータまで。amd64 (x86_64) および arm64 Linux で利用可能。

Copyright: © 2026 Koji Nakano. All rights reserved.

# QUBO++ ソルバー: Easy Solver, Exhaustive Solver, ABS3 Solver
## Easy Solver
* **QUBO/HUBO に最適化されたヒューリスティックソルバー**: マルチコア CPU 上で QUBO/HUBO モデルの解を探索します。
* **マルチスレッド加速**: Intel oneTBB による並列探索。
* **任意精度の整数係数**: 任意の大きさの整数係数をサポート。

## Exhaustive Solver
* マルチコア CPU と CUDA GPU 上で QUBO/HUBO 定式化の**全解を列挙**します。
* **最適性保証**: 大域最適解が発見・証明されます。
* **マルチスレッド加速**: Intel oneTBB による並列探索。
* **任意精度の整数係数**: 任意の大きさの整数係数をサポート。
* **GPU 加速**: CUDA GPU が利用可能な場合、GPU ワーカーが CPU スレッドと並行して探索に参加します。128ビット整数までの係数で GPU 加速が利用可能で、それ以上の係数は CPU のみで実行されます。

## ABS3 Solver
* **マルチコア CPU と CUDA GPU 上のヒューリスティックソルバー**: CPU スレッドと CUDA GPU の両方を使用して QUBO/HUBO インスタンスの解を探索します。
* **任意精度の整数係数**: 任意の大きさの整数係数をサポート。
* **マルチ GPU スケーリング**: Linux ホスト上の検出されたすべての GPU を使用。128ビット整数までの係数で GPU 加速が利用可能で、それ以上の係数は CPU のみで実行されます。

### **ABS3 対応 GPU アーキテクチャ**
  - **sm_80** : NVIDIA A100 (Ampere)
  - **sm_86** : NVIDIA RTX A6000, GeForce RTX 3090/3080/3070 (Ampere)
  - **sm_89** : NVIDIA RTX 6000 Ada, GeForce RTX 4090/4080/4070 (Ada)
  - **sm_90** : NVIDIA H100 / H200 / GH200 (Hopper)
  - **sm_100** : NVIDIA B200 / GB200 (Blackwell, データセンター)
  - **sm_120** : GeForce RTX 5090/5080/5070(Ti)/5060(Ti)/5050、RTX PRO 6000/5000/4500/4000/2000 Blackwell (ワークステーション)
  - **検証について**: 上記アーキテクチャの一部のみが実機で検証済みです。

### **性能に関する注意**
  - パフォーマンスを最大化するため、算術オーバーフローチェックは省略されています。

## ビルド環境
以下の環境を使用して QUBO++ をビルドしています。
**QUBO++ は Ubuntu 20.04 に限定されません**。Ubuntu 22.04/24.04 およびその他の Linux ディストリビューション（CentOS/RHEL系を含む）でもテスト済みです。
互換性を確保するため、以下のコンポーネントと同じかそれ以降のバージョンをお使いください。
- **オペレーティングシステム**: Ubuntu 20.04.6 LTS
- **C++ 標準**: C++17
- **glibc**: 2.31
- **コンパイラ**: g++ 9.4.0
- **Boost**: 1.81.0
- **CUDA**: 12.8

## oneTBB / TBB 依存関係
QUBO++ は TBB をバンドルしていません。パブリック API に TBB ヘッダを含みますが、デフォルトの使用ではライブラリ自体が TBB のリンクを必要としません。
- **ビルド・実行確認済み**: Ubuntu 20.04 (classic TBB 2020.1), 22.04 / 24.04 (oneTBB 2021+)。

# QUBO++ ライセンス

QUBO++ はライセンスキーなしで使用できます。
ライセンスキーが設定されていない場合、**Anonymous Trial**（7日間、1,000変数）が自動的に有効になり、すぐに QUBO++ を試すことができます。

ライセンスの有効化、ライセンスの種類、条件の詳細は **[License Management](LICENSE_MANAGEMENT)** をご覧ください。

# サードパーティライブラリ

- **oneTBB (oneAPI Threading Building Blocks)**
  - Apache License 2.0 の下でライセンスされています。
  - Copyright © Intel Corporation.
  - 詳細は <https://www.apache.org/licenses/LICENSE-2.0> をご覧ください。

- **Boost C++ Libraries**
  - Boost Software License, Version 1.0 の下でライセンスされています。
  - 詳細は <https://www.boost.org/LICENSE_1_0.txt> をご覧ください。

- **xxHash**
  - BSD 2-Clause License の下でライセンスされています。
  - Copyright © Yann Collet.
  - 詳細は <https://opensource.org/license/bsd-2-clause/> をご覧ください。

- **Gurobi Optimizer**
  - QUBO++ は Gurobi Optimizer と統合するための API を提供しています。
  - これらの API を使用するには、Gurobi Optimization, LLC から有効なライセンスを取得する必要があります。
  - Gurobi Optimizer は QUBO++ に含まれておらず、その使用は独自のライセンス契約の条件に従います。

</div>
