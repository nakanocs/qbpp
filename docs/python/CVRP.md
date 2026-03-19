---
layout: default
nav_exclude: true
title: "CVRP (PyQBPP)"
nav_order: 61
---
<div class="lang-en" markdown="1">
# Capacitated Vehicle Routing Problem (CVRP)
The **Capacitated Vehicle Routing Problem (CVRP)** aims to find a set of routes for $V$ **vehicles** that start and end at a single **depot** and collectively serve all **customers**.
We index the **locations** by $i \in \lbrace 0,1,\ldots,N-1\rbrace$, where location 0 denotes the depot and locations $1,\ldots,N-1$ are customers.
Each customer $i\in \lbrace 1,\ldots,N-1\rbrace$ has a demand $d_i$ to be delivered (and we set $d_0=0$ for the depot).
Each vehicle $v \in \lbrace 0,\ldots,V-1\rbrace$ departs from the depot, visits a subset of customers, and returns to the depot, subject to the capacity constraint that the total delivered demand on its route does not exceed the vehicle capacity $q_v$.
The objective is to minimize the total travel cost over all vehicles.

## QUBO formulation
We use a $V\times N\times N$ array $A=(a_{v,t,i})$ of binary variables,
where $a_{v,t,i}$ is 1 if and only if the $t$-th visited location of vehicle $v$ is location $i$.

### Constraints
- **Row constraint**: Each vehicle must be at exactly one location at each time $t$.
- **Column constraint**: Each customer must be visited exactly once.
- **Consecutive-depot constraint**: Once a vehicle returns to the depot, it stays there.
- **Capacity constraint**: Total demand on each route must not exceed vehicle capacity.

### Objective
The total tour cost is computed from consecutive visited locations:

$$
\begin{aligned}
\text{objective} &= \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\sum_{i=0}^{N-1}\sum_{j=0}^{N-1}c_{i,j}a_{v,t,i}a_{v,(t+1)\bmod N, j}
\end{aligned}
$$

## PyQBPP program
```python
import math
import pyqbpp as qbpp

locations = [
    (200, 200, 0),  (247, 296, 44), (31, 393, 57), (96, 398, 94),
    (391, 230, 91), (118, 95, 66),  (197, 99, 59), (224, 8, 10),
    (3, 10, 52),    (281, 379, 83)]
vehicle_capacity = [100, 200, 300]

N = len(locations)
V = len(vehicle_capacity)

a = qbpp.var("a", V, N, N)

row_constraint = qbpp.sum(qbpp.vector_sum(a) == 1)

column_sum = [toExpr(0) for _ in range(N - 1)]
for v in range(V):
    for t in range(N):
        for i in range(1, N):
            column_sum[i - 1] += a[v][t][i]
column_constraint = toExpr(0)
for i in range(N - 1):
    column_constraint += column_sum[i] == 1

consecutive_constraint = toExpr(0)
for v in range(V):
    for t in range(1, N - 1):
        consecutive_constraint += a[v][t][0] * (1 - a[v][t + 1][0])

vehicle_load = [toExpr(0) for _ in range(V)]
capacity_constraint = toExpr(0)
for v in range(V):
    for t in range(N):
        for i in range(1, N):
            vehicle_load[v] += a[v][t][i] * locations[i][2]
    capacity_constraint += qbpp.between(vehicle_load[v], 0, vehicle_capacity[v])

objective = toExpr(0)
for v in range(V):
    for t in range(N):
        next_t = (t + 1) % N
        for i in range(N):
            x1, y1 = locations[i][0], locations[i][1]
            for j in range(N):
                x2, y2 = locations[j][0], locations[j][1]
                dist = int(math.sqrt((x1 - x2) ** 2 + (y1 - y2) ** 2))
                objective += dist * a[v][t][i] * a[v][next_t][j]

f = objective + 10000 * (row_constraint + column_constraint +
                          consecutive_constraint + capacity_constraint)

ml = [(a[v][0][0], 1) for v in range(V)]
ml += [(a[v][0][i], 0) for v in range(V) for i in range(1, N)]

g = qbpp.replace(f, ml)
f.simplify_as_binary()
g.simplify_as_binary()
solver = qbpp.EasySolver(g)

sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)

print(f"row_constraint = {full_sol(row_constraint)}")
print(f"column_constraint = {full_sol(column_constraint)}")
print(f"consecutive_constraint = {full_sol(consecutive_constraint)}")
print(f"capacity_constraint = {full_sol(capacity_constraint)}")
print(f"objective = {full_sol(objective)}")

for v in range(V):
    load = full_sol(vehicle_load[v])
    route = f"Vehicle {v} : load = {load} / {vehicle_capacity[v]} : 0 "
    for t in range(1, N):
        for i in range(1, N):
            if full_sol(a[v][t][i]) == 1:
                route += f"-> {i}({locations[i][2]}) "
                break
    route += "-> 0"
    print(route)
```
This program defines a $V\times N\times N$ array `a` of binary variables.
It then defines the objective and constraint terms according to the formulation described above.

To fix the first visited location ($t=0$) of every vehicle to the depot (location 0), the program constructs a list of pairs and applies it using `replace()`.

For example, the program prints the following results:
```
row_constraint = 0
column_constraint = 0
consecutive_constraint = 0
capacity_constraint = 0
objective = 2142
Vehicle 0 : load = 91 / 100 : 0 -> 4(91) -> 0
Vehicle 1 : load = 177 / 200 : 0 -> 6(59) -> 5(66) -> 8(52) -> 0
Vehicle 2 : load = 288 / 300 : 0 -> 7(10) -> 9(83) -> 1(44) -> 3(94) -> 2(57) -> 0
```

## Visualization using matplotlib
The following code visualizes the CVRP solution:
```python
import matplotlib.pyplot as plt

vehicle_colors = ["#e74c3c", "#3498db", "#2ecc71"]
plt.figure(figsize=(6, 6))
for i, (lx, ly, q) in enumerate(locations):
    plt.plot(lx, ly, "ko", markersize=8)
    plt.annotate(f"{i}" + (f"({q})" if q > 0 else ""),
                 (lx, ly), textcoords="offset points", xytext=(5, 5))

for v in range(V):
    route_nodes = [0]
    for t in range(1, N):
        for i in range(1, N):
            if full_sol(a[v][t][i]) == 1:
                route_nodes.append(i)
                break
    route_nodes.append(0)
    for k in range(len(route_nodes) - 1):
        fr, to = route_nodes[k], route_nodes[k + 1]
        plt.annotate("", xy=(locations[to][0], locations[to][1]),
                     xytext=(locations[fr][0], locations[fr][1]),
                     arrowprops=dict(arrowstyle="->", color=vehicle_colors[v],
                                    lw=2))
plt.title("CVRP Solution")
plt.savefig("cvrp.png", dpi=150, bbox_inches="tight")
plt.show()
```

Each vehicle's route is shown in a different color, with arrows indicating the direction of travel.
</div>

<div class="lang-ja" markdown="1">
# 容量制約付き配車計画問題（CVRP）
**容量制約付き配車計画問題（CVRP）**は、1つの**デポ**を出発・帰着点とし、すべての**顧客**を訪問する $V$ 台の**車両**の経路集合を求める問題です。
**地点**を $i \in \lbrace 0,1,\ldots,N-1\rbrace$ でインデックス付けし、地点0をデポ、地点 $1,\ldots,N-1$ を顧客とします。
各顧客 $i\in \lbrace 1,\ldots,N-1\rbrace$ には配送すべき需要量 $d_i$ があります（デポについては $d_0=0$ とします）。
各車両 $v \in \lbrace 0,\ldots,V-1\rbrace$ はデポを出発し、顧客の部分集合を訪問してデポに戻ります。このとき、経路上の配送需要の合計が車両容量 $q_v$ を超えないという容量制約を満たす必要があります。
目的は、すべての車両の総移動コストを最小化することです。

## QUBO定式化
$V\times N\times N$ のバイナリ変数の配列 $A=(a_{v,t,i})$ を使用します。
$a_{v,t,i}$ は車両 $v$ の $t$ 番目の訪問地点が地点 $i$ である場合にのみ1となります。

### 制約
- **行制約**: 各車両は各時刻 $t$ においてちょうど1つの地点にいなければならない。
- **列制約**: 各顧客はちょうど1回訪問されなければならない。
- **連続デポ制約**: 車両がデポに戻ったら、そこに留まらなければならない。
- **容量制約**: 各経路上の総需要量が車両容量を超えてはならない。

### 目的関数
総巡回コストは連続する訪問地点から計算されます：

$$
\begin{aligned}
\text{objective} &= \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\sum_{i=0}^{N-1}\sum_{j=0}^{N-1}c_{i,j}a_{v,t,i}a_{v,(t+1)\bmod N, j}
\end{aligned}
$$

## PyQBPPプログラム
```python
import math
import pyqbpp as qbpp

locations = [
    (200, 200, 0),  (247, 296, 44), (31, 393, 57), (96, 398, 94),
    (391, 230, 91), (118, 95, 66),  (197, 99, 59), (224, 8, 10),
    (3, 10, 52),    (281, 379, 83)]
vehicle_capacity = [100, 200, 300]

N = len(locations)
V = len(vehicle_capacity)

a = qbpp.var("a", V, N, N)

row_constraint = qbpp.sum(qbpp.vector_sum(a) == 1)

column_sum = [toExpr(0) for _ in range(N - 1)]
for v in range(V):
    for t in range(N):
        for i in range(1, N):
            column_sum[i - 1] += a[v][t][i]
column_constraint = toExpr(0)
for i in range(N - 1):
    column_constraint += column_sum[i] == 1

consecutive_constraint = toExpr(0)
for v in range(V):
    for t in range(1, N - 1):
        consecutive_constraint += a[v][t][0] * (1 - a[v][t + 1][0])

vehicle_load = [toExpr(0) for _ in range(V)]
capacity_constraint = toExpr(0)
for v in range(V):
    for t in range(N):
        for i in range(1, N):
            vehicle_load[v] += a[v][t][i] * locations[i][2]
    capacity_constraint += qbpp.between(vehicle_load[v], 0, vehicle_capacity[v])

objective = toExpr(0)
for v in range(V):
    for t in range(N):
        next_t = (t + 1) % N
        for i in range(N):
            x1, y1 = locations[i][0], locations[i][1]
            for j in range(N):
                x2, y2 = locations[j][0], locations[j][1]
                dist = int(math.sqrt((x1 - x2) ** 2 + (y1 - y2) ** 2))
                objective += dist * a[v][t][i] * a[v][next_t][j]

f = objective + 10000 * (row_constraint + column_constraint +
                          consecutive_constraint + capacity_constraint)

ml = [(a[v][0][0], 1) for v in range(V)]
ml += [(a[v][0][i], 0) for v in range(V) for i in range(1, N)]

g = qbpp.replace(f, ml)
f.simplify_as_binary()
g.simplify_as_binary()
solver = qbpp.EasySolver(g)

sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(sol)
full_sol.set(ml)

print(f"row_constraint = {full_sol(row_constraint)}")
print(f"column_constraint = {full_sol(column_constraint)}")
print(f"consecutive_constraint = {full_sol(consecutive_constraint)}")
print(f"capacity_constraint = {full_sol(capacity_constraint)}")
print(f"objective = {full_sol(objective)}")

for v in range(V):
    load = full_sol(vehicle_load[v])
    route = f"Vehicle {v} : load = {load} / {vehicle_capacity[v]} : 0 "
    for t in range(1, N):
        for i in range(1, N):
            if full_sol(a[v][t][i]) == 1:
                route += f"-> {i}({locations[i][2]}) "
                break
    route += "-> 0"
    print(route)
```
このプログラムは $V\times N\times N$ のバイナリ変数の配列 `a` を定義します。
次に、上記の定式化に従って目的関数と制約項を定義します。

すべての車両の最初の訪問地点（$t=0$）をデポ（地点0）に固定するため、ペアのリストを構築し `replace()` を使って適用します。

例えば、このプログラムは以下の結果を出力します：
```
row_constraint = 0
column_constraint = 0
consecutive_constraint = 0
capacity_constraint = 0
objective = 2142
Vehicle 0 : load = 91 / 100 : 0 -> 4(91) -> 0
Vehicle 1 : load = 177 / 200 : 0 -> 6(59) -> 5(66) -> 8(52) -> 0
Vehicle 2 : load = 288 / 300 : 0 -> 7(10) -> 9(83) -> 1(44) -> 3(94) -> 2(57) -> 0
```

## matplotlibによる可視化
以下のコードはCVRPの解を可視化します：
```python
import matplotlib.pyplot as plt

vehicle_colors = ["#e74c3c", "#3498db", "#2ecc71"]
plt.figure(figsize=(6, 6))
for i, (lx, ly, q) in enumerate(locations):
    plt.plot(lx, ly, "ko", markersize=8)
    plt.annotate(f"{i}" + (f"({q})" if q > 0 else ""),
                 (lx, ly), textcoords="offset points", xytext=(5, 5))

for v in range(V):
    route_nodes = [0]
    for t in range(1, N):
        for i in range(1, N):
            if full_sol(a[v][t][i]) == 1:
                route_nodes.append(i)
                break
    route_nodes.append(0)
    for k in range(len(route_nodes) - 1):
        fr, to = route_nodes[k], route_nodes[k + 1]
        plt.annotate("", xy=(locations[to][0], locations[to][1]),
                     xytext=(locations[fr][0], locations[fr][1]),
                     arrowprops=dict(arrowstyle="->", color=vehicle_colors[v],
                                    lw=2))
plt.title("CVRP Solution")
plt.savefig("cvrp.png", dpi=150, bbox_inches="tight")
plt.show()
```

各車両の経路は異なる色で表示され、矢印が移動方向を示しています。
</div>
