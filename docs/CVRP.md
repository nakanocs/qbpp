---
layout: default
title: "CVRP"
---

# Capacitated Vehicle Routing Problem (CVRP)
The **Capacitated Vehicle Routing Problem (CVRP)** aims to find a set of routes for $V$ identical vehicles that start and end at a single depot and collectively serve all customers.  
We index the locations by $i \in \lbrace 0,1,\ldots,N-1\rbrace$, where location 0 denotes the depot and locations $1,\ldots,N-1$ are customers.  
Each customer $i\in \lbrace 1,\ldots,N-1\rbrace$ has a demand $d_i$ to be delivered (and we set $d_0=0$ for the depot).  
Each vehicle $v \in \lbrace 0,\ldots,V-1\rbrace$ departs from the depot, visits a subset of customers, and returns to the depot, subject to the capacity constraint that the total delivered demand on its route does not exceed the vehicle capacity $q_v$.  
The objective is to minimize the total travel cost over all vehicles.

Here, we assume that the locations are in the two-dimensional plane and that the travel cost between two locations is given by the Euclidean distance,
and let $d_{i,j}$ denote the distance of localtion $i$ and location $j$.

We use a $V\times N\times N$ array $a=(a_{v,t,i})$ of binary variables to formulate the CVRP as a QUBO expression,
where $a_{v,t,i}$ is 1 if and only if the $t$-th visiting location of vehicle $v$ is location $i$.

Since each vehicle visit exactly one location at each time, the following constraint must be satisfied:

$$
\begin{aligned}
\text{row\_constraint} & = \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\bigr(\sum_{i=0}^{N-1} a_{v,t,i} = 1\bigl)\\
 &= \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\bigr(1-\sum_{i=0}^{N-1} a_{v,t,i}\bigl)^2
\end{aligned}
$$

\text{row\_constraint} attains a minimum value of 0 if all veicles visit exactly one location at each time.

Note that the 0-th visiting location is location 0 where the depot is placed:

$$
\begin{aligned}
a_{v,0,0}& = 1 && (0\leq v\leq V-1) \\
a_{v,0,i}& = 0 && (0\leq v\leq V-1, 1\leq i\leq N-1)
\end{aligned}
$$

It makes no sense for vehicles to go back to the depot and visit customers again.
Thus, after going back to the depot, the vehicle stay at the depot.
Therefore, if a vehicve $v$ visit $k$ customers, the value of $x$ satisfies the following consition:

$$
\begin{aligned}
a_{v,t,0}& = 0 && (1\leq t\leq k) \\
a_{v,t,0}& = 1 && (k+1\leq t\leq N-1)
\end{aligned}
$$

In other words, we have the following constraint:

$$
\begin{aligned}
\text{consecutive\_constraint} &= \sum_{v=0}^{V-1}\sum_{t=1}^{N-2} (1-a_{v,t})a_{v,t+1}
\end{aligned}
$$

\text{consecutive\_constraint} attains a minimum value of 0 if and only if
the sequence
$$
a_{v,1,0}, a_{v,2,0}, \ldots, a_{v,N-1,0}
$$
have consecutive 0's followed by consecutive 1's.

Each customer must be visited by exactly one of the vehicles.
Thus the following constraint must be satisfied:

$$
\begin{aligned}
\text{column\_constraint}
& = \sum_{i=1}^{N-1}\bigr(\sum_{v=0}^{V-1}\sum_{t=0}^{N-1} a_{v,t,i} = 1\bigl)\\
 &= \sum_{i=1}^{N-1}\bigr(1-\sum_{v=0}^{V-1}\sum_{t=0}^{N-1} a_{v,t,i}\bigl)^2
\end{aligned}
$$

\text{column\_constraint} attains a minimum value of 0 if all customers are visited by exactly one vehicle.

The sum of demands for each veihicle $v$ should not exceed capacity $q_v$:
Thus the following constraints must be satisfied:

$$
\begin{aligned}
\text{vehicle\_constraint} &= \sum_{v=0}^{V-1}\Bigr(0\leq \sum_{t=1}^{N-1}\sum_{i=1}^{N-1}d_ix_{t,i}\leq q_v\Bigl)
\end{aligned}
$$

\text{vehicle\_constraint} attains a minimum value of 0 if all vehicles do not exceed their capacity.

Finally, we show how the total tour const is computed:

$$
\begin{aligned}
\text{objective} &= \sum_{v=0}^{V-1}\sum_{t=0}^{N-1}\sum_{i=0}^{N-1}\sum_{j=0}^{N-1}d_{i,j}x_{v,t,i}x_{v,(t+1)\bmod N, j}
\end{aligned}
$$

All constraints and the objective combined, we have the QUBO formulation $f$:

$$
\begin{aligned}
f &= \text{objective} + P\cdot (\text{row\_constraint}+\text{consecutive\_constraint}+\text{column\_constraint}+\text{vehicle\_constraint}),
\end{aligned}
$$
where $P$ is an enough large constant to prioritize the constraints over the objective.











```
row_constraint = 0
column_constraint = 0
consecutive_constraint = 0
vehicle_constraint = 0
objective = 2142
Vehicle 0 : load = 91 / 100 : 0 -> 4(91) -> 0
Vehicle 1 : load = 177 / 200 : 0 -> 6(59) -> 5(66) -> 8(52) -> 0
Vehicle 2 : load = 288 / 300 : 0 -> 7(10) -> 9(83) -> 1(44) -> 3(94) -> 2(57) -> 0
```