---
title: Relations
permalink: /relations
sidebar: docs_sidebar
folder: docs
---

# Relation Representation in Souffle

Relations are represented using different data-structures in Souffle.
There is an default selection strategy that can be overwritten by users. 
The user can specify a data-structure via relation qualifiers. 

## BTree Relations 

Direct/Indirect indexing
```
.decl A(x:number, y:symbol) btree
```


## Brie Relations 

```
.decl A(x:number, y:symbol) brie
```

## Equivalence Relations

The following demo demonstrates the special relation `eqrel_fast` which is equivalent to the `eqrel_slow` relation, yet the former carries an upper bounded quadratic speedup & memory improvement over the latter.

```
.decl eqrel_fast(x : number, y : number) eqrel
eqrel_fast(a,b) :- rel1(a), rel2(b).


// this does the same but is slow
.decl eqrel_slow(x : number, y : number)
eqrel_slow(a,b) :- rel1(a), rel2(b).
eqrel_slow(a,a) :- rel1(a).             // reflexivity
eqrel_slow(b,a) :- eqrel_slow(a,b).     // symmetry
eqrel_slow(a,c) :- eqrel_slow(a,b), eqrel_slow(b, c).   // transitivity
```


## Auto Selection
Strategy ...

{% include links.html %}
