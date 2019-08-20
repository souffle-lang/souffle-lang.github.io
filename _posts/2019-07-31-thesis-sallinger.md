---
sidebar: news_sidebar
title: 'Provenance Tracking for Static Analysis with Datalog'
date: 2019-07-31
author: b-scholz
categories: [thesis]
permalink: sallinger19.html
layout: post
---

Sarah has just finished her MPhil thesis on provenance in Datalog. 
She improved David's provenance system by using a neat trick:
storing the arguments of the max-calculation to speed up 
the proof-generation time. Well done Sarah!

You find her thesis [Provenance Tracking for Static Analysis
with Datalog](/pdf/thesis_sallinger.pdf) here.  

## Abstract 
Logic programming languages such as Datalog are gaining popularity for industrial static program
analysis. This rise in popularity is due to the ease of expressing analyses in a declarative
manner and to the availability of Datalog solvers that allow for performance characteristics
similar to those of hand crafted analyses.
A major challenge in using Datalog for program analysis is the generation of valuable information
about generated alarms to give useful feedback to the users. A first step towards
obtaining this information is the computation of provenance information for given analysis
alarms. The state-of-the-art Datalog engine Soufflé provides this functionality in a system
component that allows users to construct proof trees for arbitrary alarms.
Other than the Datalog evaluation itself, this component is not fine-tuned for performance
which results in unnecessarily long proof tree construction times. Soufflé’s proof tree constructionmechanism
relies on annotations that are added to the generated tuples during Datalog
evaluation, describing the height of the tuple’s proof tree.
This thesis introduces subtree-heights provenance, an alternative proof tree construction
mechanism that additionally annotates tuples with the heights of the proof trees of the tuples
that where used in the generation, i.e. the heights of the first level subtrees.
The alternative provenance mechanism was implemented in Soufflé and evaluated on a
set of different Datalog program analyses over real-world programs, showing significant reductions
of the proof tree computation times in all setups and reaching reductions of up to
80% in some setups.


{% include links.html %}

