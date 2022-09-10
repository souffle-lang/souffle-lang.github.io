---
sidebar: news_sidebar
title: 'Building a Join Optimizer for Soufflé'
date: 2022-09-10
author: Sam Arch 
categories: [paper]
permalink: lopstr22.html
layout: post
---
The paper [Building a Join Optimizer for Soufflé](/pdf/lopstr2022.pdf) has
been published in LOPSTR 2022: The 32nd International Symposium on Logic-Based Program Synthesis and Transformation. 

The paper describes the design of a feedback-directed join optimizer for compiling Datalog engines
such as Soufflé. The join optimizer automatically finds high-performance join orders that are competitive
with hand-tuned orders found by experts. As a result, Datalog programmers can unlock high-performance
execution without understanding the engine's internals. 

The work is available in the main branch of Soufflé and documented in the [Auto-Tuning](/autotuning) section of the Soufflé wiki. 

## Abstract 
{% include callout.html content="

Datalog has grown in popularity as a domain-specific language (DSL) for real-world
applications. Crucial to its resurgence has been the advent of high-performance Datalog
compilers, including Soufflé. Yet this high performance is unobtainable for users unless
they provide performance hints such as join orders for rules.

In this paper, we develop a join optimizer for Soufflé that automatically computes
high-quality join orders using a feedback-directed optimization strategy:
In a profiling stage, the compiler obtains join size estimates, and in a join
ordering stage, an offline join optimizer derives cost-optimal join orders. The
performance of the automatically optimized joins is demonstrated using complex
real-world applications, including DOOP, DDISASM, and VPC, surpassing the performance
of un-tuned join orders by a geometric mean speedup of 12.07×.
"  type="primary"%} 

{% include links.html %}
