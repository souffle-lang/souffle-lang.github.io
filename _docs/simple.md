---
layout: docs
title: A Simple Example
permalink: /docs/simple/
---
Create the following Datalog program and save it to `reachable.dl`.

```
.symbol_type Node

//
// Edge
//
.decl A(n:Node, m:Node)

Edge("n0","n1").
Edge("n1","n2").
Edge("n2","n3").
Edge("n1","n4").

//
// Reachable 
//

.decl Reachable(n:Node, m:Node) output

// Relation Edge is a subset of relation Reachable
Reachable(x,y) :- Edge(x,y).

// Reflexive rules
Reachable(x,x) :- Edge(x,_). 
Reachable(y,y) :- Edge(_,y). 

// Transitive rule
Reachable(x,y) :- Edge(x,z), Reachable(z,y).
```

The relation ```edge``` is defined by facts in the source code. 
The relation ```reachable``` computes the transitive closure of the relation ```edge```, inductively. The first rule ensures that all pairs in relation ```Edge``` are in the relation ```Reachable```. The second and third rules create reflexive edges, and the fourth rule creates transitive edges. Note that the underscore in the second and third rule refer to an unnamed variable, i.e., the actual variable value of an unnamed variable is ignored. 

With the command
```
souffle -D- reachable.dl
```
the program is executed and the result is printed out standard output. The output is


```
---------------
Reachable
===============
n0 n1
n1 n2
n2 n3
n1 n4
n0 n0
n1 n1
n2 n2
n3 n3
n4 n4
n0 n2
n0 n4
n1 n3
n0 n3
===============
```

The execution and command line options are described on the next page.

## [More Datalog Examples ...](https://github.com/souffle-lang/souffle/tree/master/tests/evaluation)


