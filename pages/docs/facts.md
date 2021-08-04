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

## Syntax 
In the following, we define facts in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced using [Bottlecaps](https://www.bottlecaps.de/rr/ui).


![Qualifier Name](https://souffle-lang.github.io/img/qualified_name.svg)

A qualified name is a sequence of identifiers separated by `.` to disambiguate relations that are instantiated by components.

```ebnf
qualified_name ::= IDENT ( '.' IDENT )*
```

### Atom

An atom is a predicate name followed by its arguments. The arguments must be constants and for nullary predicates there are no arguments. 

![Atom](https://souffle-lang.github.io/img/atom.svg)

```ebnf
atom ::= qualified_name '(' ( argument ( ',' argument )* )? ')'
```

### Fact

A fact is an atom followed by a dot.

![Fact](https://souffle-lang.github.io/img/fact.svg)

```ebnf
fact ::= atom '.'
```

{% include links.html %}
