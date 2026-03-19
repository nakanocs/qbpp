---
layout: default
nav_exclude: true
title: "Shift Scheduling (PyQBPP)"
nav_order: 71
---
<div class="lang-en" markdown="1">
# Shift Scheduling Problem
Consider the following **shift scheduling problem**, which aims to find a schedule that minimizes the total worker cost.

- There are 6 workers and a planning horizon of 31 days, from day 1 to day 31.
For simplicity, we assume that all workers are off on day 0 and day 32.
- Exactly 4 workers must be scheduled on each day from day 1 to day 31.
- The following constraints must be satisfied for each worker:
  - works for either 20 or 21 days,
  - works no more than 6 consecutive days,
  - works no fewer than 3 consecutive days,
  - has no isolated day off; days off must be consecutive.

## PyQBPP program for the shift scheduling
```python
import pyqbpp as qbpp

days = 31
worker_cost = [13, 13, 12, 12, 11, 10]
workers = len(worker_cost)

x = qbpp.var("x", workers, days + 2)

workers_each_day = qbpp.vector_sum(x, 0)
each_day_4_workers = toExpr(0)
for j in range(1, days + 1):
    each_day_4_workers += workers_each_day[j] == 4

workers_working_days = qbpp.vector_sum(x)
work_20_21_days = toExpr(0)
for i in range(workers):
    work_20_21_days += qbpp.between(workers_working_days[i], 20, 21)

no_more_than_6 = toExpr(0)
for w in range(workers):
    for j in range(days - 5):
        no_more_than_6 += (x[w][j] * x[w][j+1] * x[w][j+2] *
                           x[w][j+3] * x[w][j+4] * x[w][j+5] * x[w][j+6])

no_less_than_3 = toExpr(0)
for w in range(workers):
    for j in range(days - 1):
        no_less_than_3 += (1 - x[w][j]) * x[w][j+1] * x[w][j+2] * (1 - x[w][j+3])
    for j in range(days):
        no_less_than_3 += (1 - x[w][j]) * x[w][j+1] * (1 - x[w][j+2])

no_single_day_off = toExpr(0)
for w in range(workers):
    for j in range(days):
        no_single_day_off += x[w][j] * (1 - x[w][j+1]) * x[w][j+2]

total_worker_cost = toExpr(0)
for i in range(workers):
    total_worker_cost += worker_cost[i] * workers_working_days[i]

constraints = (work_20_21_days + no_less_than_3 + no_more_than_6 +
               no_single_day_off + each_day_4_workers)
f = total_worker_cost + 10000 * constraints

ml = [(x[i][0], 0) for i in range(workers)]
ml += [(x[i][days + 1], 0) for i in range(workers)]
f.simplify_as_binary()

g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.EasySolver(g)
solver.time_limit(5.0)
solver.target_energy(0)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(ml)
full_sol.set(sol)

for i in range(workers):
    wd = full_sol(workers_working_days[i])
    bits = "".join(str(full_sol(x[i][j])) for j in range(1, days + 1))
    print(f"Worker {i}: {wd} days worked: {bits}")

day_counts = "".join(str(full_sol(workers_each_day[d])) for d in range(1, days + 1))
print(f"Workers each day        : {day_counts}")
print(f"Total worker cost: {full_sol(total_worker_cost)}")
print(f"Constraints violations: {full_sol(constraints)}")
```
The obtained solution is as follows:
```
Worker 0: 20 days worked: 0001111001110011111001111001111
Worker 1: 20 days worked: 1111001111110001111110011110000
Worker 2: 21 days worked: 0000111100111110011111100111111
Worker 3: 21 days worked: 1111110011111100111000111111000
Worker 4: 21 days worked: 1111100111001111000111000111111
Worker 5: 21 days worked: 1110011110001111100111111000111
Workers each day        : 4444444444444444444444444444444
Total worker cost: 1465
Constraints violations: 0
```
</div>

<div class="lang-ja" markdown="1">
# シフトスケジューリング問題
以下の**シフトスケジューリング問題**を考えます。この問題は、総労働者コストを最小化するスケジュールを求めることを目的とします。

- 6人の労働者がおり、計画期間は1日目から31日目までの31日間です。
簡単のため、すべての労働者は0日目と32日目は休みとします。
- 1日目から31日目までの各日に、ちょうど4人の労働者を配置する必要があります。
- 各労働者について以下の制約を満たす必要があります：
  - 20日または21日勤務する、
  - 連続勤務日数は6日以下、
  - 連続勤務日数は3日以上、
  - 孤立した休日がない（休日は連続でなければならない）。

## シフトスケジューリングのPyQBPPプログラム
```python
import pyqbpp as qbpp

days = 31
worker_cost = [13, 13, 12, 12, 11, 10]
workers = len(worker_cost)

x = qbpp.var("x", workers, days + 2)

workers_each_day = qbpp.vector_sum(x, 0)
each_day_4_workers = toExpr(0)
for j in range(1, days + 1):
    each_day_4_workers += workers_each_day[j] == 4

workers_working_days = qbpp.vector_sum(x)
work_20_21_days = toExpr(0)
for i in range(workers):
    work_20_21_days += qbpp.between(workers_working_days[i], 20, 21)

no_more_than_6 = toExpr(0)
for w in range(workers):
    for j in range(days - 5):
        no_more_than_6 += (x[w][j] * x[w][j+1] * x[w][j+2] *
                           x[w][j+3] * x[w][j+4] * x[w][j+5] * x[w][j+6])

no_less_than_3 = toExpr(0)
for w in range(workers):
    for j in range(days - 1):
        no_less_than_3 += (1 - x[w][j]) * x[w][j+1] * x[w][j+2] * (1 - x[w][j+3])
    for j in range(days):
        no_less_than_3 += (1 - x[w][j]) * x[w][j+1] * (1 - x[w][j+2])

no_single_day_off = toExpr(0)
for w in range(workers):
    for j in range(days):
        no_single_day_off += x[w][j] * (1 - x[w][j+1]) * x[w][j+2]

total_worker_cost = toExpr(0)
for i in range(workers):
    total_worker_cost += worker_cost[i] * workers_working_days[i]

constraints = (work_20_21_days + no_less_than_3 + no_more_than_6 +
               no_single_day_off + each_day_4_workers)
f = total_worker_cost + 10000 * constraints

ml = [(x[i][0], 0) for i in range(workers)]
ml += [(x[i][days + 1], 0) for i in range(workers)]
f.simplify_as_binary()

g = qbpp.replace(f, ml)
g.simplify_as_binary()

solver = qbpp.EasySolver(g)
solver.time_limit(5.0)
solver.target_energy(0)
sol = solver.search()

full_sol = qbpp.Sol(f)
full_sol.set(ml)
full_sol.set(sol)

for i in range(workers):
    wd = full_sol(workers_working_days[i])
    bits = "".join(str(full_sol(x[i][j])) for j in range(1, days + 1))
    print(f"Worker {i}: {wd} days worked: {bits}")

day_counts = "".join(str(full_sol(workers_each_day[d])) for d in range(1, days + 1))
print(f"Workers each day        : {day_counts}")
print(f"Total worker cost: {full_sol(total_worker_cost)}")
print(f"Constraints violations: {full_sol(constraints)}")
```
得られた解は以下の通りです：
```
Worker 0: 20 days worked: 0001111001110011111001111001111
Worker 1: 20 days worked: 1111001111110001111110011110000
Worker 2: 21 days worked: 0000111100111110011111100111111
Worker 3: 21 days worked: 1111110011111100111000111111000
Worker 4: 21 days worked: 1111100111001111000111000111111
Worker 5: 21 days worked: 1110011110001111100111111000111
Workers each day        : 4444444444444444444444444444444
Total worker cost: 1465
Constraints violations: 0
```
</div>
