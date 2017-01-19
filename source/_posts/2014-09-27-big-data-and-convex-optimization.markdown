---
layout: post
title: "Big Data and Convex Optimization"
date: 2014-09-27 16:38
updated: 2014-09-28 18:51
comments: true
published: true
categories: [Big Data, Convex Optimization, Mathematical Optimization, Machine Learning, Statistics]
sharing: true
---

In the last months, I came several times across different definitions of **Big Data**.
However, when someone asks me what Big Data means in practice, I am never
able to give a satisfactory explanation. Indeed, you can easily find a flood
of posts on twitter, blogs, newspaper, and even scientific journals and conferences,
but I always kept feeling that **Big Data** is a buzzword.

By sheer serendipity, this morning I came across [three paragraphs](http://web.stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf) clearly
stating the importance of Big Data from a scientific standpoint, that I like to **cross-post** here (the following paragraphs appear in the introduction of [1]):

*In all applied fields, it is now commonplace to attack problems through data analysis, particularly through the use of statistical and machine learning algorithms on what are often large datasets. In industry, this trend has been referred to as ‘Big Data’, and it has had a significant impact in areas as varied as artificial intelligence, internet applications, computational biology, medicine, finance, marketing, journalism, network analysis, and logistics.*

*Though these problems arise in diverse application domains, they share some key characteristics. First, the datasets are often extremely large, consisting of hundreds of millions or billions of training examples; second, the data is often very high-dimensional, because it is now possible to measure and store very detailed information about each example; and third, because of the large scale of many applications, the data is often stored or even collected in a distributed manner. As a result, it has become of central importance to develop algorithms that are both rich enough to capture the complexity of modern data, and scalable enough to process huge datasets in a parallelized or fully decentralized fashion. Indeed, some researchers have suggested that even highly complex and structured problems may succumb most easily to relatively simple models trained on vast datasets.*

> Many such problems can be posed in the framework of **Convex Optimization**. 

*Given the significant work on decomposition methods and decentralized algorithms in the optimization community, it is natural to look to parallel optimization algorithms as a mechanism for solving large-scale statistical tasks. This approach also has the benefit that one algorithm could be flexible enough to solve many problems.*

Even if I am not an expert of Convex Optimization [2], I do have my own **mathematical optimization** bias. 
Likely, you may have a different opinion (that I am always happy to hear), but, honestly, the above paragraphs
are the best content that I have read so far about **Big Data**.

### References

[1] S. Boyd, N. Parikh, E. Chu, B. Peleato, J. Eckstein.
*Distributed Optimization and Statistical Learning via the Alternating Direction Method of Multipliers*.
Foundations and Trends in Machine Learning. Vol. 3, No. 1 (2010) 1–122. [[pdf]](http://web.stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf)

[2] If you like to have a speedy overview of *Convex Optimization*, you may read a [J.F. Puget's blog post](https://www.ibm.com/developerworks/community/blogs/jfp/entry/convex_optimization?lang=en).
