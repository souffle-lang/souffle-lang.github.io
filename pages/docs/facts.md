---
title: Facts
permalink: /facts
sidebar: docs_sidebar
folder: docs
---

Facts are clauses that unconditional hold; they are rules with a head,
but no rule body. In facts, all arguments must be constant terms. 

In the example,
```prolog
.decl A(x:number, y:number)  // declaration of relation A
A(1,2).                      // facts of relation A
A(2,3).
.output A
```
the relation `A` has two facts: `A(1,2).` and `A(2,3)`.  Note that facts can also be 
loaded with the [input directive](directives).

{% include links.html %}
