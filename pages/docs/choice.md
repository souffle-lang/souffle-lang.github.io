---
title: Choice Construct
permalink: /choice
sidebar: docs_sidebar
folder: docs
---


Souffle's choice construct impose one or more functional dependency constraints for a relation. With the choice
construct, worklist algorithms such as spanning trees, can be expressed in few lines. 

A functional dependency `x -> y` on relation `R(x:number, y:number)` ensures that
each `x` in `R` will uniquely define a value of `y`.
For example, during the evaluation, if a set of tuples `{(1,1), (1,2), (1,3)}`
are about to be inserted into `R`, Souffle only chooses arbitrary one of them
(the first one in the evaluation order). Any subsequent functionally dependent
tuples will be ommitted.

Functional dependency is defined using the keyword `choice-domain` in a relation
declaration.  For the sake of brevity, our syntax omits the co-domain (i.e. the
right hand side of the arrow).  Therefore, a choice-domain `D` for a relation with
attribute set `X` implicitly defines a functional dependency of `D -> X \ D`.

The following example,
```prolog
.decl edge(v:symbol, u:symbol)
edge("L1","L2"). edge("L2","L10"). edge("L2","L3").
edge("L3","L4"). edge("L4","L8"). edge("L3","L6").
edge("L6","L8"). edge("L8","L2").

.decl st(v:symbol, u:symbol) choice-domain u
st("r","L1").
st(v,u) :- st(_, v), edge(v,u).
.output st
```
computes the spanning tree `st` from the graph relation `edge`.
The rule `st(v,u) :- st(_, v), edge(v,u)` states that, an edge 
`(u, v)` is in the spanning tree: if `v` is
reachable in the spanning tree and there is an edge between `u` and `v`.
The `choice-domain u` on `st` ensures that the value of `u` is unique, i.e.,
each node can be visited only once in the spanning tree, and 
there is at
most one in-coming edge for a node in the spanning tree. 
For example, if the relation `st` already contains the tuple
`(L5, L9)`, a subsequent insertion of tuple `(L7, L9)` will be 
ignored/rejected because the attribute value `L9` of attribute u
has been allocated by the previously inserted
tuple `(L5, L9)`. Note that the order of insertion cannot be controlled 
by the program. It is a non-deterministic
choice, which tuple will be inserted first. 

The output of the example is the following, 
```
---------------
st
v	u
===============
root	L1
L1	L2
L2	L10
L2	L3
L3	L4
L4	L8
L3	L6
===============
```

Here is another example,
```prolog
.decl student (s:symbol, majr:symbol, year:number)
.decl professor (s:symbol, majr:symbol)
.decl advisor (s:symbol, year:number, p:symbol) choice-domain (s, year) // functional dependency: (s,year) -> p

advisor(s, y, p) :- student(s, y, m), professor(p, m).
```
which allocates an advisor for each student. The `choice-domain (s, year)`
on `advisor` makes sure that the combination of `(student, year)` is unique,
i.e., student from each year is assigned to a professor only once.

In the following example,
```prolog
.decl d(x:symbol)
.decl list(prev:symbol, next:symbol) choice-domain prev, next // functional dependencies: prev -> next & next -> prev

list(p, n) :- d(n), list(_, p).
```
an unordered set of elements is ordered using a list.
The program effectively computes a total order over the set.
The rule states that, `p` is before `e` if `p` is already assigned into the total order list.
With the help of the two choice-domains `prev` and `next`:
`prev` makes sure that for each element, there can only be one unique element after
it; similarly, `next` makes sure for each element, there can be only one unique
element before it. Those constraints defines the property of a total order set.


## Publication

 * Xiaowen Hu, Joshua Karp, David Zhao, Abdul Zreika, Xi Wu, Bernhard Scholz:
The Choice Construct in the Souffle Language. APLAS 2021: 161-181; ([link](https://link.springer.com/chapter/10.1007/978-3-030-89051-3_10)).

## Syntax

### Choice-Domain
The choice-construct is expressed as a relational qualifier, and is expressed
by a choice-domain, which imposes one or more functional dependency constraints. 
A functional dependency constraint is expressed by its domain only. 
A domain can be either a single attribute 
or a subset of attributes. The choice-domain is specified as an relational qualifier.

![Choice Domain](https://souffle-lang.github.io/img/choice_domain.svg)

```ebnf
choice_domain ::= ( 'choice-domain' ( IDENT | '(' IDENT ( ',' IDENT )* ')' ) ( ',' ( IDENT | '(' IDENT ( ',' IDENT )* ')' ) )* )?
```

{% include links.html %}
