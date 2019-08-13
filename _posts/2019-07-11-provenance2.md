---
sidebar: news_sidebar
title: 'Provenance for Large-scale Datalog'
date: 2019-07-11
author: b-scholz
categories: [paper]
permalink: provenance2.html
layout: post
---
This is the [Provenance for Large-scale Datalog](/pdf/provenance-tr.pdf) paper describing how to implement a fast
provenance system in Datalog. Provenance is a mechanism to describe the existence of tuples (or their lack off it). 
It is on [Arxiv.org](https://arxiv.org/abs/1907.05045). The co-authors are 
David Zhao, Pavle Subotic, and Bernhard Scholz.

## Abstract 
Modern Datalog engines are employed in industrial applications 
such as graph databases, networks, and static program analysis. 
To cope with the vast amount of data in these 
applications, Datalog engines must employ specialized parallel 
data structures. In this paper, we introduce the Brie, a 
specialized data structure for high-density relations storing 
large data volumes. It effectively compresses dense data in a 
lock-free fashion and obtains up to 15× higher performance in 
parallel insertion benchmarks compared to state-of-the-art 
alternatives. Furthermore, when integrated into a Datalog 
engine running an industrial points-to analysis, runtime 
improves by a factor of 4× with a compression ratio of 
up to 3.6× are obtained.

{% include links.html %}

