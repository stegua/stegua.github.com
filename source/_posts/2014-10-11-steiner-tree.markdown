---
layout: post
title: "Steiner Tree, Cutting Planes, and the importance of an Epsilon"
date: 2014-10-11 16:14
comments: true
categories: Steiner Tree, branch-and-cut, cutting planes, integer programming, Gurobi, Lemon Graph Library
published: false

---

This is a technical post about the following question:

> When solving the [Steiner Tree problem]() via [branch-and-cut](), which is the impact of adding **a small epsilon** to the weights of the separation problem?

The answer is: the epsilon has a **HUGE** impact! 

If you don't believe me, look at the plot below: it reports the dual bound as function of the number of iterations, with the blue and red lines refer to adding or not adding a small epsilon to the weighs of the separation problem (which is a standard **max flow** problem).

If you are curious about the reason of such impact, maybe you can find this post interesting.
 
If you are not curious, but you need a starting point to implement a branch-and-cut solver for the Steiner Tree problem, based on [Gurobi]() and the [Lemon Graph Library](), you can jump directly to [my github repo]() with the code used to write this post, while you can go to [Steiner Tree Lib]() to get a number of standard instances.

If you are not curious and you are not interested about the Steiner Tree problem, you can just stop to read this post.

## Steiner Tree via Branch-and-Cut

The Steiner Tree problem is defined on an undirected graph $G=(V,E)$ and a set of terminal
nodes $T \subseteq V$. Every edge $e={i,j} \in E$ has a cost $c_e$. The problem asks to find a tree in $G$ that spans the nodes in $T$.

In this post, we use a Integer Linear Programming model defined on the directed graph $H=(V,A)$, obtained from $G$ by inserting in $A$ the two arcs $(i,j)$ and $(j,i)$ for each edge ${i,j}  in E$; the cost of the arcs are equal to the cost of the corresponding edges.
In addition, we select a vertex that plays the role of the **root** vertex $r$: the Steiner Tree problem becomes the problem of finding an arborescence rooted in $r$ that **reach** every node in $T$. 

Formally, we introduce a 0-1 variable $x_e$ for each arc in $A$, and write the following model:

$$(P) = \min \sum_{e \in A} c_e x_e \mid \sum_{e \in \delta^+(S)} x_e \geq 1, \forall S \subset V, r \in S, \exists t \in T: t \in V \setminus S$$

All the magic of this model is in the **reachibility** constraints: for each cut of the digraph with the root node on one side (i.e., in $r \in S$) and any terminal node $t$ on the other side (i.e., $t \in V \setminus S$), we must have at least 1 arc variable $x_e$ that traverses the cut equal to 1. Indeed, these constraints are exponential in number, since the number of subsets of $V$ is $O(2^{|V|})$: 

> the only option to consider this exponential number of constraints is via branch-and-cut

We solve a relaxation of the problem with a small subset of constraints, and we add new constraints "on demand", only when we find a candidate solution that violates any of these constraints.

The problem of looking for a violated constrained is usually called the **separation problem**, and here is defined as follows:

Given a $x^*$ solution to (P), define for each arc a capacity $u_e$ equal to $x_e$.
At this point, for each terminal node $t$, find the maximal flow from $r$ to $t$. Remember that for duality, to each maximal flow corresponds a minimum cut, and note that if the capacity of this cut is strictly smaller than 1, you have find a violated inequality of the problem.

Here is they key part of this post. Given $\bar{x}$, we set each arc capacity $u_e=\bar{x}_e$
and we solve a sequence of max-flow problem to look for a cut of minimum capacity.

> What does it happen if in the separation problem we set $u_e=\bar{x}_e + \epsilon$ ??

> How significant can be the impact of an epsilon?


 about a short code to solve via branch-and-cut the classical [Steiner Tree Problem](), one of those fundamental Combinatorial Optimization problems that are not solvable in polynomial time (technically, it is [NP-hard]()).

