---
layout: page
title: Fantasy Football Scheduling Problem
---

We have $16$ teams $i \in T$ dividied into $4$ divisions $N, E, S, W$ and each team
has a position within its division $d_i$ from last year $p_i \in \{1, 2, 3, 4\}$.
Every division $D$ is assigned a subdivision $S_D$ in which each team from $D$ has 
to play two home and two away games (while playing each team from $S_D$ exactly once).
Every team has also to play twice against each other team of its own division $D_i$
(once at home, once away) and against each team that had the same position in its
respective division (thus, there are four teams that will be played twice---once at 
home, once away---by a team: The three teams in its own division and the team $j$ in 
$i$s subdivision $S_{D_i}$ that cam in the same place $p_i = p_j$ last year). This 
makes in total $13$ games for each team on $13$ game days $d \in G$.

We use the following types of variables:
  * $x_{ij}^d \in \{0, 1\}$ is $1$ iff team $i$ plays team $j$ on day $d$.
  * $y_{ij} \in \{0, 1\}$ is $1$ iff team $i$ plays team $j$ in two consecutive weeks.

The model is defined as following
$$
\begin{align}
	\text{minimize }	& \sum_{i \in T} \sum_{j \in T} y_{ij}																\label{ffsp:obj}\\
	\text{subject to }	& \sum_{j \in T} x_{ij}^d + x_{ji}^d = 1					& i \in T, d \in G						\label{ffsp:oneopp}\\
						& x_{ij}^d + x_{ji}^{d+1} \leq 1 + y_{ij}					& i, j \in T, d \in G					\label{ffsp:penalty}\\
						& \sum_{d \in G} x_{ij}^d = 1								& i \in T, j \in D_i					\label{ffsp:divhome}\\
						& \sum_{d \in G} x_{ji}^d = 1								& i \in T, j \in D_i					\label{ffsp:divaway}\\
						& \sum_{d \in G} x_{ij}^d + x_{ji}^d \geq 1					& i \in T, j \in S_{D_i}				\label{ffsp:sdiv}\\
						& \sum_{j \in S_{D_i}} \sum_{d \in G} x_{ij}^d \geq	2		& i \in T								\label{ffsp:sdivhome}\\
						& \sum_{j \in S_{D_i}} \sum_{d \in G} x_{ji}^d \geq	2		& i \in T								\label{ffsp:sdivaway}\\
						& \sum_{d \in G} x_{ij}^d + x_{ji}^d \geq 1					& i, j \in T, p_i = p_j					\label{ffsp:pos}\\
						& \sum_{d \in G} x_{ij}^d + x_{ji}^d = 2					& i, j \in T, j \in S_{D_i}, p_i = p_j	\label{ffsp:sdivpos}\\
						& \sum_{j \in T, p_i = p_j} \sum_{d \in G} x_{ij}^d \geq 1	& i \in T								\label{ffsp:poshome}\\
						& \sum_{j \in T, p_i = p_j} \sum_{d \in G} x_{ji}^d \geq 1	& i \in T								\label{ffsp:posaway}\\
						& \sum_{j \in T} x_{ij}^d + \sum_{j \in T} x_{ij}^{d+1} 
						+ \sum_{j \in T} x_{ij}^{d+2} \leq 2						& i \in T, d \in G						\label{ffsp:max2home}\\
						& \sum_{j \in T} x_{ji}^d + \sum_{j \in T} x_{ji}^{d+1} 
						+ \sum_{j \in T} x_{ji}^{d+2} \leq 2						& i \in T, d \in G						\label{ffsp:max2away}\\
						& \sum_{d \in G} x_{ij}^d \leq 1							& i, j \in T							\label{ffsp:1home1away}\\
						& \sum_{j \in S_{D_i}} x_{ij}^{12} + x_{ji}^{12}
						+ \sum_{j \in S_{D_i}} x_{ij}^{13} + x_{ji}^{13} \geq 2		& i \in T								\label{ffsp:last2weeks}\\
						& x_{ij}^d \in \{0, 1\}										& i, j \in T, d \in G\\
						& y_{ij} \in \{0, 1\}										& i, j \in T
\end{align}
$$
Note that \eqref{ffsp:last2weeks} ensures that the last two weeks are divisional matchups.
