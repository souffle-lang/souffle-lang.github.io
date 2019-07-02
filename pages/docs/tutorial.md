---
title: Tutorial
permalink: /tutorial
sidebar: docs_sidebar
folder: docs
---

## Introduction to Datalog
### Overview
Datalog is a (declarative) logic-based query language, allowing the user to perform recursive queries.
It adopts syntax in the style of Prolog.
In its pure form, it is based on a decidable fragment of first-order logic (FIL).
Here, the universe -- the colleciton of elements by which computation can be performed within -- is finite, and functors are not permitted.
Applications of Datalog include program analysis, security, graph databases, and declarative networking.

### Syntax
Computations are represented in terms of Horn clauses.
TODO?

## Soufflé: The Language
### Motivation
There does not exist a unified standard for the formal specification of Datalog. Indeed, each implementation implicitly embodies its own specification. TODO: correct?

The syntax of Soufflé is inspired by implementations of Datalog, namely [bddbddb](http://bddbddb.sourceforge.net/) and [muZ in Z3](https://github.com/Z3Prover/z3/wiki).
A principle goal of the Soufflé project is speed, tailoring to multi-core servers with large amounts of memory.
With this in mind, Soufflé provides software engineering features (components, for example.) for large-scale logic-oriented programming
For practical usage, Soufflé extends Datalog to make it Turing-equivalent through arithmetic functors.
Although this may lead to termination issues, the increased expressiveness is useful TODO: eg?

### First example
Soufflé supports UNIX, FreeBSD, Mac OS X and Windows, and can be installed as per [these instructions](/install).

Soufflé is invoked by command line in the format ``souffle <flags> <program>.dl``, where ``<program>.dl`` is the input program that is to be evaluated.
The input fact directory is set with flag ``-F <dir>``, specifying the input directory for relations. The default directory is the current directory.
The output directory for relations is set with flag ``-D <dir>``, and also defaults to the current directory. Note that if ``<dir>`` is set to ``-``, the output will be written to stdout.

#### Transitive closure
TODO: definition of transitive closure
```prolog
.decl edge(n: symbol, m: symbol)
edge("a", "b"). /* facts of edge */
edge("b", "c").
edge("c", "b").
edge("c", "d").
.decl reachable (n: symbol, m: symbol)
.output reachable // output relation reachable
reachable(x, y):- edge(x, y). // base rule
reachable(x, z):- edge(x, y), reachable(y, z). // inductive rule
```

Exercises:
- Extend the previous code to add a new relation SCC(x, y), that is defined as: if node x reaches node y and node y reaches node x, then (x, y) is in SCC.
- Check whether a node is cyclic.
- Check whether the graph is acyclic.
- Omit the flag ``-D-`` - where is the output?

Solutions:
TODO

#### Same generation example
Given a tree (a directed acyclic graph with a particular root node), the goal is to find which nodes are on the same level.

Solution:
```prolog
.decl Parent(n: symbol, m: symbol)
Parent("d", "b"). Parent("e", "b"). Parent("f","c").
Parent("g", "c"). Parent("b", "a"). Parent("c","a").
.decl Person(n: symbol)
Person(x) :- Parent(x, _).
Person(x) :- Parent(_, x).
.decl SameGeneration (n: symbol, m: symbol)
SameGeneration(x, x):- Person(x).
SameGeneration(x, y):- Parent(x,p), SameGeneration(p,q), Parent(y,q).
.output SameGeneration
```

#### Data-flow analysis example
[Data-flow analysis](https://en.wikipedia.org/wiki/Data-flow_analysis) (DFA) aims to determine *static* properties of programs.
DFA is a unified theory, providing information for global analysis about a program.
DFA is based on [control flow graphs](https://en.wikipedia.org/wiki/Control-flow_graph) (CFGs), where analysis of the program is derived from properties of nodes and the graph.

An example is a reaching definition - that is, whether a definition of a variable reaches a given point in the program. Note that an assignment of a variable can directly affect the value at another point in the program.
To cope, we consider an unambiguous definition ``d`` of a variable ``v`` as defined as follows:
```
d: v = <expression>;
```
Then, a definition ``d`` of a variable ``v`` is considered to reach a statement ``u`` if all paths from ``d`` to ``u`` do not contain any unambiguous definition of ``v``. We note that functions can have side effects to variables even in the absence of explicit unambiguous definitions.

Example: reaching definition - TODO image

```prolog
// define control flow graph
// via the Edge relation
.decl Edge(n: symbol, m: symbol)
Edge("start", "b1"). 
Edge("b1", "b2"). 
Edge("b1", "b3"). 
Edge("b2", "b4").  
Edge("b3", "b4"). 
Edge("b4", "b1"). 
Edge("b4", "end").

// Generating Definitions
.decl GenDef(n: symbol, d:symbol) 
GenDef("b2", "d1"). 
GenDef("b4", "d2"). 

// Killing Definitions
.decl KillDef(n: symbol, d:symbol) 
KillDef("b4", "d1"). 
KillDef("b2", "d2"). 

// Reachable 
.decl Reachable(n: symbol, d:symbol) 
Reachable(u,d) :- GenDef(u,d). 
Reachable(v,d) :- Edge(u,v), Reachable(u,d), !KillDef(u,d). 

.output Reachable 
```

#### Remarks on Input and C-Preprocessor.
Like in C++, Soufflé uses two types of comments:
- Type 1: ``// this is a remark``
- Type 2: ``/* this is a remark as well */``

C preprocessor processes input from Soufflé. For example:
```
#include "myprog.dl"
#define MYPLUS(a,b) (a+b)
```

### Relations
#### Relation declaration
A relation must be declared in order to be able to be used. We briefly note that the order in which statements (TODO: ??) are written in Soufflé code does not affect correctness. The statement:
```prolog
.decl edge(a:symbol, b:symbol)
```
Declares the relation ``edge`` with two fields, ``a`` and ``b``, each of which are symbols (strings).

#### I/O Directives
Users can specify directives for loading input and writing output to a file:
- The input directive ``.input <relation-name>`` reads from the ``<relation-name>.facts``, which is assumed to be tab-separated by default.
### Type system for attributes
### Arithmetic expressions
### Aggregation
### Records
### Components
### Performance and profiling facilities


{% include links.html %}
