---
title: Types
permalink: /types
sidebar: docs_sidebar
folder: docs
---
Soufflé is a statically typed language. The main purpose of Souffle's 
type system is the correct usage of relations and records in rules.  
The type system defines types for relation attributes and records
and is enforced at compiletime. Static types have the advantage 
that they do not have runtime overheads while evaluating a 
logic program. 

Our type system is based on the idea of sets. A type can be 
either the universe of all possible values of a primitive type, 
or a subset of it; subsets can be composed of other subsets.

## Primitive Types
Soufflé has two primitive types:
* Symbol type: `symbol`
* Signed number type: `number`
* Unsigned number type: `unsigned`
* Float number type: `float`

The wordsize of a primitive type is 32 bit. The word-size can be 
changed to 64 bit by appropriately configuring Soufflé using
the configuration option ```--enable-64bit-domain``` in the
```configure``` script. 

### Symbol Type
The symbol type consists of the universe of all strings.
Internally, the symbol type is represented by an ordinal number.
The ordinal number for a symbol can be determined by using the `ord` functor, e.g., 
`ord("hello")` represents the ordinal number for `"hello"`.

### Number Type
The number type consists of the universe of all numbers.
The accepted range of numbers is given by the two-complement 
scheme of the word-size. 

### Unsigned Type
The number type consists of the universe of all positive numbers including zero.
The range is given by the word-size. 

### Float Type
The number type consists of the universe of floating point numbers. 
The precision is given by the wordsize, i.e., 32 bit or 64 bit. 

### Primitive Type Example
```
.decl Name(n: symbol)
Name("Hans").
Name("Gretl").

.decl Translate(n: symbol , o: number)
.output Translate
Translate(x,ord(x)) :- Name(x).

.decl Magic(x:number, y:unsigned, z:float)
Magic(-1,1,2.718).
.output Magic

```

## Equivalence Types 

You can define equivalence types in Soufflé using the directive ```.type <name> = <other-type>```
defining a new type ```<name>``` that is equivalent to type ```<other-type>```.  For example, 
```
.type myNumber = number
```
introduces the new type ```myNumber``` that is equivalent to the primitive type ```number```.

## Ontologies with Base and Union Types
The sole usage of primitive types (and equivalent types) is 
hazardous for large projects.
Binding wrong attributes via name equivalences is a 
common mistake writing large logic code bases. 
To avoid wrong bindings,  
base types and union types have been introduced. Base types are subtypes 
of primitive types and union types permit merging 
several base types. With base and union types, 
partial orders over subsets permit the formation of 
type ontologies.

### Base Type
Base types are defined by the directive ```.type <name> <: <primitive-type>```  where
```<name>``` is the name of the base type and ```<primitive-type>``` is one of the
four primitive types. For example, 
```
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
```
defines base type ```City```, ```Town```, and ```Village```. Although 
all three types are derived from the same primitive type ```symbol```, they are 
distinct sets of symbols from the universe of possible symbols. 

![location types as a subset of the universe](https://souffle-lang.github.io/img/universe_symbol_base.svg)


### Union Type
The Union type unifies types including base and union types, as long as all are derived from the same primitive type. The union type declaration is given below
```
.type <ident> = <ident-1> | <ident-2> | ... | <ident-k>
```
where ```<ident>``` is the name of the union type and ```<ident-...>``` referes to the base and other union types used in the union type.  For example, the union type ```Place```
```
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
.type Place = City | Town | Village
```
is the union of base types ```City```, ```Town```, and ```Village```. 
The union type ```PostalCode```,
```
.type PostCodes <: number
.type ZipCodes <: number
.type PostalCode = PostCodes | ZipCodes
```
unifies the base types ```Postcodess``` and ```ZipCodes```.

However, the following union type is not allowed
```
.type Weekdays <: symbol 
.type Dates <: number
.type Days = Weekdays | Dates // error
```
and while result in a compile error.

![Place as a union of the location types in the universe of symbols](https://souffle-lang.github.io/img/universe_symbol_place.svg)

We can bring these together to define attributes that better describe a relation, e.g.,
```
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
.type Place = City | Town | Village
.decl Data(c:City, t:Town, v:Village)
Data(“Sydney”, ”Ballina”, “Glenrowan”).

.decl Location(p:Place)
.output Location
Location(p) :- Data(p,_,_); Data(_,p,_); Data(_,_,p).
```

## Pitfalls 

As an example,  consider the following code fragment:
```
.type even = number
.type odd = number
.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X)
```
In this example, the types `even` and `odd` are defined as equivalence types for `number`. Thus, they are effectively equivalent and no typing error will result from the rule stated for relation `A`, which copies odd numbers into an even domain. 

When the aim of defining the two types `even` and `odd` was to enforce that those are two disjoint sets of integer values, the rule stated for `A` should trigger a type clash.

The above example will prodcue a type clash
```
.type even <: number 
.type odd <: number

.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X) // type clash
```
In this example, `even` and `odd` are defined as two disjoint sub-types of the type number, each exhibiting its own domain of values. From this definition Soufflé can deduce that the users intention was to define two disjoint sets and will report an error for the rule in the last line. For logic programmers it is crucial to accurately model the categories of values in form of a type system -- as has been illustrated by the example above.

## Legacy Syntax

In older versions of Soufflé we used 
```
.number_type Even
.symbol_type Place
.type Town
``` 
to define base types that are equivalent to 
```
.type Even <: number
.type Place <: symbol
.type Town <: symbol
```
in recent versions.
