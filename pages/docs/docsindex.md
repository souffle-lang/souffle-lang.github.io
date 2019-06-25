---
title: Welcome
permalink: docs.html
redirect_from: docs
toc: false
sidebar: docs_sidebar
folder: docs
---
Soufflé is a variant of Datalog for tool designers crafting static analyses. 
Soufflé provides the ability to rapid prototype and make deep design space explorations possible.
Applications include the points-to analysis framework [DOOP](https://bitbucket.org/yanniss/doop), [security analyses for large-scale problems](https://labs.oracle.com/pls/apex/f?p=labs:49:::::P49_PROJECT_ID:122), parallelizing compiler framework [Insieme](http://www.insieme-compiler.org), security analysis for cloud computing, and security analysis for smart contracts [VANDAL](https://github.com/usyd-blockchain/vandal).
Soufflé commenced at [Oracle Labs in Brisbane](https://github.com/oracle/souffle/wiki/Contributors), and was open-sourced in March 2016. It is actively supported by universities and industrial research labs including [The University of Sydney](http://sydney.edu.au), the [University of Innsbruck](https://www.uibk.ac.at/index.html.en), the [University College London](https://www.ucl.ac.uk), and the [University of Athens](http://www.di.uoa.gr/).

One of the major challenges in logic programming is scalability. 
Soufflé applies advanced compilation techniques for logic programs over finite domains including semi-naïve evaluation, Futamura Projections, staged-compilation with a new abstract machine, partial evaluation, and automated parallelisation for achieving high performance.

Soufflé has been designed such that, 

* declarative rules are efficiently translated to imperative programs on modern computer hardware, including multi-core computers, and

* the domain specific language extensions of Soufflé support the tool designer crafting static program analyses.

# Why the name Soufflé?
Soufflé  is short for **Systematic, Ontological, Undiscovered Fact Finding Logic Engine**. The EDB represents the
uncooked Soufflé  and the IDB causes the Soufflé  to rise, i.e., monotonically increasing knowledge. When it stops rising and a fixed-point is reached, the result is a puffed-up ready-to-eat Soufflé. Big thanks to Nicholas Allen and Diane Corney from [Oracle Labs/Brisbane](https://labs.oracle.com/pls/apex/f?p=labs:23:::::P23_LOCATION_ID:46) for finding a translation :+1:.


## Improving Documentation

If there are errors, and/or explanations in the documentation can be improved, please let us know.
You can either click on the ```Improve this page``` button at the top right-hand side, and trigger a pull request for your improved documentation, or please [file an issue]({{ site.repository }}/issues/new).

{% include links.html %}
