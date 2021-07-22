---
title: Welcome
permalink: docs.html
redirect_from: docs
toc: false
sidebar: docs_sidebar
folder: docs
---
Soufflé is a logic programming language inspired by Datalog. It overcomes some of the limitations in classical Datalog.
For example, programmers are not restricted to finite domains, and the usage of functors (intrinsic, user-defined, records/constructors, etc.) are permitted. Soufflé has a component model so that large logic projects can be 
expressed. Initially, Soufflé has been designed for crafting static analysis in logic at Oracle Labs. 
Recently, there are many other applications written in the Soufflé language, including applications in network analysis and data analytics. 

Soufflé provides the ability to rapid prototype and make deep design space explorations possible.
A wide range of applications have been implemented in the Soufflé language, e.g., [DOOP](https://bitbucket.org/yanniss/doop), security analyses for Java JDK at Oracle Labs, parallelizing compiler framework [Insieme](http://www.insieme-compiler.org), security analysis for cloud computing, and security analysis for smart contracts [VANDAL](https://github.com/usyd-blockchain/vandal), [Gigahorse](https://github.com/nevillegrech/gigahorse-toolchain), [Securify](https://github.com/eth-sri/securify), [Secuify V2.0](https://github.com/eth-sri/securify2), and binary disassembler [DDISASM](https://github.com/GrammaTech/ddisasm).  
Soufflé language project is led by [Bernhard Scholz](http://b-scholz.github.io), and commenced at [Oracle Labs in Brisbane](https://github.com/oracle/souffle/wiki/Contributors). Soufflé was open-sourced in March 2016. It is actively supported by universities and industrial research labs. The main contributors to this project have been [The University of Sydney](http://sydney.edu.au), the [University of Innsbruck](https://www.uibk.ac.at/index.html.en), the [University College London](https://www.ucl.ac.uk), the [University of Athens](http://www.di.uoa.gr/), [Oracle Labs, Brisbane](http://https://labs.oracle.com/). 

One of the major challenges in logic programming is scalability. 
Soufflé applies advanced compilation techniques for logic programs. We use a range of techniques to achieve high-performance: Futamura Projections, staged-compilation with a new abstract machine, partial evaluation, and parallelization with highly-parallel data-structures. 

Soufflé has been designed such that, 

* declarative rules are efficiently translated to efficient C++ programs on modern computer hardware, including multi-core computers, and

* the domain specific language extensions of Soufflé support the tool designer to structure projects effectively and give sufficient expressiveness to the users

# Why the name Soufflé?
Soufflé  is short for **Systematic, Ontological, Undiscovered Fact Finding Logic Engine**. The EDB represents the
uncooked Soufflé  and the IDB causes the Soufflé  to rise, i.e., monotonically increasing knowledge. When it stops rising and a fixed-point is reached, the result is a puffed-up ready-to-eat Soufflé. Big thanks to Nicholas Allen and Diane Corney from [Oracle Labs/Brisbane](https://labs.oracle.com/pls/apex/f?p=labs:23:::::P23_LOCATION_ID:46) for finding a translation.


## Improving Documentation

If there are errors, and/or explanations in the documentation can be improved, please let us know.
You can either click on the ```Edit me``` button at the top of the relevant page, and trigger a pull request for your improved documentation, or please [file an issue]({{ site.repository }}/issues/new).

{% include links.html %}
