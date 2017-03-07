---
layout: docs
title: Type System
permalink: /docs/types2/
---
Soufflé's type system is static.
Types are used to define the attributes of a relation.
Static types have the advantage that they can be enforced at compile-time; evaluation speed is improved as no dynamic checks are needed.
Another advantage of the type system is that it supports the programmer by enforcing the correct use of relations.

The Type system is based on the idea of sets.
A Type can be either the universe of all possible values, or a subset.
Subsets can be composed of other subsets.

## Primitive Types
Soufflé has two primitive types:
* Symbol type: `symbol`
* Number type: `number`

### Symbol type
The symbol type consists of the universe of all strings.
Internally, the symbol type is represented by an ordinal number.
The ordinal number for a symbol can be determined by using the `ord` command, e.g., ord("hello") represents the ordinal number for "hello".

### Number type
The number type consists of the universe of all numbers
The accepted range of numbers is restricted by architecture.
Soufflé accepts 32 bit signed numbers.

### Primitive type usage
```
decl Name(n: symbol ) 
Name("Hans").
Name("Gretl").

.decl Translate(n: symbol , o: number )
.output Translate
Translate(x,ord(x)) :- Name(x).
```
## Beyond Primitive Types
Primitive types are insufficient for large projects.
There is a need for a way to ensure that the wrong attributes will not be bound.
Symbols of different types need to be differentiated.
The ability to form partial orders over subsets allows the formation of ontologies.

### Base Type
Symbol types for attributes are defined by the `.symbol_type` declarative, e.g.,
```
.symbol_type City
.symbol_type Town
.symbol_type Village
```
Here we have defined distinct sets of symbols from the universe of possible symbols.

![location types as a subset of the universe](/img/symbol_universe_base.png)
### Union Type
The Union type unifies a fixed number of symbol set types, of either base or union types.
```
.type <ident1> = <ident1> | <ident2> | ... | <identk>
```
For example,
```
.type Place = City | Town | Village
```
![Place as a union of the location types in the universe of symbols](/img/symbol_universe_place.png)

We can bring these together to define attributes that better describe a relation, e.g.,
```
.symbol_type City
.symbol_type Town
.symbol_type Village
.type Place = City | Town | Village
.decl Data(c:City, t:Town, v:Village) 
Data(“Sydney”, ”Ballina”, “Glenrowan”).

.decl Location(p:Place)
.output Location
Location(p) :- Data(p,_,_); Data(_,p,_); Data(_,_,p).
```
