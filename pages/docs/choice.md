---
title: Choice Construct
permalink: /choice
sidebar: docs_sidebar
folder: docs
---

Souffle's choice construct impose one or more functional dependency constraints for a relation. With the choice
construct, worklist algorithms such as spanning trees, can be expressed in few lines. 

The following example,
```
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
The output is the following, 
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

The functional dependency constraint ```choice-domain u``` ensures that for each edge in the spanning tree there is at
most one in-coming edge for a node in the spanning tree. For example, if the relation `st` already contains the tuple
`(L5, L9)`, a subsequent insertion of tuple `(L7, L9)` will be ignored/rejected because the attribute value `L9` of attribute u
has been allocated by the previously inserted
tuple `(L5, L9)'. Note that the order of insertion cannot be controlled by the program. It is a non-deterministic
choice, which tuple will be inserted first. 

## Syntax

### Choice-Domain
A choice-domain imposes a functional dependency constraint. The functional dependency 
constraint is expressed by its domain only. A domain can be either a single attribute 
or a subset of attributes. The choice-domain is specified as an relational qualifier.

![Choice Domain](https://souffle-lang.github.io/img/choice_domain.svg)

```ebnf
choice_domain ::= ( 'choice-domain' ( IDENT | '(' IDENT ( ',' IDENT )* ')' ) ( ',' ( IDENT | '(' IDENT ( ',' IDENT )* ')' ) )* )?
```


{% include links.html %}
