---
layout: post
title: "In Freshmen's Terms, What is a Matroid? (II)"
tagline: "Ranks, Spans, Flats and all that"
date: 2026-08-30 13:00:00 -0000
categories: matroids
---

In the [first posts](/blog/2026/05/20/in-freshmen-s-terms-what-is-a-matroid) on matroids, we gave some basic definitions on the combinatorial structure that is called a matroid.
The focus here was to lift the notion of *independence*, a term borrowed from linear algebra, to arbitrary combinatorial structures that allow for a reasonable definition of that word that look similar to the notion of independence in linear algebra.
Today, I want to dive into some further notions that can be found in matroid theory.
Each of which can be used to give a different axiomatic definition of matroids that are all equivalent.
One also uses the term *cryptomorphic* for those axiom systems (roughly: *equivalent but not obviously equivalent*).

There is a quote, which I once read on the fact that matroids have so many characterizations or cryptomorphisms which makes them really nice to work with.
I unfortunately cannot find the exact quote or its author anymore but it roughly went like this:

> There are notions of independence, dimension (rank), bases, closure, and duality, all coexisting in one theory.
> One might think such a theory too good to be true, yet it exists: the theory of matroids.

My best guess for the author of this quote is Gian-Carlo Rota, who promoted the term cryptomorphism in the 1960s.

The goal of this blog post is to define certain families of sets (or functions) for matroids, give some intuition and then show how they allow for a cryptomorphism, i.e. a semantically equivalent axiomatization of matroids.


### Bases
In [Part I](/blog/2026/05/20/in-freshmen-s-terms-what-is-a-matroid), we already defined a what a *base* of a matroid is: a maximal independent set.
By the independence axiom (I3), we must have that every base has the same cardinality (otherwise, if \\(B_1, B_2\\) are bases but \\(|B_1| < |B_2|\\), then there exists by (I3) some \\(e \in B_2 \setminus B_1\\) such that \\(B_1+e\\) is still independent, i.e. \\(B_1\\) was not maximal independent).
We denote the collection of all bases of a matroid by \\(\B\\).
Then, we can define an axiom system for matroids based on bases:

**Proposition 1.** A pair \\(M = (E, \B)\\) on a finite ground set \\(E\\) with a collection of its bases \\(\B \subseteq 2^E\\) is a matroid iff the following properties hold
* (B1) \\(\B \neq \emptyset\\) and
* (B2) for all \\(B_1, B_2 \in \B\\) and \\(e \in B_1 \setminus B_2\\), there exists some \\(f \in B_2 \setminus B_1\\) such that \\(B_1-e+f \in \B\\).

*Proof.* From \\(\B\\), we can construct the independent sets as follows:

$$\I = \{I \subseteq 2^E \mid I \subseteq B \text{ for some } B \in \B\}.$$

Then we can verify the independence axioms:
(I1) holds since due to (B1), we have \\(\B \neq \emptyset\\) and hence, there must exist a \\(B \in \B\\) for which \\(\emptyset \subseteq B\\) trivially holds.
For (I2), observe that for every \\(I_1 \subseteq I_2 \in \I\\), we have some \\(B \in \B\\) with \\(I_2 \subseteq B\\) and hence, \\(I_1 \subseteq B\\).
Thus, \\(I_1 \in \I\\).
Finally, for (I3), let \\(I_1, I_2 \in \I\\) with \\(|I_1| < |I_2|\\).
Without loss of generality, we may assume that \\(|I_1| = |I_2| - 1\\).
Then there exist \\(B_1, B_2\\) with \\(B_i \supseteq I_i\\) for \\(i \in \\\{1, 2\\\}\\).
Let 

$$
	\begin{aligned}
		I_1 &= \{e_1, \ldots, e_k\},\\
		B_1 &= \{e_1, \ldots, e_k, a_1, \ldots, a_q\},\\
		I_2 &= \{f_1, \ldots, f_k, f_{k+1}\},\\
		B_2 &= \{f_1, \ldots, f_k, f_{k+1}, b_1, \ldots, b_{q-1}\}.\\
	\end{aligned}
$$

Now consider \\(B_1 - a_q\\).
Then there exists some \\(x \in B_2\\) such that \\(B_1-a_q+x \in \B\\) by (B2).
If \\(x \in I_2\\) then \\(I_1+x \in \I\\) and (I3) is satisfied.
Otherwise, if \\(x \notin I_2\\), consider \\(B_1 - a_q + x - a_{q-1}\\).
Again by (B2), we must have some \\(y \in B_2\\) such that \\(B_1 - a_q + x - a_{q-1} + y \in \B\\) and if \\(y \in I_2\\) we have (I3) satisfied.
Otherwise we continue making those exchanges between \\(B_1\\) and \\(B_2\\).
Since \\(|\\\{a_1, \ldots, a_q\\\}| > |\\\{b_1, \ldots, b_{q-1}\\\}|\\) by the \\(q\\)-th iteration latest we must have the situation where we can replace \\(a_i\\) by some member of \\(I_2\\).
Hence, \\(\I\\) satisfies all three independence axioms. \\(\square\\)

Due to a theorem of Brualdi [Bru69] the second axiom can actually be strengthened which comes in very handy in a lot of proofs:
* (B2) for all \\(B_1, B_2 \in \B\\) and \\(e \in B_1 \setminus B_2\\), there exists some \\(f \in B_2 \setminus B_1\\) such that \\(B_1-e+f \in \B\\) and \\(B_2-f+e \in \B\\).

Under abuse of notation, we may write \\(M = (E, \I) = (E, \B)\\) etc. to refer to the same matroid where the actual symbol (\\(\I, \B, \ldots\\)) tells us whether we are looking at independent sets, bases, etc.


### Circuits
We introduced circuits already as the *minimal dependent sets* of a matroid.
It helps a lot to think of graphic matroids here, where circuits are exactly those sets of edges that form a cycle in a graph.
The collection of circuits of a given matroid is denoted by \\(\C\\) and also such a family of circuits uniquely determines the matroid.

**Proposition 2.** A pair \\(M = (E, \C)\\) on a finite ground set \\(E\\) with a collection of circuits \\(\C \subseteq 2^E\\) is a matroid iff the following properties hold
* (C1) \\(\emptyset \notin \C\\),
* (C2) for all \\(C_1, C_2 \in \C\\) with \\(C_1 \subseteq C_2\\) it holds that \\(C_1 = C_2\\), and
* (C3) for all \\(C_1, C_2 \in \C\\) with \\(e \in C_1 \cap C_2\\), there exists come \\(C_3 \in \C\\) with \\(C_3 \subseteq (C_1 \cup C_2) - e\\).

*Proof.* From \\(\C\\), we can construct the indepedent sets

$$\I = \{I \subseteq 2^E \mid I \not\supseteq C \text{ for all } C \in \C\}$$

and verify the independence axioms.
(I1) required \\(\emptyset \in \I\\), which obvioulsy holds using (C2) and the fact that the empty set contains nothing.
Similarly, for (I2) we have that if some subset \\(I\\) does not include any circuit from \\(\C\\), then this also holds for all subsets of \\(I\\).
Finally, for (I3), suppose there exists \\(I_1, I_2 \in \I\\) with \\(|I_1| < |I_2|\\) but no \\(e \in I_2 \setminus I_1\\) such that \\(I_1+e \in \I\\).
Then for each such \\(e\\), there exists a circuit \\(C_e \in \C\\) with \\(C_e \subseteq I_1+e\\).
As \\(I_1 \in \I\\), we must have \\(C_e \not\subseteq I_1\\) and hence, \\(e \in C_e\\).
Among all these circuits, choose one, say \\(C_e\\), minimizing \\(|C_e \setminus I_2|\\).
Since \\(e \in I_2\\), we have \\(C_e \setminus I_2 \subseteq I_1 \setminus I_2\\).
If \\(C_e \subseteq I_2\\), then \\(I_2 \notin I_2\\), hence, we can choose \\(f \in C_e \setminus I_2\\).
Since \\(f \in I_1\\) and \\(|I_1| < |I_2|\\), there exists some \\(g \in I_2 \setminus I_1\\).
By our assumption, \\(I_1+g\\) contains a circuit \\(C_g\\) with \\(g \in C_g\\).
Now \\(f \neq g\\) as \\(f \in I_1\\) and \\(g \notin I_1\\).
Moreover, \\(f \in C_e \cap C_g\\), since \\(f \in C_e\\) and \\(C_g-g \subseteq I_1\\), so \\(f \in C_g\\).
We now apply (C3)
