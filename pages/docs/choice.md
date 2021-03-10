---
title: Choice and non-deterministic algorithm
permalink: /choice
sidebar: docs_sidebar
folder: docs
toc: false
---

## Choice

Choice construct is now available in Souffle to support the notion of non-determinism.
With choice, it is now much easier to express worklist algorithm in Souffle by
specifying functional dependencies on the relations.

A functional dependency `x -> y` on relation `R(x:number, y:number)` means that
each `x` in `R` will uniquely define a value of `y`.
For example, during the computation, if a set of tuples `{(1,1), (1,2), (1,3)}`
are about to be inserted into `R`, Souffle only chooses arbitrary one of them
(hence the "non-determinism"), since inserting more than one would break the
functional dependency.

Functional dependency can be enforced on relation using the keyword `choice-domain` during relation
declaration in Souffle with the following syntax:

```
<relation-declaration>      ::= ".decl" <relation-name> "(" <attributes> ")" <choice-domain>
<choice-domain>            ::= "" | "choice-domain" (<constraint>)+
<constraint>               ::= <attribute> | "(" <attributes> ")"
```

Note here, for the sake of brevity, our syntax omits the co-domain (i.e. the right hand side of the arrow). 
Therefore, a choice-domain `D` for a relation with attribute set `X` implicitly defines a functional dependency of `D -> X \ D`.


## Examples

### Spanning Tree
```
.decl edge(u:symbol, v:symbol)
.decl st(u:symbol, v:symbol) choice-domain v
.decl startNode(x:symbol)

st("root", x) :- startNode(x).
st(u, v) :- st(_, v), edge(u, v).
```

The above program calculates a spanning tree on a connected component of an undirected graph.
The first rule simply chooses a start node.
The second rule states that, an edge `(u, v)` is in the spanning tree: if `v` is
reachable in the spanning tree and there is an edge between `u` and `v`.
The `choice-domain v` on `st` ensures that the value of `v` is unique, i.e.,
each node can be visited only once in the spanning tree.

### Eligible advisors
```
.decl student (s:symbol, majr:symbol, year:number)
.decl professor (s:symbol, majr:symbol)
.decl advisor (s:symbol, year:number, p:symbol) choice-domain (s, year)

advisor(s, y, p) :- student(s, y, m), professor(p, m).
```
The above program allocates an advisor for each student. The `choice-domain (s, year)`
on `advisor` makes sure that the combination of `(student, year)` is unique,
i.e., student from each year is assigned to a professor only once.

### Total order.
```
.decl d(x:symbol)
.decl list(prev:symbol, next:symbol) choice-domain prev, next

list(p, n) :- d(n), list(_, p).
```
Given an unordered set of elements, the above program assigns them into a list,
effectively computes a total order over the set.
The rule states that, `p` is before `e` if `p` is already assigned into the total order list.
With the help of the two choice-domains `prev` and `next`: 
`prev` makes sure that for each element, there can only be one unique element after
it; similarly, `next` makes sure for each element, there can be only one unique
element before it. Those constraints defines the property of a total order set.
