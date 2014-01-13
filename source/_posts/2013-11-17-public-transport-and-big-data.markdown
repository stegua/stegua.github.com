---
layout: post
title: "Public Transport and Big Data"
date: 2013-11-17 14:10
updated: 2013-11-17 14:10
comments: true
published: true
sharing: true
categories: [Public Transport, Big Data, Mathematical Optimization, Analytics]
description: "Is it possible to improve the quality of Public Transport while keeping its cost competitive?"
cover:
---

**Big Data** is nowadays a *buzzword*. A simple query for "Big Data" on Google gives about 26,700,000 results. 

**Public Transport** is not really a *buzzword*, but still on Google you can get almost the same number as with "Big Data": 26,400,000 results.

### Why is Public Transport so important?

Because *many* of us use [Public Transport](http://en.wikipedia.org/wiki/Public_transport) every day, but *most* of us still use their own car to go to work, to bring child at school, and to go shopping. This has a negative impact on the quality of life of everyone and is clearly inefficient since it does cost more: 

1. More money.
2. More pollution.
3. More time.

(Well, for time, it is not always true, but it happens more often than commonly perceived). 

Thus, an important **challenge** is to improve the quality of Public Transport while keeping 
its cost competitive. The ultimate goal should be to increase the number of people that **trust** and use Public Transport.

How is it possible to achieve this goal?

### Transport Operators are **Big Data** producers (are they?)

Modern transport operators have installed so called *Automatic Vehicle Monitoring (AVM)* systems that use several technologies to monitor the fleet of vehicles that operates the service (e.g., metro coaches, buses, metro trains, trains, …).

The stream of data produced by an AVM might be considered as **Big Data** because of its **volume** and **velocity** (see [Big Data For Dummies](https://www.ibm.com/developerworks/community/blogs/jfp/entry/big_data_for_dummies23?lang=en), by J.F. Puget). Each vehicle produces at regular intervals (measured in seconds) data concerning its position and status. This information is stored in remote data centers. The data for a single day might not be considered as "Big", however once you start to analyze the historical data, the volume increases significantly. For instance, a public transport operator could easily have around 2000 thousands vehicles that operate 24 hours a day, producing data potentially every second.

At the moment, this stream of data misses the third dimension of Big Data that is **variety**. However, new projects that aim at integrating this information with the stream of data coming from social networks are quickly reaching maturity. One of such project is [SuperHub](http://www.superhub-project.eu/), a FP7 project that has recently won the best exhibit award in Cluster 2 *"Smart and sustainable cities for 2020+"*, at the [ICT2013 Conference](http://www.superhub-project.eu/media-centre/124-ict2013-conference-best-exhibit-award-.html) in Vilnius.

I don't know whether transport operators are really **Big Data** producers or they are merely **Small Data** producers, but data collected using AVMs are nowadays mainly used to report and monitor the daily activities.

In my own opinion, the data produced by transport operators, integrated with input coming from social networks, should be used to improve the quality of the public transport, for instance, by trying to better tackle **Disruption Management** issues.

So, I am curious:

> Do you know any project that uses AVM data, combined with Social Network inputs (e.g., from [Twitter](http://www.twitter.com)), to elaborate **Disruption Management** strategies for Public Transport? If yes, do they use **Mathematical Optimization** at all?
