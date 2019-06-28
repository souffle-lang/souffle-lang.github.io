---
title: Synthesis
permalink: /translate
sidebar: docs_sidebar
folder: docs
---

Soufflé synthesizes a native parallel C++ program from a logic specification. 
The synthesis is performed by several applications of the [Futamura Projection](http://blog.sigfpe.com/2009/05/three-projections-of-doctor-futamura.html): The semi-naïve evaluation of Datalog programs becomes the interpreter, the IDB is the static program that is executed on the interpreter, and the EDB the input of the static program. Soufflé mixes the 
semi-naïve evaluation with the IDB resulting in a relational algebra program
that is imperative and relational. After further optimisations, the relational algebra program is transformed to a templatized C++ program applying 
another Futamura projections, again. 
As a result, we obtain a highly optimized parallel executable. 
Note that no persistent storage is  provided 
resulting in better performance than traditional 
relational database management systems.
The stages of the synthesis are shown below:
![Stages](/img/souffle-compiler.png)

Datalog program is transformed to an relational algebra machine applying a Futamura Projection on the semi-naïve evaluation scheme and the IDB. 
The relational algebra machine is an abstract machine, that can express the imperative execution of relational algebra operations and
fixed-point computations for the evaluation of Datalog programs. 
The resulting relational algebra machine program is 
further transformed to a C++ program applying another specialization / optimisation.  By lowering the
Datalog program to heavily templatized C++ code, computations can be moved from runtime to compile-time. 
This new compilation technique overcomes performance bottlenecks observed in traditional
Datalog implementations that are (1) implemented as a generic semi-naïve evaluation interpreter and (2) perform the relational algebra operations on disk using a relational database.

