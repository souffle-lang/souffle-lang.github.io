---
sidebar: news_sidebar
title: 'An efficient tunable selective points-to analysis for large codebases'
date: 2017-06-18
author: psubotic
categories: [paper]
permalink: soap-paper.html
layout: post
---
This is the [paper that uses Souffle](/pdf/soap117.pdf).  It was published in SOAP@PLDI'17.

## Abstract
Points-to analysis is a fundamental static program analysis technique for tools including compilers and bug-checkers. Although 
object-based context sensitivity is known to improve precision of points-to analysis, scaling it for large Java codebases remains 
a challenge.

In this work, we develop a tunable, client-independent, object-sensitive points-to analysis framework where heap cloning is 
applied selectively. This approach is aimed at large codebases where standard analysis is typically expensive. Our design includes 
a pre-analysis that determines program points that contribute to the cost of an object-sensitive points-to analysis. A subsequent 
analysis then determines the context depth for each allocation site. While our framework can run standalone, it is also possible 
to tune it – the user of the framework can use the knowledge of the codebase being analysed to influence the selection of expensive 
program points as well as the process to differentiate the required context-depth. Overall, the approach determines where the cloning 
is beneficial and where the cloning is unlikely to be beneficial.

We have implemented our approach using Souffle (a Datalog compiler) and an extension of the DOOP framework. Our experiments on 
large programs, including OpenJDK, show that our technique is efficient and precise. For the OpenJDK, our analysis reduces 27% 
of runtime and 18% of memory usage in comparison with 2O1H points-to analysis for a negligible loss of precision, while for 
Jython from the DaCapo benchmark suite, the same analysis reduces 91% of runtime for no loss of precision.

{% include links.html %}
