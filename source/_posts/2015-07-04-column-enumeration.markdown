---
layout: post
title: "Graph Coloring: Column Generation or Column Enumeration?"
date: 2015-07-04 16:45
updated: 2015-07-15 16:46
comments: true
published: true
categories: [integer programming, graph coloring, column generation, column enumeration, CPLEX, GUROBI]
sharing: true
description: "A simple idea on solving Graph Coloring using a MIP solver via Column Enumeration"
cover: 
---

<style type="text/css">
table { width:100%; }
thead {
   background-color: rgba(0,0,255,0.3);
   color: black;
   text-indent: 14px;
   text-align: left;
}
td { padding:4px; }
tbody tr:nth-child(odd) { background-color: rgba(0, 0, 100, 0.2);  }
tbody tr:nth-child(even) { background-color: rgba(0, 0, 100, 0.1); }
.title { color: #07235F; }
.journal { font-style: italic; }
</style>

In this post, I like to share a simple idea on how to solve to optimality some [hard instances](http://www.info.univ-angers.fr/pub/porumbel/graphs/) of the [Graph Coloring](http://en.wikipedia.org/wiki/Graph_coloring) problem. This simple idea yields a "new time" record for a couple of hard instances. 

To date, the best **exact** approach to solve Graph Coloring 
is based on *[Branch-and-Price](http://en.wikipedia.org/wiki/Branch_and_price)* [1, 2, 3].
The branch-and-price method is completely different from the Constraint Programming approach I discussed in a [previous post](http://stegua.github.io/blog/2013/06/28/gecol/). A key component of Branch-and-Price is the *column generation* phase, which is 
intuitively quite simple, but mathematically rather involved for a short blog post.

Here, I want to show you that a modern Mixed Integer Programming (MIP) solver, such as [Gurobi](http://www.gurobi.com/) or [CPLEX](http://www-01.ibm.com/software/commerce/optimization/cplex-optimizer/), can solve a few hard instances of graph coloring with the following *"null implementation effort"*:

1. Enumerate all possible columns
2. Build a [.mps](http://en.wikipedia.org/wiki/MPS_%28format%29) instance with those columns
3. Use a MIP solver to solve the .mps instance

Indeed, in this post we try to answer to the following question:


> Is there any hope to solve any hard graph coloring instances with this naive approach?


## Formulation
Given an undirected graph $$G=(V,E)$$ and a set of colors $$K$$, 
the minimum (vertex) graph coloring problem consists of assigning a color to each vertex,
while every pair of adjacent vertices gets a different color. The objective is to minimize the number of colors used in a solution.

The branch-and-price approach to graph coloring is based on a *set covering* formulation.
Let $$S$$ be the collection of all the maximal stable sets of $$G$$,
and let $$S_i \subseteq S$$ be the maximal stable sets that contain the vertex $$i$$.
Let $$\lambda_s$$ be a 0-1 variable equal to 1 if all the vertices in the maximal stable set $$s \in S$$ 
get assigned the same color. Hence, the set covering model is:

$$\min \sum_{s \in S} \lambda_s \mbox{ such that } \sum_{s \in S_i} \lambda_s \geq 1, \forall i \in N, \lambda_s \in \{0,1\}, \forall s \in S.$$

Indeed, we *"cover"* every vertex of $$G$$ with the minimal number of maximal stable sets.
The issue with this model is the total number of maximal stable sets in $$G$$, which *is exponential in the number of vertices of G*.

Column Generation is a "mathematically elegant" method to by-pass this issue:
it lets you to solve the set covering model by *generating* a very small subset of the elements in $$S$$. This happens by repeatedly solving an auxiliary problem,
called the *pricing* subproblem. For graph coloring, the pricing subproblem consists of a Maximum Weighted Stable Set problem.
If you are interested in Column Generation, I recommend you to look at the first chapter of
the [Column Generation book](http://www.springer.com/gp/book/9780387254852),
which contains a nice tutorial on the topic, and I would **strongly** recommend reading the nice survey "Selected Topics in Column Generation", [4].

> How many maximal stable sets are in a hard graph coloring instance?

If this number were not so high, we could enumerate all the stable sets in $$S$$
and attempt to directly solve the set covering model without resorting to column generation.
However, *"high"* is a subjective measure, so let me do some computations on my laptop and give you some precise numbers.

## Hard instances
Among the [DIMACS instances](http://mat.gsia.cmu.edu/COLOR/instances.html) of Graph Coloring, there are a few instances
proposed by David Johnson, which are still unsolved (in the sense that we have not a computational proof of optimality of the best known upper bounds).

The table below shows the dimensions of these instances. The name of instances are DSJC{n}.{d}, where {n} is the number of vertices and {d} gives the density of the graph (e.g., DSJC125.9 has 125 vertices and 0.9 of density).

|---------|------------|---------|---------|----------|-----------|
| Graph | Nodes | Edges | Max stable sets | Enumeration Time |
|:--------|:----------:|:-------:|:-------:|:--------:|:---------:|
| DSJC125.9 | 125 | 6,961 | 524 | 0.00
| DSJC250.9 | 250 | 27,897 | 2,580 | 0.01
| DSJC500.9 | 500 | 112,437 | 14,560 | 0.12
| DSJC1000.9 | 1,000 | 449,449| 100,389 | 2.20
| DSJC125.5 | 125 | 3,891 | 43,268 | 0.53
| DSJC250.5 | 250 | 15,668 | 1,470,363 | 43.16
| DSJC500.5 | 500 | 62,624 | ? | out of memory
| DSJC1000.5 | 1,000 | 249,826 | ? | out of memory
| DSJC125.1 | 125 | 736 |  ? | out of memory
| DSJC250.1 | 250 | 3,218 |  ? | out of memory
| DSJC500.1 | 500 | 12,458 | ? | out of memory
| DSJC1000.1 | 1,000 | 49,629 | ? | out of memory
|---------|------------|---------|---------|----------|-----------|

<br>
As you can see the number of maximal stable sets (i.e. the cardinality of $$S$$)
of several instances is not so high, above all for very dense graphs, where the number of stables set is less than the number of edges. However, for sparse graphs, the number of maximal stable sets is too large for the memory available in my laptop.

Now, let me re-state the main question of this post:

> Can we **enumerate** all the maximal stable sets of $$G$$ and use a  MIP solver such as [Gurobi](http://www.gurobi.com/) or [CPLEX](http://www-01.ibm.com/software/commerce/optimization/cplex-optimizer/) to solve any Johnson's instance of Graph Coloring?

## Results
I have written a small script which uses [Cliquer](http://users.aalto.fi/~pat/cliquer.html) to enumerate all the maximal
stable sets of a graph, and then I generate an [.mps](http://en.wikipedia.org/wiki/MPS_%28format%29)
instance for each of the DSJC instance where I was able to store all maximal stable sets.
The .mps file are on my public [GitHub repository for this post](https://github.com/stegua/MyBlogEntries/tree/master/Coloring).

The table below shows some numbers for the sparse instances obtained using Gurobi (v6.0.0)
with a timeout of 10 minutes on my laptop. If you compare these numbers with the results published in the literature, you can see that they are not bad at all.

Believe me, these number are not bad at all, and establish a new **TIME RECORD**.

For example, the instance DSJC250.9 was solved to optimality only recently in
11094 seconds by [3], while the column enumeration approach solves the same instance on a similar hardware in only 23 seconds (!), and, honestly, our work in [2] did not solve this instance to optimality at all.

|---------|------------|---------|---------|----------|-----------|------|------|-----|
| Graph | Best known | Enum. Time | Run time | LB | UB | Time [2] | LB[2] | UB [2]  
|:--------|:----------:|:-------:|:-------:|:--------:|:---------:|:----:|:----:|:---:|
| DSJC125.9 |   **44** | 0.00  |  **0.44** | **44** | **44** | 44 | **44** | **44** | 
| DSJC250.9 |   **72** | 0.01  | **23**  | **72** | **72** |  timeout | 71 | 72 |
| DSJC500.9 |   128  | 0.12  |  timeout | 123 | **128** | timeout | 123 | 136
| DSJC1000.9 |   222  | 2.20  | timeout  | 215 | **229** | timeout | 215 | 245 |
| DSJC125.5 |   **17**   | 0.53  | **70.6** | **17** | **17** | 19033 | **17** | **17** |
| DSJC250.5 |   28  | 43.16  | timeout  | 26 | 33 | timeout | 26 | 31 |
|---------|------------|---------|---------|----------|-----------|------|------|-----|

<br>

> Can we ever solve to optimality DSJC500.9 and DSJC1000.9 via Column Enumeration?

I would say:

> "Yes, we can!"

... but likely we need to be smarter while branching on the decision variables, since the default branching strategy of a generic MIP solver does not exploit the structure of the problem. If I had the time to work again on Graph Coloring, I would likely use the same branching scheme used in [2], where we combined a Zykov's branching rule with a randomized [iterative deepening depth-first search](http://en.wikipedia.org/wiki/Iterative_deepening_depth-first_search) (randomised because at each restart we were using a different initial pool of columns). Another interesting option would be to tighten the set covering formulation with valid inequalities, by starting with those studied in [5]. 

In conclusion, I believe that enumerating all columns can be a simple but good starting point to attempt to solve to optimality at least the instances DSJC500.9 and DSJC1000.9.

Do you have some spare time and are you willing to take up the challenge?

## References
1. <p>A Mehrotra, MA Trick.
<span class="title">A column generation approach for graph coloring</span>.
<span class="journal">INFORMS Journal on Computing</span>. Fall 1996 vol. 8(4), pp.344-354. 
<a href="http://joc.journal.informs.org/content/8/4/344.short ">[pdf]</a></p>

2. <p>S. Gualandi and F. Malucelli.
<span class="title">Exact Solution of Graph Coloring Problems via Constraint Programming and Column Generation</span>.
<span class="journal">INFORMS Journal on Computing</span>. Winter 2012 vol. 24(1), pp.81-100. 
<a href="http://joc.journal.informs.org/content/24/1/81.short">[pdf]</a>
<a href="http://www.optimization-online.org/DB_FILE/2010/03/2568.pdf">[preprint]</a></p>

3. <p>S. Held, W. Cook, E.C. Sewell.
<span class="title">Maximum-weight stable sets and safe lower bounds for graph coloring</span>.
<span class="journal">Mathematical Programming Computation</span>. December 2012, Volume 4, Issue 4, pp 363-381.
<a href="http://link.springer.com/content/pdf/10.1007%2Fs12532-012-0042-3.pdf">[pdf]</a></p>

4. <p>M. Lubbecke and J. Desrosiers.
<span class="title">Selected topics in column generation</span>.
<span class="journal">Operations Research</span>. 2005, Volume 53, Issue 6, pp 1007-1023.
<a href="http://pubsonline.informs.org/doi/abs/10.1287/opre.1050.0234">[pdf]</a></p>

5. <p>
<span class="title">Set covering and packing formulations of graph coloring: algorithms and first polyhedral results</span>.
<span class="journal">Discrete Optimization</span>. 2009, Volume 6, Issue 2, pp 135-147.
<a href="http://www.sciencedirect.com/science/article/pii/S1572528608000716">[pdf]</a></p>
