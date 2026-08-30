---
layout: post
title: "In Freshmen's Terms, What is a Matroid? (III)"
tagline: "The Greedy Algorithm"
date: 9999-08-30 13:00:00 -0000
categories: matroids
---

In the [first posts]() on matroids, we gave some basic definitions on the combinatorial structure that is called a matroid.
While one can admire the beautiful theory behind matroids just like that, there is a killer feature that matroids have which makes them special in a very applied setting:
It is the fact that the there is a greedy algorithm (I personally would even refer to it as *the* greedy algorithm) that will find a maximum (or minimum) weight base for every linear weight function $$w\colon E \to \R_+$$ on a matroid $$M = (E, \I)$$.

To put this more explicit, we consider the following problem:

**MAXIMUM WEIGHT MATROID BASE**  
**Given:** A matroid $$M = (E, \I)$$ and a weight function $$w\colon E \to \R_+$$.  
**Find:** An independent set $$I \in \I$$ of $$M$$ with $$w(I) = \sum_{e \in I} w(e)$$ maximum.

It is clear that if all weights are indeed non-negative, then $$I$$ can always be chosen to be a base, i.e. a maximal independent set.

Remarkably, the first algorithm that anyone would come up with to attempt to solve this problem is the right one:

```
Sort elements in \(E\) in descending order of \(w(e)\)
\(I \coloneqq \emptyset\)
for \(e \in E\):
	if \(I+e \in \I\):
		\(I \coloneqq I+e\)
return \(I\)
```

The pseudocode above starts with the empty set as a trivially independent set and then just checks if the element of maximum weight can be added to $$I$$ without making it a dependent set.
We previously used graphic matroids as a running example for matroids, which have spanning trees as bases; and indeed the matroid greedy algorithm when applied to a graphic matroid boils down to [Kruskal's Algorithm]() for the Maximum (or Minimum) Spanning Tree problem.
This greedy algorithm can be used of course for any independence system, not just matroids, but only if it the independence system is a matroid the algorithm is guaranteed to return an optimal solution.

**Theorem 1.** Let $$M = (E, \I)$$ be a matroid and $$w \colon E \to \R_+$$ a weight function on $$M$$.
Then the greedy algorithm returns a base of $$M$$ of maximum weight.

*Proof.* Let $$B$$ be the set that the algorithm returns.
Note that $$B$$ is indeed a base of $$M$$, i.e. maximal independent.
Independence is trivial as, we only add an element if the resulting set stays independent.
If it was not maximal, then there exists by the augmentation axiom (I3) an element $$e$$ that we can add to $$B$$ and stay independent.
This element however, would have been rejected by the greedy algorithm, which only happens if its addition to the current subset $$I$$ (which is a subset of $$B$$) renders a dependent set.  
Towards a contradiction let us now assume that $$B$$ has not maximum weight.\\(\def\tB{\tilde B}\\)
Then let $$\tB$$ be a base of maximum weight such that $$|B \cap \tB|$$ is maximal.
Choose any $$e \in B \setminus \tB$$; such an element must exist as otherwise $$B = T$$ and $$B$$ has maximum weight.
Since $$\tB$$ is a base, we have that $$\tB+e$$ is dependent and we consider its fundamental circuit $$C \coloneqq C(\tB, e)$$.
Note that $$e \in C$$ (otherwise $$C \subseteq \tB$$ which would mean that $$\tB$$ is dependent).
Now choose any $$f \in C \setminus B$$.
We then have that $$C-f$$ is independent and $$C-f \subseteq \tB+e$$.
Let $$A_0 \coloneqq C-f$$.
For all $$i \in [r]$$ with $$r \coloneqq |\tB|-|A_0|$$ we can use (I3) to find a $$g_i \in \tB \setminus A_i$$ such that $$A_{i+1} \coloneqq A_i+g_i$$ is independent.
Finally, let $$\tB' \coloneqq A_r$$.
Note that

$$C-f \subseteq \tB' \subseteq \tB+e$$

with $$f \notin \tB'$$ and $$|\tB'| = |T|$$.
Thus, $$\tB' = (T-f)+e$$ is a base.
As $$\tB$$ has maximum weight, we have $$w(\tB') \leq w(\tB)$$ and hence, $$w(f) \geq w(e)$$.
On the other hand, the algorithm selected $$e$$ instead of $$f$$ and hence, $$w(e) \geq w(f)$$.
Therefore $$w(e) = wf(f)$$ and $$w(\tB') = w(\tB)$$.
However, $$|\tB' \cap B| > |\tB \cap B|$$; this is a contradiction to the choice of $$\tB$$. $$\square$$

The story does not end here; for any non-matroidal independence system we can find a weight function that causes the greedy algorithm to fail to find an optimal solution.
This means that *guaranteed optimality through greedy algorithm* is yet another way to define matroids.

**Theorem 2.** Let $$M = (E, \I)$$ be an independence system.
If the greedy algorithm finds an optimal solution for every possible weight function $$w\colon E \to R_+$$, then $$M$$ is a matroid.

*Proof.* The axioms (I1) and (I2) hold by definition.
For (I3) we choose $$I_1, I_2$$ with $$|I_1| < |I_2|$$.
Let $$t \coloneqq |I_1 \cap I_2|$$ and $$s \coloneqq |I_1 \setminus I_2| = |I_1| - t$$, and $$r \coloneqq |I_2| - |I_1| = |I_2| - t - s$$.
We define the following weight function $$w\colon E \to \R_+$$ by

$$
w(e) = \begin{cases}
	s+r+2 & e \in I_1,\\
	s+2 & e \in I_2 \setminus I_1,\\
	0 & e \notin I_1 \cup I_2.
\end{cases}
$$

If we apply the algorithm to $$M$$ with weight function $$w$$, then it will first choose all the elements in $$I_1$$.
By our assumption, the algorithm finds a maximum weight independent set for every possible $$w$$.
However,

$$
\begin{aligned}
w(I_2)	&= t(s+r+2) + (s+r)(s+2)\\
		&= t(s+r+2) + (s+r+2) + 2r\\
		&= w(I_1) + 2r\\
		&> w(I_1).
\end{aligned}
$$

Thus, the algorithm also adds at least one more element $$e \notin I_1$$ after all elements in $$I_1$$ with $$w(e) > 0$$ and $$I_1+e \in \I$$.
By definition of $$w$$ we must have $$e \in I_2$$ and hence, (I3) follows. $$\square$$
