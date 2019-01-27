---
layout: post
title: "An informal and biased Tutorial on Kantorovich-Wasserstein distances"
date: 2018-12-31 13:13
update: 2018-12-31 13:13
comments: true
published: true
categories: [Optimal Transport, Kantorovich-Wasserstein Distance, Linear Programming, Computational Combinatorial Optimization, Network Simplex]
sharing: true
description: "A biased tutorial on Kantorovich-Wasserstein distances in discrete settings"
cover:
---

Two years ago, I started to study **Computational Optimal Transport (OT)**, and, now, it is time to wrap up informally the main ideas by using an *Operations Research (OR)* perspective with a *Machine Learning (ML)* motivation.

Yes, an [OR](https://twitter.com/hashtag/orms) perspective.

> Why an OR perspective?

Well, because most of the current theoretical works on Optimal Transport have a strong functional analysis bias, and, hence, the are pretty far to be an "easy reading" for anyone working on a different research area. Since I'm more comfortable with "summations" than with "integrations", in this post I focus only on Discrete Optimal Transport and on Kantorovich-Wasserstein distances between a pair of discrete measure.

> Why an [ML](https://nips.cc/Conferences/2017/Schedule?showEvent=8758) motivation?

Because measuring the **similarity** between complex objects is a crucial basic step in several [#machinelearning](https://twitter.com/hashtag/machinelearning?vertical=news&src=hash) tasks. Mathematically, in order to measure the similarity (or dissimilarity) between two objects we need a metric, i.e., a distance function. And *Optimal Transport* gives us a powerful similarity measure based on the solution of a **Combinatorial Optimization** problem, which can be formulated and solved with **Linear Programming**.

**The main Inspirations for this post are:**

*. My current favorite tutorial is [Optimal Transport on Discrete Domains](https://arxiv.org/abs/1801.07745), by [Justin Solomon](http://people.csail.mit.edu/jsolomon/).
*. Two years ago, I started to study this topic thanks to [Giuseppe Savaré](https://www-dimat.unipv.it/savare/), and I wrote this post by looking also to [his set of slides](https://www-dimat.unipv.it/savare/Ravello2010/ravelloB.pdf). He has several interesting papers and you can look at his [publications list](https://www-dimat.unipv.it/savare/pubblicazioni/all.html).
*. For a complete overview of this topic look at the [Computational Optimal Transport](https://optimaltransport.github.io/book/) book, by [Gabriel Peyré](http://www.gpeyre.com/) and [Marco Cuturi](http://marcocuturi.net/). These two researchers, together with Justin Solomon, where among the organizers of two [#NeurIPS](https://en.wikipedia.org/wiki/Conference_on_Neural_Information_Processing_Systems) workshops, in [2014](http://www.iip.ist.i.kyoto-u.ac.jp/OTML2014/doku.php?id=home) and in [2017](http://otml17.marcocuturi.net/), on *Optimal Transport and Machine Learning*.
*. For a broader history of Combinatorial Optimization till 1960, see [this manuscript by Alexander Schrijver](https://homepages.cwi.nl/~lex/files/histco.pdf).

**DISCLAIMER 1:** This is a long "ongoing" post, and, despite my efforts, it might contain errors of any type. If you have any suggestion for improving this post, please (!), let me know about: I will be more than happy to mention you (or any of your avatars) in the acknowledgement section. Otherwise, if you prefer, I can offer you a drink, whenever we will meet in real life.

**DISCLAIMER 2:** I wrote this post while reading the book **[Ready Player One](https://www.goodreads.com/book/show/9969571-ready-player-one)**, by Ernest Cline.

**DISCLAIMER 3:** I'm recruiting postdocs. If you like the topic of this post and you are looking for a postdoc position, write me an email.

## Similarity measures and distance functions
A **metric** is a function, usually denoted by $$d$$, between a pair of objects belonging to a space $$X$$:

$$d : X \times X \rightarrow \mathbb{R}_+$$

Given any triple of points $$x,y,z \in X$$, the conditions that $$d$$ must satisfy in order to be a metric are:

1. $$d(x,y) \geq 0$$ *(non negativity)*
2. $$d(x,y) = 0 \Leftrightarrow x=y$$ *(identity of indiscernibles)*
3. $$d(x,y) = d(y,x)$$ *(symmetry)*
4. $$d(x,z) \leq d(x,y) + d(y,z)$$ *(triangle inequality or subadditivity)*

If the space $$X$$ is $$\mathbb{R}^k$$, then $$\mathbf{x}, \mathbf{y}, \mathbf{z}$$ are vectors of $$k$$ elements, and the most common distance is indeed the Euclidean function

$$d(\mathbf{x},\mathbf{y}) = \sqrt{\sum_{i = 1}^k(x_i - y_i)^2}$$

where $$x_i$$ is the $$i$$-th component of the vector $$\mathbf{x}$$. Clearly, the algorithmic complexity of computing (in finite precision) this distance is linear with the dimension of $$X$$.

**QUESTION:** What if we want to compute the distance between a pair of clouds of $$n$$ points defined in $$\mathbb{R}^k$$?

If we want to compute the distance between the two vectors, that represent the two clouds of $$n$$ points, we need to define a distance function.

Let me fix the notation first. If $$\mathbf{x}$$ is a vector, then $$x_i$$ is the $$i$$-th element. Suppose we have two matrices $$\mathbf{X}$$ and $$\mathbf{Y}$$ with $$n \times k$$ elements, which represent $$n$$ points in $$\mathbb{R}^k$$. We denote by $$\mathbf{x}_i$$ the $$i$$-th row of matrix $$\mathbf{X}$$, and by $$x_{ij}$$ the $$j$$-th element of row $$i$$. Indeed, the rows $$\mathbf{x}_i$$ and $$\mathbf{y}_i$$ of the two matrices give the coordinates $$x_{i1},\dots,x_{ik}$$ and $$y_{i1},\dots,y_{ik}$$ of the two corresponding points.

Whenever $$k=1$$, a simple choice is to consider the [Minkowski Distance](https://en.wikipedia.org/wiki/Minkowski_distance), which is a metric for normed vector spaces:

$$M_p(\mathbf{x},\mathbf{y}) = \left( \sum_{i=1}^n \mid x_i - y_i\mid^p \right)^{\frac{1}{p}}$$

where typical values of $$p$$ are:

* $$p=1$$ (Manhattan distance)
* $$p=2$$ (Euclidean distance, see above)
* $$p=\infty$$ (Infinity distance)

We have also the Minkowski norm, that is a function

$$\ell_p : \mathbb{R} \rightarrow \mathbb{R}_+$$

computed as

$$\ell_p(\mathbf{x}) = \left( \sum_{i=1}^n \mid x_i \mid^p\right)^{\frac{1}{p}}$$

Whenever $$k>1$$, we have to consider a more general distance function:

$$D : \mathbb{R}^{n \times k} \times \mathbb{R}^{n \times k} \rightarrow \mathbb{R}_+$$

such that the relations (1)-(4) are satisfied. We could use as distance function $$D(\mathbf{X},\mathbf{Y})$$ any [matrix norm](https://en.wikipedia.org/wiki/Matrix_norm), but, to begin with, we can use the Minkowski distance twice in cascade as follows.

1. First, we compute the distance between a pair of points in $$\mathbb{R}^k$$, which we call the **ground distance**, using $$M_q$$, with $$q\geq 1$$. By applying this function to all the $$n$$ pairs of points, we get a vector $$\mathbf{z}$$ of $$n$$ non-negative values:

$$ z_i = M_q(\mathbf{x}_i,\mathbf{y}_i), \quad i=1,\dots, n$$

2. Second, we apply the Minkowski norm $$\ell_p$$ to the vector $$\mathbf{z}$$.

Composing these two operations, we can define a distance function between a pair of vectors of points (i.e., pair of matrices) as follows:

$$D_{p,q}(\mathbf{X},\mathbf{Y}) = \ell_p(\mathbf{z}) = \ell_p(\left< M_q(\mathbf{x}_i,\mathbf{y}_i)\right>) = \left( \sum_{i=1}^n \left( \sum_{j = 1}^n \left(x_{ij} - y_{ij}\right)^q  \right)^{\frac{p}{q}} \right)^{\frac{1}{p}}$$

Note that for $$p=q=2$$, we get

$$D_{2,2}(\mathbf{X},\mathbf{Y}) = \sqrt{\sum_{i=1}^n \sum_{j = 1}^n \left(x_{ij} - y_{ij}\right)^2} = \mid\mid \mathbf{X} - \mathbf{Y}\mid\mid_F$$

which is the [Frobenius Norm](https://en.wikipedia.org/wiki/Matrix_norm#Frobenius_norm) of the element wise difference $$\mathbf{X} - \mathbf{Y}$$.

The main drawback of this distance function is that it implicitly relies on the order (position) of the single points in the two input vectors: any permutation of one (or both) of the two vectors will yield a different value of the ground distance. This happens because the distance function between the two input vectors considers only "interactions" between the $$i$$-th pair of points stored at the same $$i$$-th position in the two vectors.

**IMPORTANT.** Here is where Discrete Optimal Transport comes into action: it offers an alternative distance function based on the solution of a **Combinatorial Optimization** problem, which is, in the simplest case, formulated as the following **Linear Program**:

$$\begin{aligned}\mathcal{W}_d(\mathbf{X},\mathbf{Y}) := \min \;\;& \sum_{i=1}^n\sum_{j=1}^n d(\mathbf{x}_i,\mathbf{y}_j) \pi_{ij} \\
 & \sum_{i=1}^n \pi_{ij} = 1& j=1,\dots,n \\
 & \sum_{j=1}^n \pi_{ij} = 1& i=1,\dots,n \\
 & \pi_{ij} \geq 0 & i=1,\dots,n, j=1,\dots,n .
 \end{aligned}$$

If you have a minimal [#orms](https://twitter.com/hashtag/orms) background at this point you should have recognized that this problem is a standard [Assignment Problem](https://en.wikipedia.org/wiki/Assignment_problem): we have to assign each point of the first vector $$\mathbf{x}$$ to a single point of the second vector $$\mathbf{y}$$, in such a way that the overall cost is minimum. From an optimal solution of this problem, we can select among all possible permutations of the rows of $$\mathbf{Y}$$, the permutation that gives the minimal value of the Frobenius norm.

Whenever the ground distance $$d(\mathbf{x},\mathbf{y})$$ is a metric, then $$\mathcal{W}_d(\mathbf{X},\mathbf{Y})$$ is a metric as well. In other terms, the optimal value of this problem is a measure of distance between the two vectors, while the optimal values of the decision variables $$\mathbf{\pi}$$ gives a mapping from the rows of $$\mathbf{X}$$ to the rows of $$\mathbf{Y}$$ (in OT terminology, an *optimal plan*). This is possible because the LP problem has a [Totally Unimodular](https://en.wikipedia.org/wiki/Unimodular_matrix) coefficient matrix, and, hence, every basic optimal solution of the LP problem has integer values.

> **WAIT, LET ME STOP HERE FOR A SECOND!**

I am being too technical, too early. Let me take a step back in *History*.

## Once Upon a Time: from Monge to Kantorovich
The History of Optimal Transport is quite fascinating and it begins with the [*Mémoire sur la théorie des déblais et des remblais*](https://gallica.bnf.fr/ark:/12148/bpt6k35800/f796) by [Gaspard Monge](https://en.wikipedia.org/wiki/Gaspard_Monge) (1746-1818). I like to think of Gaspard as visiting the [Dune of Pilat](https://en.wikipedia.org/wiki/Dune_of_Pilat), near Bordeaux, and then writing his *Mémoire* while going back to home... but this is only my imagination. Still, particles of sand give me the most concrete idea for passing from a continuous to a discrete problem.

{% img center ../../../../../../images/dune_pillat.JPG %}

In his *Mémoire*, Gaspard Monge considered the problem of transporting *"des terres d'un lieu dans un autre"* at minimal cost. The idea is that we have first to consider the cost of transporting a single molecule of *"terre"*, which is proportional to its weight and to the distance from its initial and final position. The total cost of transportation is given by summing up the transportation cost of each single molecule. Using the Lex Schrijver's words, Monge's transportation problem was [camouflaged as a continuous problem](https://homepages.cwi.nl/~lex/files/histco.pdf).

The idea of Gaspard is to assign to each initial position a single final destination: it is not possible to split a molecule into smaller parts. This unsplittable version of the problem posed a very challenging problem where [*"the direct methods of the calculus of variations fail spectacularly"*](https://math.berkeley.edu/~evans/Monge-Kantorovich.survey.pdf) ([Lawrence C. Evans](https://math.berkeley.edu/~evans/), see link at page 5). This challenge stayed unsolved until the work of [Leonid Kantorovich](https://en.wikipedia.org/wiki/Leonid_Kantorovich) (1912-1986).

Curiously, Leonid did not arrive to the transportation problem while studying directly the work of Monge. Instead, he was asked to solve an industrial resource allocation problem, which is more general than Monge's problem. Only a few years later, he reconsidered his contribution in terms of the continuous probabilistic version of Monge's transportation problem. However, I am unable to state the true technical contributions of Leonid with respect to the work of Gaspard in a short post (well, honestly, I would be unable even in an infinite post), but I recommend you to read the [Long History of the Monge-Kantorovich Transportation Problem](https://link.springer.com/article/10.1007%2Fs00283-013-9380-x).

Anyway, I have pretty clear the two main concepts that are the foundations of the work by Kantorovich:

1. **RELAXATION**: He relaxes the problem posed by Gaspard and he proves that an optimal solution of the relaxation equals an optimal solution of the original problem (Here is the link to the very unofficial soundtrack of his work: ["Relax, take it easy!"](https://www.youtube.com/watch?v=RVmG_d3HKBA)). Indeed, Leonid relaxed formulation allows each molecule to be split across several destinations, differently from the formulation of Monge. In OR terms, he solves a Hitchcock Problem, not an Assignment Problem. For more details, see Chapter 1 of the [Computational Optimal Transport](https://optimaltransport.github.io/book/).

{% img center ../../../../../../images/relax_easy.PNG %}

2. **DUALITY**: Leonid uses a dual formulation of the relaxed problem. Indeed, he *invented* the dual potential functions and he sketched the first version of dual simplex algorithm. Unfortunately, I studied Linear Programming duality without having an historical perspective, and hence duality looks like *obvious*, but at the time is was clearly a new concept.

For the records, Leonid Kantorovich won the Nobel Prize, and [his autobiography](https://www.nobelprize.org/prizes/economic-sciences/1975/kantorovich/25950-autobiography-1975/) merits to be read more than once.

Well, I have still so much to learn from the past!

### Two Fields medal winners: Alessio Figalli and Cedric Villani
If you think that Optimal Transport belongs to the past, you are wrong!

Last summer (2018), in Rio de Janeiro, [Alessio Figalli](https://people.math.ethz.ch/~afigalli/) won the [Fields Medal](https://en.wikipedia.org/wiki/Fields_Medal) with this citation:

> *"for his contributions to the **theory of optimal transport**, and its application to partial differential equations, metric geometry, and probability"*

Alessio is not the first Fields medalist who worked on Optimal Transport. Already [Cedric Villani](https://cedricvillani.org/for-mathematicians/), who wrote the [most cited book](http://cedricvillani.org/wp-content/uploads/2012/08/B07.StFlour.pdf) on Optimal Transport [1], won the Fields Medal in 2010. I strongly suggest you to look any of [his lectures available on Youtube](https://www.youtube.com/watch?v=Kc0Kthyo0hU). And ... do you know that Villani spent a short period of time during his PhD in my current Math Dept. at University of Pavia?

As an extra bonus, if you don't know what a Fields Medal is, you can have Robin Williams to explain the prize in this clip taken from [Good Will Hunting](https://www.youtube.com/watch?v=TRU4YYxM7vU).

## Discrete Optimal Transport and Linear Programming
It is time of being technical again and to move from the Monge assignment problem, to the Kantorovich "relaxed" transportation problem. In the assignment model presented above, we are implicitly considering that all the positions are occupied by a single molecule of unitary weight. If we want to consider the more general setting of Discrete Optimal Transport, we need to consider the "mass" of each molecule, and to formulate the problem of transporting the total mass at minimum cost. Before presenting the model, we define formally the concept of a **discrete measure** and of the **cost matrix** between all pairs of molecule positions.

**DISCRETE MEASURES:** Given a of vector of $$n$$ positions $$\mathbf{x}_i$$, and given the [Dirac delta function](https://en.wikipedia.org/wiki/Dirac_delta_function), we can define the *Dirac measures* $$\delta_{\mathbf{x}_i}$$ as

$$\delta_{\mathbf{x}_i}(A) = \left\{ \begin{array}{ll} 1 & \text{if} \;\;\mathbf{x}_i \in A \subseteq X \\ 0 & \text{if} \;\;\mathbf{x}_i \notin A \subseteq X \\  \end{array}\right.$$

Given a vector of $$n$$ weights $$\mu_i$$, one associated to each element of $$\mathbf{x}$$, we can define the **discrete measure** $$\mathbf{\mu}$$ as

$$\mu(A) = \sum_{i=1}^n \mu_i \delta_{\mathbf{x}_i}$$

Note the $$\mu$$ is a function of type: $$\mu : A \rightarrow \mathbb{R}_+$$, for any subset $$A$$ of $$X$$. The vector $$\mathbf{x}$$ is called the support of the measure $$\mathbf{\mu}$$. In Computer Science terms, *a discrete measure is defined by a vector of pairs*, where each pair contains a positive number $$\mu_i$$ (the measured value) and its support point $$\mathbf{x}_i$$ (the location where the measure occurred). Note that $$\mathbf{x}_i$$ is a (small?) vector storing the coordinates of the $$i$$-th point.

**COST MATRIX:** Given two discrete measures $$\mathbf{\mu}$$ and $$\mathbf{\nu}$$, the first with support $$\mathbf{x}$$ and the second with support $$\mathbf{y}$$, we can define the following cost matrix:

$$c_{ij} = d(\mathbf{x}_i, \mathbf{y}_j), \quad \text{for } i=1,\dots,n \text{ and } j= 1,\dots, m$$

where $$d$$ is a distance function, such as, for instance, the Minkowski distance $$M_q(\mathbf{x},\mathbf{y})$$ defined before. Note that $$\mathbf{\mu}$$ has $$n$$ elements, and $$\mathbf{\nu}$$ has $$m$$ elements.

### Kantorovich-Wasserstein distances between two discrete measures
At this point, we have all the basic elements to define the **Kantorovich-Wasserstein distance function between discrete measures** in terms of the solution of a (huge) Linear Program.

**INPUT:** Two discrete measures $$\mathbf{\mu}$$ and $$\mathbf{\nu}$$ defined on a metric space $$X$$, and the corresponding supports $$\mathbf{x}$$ and $$\mathbf{y}$$, having $$n$$ and $$m$$ elements, respectively. A distance function $$d$$, which permits to compute the cost $$c_{ij}$$.

**OUTPUT:** A transportation plan $$\mathbf{\pi}$$ and a value of distance of $$\mathcal{W}(\mathbf{\mu}, \mathbf{\nu})$$ that corresponds to an optimal solution of the following Linear Program:

$$\begin{aligned}
\mathcal{W}(\mathbf{\mu},\mathbf{\nu}) := \min \;\;& \sum_{i=1}^n\sum_{j=1}^m c_{ij} \pi_{ij} \\
 & \sum_{j=1}^m \pi_{ij} \geq \mu_i & i=1,\dots,n \\
 & \sum_{i=1}^n \pi_{ij} \leq \nu_j& j=1,\dots,m \\
 & \pi_{ij} \geq 0 & i=1,\dots,n, j=1,\dots,m.
 \end{aligned}$$

This Linear Program is indeed a special case of the [Transportation Problem](https://www.jstor.org/stable/2627172?seq=1#page_scan_tab_contents), known also as the Hitchcock-Koopmans problem. It is a special case because the cost vector has a strong structure that should be exploited as much as possible.

**Computational Challenge:** While the previous problem is polynomially solvable, the size of practical instances is very large. For instance, if you want to compute the distance between a pair of grey scale images of resolution $$512 \times 512$$ pixels, you end up with an LP with $$512^4 = 68\;719\;476\;736$$ cost coefficients. Hence, these problems must be handled with care. If you want to see how the solution time and memory requirement scale for grey scale image, please, have a look at the [slides of my talk at Aussois (2018)](http://www.iasi.cnr.it/aussois/web/uploads/2018/slides/gualandis.pdf).

{% img center ../../../../../../images/ot_challenge.png %}

### KANTOROVICH-WASSERSTEIN DISTANCE

Whenever

1. The two measure are discrete probability measures, that is, both $$\sum_{i=1}^n \mu_i = 1$$ and $$\sum_{j = 1}^m \nu_j = 1$$ (i.e., $$\mathbf{\mu}$$ and $$\mathbf{\nu}$$ belongs to the probability simplex), and,
2. The cost vector is defined as the $$p$$-th power of a distance,

then we define the **Kantorovich-Wasserstein distance of order $$p$$** as the following functional:

$$\mathcal{W}_p(\mathbf{\mu},\mathbf{\nu}) := \left(\min_{\pi \in U} \sum_{i=1}^n\sum_{j=1}^m d(\mathbf{x}_i, \mathbf{y}_j)^p \pi_{ij} \right)^{\frac{1}{p}}$$

where the set $$U$$ is defined as:

$$U := \left\{\begin{array}{ll}
 \sum_{j=1,\dots,m} \pi_{ij} \leq \mu_i & i=1,\dots,n \\
 \sum_{i=1,\dots,n} \pi_{ij} \geq \nu_j& j=1,\dots,m \\
 \pi_{ij} \geq 0 & i=1,\dots,n, j=1,\dots,m. \\
 \end{array}
 \right\}
$$

From a mathematical perspective, the most interesting case is the order $$p=2$$, which *generalizes* the Euclidean distance to discrete probability vectors. Note that in this formulation, the two constraint sets defining $$U$$ could be replaced with equality constraints, since $$\mathbf{\mu}$$ and $$\mathbf{\nu}$$ belongs to the probability simplex. In addition, any Combinatorial Optimization algorithm must be used with care, since all the cost and constraint coefficients are not integer.
**Note:** The $$p$$ power used for the ground distance must not be confused with the order (power) of a Kantorovich-Wasserstein distance.

### EARTH MOVER DISTANCE

A particular case of the Kantorovich-Wasserstein distance very popular in the Computer Vision research community, is the so-called **Earth Mover Distance (EMD)** [2], which is used between a pair of $$k$$-dimensional histograms obtained by preprocessing the images of interest. In this case, (i) we have $$n=m$$, (ii) we do not require the discrete measures to belong to the probability simplex, and (iii) we do not even require that the two measures are *balanced*, that is, $$\sum_{i=1}^n \mu_i = \sum_{j=1}^n \nu_j$$. In Optimal Transport terminology, the Earth Mover Distance solves an **unbalanced optimal transport problem**. For the EMD, the feasibility set $$U$$ is replaced by the set:

$$V := \left\{\begin{array}{ll}
 \sum_{i=1}^n \sum_{j=1}^n \pi_{ij} = \min \{ \sum_{i=1}^n \mu_i, \sum_{j=1}^n \nu_j \}& \\
 \sum_{j=1,\dots,n} \pi_{ij} \geq \mu_i & i=1,\dots,n \\
 \sum_{i=1,\dots,n} \pi_{ij} \geq \nu_j& j=1,\dots,n \\
 \pi_{ij} \geq 0 & i=1,\dots,n, j=1,\dots,n. \\
 \end{array}
 \right\}
$$

The cost function is taken with the order $$p=1$$:

$$EMD(\mathbf{x},\mathbf{y}) := \min_{\pi \in V} \sum_{i=1}^n\sum_{j=1}^m d(x_i, y_j) \pi_{ij}$$

The most used function $$d$$ is the Minkowski distance induced by the $$\ell_p$$ norm.

### WORD MOVER DISTANCE

A very interesting application of discrete optimal transport is the definition of a metric for text documents [3]. The main idea is, first, to exploit a [word embedding](https://en.wikipedia.org/wiki/Word_embedding) obtained, for instance, with the popular [word2vec](https://www.tensorflow.org/tutorials/representation/word2vec) neural network [4], and, second, to formulate the problem of "transporting" a text document into another at minimal cost.

> Yes, but ... how we compute the ground distance between two words?

A word embedding associates to each word of a given vocabulary a vector of $$\mathbb{R}^k$$. For the pre-trained embedding made available by Google at [this archive](https://code.google.com/archive/p/word2vec/), which contains the embedding of around 3 millions of words, $$k$$ is equals to 300. Indeed, given a vocabulary of $$n$$ words and fixed a dimension $$k$$, a word embedding is given by a matrix $$\mathbf{X}$$ of dimension $$n \times k$$: Row $$\mathbf{x}_i$$ gives the $$k$$-dimensional vector representing the word embedding of word $$i$$.

In this case, instead of having discrete measures, we deal with normalized bag-of-words (nBOW), which are vector of $$\mathbb{R}^n$$, where $$n$$ denotes the number of words in the vocabulary. If a text document $$\mathbf{\mu} \in \mathbb{R}^n$$ contains $$t_i$$ times the word $$i$$, then $$\mathbf{\mu}_i = \frac{t_i}{\sum_{j=1}^n t_j}$$. At this point is clear that the **ground distance** between a pair of words is given by the distance between the corresponding embedding vectors in $$\mathbb{R}^k$$, that is, given two words $$i$$ and $$j$$, then

$$c(i,j) = \ell_2(\mathbf{x}_i - \mathbf{x}_j)$$

Finally, given two text documents $$\mathbf{\mu}$$ and $$\mathbf{\nu}$$, we can formulate the Linear Program that gives the **Word Mover Distance** as:

$$\text{WMD}(\mathbf{\mu},\mathbf{\nu}) := \min_{\pi \in U} \sum_{i=1}^n\sum_{j=1}^n c(\mathbf{x}_i, \mathbf{x}_j) \pi_{ij}$$

where the set $$U$$ is defined as:

$$U := \left\{\begin{array}{ll}
 \sum_{j=1,\dots,n} \pi_{ij} = \mu_i & i=1,\dots,n \\
 \sum_{i=1,\dots,n} \pi_{ij} = \nu_j& j=1,\dots,n \\
 \pi_{ij} \geq 0 & i=1,\dots,n, j=1,\dots,n. \\
 \end{array}
 \right\}
$$

If you are serious reader, and you are still reading this post, then it is clear that the Word Mover Distance is exactly a Kantorovich-Wasserstein distance of order 1, with an Euclidean ground distance. If you are interested in the quality of this distance when used within a nearest neighbor heuristic for a text classification task, we refer to [3]. *(While writing this post, I started to wonder how would perform a **WMD** of order 2, but this is another story...)*


### Interesting Research Directions
The following are the research topics I am interested in right now. Each topic deserves its own blog post, but let me write here just a short sketch.

* **Computational Challenges:** The development of efficient algorithms for the solution of Optimal Transport problems is an active area of research. Currently, the preferred (heuristic) approach is based on so-called [regularized optimal transport](http://marcocuturi.net/SI.html), introduced originally in [5]. Indeed, regularized optimal transport deserves its own blog post.
In two recent works, together with my co-authors, we tried to revive the [Network Simplex](https://scholar.google.it/scholar?q=dantzig+network+simplex&hl=en&as_sdt=0&as_vis=1&oi=scholart) for two special cases: Kantorovich-Wasserstein distances of order 1 for $$k$$-dimensional histograms [6] and Kantorovich-Wasserstein distances of order 2 for decomposable cost functions [7]. The second paper  was presented as a [poster at NeurIPS2018](https://github.com/stegua/dpartion-nips2018/blob/master/poster/PosterNIPS2018.pdf). *(If you ever read any of the two papers, please, let me know what you think about them)*

* **Unbalanced Optimal Transport:** The computation of Kantorovich-Wasserstein distance for pair of unbalanced discrete measures is very challenging. Last year, a brilliant math student finished a nice project on this topic, which I hope to finalize during the next semester.

* **Barycenters of Discrete Measures:** The Kantorovich-Wasserstein distance can be used to generalize the concept of *barycenters*, that is, the problem of finding a discrete measure that is the closest (in Kantorovich terms) to a given set of discrete measures. The problem of finding the barycenter can be formulated as a Linear Program, where the unknowns (the decision variables) are both the transport plan and the discrete measure representing the barycenter. For instance, the following images, taken from our last draft paper, represents the barycenters of each of the 10 digits of the [MNIST](http://yann.lecun.com/exdb/mnist/) data set of handwritten images (each image is the barycenter of other $$3\;200$$ images).

{% img center ../../../../../../images/mnist_bary.PNG %}

* **Distances between Discrete Measures defined on different metric spaces:** This topic is at the top of my 2019 resolutions. There are a few papers by [Facundo Mémoli](https://people.math.osu.edu/memoli.2/) on this topic, which are based on the so-called **Gromov-Wasserstein distance** and that requires the solution of a **Quadratic Assignment Problem (QAP)**.

Now, it's time to close this post with final remarks.

## Optimal Transport: A brilliant future?
Given the number and the quality of results achieved on the Theory of Optimal Transport by "pure" mathematicians, it is the time to turn these theoretical results into a set of useful algorithms implemented in efficient and scalable solvers. So far, the only public library I am aware of is [POT: Python Optimal Transport](https://github.com/rflamary/POT).

On [Medium](https://medium.com/), C.E. Perez claims that [Optimal Transport Theory (is) the New Math for Deep Learning](https://medium.com/intuitionmachine/optimal-transport-theory-the-new-math-for-deep-learning-2520395fc183). In his short post, he explains how Optimal Transport is used in Deep Learning algorithms, specifically in **Generative Adversarial Networks (GANs)**, to replace the **Kullback-Leibler (KL) divergence**.

Honestly, I do not have a clear idea regarding the potential impact of Kantorovich distances on **GANs** and **Deep Learning** in general, but I think there are a lot of research opportunities for everyone with a strong passion for **Computational Combinatorial Optimization**.

And you, what do you think about the topics presented in this post?

As usual, I will be very happy to hear from you, in the meantime...

> **GAME OVER**

### Acknowledgement
I would like to thank [Marco Chiarandini](https://imada.sdu.dk/~marco/), [Stefano Coniglio](https://www.southampton.ac.uk/maths/about/staff/sc2r15.page), and [Federico Bassetti](http://www-dimat.unipv.it/~bassetti/) for constructive criticism of this blog post.

### References

1. <p>Villani, C. <span class="title">Optimal transport, old and new.</span> Grundlehren der mathematischen Wissenschaften, Vol.338, Springer-Verlag, 2009. <a href="http://cedricvillani.org/wp-content/uploads/2012/08/B07.StFlour.pdf">[pdf]</a></p>
2. <p>Rubner, Y., Tomasi, C. and Guibas, L.J., 2000. <span class="title">The earth mover's distance as a metric for image retrieval.</span> International journal of computer vision, 40(2), pp.99-121. <a href="https://idp.springer.com/authorize/casa?redirect_uri=https://link.springer.com/content/pdf/10.1023/A:1026543900054.pdf&casa_token=Ysv8VHVK29EAAAAA:nIcUgwC_iwnFqj7C1DUcKcVi-JY4FvCt9GQOhNQsa4nmVe_H3BfpO197Y_sakGDwqCfziSHEt3q1Mg">[pdf]</a></p>
3. <p>Kusner, M.J., Sun, Y., Kolkin, N.I., Weinberger, K.Q. <span class="title">From Word Embeddings To Document Distances.</span> Proceedings of the 32 nd International Conference on Machine Learning, Lille, France, 2015. <a href="http://proceedings.mlr.press/v37/kusnerb15.pdf">[pdf]</a></p>
4. <p>Mikolov, T., Sutskever, I., Chen, K., Corrado, G.S. and Dean, J., 2013. <span class="title">Distributed representations of words and phrases and their compositionality.</span> In Advances in neural information processing systems (pp. 3111-3119). <a href="https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf">[pdf]</a></p>
5. <p>Cuturi, M., 2013. <span class="title">Sinkhorn distances: Lightspeed computation of optimal transport.</span> In Advances in neural information processing systems (pp. 2292-2300). <a href="https://papers.nips.cc/paper/4927-sinkhorn-distances-lightspeed-computation-of-optimal-transport.pdf">[pdf]</a></p>
6. <p>Bassetti, F., Gualandi, S. and Veneroni, M., 2018. <span class="title">On the Computation of Kantorovich-Wasserstein Distances between 2D-Histograms by Uncapacitated Minimum Cost Flows.</span> arXiv preprint arXiv:1804.00445. <a href="https://arxiv.org/pdf/1804.00445">[pdf]</a></p>
7. <p>Auricchio, G., Bassetti, F., Gualandi, S. and Veneroni, M., 2018. <span class="title">Computing Kantorovich-Wasserstein Distances on d-dimensional histograms using (d+1)-partite graphs. </span> NeurIPS, 2018.<a href="https://arxiv.org/pdf/1805.07416">[pdf]</a></p>
