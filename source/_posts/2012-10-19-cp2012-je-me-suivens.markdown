---
layout: post
title: "CP2012: Je me suivens"
date: 2012-10-19 13:24
comments: true
published: false
categories: [Conference report, Constraint Programming, Optimization, Smart Grids]
sharing: true
---

Last week in Quebec City, there was the 
[18th International Conference on Principles and Practice of Constraint Programming](http://www.cp2012.org/).
This year the conference had a record of submissions (186 in total) and the program committee
made a vey nice job in organizing the plenary sessions and the tutorials.
You can find [very nice pictures](http://4c.ucc.ie/~hsimonis/CP2012/index.htm) of the conference on [Helmut's web page](http://4c.ucc.ie/~hsimonis/).

During the conference, the weather outside was pretty cold, but at the conference site
the discussions were warm and the presentations were intriguing.

In this post, I share an informal report of the conference as [_"Je me suivens"_](http://en.wikipedia.org/wiki/Je_me_souviens). 

### Challenges in Smart Grids
The invited talks were excellent and my favorite one was given by Miguel F. Anjos
on Optimization Challenges in Smart Grid Operations.
Miguel is not exactly a CP programmer, he is more on discrete non linear 
optimization, but his talk was a perfect mixed of applications, modeling, and solution techniques.
Please, read and enjoy [his slides](http://www.cp2012.org/slides/Anjos-OptChalSmartGrids-CP2012.pdf).

I like to mention just one of his observations. Nowadays, electric cars are becoming more and more
present. What would happen when each of us will have an electric car?
Likely, during the night, while sleeping, we will connect our car to the grid to recharge the
car batteries. This will lead to high variability in night peaks of energy demand.

How to manage these peaks?

Well, what Miguel has reported as a possible challenging option is to think of the collection of cars connected to the grid as a kind of huge battery. This sort of *collective* battery could be used to better handle the peaks of energy demands. Each car would play the game with a double role: if there is not an energy demand peak, you can recharge the car battery; otherwise, the car battery could be used as a power source and it could supply energy to the grid. This is an oversimplification, but as you can image there would be great challenges and opportunities for any _constraint optimizer_ in terms of modeling and solution techniques. 

I am curious to read more about, do you?

### Sessions and Talks
This year CP had the thicker conference proceedings, ever. Traditionally, the papers are presented in two parallel sessions. Two is not that much when you think that this year at [ISMP](http://ismp2012.mathopt.org/) there were 40 parallel sessions...
but still, you always regret that you could not attend the talk in the other session. Argh!

Here I like to mention just two works. However, the program chair is trying 
to make [all the slides](http://www.cp2012.org/accepted_papers.php) available.
Have a look at the program and at the slides: there are many good papers.

In the application track, Deepak Mehta gave a nice talk about a joint work with Barry O'Sullivan and Helmut Simonis
on [_Comparing Solution Methods for the Machine Reassignment Problem_](http://4c.ucc.ie/~hsimonis/reassignment.pdf), a problem
that Google has to solve every day in its 
[data centers](http://www.google.com/about/datacenters/gallery/#/tech) and that was the subject of the [Google/Roadef Challenge 2012](http://challenge.roadef.org/2012/en/sujet.php).
The true challenge is given by the HUGE size of the instances and the very short timeout (300 seconds). The work presented by Deepak is really interesting and they got excellent results using CP-based Large Neighborhood Search: they classified second at the challenge.

Related to the **Machine Reassignment Problem** there was a second interesting talk entitled 
_[Weibull-based Benchmarks for Bin Packing](http://www.springerlink.com/content/52j3197311333658/)_, 
by Ignacio Castineiras, Milan De Cauwer and Barry O'Sullivan. 
They have designed a parametric instance generator for bin packing problems based on the Weibull distribution.
Having a parametric generator is crucial to perform exhaustive computational results and to
identify those instances that are challenging for a particular solution technique.
For instance, they have considered a CP-approach to bin packing problems and they have identified those Weibull shape values that yield challenging instances for such an approach.
A nice feature is that their generator is able to create instances similar to those of the Google challenge...
I hope they will release their generator soon!

### The Doctoral Program
Differently from other conferences (as for instance [IPCO](http://ipco2013.dim.uchile.cl/)), CP gives PhD
students the opportunity to present their ongoing work within a [Doctoral Program](http://zivny.cz/dp12/).
The sponsors cover part of the costs for attending the conference.
During the conference each student has a mentor who is supposed to
help him. This year there were around 24 students and only very few of them
had a paper accepted at the main conference. This means that without
the Doctoral Program, most of these students would not had the opportunity to attend the conference.

Geoffrey Chu awarded the 2012 ACP Doctoral Research Award for his thesis
**_Improving Combinatorial Optimization_**. To give you an idea about the amount of his contributions, consider that after his thesis presentation, someone in the audience asked:

_"And you got only **one PhD** for all this work?"_

_Chapeau!_ Among other things, Chu has implemented [Chuffed](http://www.g12.csse.unimelb.edu.au/minizinc/challenge2011/description_chuffed.txt) one of the most efficient CP solver
that uses _lazy clause generation_ and that ranked very well at the last
[MiniZinc Challenge](http://www.g12.cs.mu.oz.au/minizinc/challenge2012/results2012.html), even if it was not one of the official competitors. 

For the record, the winner of the MiniZinc challenge of this year is (again) the [Gecode team](http://www.gecode.org). Congratulations!

### Next Year
Next year CP will be held in Sweden, at Uppsala University on 16-20 September 2013.
Will you be there? I hope so...

In the meantime, if you were at the conference, which was your favorite talk and/or paper?
