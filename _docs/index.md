---
layout: docs
title: Welcome
permalink: /docs/home/
redirect_from: /docs/index.html
---
Soufflé is a variant of Datalog for tool designers crafting static analyses. 
Soufflé provides the ability to rapid prototype and make deep design space explorations possible.
Applications include points-to, taint, constant-propagation, and security analyses for large-scale problems.

One of the major challenges in logic programming is scalability. 
Souffé applies advanced compilation techniques for logic programs over finite domains including semi-naïve evaluation, Futamura Projections, staged-compilation with a new abstract machine, partial evaluation, and automated parallelisation for achieving high performance.

Soufflé has been to designed such that, 

* declarative rules are efficiently translated to imperative programs efficiently on modern computer hardware including multi-core computers, and

* the domain specific language extensions of Soufflé support the tool designer
crafting static program analyses.

# Why the name Soufflé?
Soufflé  is short for **Systematic, Ontological, Undiscovered Fact Finding Logic Engine**. The EDB represents the
uncooked Soufflé  and the IDB causes the Soufflé  to rise, i.e., monotonically increasing knowledge. When it stops rising and a fixed-point is reached, the result is a puffed-up ready-to-eat Soufflé. Big thanks to Nicholas Allen and Diane Corney from [Oracle Labs/Brisbane](https://labs.oracle.com/pls/apex/f?p=labs:23:::::P23_LOCATION_ID:46) for finding a translation :+1:.


## Improving Documentation

If there are errors, and/or explanations in the documentation can be 
improved, please let us know. You can either click on the 
```Improve this page``` button at the top right-hand side,  
and trigger a pull request for your improved documentation, or please 
[file an issue]({{ site.repository }}/issues/new).
