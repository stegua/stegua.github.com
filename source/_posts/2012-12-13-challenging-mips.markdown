---
layout: post
title: "Challenging MIPs instances"
date: 2012-12-03 18:11
comments: true
categories: branch-and-cut heuristic AMPL Gurobi
published: true
comments: true
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

Today, I share seven challenging MIP instances as 
[.mps files](http://en.wikipedia.org/wiki/MPS_%28format%29)
along with the AMPL [model and data files](https://github.com/stegua/MyBlogEntries/tree/master/Roadef2012) 
I used to generate them. 
While I like the [MIPLIBs](http://miplib.zib.de/), I do prefer problem libraries like the 
[CSPLIB](http://www.csplib.org/) where
you get both a problem description **and** a set of data. This allows anyone to try
with her new model and/or method.

The MIP instances I propose come from my formulation of
the [Machine Reassignment Problem](http://challenge.roadef.org/2012/en/sujet.php) 
proposed for the [Roadef Challenge](http://roadef.org/content/index.htm) sponsored by Google last year. 
As I wrote in a [previous post](http://stegua.github.com/blog/2012/10/19/cp2012-je-me-souviens/), 
the Challenge had **huge** instances and a _micro_ time limit of 300 seconds.
I said _micro_ because I have in mind exact methods: there is little you can do in 300 seconds when you
have a problem with potentially as many as $50000 \times 5000$ binary variables. 
If you want to use math programming and start with the solution of a linear programming relaxation of the problem,
you have to be careful: it might happen that you cannot even solve the LP relaxation at the root node within 300 seconds.

That is why most of the participants tackled the Challenge mainly with heuristic algorithms.
The only _general purpose_ solver that qualified for the challenge is [Local Solver](http://www.localsolver.com),
which has a nice abstraction ("somehow" similar to AMPL) to well-known local search algorithms and move operators.
The Local Solver script used in the qualification phase is available 
[here](http://www.localsolver.com/misc/google_machine_reassignment.lsp).

However, in my own opinion, it is interesting to try to solve at least the instances of the qualification phase
with Integer Linear Programming (ILP) solvers such as 
[Gurobi](http:\\www.gurobi.com) and [CPLEX](http:\\http://www-01.ibm.com/software/integration/optimization/cplex-optimizer/).
Can these branch-and-cut commercial solvers be competitive on such problems? 

## Problem Overview

Consider you are given a set of processes $P$, a set of machines $M$,
and an initial mapping $\pi$ of each process to a single machine 
(i.e., $\pi_p = i$ if process $p$ is initially assigned to machine $i$).
Each process consumes several _resources_, e.g., CPU, memory, and bandwidth.
In the challenge, some processes were defined to be
_transient_: they consume resources both on the machine where they are initially located,
and in the machine they are going to be after the reassignment.
The problem asks to find a new assignment of processes to machines that minimizes a rather involved cost function.

A basic ILP model will have a 0-1 variable $x_{pi}$ equals to 1 if you
(re)assign process $p$ to machine $i$. The number of processes and the number of machines give
a first clue on the size of the problem. 
The constraints on the resource capacities yield a multi-dimensional knapsack subproblem for each machine.
The Machine Reassignment Problem has other constraints (kind of logical 0-1 constraints), 
but I do not want to bore you here with a full problem description. 
If you like to see my model, please read the 
[AMPL model file](https://github.com/stegua/MyBlogEntries/blob/master/Roadef2012/ampl-scripts/roadef2012.mod). 

## A first attempt with Gurobi
In order to convince you that the proposed instances are challenging, I report some computational results.

The table below reports for each instance the best result obtained by the participants
of the challenge (second column). The remaining four columns give 
the upper bound (UB), the lower bound (LB), the number of branch-and-bound nodes, and the computation time in seconds
obtained with Gurobi 5.0.1, a timeout of 300 seconds, and the default parameter setting on a rather old desktop
(single core, 2Gb of RAM).

|----------|---------------|---------------|---------------|-------|-------|
| Instance | Best Known UB | Upper Bound   | Lower Bound   | Nodes | Time  |
|:--------:|--------------:|--------------:|--------------:|------:|------:|
| a1-1     |    44,306,501 |**44,306,501** |**44,306,501** | 0     | 0.05  |
| a1-2     |   777,532,896 |   780,511,277 |   777,530,829 | 537   | -     |
| a1-3     |   583,005,717 |**583,005,720**|**583,005,715**| 15    | 48.76 |
| a1-4     |   252,728,589 |   320,104,617 |   242,404,632 | 24    | -     |
| a1-5     |   727,578,309 |**727,578,316**|**727,578,296**| 221   | 2.43  |
| a2-1     |           198 |    54,350,836 |           110 | 0     | -     |
| a2-2     |   816,523,983 | 1,876,768,120 |   559,888,659 | 0     | -     |
| a2-3     | 1,306,868,761 | 2,272,487,840 | 1,007,955,933 | 0     | -     |
| a2-4     | 1,681,353,943 | 3,223,516,130 | 1,680,231,407 | 0     | -     |
| a2-5     |   336,170,182 |   787,355,300 |   307,041,984 | 0     | -     |
|----------|---------------|---------------|---------------|-------|-------|

<br>
Instances **a1-1**, **a1-3**, **a1-5** are solved to optimality within 300 seconds
and hence they are not further considered.

The remaining seven instances are the challenging instances mentioned at the begging of this post.
The instances **a2-x** are embarrassing: they have an UB that is far away from both the best known UB
and the computed LB.
Specifically, look at the instance **a2-1**: the best result of the challenge has value 198, value Gurobi
(with my model) finds a solution with cost 54,350,836: you may agree that this is "_slightly_" more than 198.
At the same time the LB is only 110. 

Note that for all the **a2-x** instances the number of branch-and-bound nodes is zero.
After 300 seconds the solver is still at the root node trying to generate cutting planes and/or
running their primal heuristics. Using CPLEX 12.5 we got pretty similar results.

This is why I think these instances are challenging for branch-and-cut solvers. 

## Search Strategies: Feasibility vs Optimality
Commercial solvers have usually a meta-parameter that controls the search focus by setting other parameters
(but how they are precisely set is undocumented: do you know more about?).
The two basic options of this parameter are (1) to focus on looking for feasible solution
or (2) to focus on proving optimality.
The name of this parameter is **MipEmphasis** in CPLEX and **MipFocus** in Gurobi. 
Since the LPs are quite time consuming and after 300 seconds the solver is still at the root node, 
we can wonder whether generating cuts is of any help on these instances.

If we set the MipFocus to **feasibility** and we explicitly **disable all cut generators**, would we get better results?

Look at the table below:
the values of the upper bounds of instances **a1-2**, **a1-4**, and **a2-3** are slightly better than before: 
this is a good news. However, for instance **a2-1** the upper bound is worse, and for the other three instances there is no difference. Moreover, the LB are always weaker: as expected, there is no free lunch!

|----------|---------------|---------------|---------|-------| 
| Instance | Upper Bound   | Lower Bound   | Gap     | Nodes |
|:--------:|--------------:|--------------:|--------:|------:| 
| a1-2     |   779,876,897 |   777,530,808 |   0.30% |   324 | 
| a1-4     |   317,802,133 |   242,398,325 |  23.72% |    48 | 
| a2-1     |    65,866,574 |            66 |  99.99% |    81 | 
| a2-2     | 1,876,768,120 |   505,443,999 |  73.06% |     0 | 
| a2-3     | 1,428,873,892 | 1,007,955,933 |  29.45% |     0 | 
| a2-4     | 3,223,516,130 | 1,680,230,915 |  47.87% |     0 | 
| a2-5     |   787,355,300 |   307,040,989 |  61.00% |     0 | 
|----------|---------------|---------------|---------|-------| 

<br>
If we want to keep a timeout of 300 seconds, there is little we can do, unless we develop an ad-hoc decomposition approach.

> Can we improve those results with a branch-and-cut solver using a longer timeout?

Most of the papers that uses branch-and-cut to solve hard problems have a timeout
of at least one hour, and they start by running an heuristic for around 5 minutes.
Therefore, we can think of using the best results obtained by the participants of the 
challenge as starting solution. 

So, let us make a step backward: we enable all cut generators and we set all parameters at the default value.
In addition we set the time limit to one hour. The table below gives the new results.
With this setting we are able to "prove" near-optimality of instance **a1-2**, and we reduce
significantly the gap of instance **a2-4**.
However, the primal solutions are never improved by the solver: this means that we have not improved the results
obtained in the qulification phase of the challenge.
Note also that the number of nodes explored is still rather smalli despite the longer timeout.

|----------|---------------|---------------|---------|-------| 
| Instance | Upper Bound   | Lower Bound   | Gap     | Nodes |
|:--------:|--------------:|--------------:|--------:|------:| 
| a1-2     |   777,532,896 |   777,530,807 | ~0.001% |    0  |
| a1-4     |   252,728,589 |   242,404,642 |   4.09% |  427  |
| a2-1     |           198 |           120 |  39.39% | 2113  | 
| a2-2     |   816,523,983 |   572,213,976 |  29.92% |   18  | 
| a2-3     | 1,306,868,761 | 1,068,028,987 |  18.27% |   69  | 
| a2-4     | 1,681,353,943 | 1,680,231,594 |   0.06% |  133  | 
| a2-5     |   336,170,182 |   307,042,542 |   8.66% |  187  | 
|----------|---------------|---------------|---------|-------| 

<br>
What if we disable all cuts and set the **MipFocus** to feasibility again?

|----------|---------------|---------------|---------|-------| 
| Instance | Upper Bound   | Lower Bound   | Gap     | Nodes |
|:--------:|--------------:|--------------:|--------:|------:|
| a1-2     |   777,532,896 |   777,530,807 | ~0.001% |     0 |
| a1-4     |   252,728,589 |   242,398,708 |  4.09%  |  1359 |
| a2-1     |       **196** |            70 | 64.28%  |   818 |
| a2-2     |   816,523,983 |   505,467,074 | 38.09%  |    81 |
| a2-3     |**1,303,662,728**| 1,008,286,290 | 22.66%  |    56 | 
| a2-4     | 1,681,353,943 | 1,680,230,918 |  0.07%  |   108 |
| a2-5     |**336,158,091**|   307,040,989 |  8.67%  |   135 |
|----------|---------------|---------------|---------|-------| 

<br>
With this parameter setting, we improve the UB for 3 instances: **a2-1**, **a2-3**, and **a2-5**.
However, the lower bounds are again much weaker. Look at instance **a2-1**: the lower bound is
now 70 while before it was 120. If you look at instance **a2-3** you can see that even if
we got a better primal solution, the gap is weaker, since the lower bound is worse.

## RFC: Any idea?
With the focus on feasibility you get better results, but you might miss the ability to prove optimality.
With the focus on optimality you get better lower bounds, but you might not improve the primal bounds.

> 1) How to balance feasibility with optimality?

To use branch-and-cut solver and to disable cut generators is counterintuitive, but if you do you, you get better
primal bounds.  

> 2) Why should I use a branch-and-cut solver then? 

Any idea out there?

### Minor Remark
While writing this post, we got 3 solutions that are better than those obtained by the participants of 
the qualification phase: 
[a2-1](https://github.com/stegua/MyBlogEntries/blob/master/Roadef2012/certificates/sol_a2_1.sol), 
[a2-3](https://github.com/stegua/MyBlogEntries/blob/master/Roadef2012/certificates/sol_a2_3.sol), and 
[a2-5](https://github.com/stegua/MyBlogEntries/blob/master/Roadef2012/certificates/sol_a2_5.sol)
(the three links give the certificates of the solutions). 
We are almost there in proving optimality of **a2-3**, and we get better lower bounds than those 
[published](http://4c.ucc.ie/~hsimonis/reassignment.pdf) in [1].


## References
1. <p> Deepak Mehta, Barry O’Sullivan, Helmut Simonis. 
   <span class="title">Comparing Solution Methods for the Machine Reassignment Problem</span>. 
   In Proc of CP 2012, Québec City, Canada, October 8-12, 2012.
   </p>

# Credits
Thanks to [Stefano Coniglio](https://plus.google.com/116327072470709585073/posts) 
and to [Marco Chiarandini](http://imada.sdu.dk/~marco/) for their passionate discussions about the posts
in this blog. 
