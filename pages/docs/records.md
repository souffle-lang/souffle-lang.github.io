---
title: Records
permalink: /records
sidebar: docs_sidebar
folder: docs
---

* **Record Types** are combining several values into a structured value similar to a `struct` in C. An example record definition is given by
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
    head : number
    tail : List
]
.decl A(x : List)
A(nil).
A([1,nil]).
A([2,[3,nil]]).
```

Union types involving records are currently not supported.
