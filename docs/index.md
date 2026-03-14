---
layout: default
title: "Home"
nav_order: 1
---
<div class="lang-en" markdown="1">

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

* 組合せ最適化問題を解くためのバイナリ変数多項式を構築する**C++ライブラリ**です。
* 現在、**amd64** (**x86_64**) および **arm64** Linux システムで利用可能です。
* 2つのCPUソルバーを同梱: **Easy Solver**（ヒューリスティック）と **Exhaustive Solver**（完全探索）。
* GPUソルバー **ABS3** を同梱。Exhaustive Solver も CUDA GPU が利用可能な場合は GPU 加速をサポートします。
* 次数無制限の**高次制約なしバイナリ最適化（HUBO）**をサポート。
* 変数の否定（$\overline{x}$ または `~x`）を $1-x$ に置き換えることなくネイティブにサポートし、高次項における項数の爆発を回避します。
* oneTBB を用いた**マルチスレッド加速**を可能な限り適用。
* **Gurobi Optimizer** を呼び出して二次制約なしバイナリ最適化（QUBO）問題を解くための API を含みます。
* Copyright: © 2026 Koji Nakano. All rights reserved.

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
