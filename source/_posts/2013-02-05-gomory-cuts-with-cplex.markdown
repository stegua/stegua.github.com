---
layout: post
title: "From blackboard to code: Gomory Cuts using CPLEX"
date: 2013-02-05 15:22
comments: true
published: true
categories: [integer programming, cutting planes, CPLEX]
sharing: true
---

On the blackboard, to solve small Integer Linear Programs with 2 variables and _less or equal_ constraints is easy,
since they can be plotted in the plane and the linear relaxation can be solved geometrically. 
You can draw the lattice of integer points, and once you have found a new cutting plane, 
you show that it _cuts off_ the optimum solution of the LP relaxation.

This post presents a naive (textbook) implementation of Gomory cuts that uses the basic solution
computed by CPLEX, the commercial Linear Programming solver used in our lab sessions.
In practice, this post is an online supplement to one of my last exercise session.

In order to solve the *"blackboard"* examples with CPLEX, it is necessary to use a couple of functions
that a few years ago were undocumented. GUROBI has very similar functions, but they are currently undocumented.

As usual, all the sources used to write this post are publicly available on 
[my GitHub repository](https://github.com/stegua/MyBlogEntries/tree/master/GomoryCut).

### The basics
Given a Integer Linear Program in the form:

$$(P) \qquad \min \{ cx \mid Ax \leq b, \, x \geq 0, \, x \mbox{ integer} \}$$

it is possible to rewrite the problem in standard form by adding slack variables:

$$(P) \qquad \min \{ cx \mid Ax + Ix_S = b, \, x \geq 0, \, x \mbox{ integer}, \, x_S \geq 0, \}$$

where $$I$$ is the identity matrix and $$x_S$$ is a vector of slack variables, one for each constraint in $$(P)$$.
Let us denote by $$(\bar{P})$$ the linear relaxation of $$(P)$$ obtained by relaxing the integrality constraint.

The optimum solution vector of $$(\bar{P})$$, if it exists and it is finite, it is used to derive a basis
(for a formal definition of **basis**, see [1] or [3]).
Indeed, the basis partition the columns of matrix $$A$$ into two submatrices
$$B$$ and $$N$$, where $$B$$ is given by the columns corresponding to the basic variables,
and $$N$$ by columns corresponding to variables out of the base (they are equal to zero in the optimal solution vector).

Remember that, by definition, $$B$$ is nonsingular and therefore is invertible. 
Using the matrices $$B$$ and $$N$$, it is easy to derive the following inequalities (for details, see any OR textbook, e.g., [1]): 

$$\begin{eqnarray}
&Ax = b & \\
&Bx_B + Nx_N = b& \\
&x_B + B^{-1}N\,x_N = B^{-1}\,b & \qquad B \mbox{ is nonsingular} \\
&x_B + \lfloor B^{-1}N \rfloor \,x_N \leq B^{-1}\,b &\qquad  \mbox{since }x \geq 0 \\
&x_B + \lfloor B^{-1}N \rfloor \,x_N \leq \lfloor B^{-1}\,b \rfloor& \qquad x \mbox{ is integer}
\end{eqnarray}$$

where the operator $$\lfloor \cdot \rfloor$$ is applied component wise to the matrix elements.
In practice, for each fractional basic variable, it is possible to generate a valid Gomory cut.

The key step to generate Gomory cuts is to get an optimal base or, even better, the inverse of the basis matrix $$B^{-1}$$
multiplied with $$A$$ and with $$b$$. Once we have that matrix, in order to generate a Gomory cut from a fractional
basic variable, we just use the last previous equation row-wise.

Given the optimal basis, the optimal basic vector is $$x_B=B^{-1}A$$, since the non basic variable are equal to zero.
Let $$i$$ be the index of a fractional basic variable, and let $$j$$ be the index of the constraint corresponding to
variable $$i$$ in the system $$x_B=B^{-1}A$$, then the Gomory cut for variable $$i$$ is:

$$x_i + \sum_{l \in N} \lfloor (B^{-1}N)_{jl} \rfloor\,x_l \leq (B^{-1}\,b)_j$$

### Using the CPLEX callable library
The CPLEX callable library (written in C) has the following _advanced_ functions:

* [CPXbinvarow](http://pic.dhe.ibm.com/infocenter/cosinfoc/v12r5/index.jsp?topic=%2Filog.odms.cplex.help%2Frefcallablelibrary%2Fhtml%2Ffunctions%2FCPXbinvarow.html) computes the *i*-th row of the tableau
* [CPXbinvrow](http://pic.dhe.ibm.com/infocenter/cosinfoc/v12r5/index.jsp?topic=%2Filog.odms.cplex.help%2Frefcallablelibrary%2Fhtml%2Ffunctions%2FCPXbinvrow.html) computes the *i*-th row of the basis inverse
* [CPXbinvacol](http://pic.dhe.ibm.com/infocenter/cosinfoc/v12r5/index.jsp?topic=%2Filog.odms.cplex.help%2Frefcallablelibrary%2Fhtml%2Ffunctions%2FCPXbinvacol.html) computes the representation of the *j*-th column in terms of the basis
* [CPXbinvcol](http://pic.dhe.ibm.com/infocenter/cosinfoc/v12r5/index.jsp?topic=%2Filog.odms.cplex.help%2Frefcallablelibrary%2Fhtml%2Ffunctions%2FCPXbinvcol.html) computes the *j*-th column of the basis inverse

Using the first two functions, Gomory cuts from an optimal base can be generated as follows:

``` c++ Gomory cut https://github.com/stegua/MyBlogEntries/blob/master/GomoryCut/cpx_gomory.c Fork Me on GitHub
printf("\nGenerate Gomory cuts:\n");
idx = 0;
cut = 0;  /// Index of cut to be added
for ( i = 0; i < m-1; ++i ) 
if ( floor(b_bar[i]) != b_bar[i] ) {
   printf("Row %d gives cut ->   ", i+1);
   POST_CMD( CPXbinvarow(env, model, i, z) );
   rmatbeg[cut] = idx;
   for ( j = 0; j < n1; ++j ) {
      z[j] = floor(z[j]); /// DANGER!
      if ( z[j] != 0 ) {
         rmatind[idx] = j;
         rmatval[idx] = z[j];
         idx++;
      }
      /// Print the cut
      if ( z[j] >= 0 ) printf("+");
      printf("%.1f x%d ", z[j], j+1);
   }
   gc_rhs[cut] = floor(b_bar[i]); /// DANGER!
   gc_sense[cut] = 'L';
   printf("<= %.1f\n", gc_rhs[cut]);
   cut++;
}
/// Add the new cuts
POST_CMD( CPXaddrows (env, model, 0, n_cuts, idx, gc_rhs, gc_sense, 
         rmatbeg, rmatind, rmatval, NULL, NULL) );
```

The code reads row by row (index *i*) the inverse basis matrix $$B^{-1}$$ multiplied with $$A$$ (line 7),
and stores the corresponding Gomory cut in the compact matrix given by vectors `rmatbeg`, `rmatind`, and `rmatval` (lines 8-15).
The array `b_bar` contains the vector $$B^{-1}b$$ (line 21). In lines 28-31, all the cuts are added at once to current LP data structure.

On GitHub you find a small program that I wrote to generate Gomory cuts for problems written as $$(P)$$.
The repository have an [example of execution](https://github.com/stegua/MyBlogEntries/blob/master/GomoryCut/README.md) of my program.


The code is simple only because it is designed for small IPs in the form
$$ \min\, \{\, cx \mid Ax\, \leq\,b,\, x\geq 0\}$$.
Otherwise, the code **must** consider the effects of preprocessing, different sense of the constraints,
and additional constraints introduced because of range constraints.
If you are interested in a **real** implementation of MIR Gomory cuts, please look at
the [SCIP source code](http://scip.zib.de/doc/html/sepa__gomory_8h.shtml).

### Additional readings
The introduction of Gomory cuts in CPLEX was **The** major breakthrough of CPLEX 6.5 and produced
the version-to-version speed-up given by the blue bars in the chart below
(source: [Bixby's slides available on the web](http://www.ferc.gov/eventcalendar/Files/20100609110044-Bixby,%20Gurobi%20Optimization.pdf)):

{% img images/cplex65.png %}

Gomory cuts are still subject of research, since they pose a number of implementation challenges. 
These cuts suffer from severe numerical issues, mainly because the computation of the inverse matrix
requires the division by its determinant. 
A recent paper by Cornuejols, Margot, and Nannicini deals with some of these issues [2].

If you like to learn more about how the basis are computed in the CPLEX LP solver, there is very nice paper
by Bixby [3]. The paper explains different approaches to get the first basic feasible solution and
gives some hints of the CPLEX implementation of that time, i.e., 1992. Though the paper does not deal with Gomory
cuts directly, it is a very pleasant reading.

## References
1. <p>C.H. Papadimitriou, K. Steiglitz.
   <span class="title">Combinatorial Optimization: Algorithms and Complexity</span>. 1998.
   <a href="http://www.amazon.com/Combinatorial-Optimization-Algorithms-Complexity-Computer/dp/0486402584">[book]</a></p>

2. <p>G. Cornuejols, F. Margot and G. Nannicini.
   <span class="title">On the safety of Gomory cut generators</span>. Submitted in 2012.
   <span class="journal">Mathematical Programming Computation, under review.</span> 
   <a href="http://faculty.sutd.edu.sg/~nannicini/papers/testing_gomory.pdf">[preprint]</a></p>

3. <p>R.E. Bixby.
   <span class="title">Implementing the Simplex Method: The Initial Basis</span>.
   <span class="journal">Journal on Computing</span> vol. 4(3), pages 267--284, 1992.
   <a href="http://joc.journal.informs.org/content/4/3/267.short">[abstract]</a></p>


