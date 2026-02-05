---
layout: default
title: "SEND+MORE=MONEY"
---

# Math Puzzle: SEND MORE MONEY

**SEND MORE MONEY** is a famouse math puzzle to find assignment a digit number to each letter satisfying

$$
\text{SEND}+\text{MORE}=\text{MONEY}
$$

The constraints are:
- Assigned digits to letters are all distinct, and
- `S` and `M` shoud not be 0.

## QUBO++ formulation

We assign a unnique index to each letter as follows:

| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| letter | S | E | N | D | M | O | R | Y | L |

Let $I(\alpha)$ 
denote the index of letter $\alpha$ ($\in \lbrace S,E,N,D,M,O,R,Y,L\rbrace$).
A $9\times 10$ $X=(x_{i,j})$ $(0\leq i\leq 8, 0\leq j\leq 9)$ is used to represent a digit number assigned to each letter
so that $x_{I(\alpha),j}=1$ if and only if letter $\alpha$ is assigned a number $j$.

First, each row of $X$ must be one hot to represent a number:

$$
\begin{aligned}
\text{Row one-hot} &=\sum_{j=0}^{9}x_{i,j}=1 && (0\leq i \leq 8) 
\end{aligned}
$$

Also each column of $X$ must be 0 or 1 so that all digits take different numbers:


$$
\begin{aligned}
\text{Column one-hot} &= 0\leq \sum_{i=0}^{8}x_{i,j}\leq 1 && (0\leq j \leq 9) 
\end{aligned}
$$

The values of $\text{SEND}$, $\text{MORE}$, and $\text{MONEY}$ are represented
by the following expressions:

$$
\begin{aligned}
\text{SEND} &= 1000\sum_{k=0}^9 kx_{I(S),k}+ 100\sum_{k=0}^9 kx_{I(E),k}+ 10\sum_{k=0}^9 kx_{I(N),k}+\sum_{k=0}^9 kx_{I(D),k}\\ 
       &= \sum_{k=0}^9k(1000x_{I(S),k}+100x_{I(E),k}+10x_{I(N),k}+x_{I(D),k})\\
\text{MORE} &= 1000\sum_{k=0}^9 kx_{I(M),k}+ 100\sum_{k=0}^9 kx_{I(O),k}+ 10\sum_{k=0}^9 kx_{I(R),k}+\sum_{k=0}^9 kx_{I(E),k}\\ 
       &= \sum_{k=0}^9k(1000x_{I(M),k}+100x_{I(O),k}+10x_{I(R),k}+x_{I(E),k})\\
\text{MONEY} &= 10000\sum_{k=0}^9 kx_{I(M),k}+1000\sum_{k=0}^9 kx_{I(O),k}+ 100\sum_{k=0}^9 kx_{I(N),k}+ 10\sum_{k=0}^9 kx_{I(E),k}+\sum_{k=0}^9 kx_{I(Y),k}\\ 
       &= \sum_{k=0}^9k(10000x_{I(M),k} 1000x_{I(O),k}+100x_{I(N),k}+10x_{I(E),k}+x_{I(Y),k})
\end{aligned}
$$

Thus, the equal constraint is represented as follows
$$
\text{SEND}+\text{MORE} = \text{MONEY}
$$

Also, since $\text{S}$ and $\text{M}$ should not be 0,
the following constraints must be satisfied:
$$
x_{I(S),0} = x_{I(M),0}= 0
$$
