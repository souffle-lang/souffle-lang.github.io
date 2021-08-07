---
title: Program
permalink: /program
sidebar: docs_sidebar
folder: docs
---

Datalog is a declarative programming language that was
introduced as a query language for deductive databases in the late 70s.
Areas including web semantics,
program analysis, and security analysis use Datalog as a domain specific
language. Datalog adapts the syntax style of [Prolog](https://en.wikipedia.org/wiki/Prolog).
A full exposition is beyond the scope of this manual -- more details are available in the book Foundations of Databases by Abiteboul,
Hull and Vianu, or [tutorial material](http://blogs.evergreen.edu/sosw/files/2014/04/Green-Vol5-DBS-017.pdf).
Datalog uses first-order predicate logic to express computations
in form of Horn clauses.

## Language

The main elements in Souffle are relations, facts, rules, and directives. For example,
the following program has two relations `A` and `B`. 
```prolog
.decl A(x:number, y:number)  // declaration of relation A
A(1,2).                      // facts of relation A
A(2,3).

.decl B(x:number, y:number)  // declaration of relation B
B(x,y) :- A(x,y).            // rules of relation B
B(x,z) :- A(x,y), B(y,z).

.output B
```
Relation ```A``` has two facts: ```A(1,2).``` and ```A(2,3)```.
A fact is a rule that holds unconditionally, i.e., a fact is a Horn Clause  ```A(1,2) ⇐ true```.
Relation ```B``` has two rules, i.e., ```B(x,y) :- A(x,y).``` and ```B(x,y) :- A(x,y), B(y,z).```.  

The directive ```.output B``` queries the relation ```B``` at the end of the execution and writes 
the result either into a file or prints it on the screen.
Several queries may show up in the program. For example, by adding the directive ```.output A``` 
the relation ```A``` is queried at the end of execution as well. The locations of the facts, 
rules, and queries (i.e., output directives) in the source code are irrelevant.

## [Relations](relations)

Soufflé requires the declaration of relations. A relation is a set
of ordered tuples (x<sub>1</sub>, ..., x<sub>k</sub>) where each
element x<sub>i</sub> is a member of a data domain defined 
by a type. In the previous example, the declaration

```
.decl A(x:number, y:number).
```

defines the relation ```A``` that contains pairs of numbers only.
The first attribute is named ``x`` and the second attribute is
named ``y``. Attributes have a type which is specified by an
identifier followed by a colon after the attribute name.
In the above example, the type is a number.

The type-checker of Soufflé will infer the type of variables in rules where name bindings in clauses are correct at compile time.
The details of the type system are covered below.

## [Types](types)

Souffle utilises a typed Datalog dialect to conduct static checks enabling the early detection of errors in Datalog query specifications. Each attribute of involved relations has to be typed. Based on those, Souffle attempts to deduce a type for all terms within all the Horn clauses within a given Datalog program. In case no type can be deduced for some terms, a typing error is reported -- indicating a likely inconsistency in the query specification.

There are four primitive types in Souffle, i.e., `symbol`, `number`, `unsigned`, and `float`. 

## [Rules](rules)

Rules are conditional logic statements. A rule starts with a head followed by a body. For example,
```
A(x,y) :- B(x,y).
```
holds for a pair (x,y) if it is in B.

Souffle may need [performance tuning](tuning) for tuples. 

## [Components](components) 

For larger projects, it is useful to arrange logic codes in [components](components).


## [User-Defined Functors](functors)

Souffle can be extended with user-defined functors, which are implemented in C/C++. There are two flavours of the user-defined functors, i.e., naive and stateful functors. Stateful functors expose [record and symbol tables](implementation). 

## [Pragma](pragmas)

Pragmas configure Souffle. For example, command-line options can be set in the source code. 

## Syntax 
In the following, we define a program more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

Soufflé utilises the same comment syntax as C/C++. Furthermore, all souffle programs are passed through the C pre-processor. As a consequence, e.g. `#include` pragmas may be utilized to organize Datalog input queries into several files and reuse common constructs within several programs. Also, constants may be defined utilizing the `#define` pragma.

Soufflé Identifiers follow the C++ naming convention, except that a question mark may appear anywhere.
- The identifier can only be composed of letters (lower or upper case), numbers, or the question mark and underscore characters. That means the name cannot contain whitespace, or any symbols other than underscores or question marks.
- The identifier must begin with a letter (lower or upper case), an underscore, or a question mark. It can not start with a number.


### Program

A program consists of [type declarations](types), [relation declarations](relations), [facts](facts), [rules](rules), [component declarations and instantiations](components), [user-defined functor declarations](functors), and [pragmas](pragmas).

![Program](https://souffle-lang.github.io/img/program.svg)

```ebnf
program  ::= 
 ( pragma | 
   functor_decl | 
   component_decl | 
   component_init | 
   directive | 
   rule | 
   fact | 
   relation_decl | 
   type_decl )*
```

{% include links.html %}
