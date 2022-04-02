---
title: Program
permalink: /program
sidebar: docs_sidebar
folder: docs
---

[Datalog](https://en.wikipedia.org/wiki/Datalog) is a declarative programming language that was
introduced as a query language for deductive databases in the late 70s. Datalog uses first-order 
predicate logic to express computations in the form of Horn clauses.
Datalog adapts the syntax style of [Prolog](https://en.wikipedia.org/wiki/Prolog).
A full exposition is beyond the scope of this manual -- more details are available in the book [Foundations of Databases by Abiteboul, Hull and Vianu](https://wiki.epfl.ch/provenance2011/documents/foundations%20of%20databases-abiteboul-1995.pdf), or [tutorial material](http://blogs.evergreen.edu/sosw/files/2014/04/Green-Vol5-DBS-017.pdf).

## Language

The main language elements in Soufflé are relation declarations, facts, rules, and directives. For example,
the following program,
```prolog
.decl A(x:number, y:number)  // declaration of relation A
A(1,2).                      // facts of relation A
A(2,3).

.decl B(x:number, y:number)  // declaration of relation B
B(x,y) :- A(x,y).            // rules of relation B
B(x,z) :- A(x,y), B(y,z).

.output B                    // Output relation B 
```
has two relations `A` and `B`.  Relations must be declare 
so that the use of attributes can be checked at compiletime. 
In the example, relation ```A``` has two facts: ```A(1,2).``` and ```A(2,3)```.
A fact is a rule that holds unconditionally, i.e., a fact is a Horn Clause  ```A(1,2) ⇐ true```.
Relation ```B``` has two rules, i.e., ```B(x,y) :- A(x,y).``` and ```B(x,y) :- A(x,y), B(y,z).```
representing the Horn clause ```B(x,y) ⇐ A(x,y)``` and ```B(x,y) ⇐ A(x,y), B(y,z)```.

The directive ```.output B``` queries the relation ```B``` at the end of the execution and writes 
its result either into a file or prints it on the screen. The programmer can choose the order of 
the relation declarations,  facts, rules, and directives in the source-code arbitrarly. 

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
named ``y``. Attributes have a type. In the above example, 
the type of attribute ``x`` and ``y`` is a number.

The type-checker of Soufflé will infer the type of variables in 
rules and check their correct use. 

## [Types](types)

Soufflé utilises a typed Datalog dialect to conduct static checks enabling the early detection of errors in Datalog query specifications. 
Each attribute of involved relations has to be typed. Based on those, Soufflé attempts to deduce a type for all terms within all the Horn clauses within a given Datalog program. In case no type can be deduced for some terms, a typing error is reported -- indicating a likely inconsistency in the query specification.

There are four primitive types in Soufflé, i.e., `symbol`, `number`, `unsigned`, and `float`. 

## [Rules](rules)

Rules are conditional logic statements. A rule starts with a head followed by a body. For example,
```
A(x,y) :- B(x,y).
```
holds for a pair (x,y) if it is in B.

The performance of rules can be automatically improved by using Soufflé's [auto-scheduler](autotuning).

Note that to achieve optimal performance of rules, [hand-tuning](handtuning) may be required.

## [Components](components) 

Soufflé has [components](components) to modularise large logic programs. A component may contain other components, relation and type declarations, 
facts, rules, and directives. A programmer can declare and instantiate components. Each component has its own name space to access its elements.
Components can have one or more super-components from which they can inherit.

## [User-Defined Functors](functors)

Programmers can declare user-defined functors for extending Soufflé. User-defined functors are implemented in C/C++. 
There are two flavours of the user-defined functors, i.e., naive and stateful functors. 
Stateful functors expose [record and symbol tables](implementation). 

## [Pragma](pragmas)

Pragmas configure Soufflé. For example, command-line options can be set in the source code. 

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
