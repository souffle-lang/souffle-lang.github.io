---
title: Relations
permalink: /relations
sidebar: docs_sidebar
folder: docs
---

# Relation Representation in Souffle

Relations can be represented using different internal data structures in Soufflé, each exhibiting different performance characteristics. By default, the B-tree is used to store tuples of a relation. However, this default selection can be overridden by users, by specifying a relation qualifier. Currently, the possible data structures are B-tree, Brie, and Eqrel (for equivalence relations).

## B-tree Relations 

The B-tree data structure is used by default, however the *direct* flavour (see below) can be forced by adding the `btree` qualifier to a relation declaration:
```
.decl A(x:number, y:symbol) btree
```

The B-tree is a good default data structure that performs reasonably well in general use cases. In Soufflé, two flavours of the B-tree are offered. The first is a *direct* version, where tuple data is stored directly in the B-tree. The second is an *indirect* version, where the B-tree stores pointers to an external table data structure which holds the actual tuple data.

By default, a relation without qualifiers will be *direct* if it has arity ≤ 6, and *indirect* if it has arity > 6. This choice is made because larger arity tuples take more space in the B-tree data structure, so cache coherence can be improved by using pointers instead.

Note, however, that adding the `btree` qualifier to the relation declaration forces the *direct* B-tree.

More details on the Soufflé B-tree can be found in [this paper](https://doi.org/10.1145/3293883.3295719).

## Brie Relations 

The Brie data structure is a specialised data structure designed for *dense* data. It can be used by adding the `brie` qualifier to a relation declaration.
```
.decl A(x:number, y:symbol) brie
```

The Brie data structure is a specialised form of a trie, or prefix tree. The benefits of using a prefix tree-style data structure for dense data can be illustrated as follows.

Consider a 3-arity relation containing tuples
```
(0,0,0), (0,0,1), (0,0,2)
```
In this case, all tuples share the common prefix `0, 0`, and differ only in the final digit. This data can be described as `dense`, since a common prefix reduces redundant stored information in the prefix tree. On the other hand, a relation containing
```
(0,0,0), (1,2,3), (4,5,6)
```
is not dense, and the prefix tree provides no benefit.

The Brie data structure in Soufflé similarly provides a performance benefit for highly dense data. However, it is slower than the B-tree in the average case, and so its use must be considered carefully.

More details on the Brie can be found in [this paper](https://doi.org/10.1145/3303084.3309490).

## Equivalence Relations

An equivalence relation is a special kind of binary relation which exhibits three properties: *reflexivity*, *symmetry*, and *transitivity*. An equivalence relation could be expressed in Datalog as follows:
```
.decl equivalence(x:number, y:number)
equivalence(a, b) :- rel1(a), rel2(b).      // every element of rel1 is equivalent to every element of rel2

equivalence(a, a) :- equivalence(a, _).     // reflexivity
equivalence(a, b) :- equivalence(b, a).     // symmetry
equivalence(a, c) :- equivalence(a, b), equivalence(b, c).  // transitivity
```

However, this expression of an equivalence relation would require a quadratic number of tuples to be stored. In Soufflé, the Eqrel data structure provides a linear representation of equivalence relations, by using a union-find based algorithm. Eqrel can be used by adding the `eqrel` qualifier to a relation declaration.

The following example demonstrates the use of Eqrel, and is semantically equivalent to the example above. However, using Eqrel carries at best a quadratic speed and memory improvement over the above example.
```
.decl eqrel_fast(x : number, y : number) eqrel
eqrel_fast(a,b) :- rel1(a), rel2(b).
```

More details on Eqrel can be found in [this paper](https://doi.org/10.1109/PACT.2019.00015).

## Nullaries

Nullary relations are special relations. Their arity is zero, i.e., they don't have attributes. 
They are defined as 
```
.decl A()
```
These relations are either empty or contain the empty tuple `()`. Internatlly, they are implemented 
as a boolean variable. 

Semantically, they can be seen as a proposition rather than a relation. 

## Auto Selection
In Soufflé, the default data structure is the B-tree, with the *direct* version for relations with arity ≤ 6, or the *indirect* version for relations with arity > 6.

{% include links.html %}
