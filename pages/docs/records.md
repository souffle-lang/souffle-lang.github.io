---
title: Records
permalink: /records
sidebar: docs_sidebar
folder: docs
---

# Record Types

In classical Datalog, a relation is a two-dimensional data-structure consisting of a set of rows and fixed number of columns. 
With a large code-base and/or a complex problem, it is convenient having more complex data-structures such as linked lists, trees, etc. 
Souffle provides such an abstraction, in form of records.  The intuition of records is similar to Pascal/C.
The syntax of a record-type definition is as follows:
```prolog
.type <name> = [ <name_1>: <type_1>, ..., <name_k>: <type_k> ]
```

Record types are combining several values into a structured value similar to a `struct` in C or a record in Pascal. In logical languages it is also known as a functor/term; and a constructor in functional languages. Record types permit to hold complex data in tuple elements. A record type may rely on other record types and allow recursive type declarations. 

An example record definition is given by
```
.type Connection = [
    from : Place,
    to : Place
]
```
defining values of ordered pairs of places. Each record type enumerates a list of nested, named fields and their types. Records may be nested as in
```
.type Cargo = [
    flight : Connection,
    mass : weight
]
```
as well as recursive, as in
```
.type Path = [
    first : Connection,
    rest : Path
]
```
Thus, a record may contain (directly or indirectly) fields of its own type. As a base case for recursive records, every record type contains the value `nil`, so we can represent lists as
```
.type List = [
    head : number,
    tail : List
]
.decl A(x : List)
A(nil).
A([1,nil]).
A([2,[3,nil]]).
.output A
```
Note that in rules, we use the bracket notation to describe a record as an argument. 

## I/O 

Record types are supported by the I/O system. You can use ```.input``` and ```.output``` directives
to print tuples of relations with record elements. If a records contains another record, 
the whole record with all its composed records will be either read or written. 

## Implementation
In the implementation, elements of a record are translated to a number.
We records with the same arity are stored in the same record pool and 
share the name number space.  During evaluation, if a record does not exist, 
it is created on the fly. Considering the example as above:
```prolog
.type Pair = [a: number, b: number]
.decl A(p: Pair)
A([1,2]).
A([3,4]).
A([4,5]).
.output A
```
Internally, these records are represented and stored as shown in this diagram:
<img src="img/record_table.png" alt="Record table example">

#### Recursive records
Recursively-defined records are allowed in Soufflé.
The recursion is terminated by the existence of a ``nil`` record.
Consider the following:
```prolog
.type IntList = [next: IntList, x: number]
.decl L(l: IntList)
L([nil,10]).
L([r1,x+10]) :- L(r1), r1=[r2,x], x < 30.
.decl Flatten(x: number)
Flatten(x) :- L([_,x]).
.output Flatten
```
Here, an ``IntList`` contains a reference to the next element, which is an ``IntList`` itself.
Internally, this is represented as follows:
<img src="img/record_recursive.png" alt="Record table with recursion example">

## Limitations
In the current verion,  union types involving records and sub-typing of record types are currently not supported.

