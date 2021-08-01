---
title: Types
permalink: /types
sidebar: docs_sidebar
folder: docs
---
Soufflé is a statically typed language. The primary purpose of Soufflé’s type system is to help programmers writing correct rules. 
The proper use of rules becomes more complex when writing large software systems in Soufflé with hundreds of rules and relations. 
The type system assists the programmer to define types for relation attributes, whose usage in rules is checked statically. 
Static typing has the advantage that it does not have runtime overheads while evaluating a logic program.

## Primitive Types

Soufflé has four primitive types:
* Symbol type: `symbol`
* Signed number type: `number`
* Unsigned number type: `unsigned`
* Float number type: `float`

The word size of a primitive type is 32 bits. The word size can be
changed to 64 bits by appropriately configuring Soufflé using
the configuration option (see [Build Soufflé](build)).

### Symbol Type
The symbol type consists of the universe of all strings.
Internally, the symbol type is represented by an 
ordinal number. The ordinal number for a symbol can 
be determined by using the `ord` functor, e.g.,
`ord("hello")` represents the ordinal number 
for `"hello"`. In an internal symbol table, the 
ordinal number is converted to a string
(and vice versa).

### Number Type
The number type consists of the universe of all numbers.
The accepted range of numbers is given by the two's complement
scheme of the word size.

### Unsigned Type
The unsigned type consists of the universe of all non-negative numbers.
The range is given by the word size.

### Float Type
The float type consists of the universe of floating point numbers.
The precision is given by the word size, i.e., 32 bit or 64 bit.

### Example
The following example shows the use of the primitive types:
```prolog
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
Relation `Name` is a set of symbols; relation `Translate` has two attributes `n` and `o` of type `symbol` and `number`, respectively; relation `Magic`'s attributes are of type `number`, `unsigned`, and `float`. 

## Equivalence Types
You can define equivalence types in Soufflé using the directive `.type <name> = <other-type>`
defining a user-defined type `<name>` that is equivalent to type `<other-type>`.  For example,
```prolog
.type myNumber = number
```
introduces the user-defined type `myNumber` that is equivalent to the primitive type `number`.
The types `myNumber` and `number` can be synomiously be used. 

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
Base types are defined by the directive `.type <name> <: <primitive-type>`  where
`<name>` is the name of the base type and `<primitive-type>` is one of the
four primitive types. For example,
```prolog
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
```
defines base type `City`, `Town`, and `Village`. Although all three types are
derived from the same primitive type `symbol`, they are distinct sets of symbols
from the universe of possible symbols.

![location types as a subset of the universe](https://souffle-lang.github.io/img/universe_symbol_base.svg)


### Union Type
The Union type unifies types including base and union types, as long as all are derived from the same primitive type. The union type declaration is given below
```prolog
.type <ident> = <ident-1> | <ident-2> | ... | <ident-k>
```
where `<ident>` is the name of the union type and `<ident-...>` refers to the base and other union types used in the union type.  For example, the union type `Place`
```prolog
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
.type Place = City | Town | Village
```
is the union of base types `City`, `Town`, and `Village`.
The union type `PostalCode`,
```prolog
.type PostCodes <: number
.type ZipCodes <: number
.type PostalCode = PostCodes | ZipCodes
```
unifies the base types `PostCodes` and `ZipCodes`.

However, the following union type is not allowed
```prolog
.type Weekdays <: symbol
.type Dates <: number
.type Days = Weekdays | Dates // error
```
and will result in a compile error, because `Weekdays` and `Dates` are not derived from the same primitive types.

![Place as a union of the location types in the universe of symbols](https://souffle-lang.github.io/img/universe_symbol_place.svg)

We can bring these together to define attributes that better describe a relation, e.g.,
```prolog
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

There can be some pitfalls. As an example,  consider the following code fragment:
```prolog
.type even = number
.type odd = number
.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X).
```
In this example, the types `even` and `odd` are defined as equivalence types for `number`. Thus, they are effectively equivalent to type `number`, and no typing error will result from the rule stated for relation `A`, which copies odd numbers into an even domain.

When the aim of defining the two types `even` and `odd` was to enforce that those are two disjoint sets of integer values, the rule stated for `A` should trigger a type clash.

The above example will correctly produce a type clash 
```prolog
.type even <: number
.type odd <: number

.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X). // type clash
```
In this example, `even` and `odd` are defined as two disjoint sub-types of the type number, each exhibiting its own domain of values. From this definition Soufflé can deduce that the users intention was to define two disjoint sets and will report an error for the rule in the last line. For logic programmers it is crucial to accurately model the categories of values in form of a type system -- as has been illustrated by the example above.

## Record Types
In classical Datalog, a relation is a two-dimensional data-structure consisting of a set of rows and fixed number of columns. 
With a large code-base and/or a complex problem, it is convenient having more complex data-structures, including linked lists, trees in form of terms. 
Souffle provides such an abstraction, which are typed records.  
The intuition of records is similar to Pascal/C. 

The syntax of a record-type definition is as follows:
```prolog
.type <name> = [ <name_1>: <type_1>, ..., <name_k>: <type_k> ]
```

Record types are combining several values into a structured value similar to a `struct` in C or a record in Pascal. In logical languages it is also known as a functor/term; and a constructor in functional languages. Record types permit to hold complex data in tuple elements. A record type may rely on other record types and allow recursive type declarations. 

An example record definition is given by
```prolog
.type Connection = [
    from : Place,
    to : Place
]
```
defining values of ordered pairs of places. Each record type enumerates a list of nested, named fields and their types. Records may be nested as in
```prolog
.type Cargo = [
    flight : Connection,
    mass : weight
]
```
as well as recursive, as in
```prolog
.type Path = [
    first : Connection,
    rest : Path
]
```
Thus, a record may contain (directly or indirectly) fields of its own type. As a base case for recursive records, every record type contains the value `nil`, so we can represent lists as
```prolog
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

Record types are supported by the I/O system. You can use ```.input``` and ```.output``` directives
to print tuples of relations with record elements. If a records contains another record, 
the whole record with all its composed records will be either read or written. 
Note that union types involving records and sub-typing of record types are currently not supported in Souffle.


##  Algebraic Data Types (ADT)
Alebraic Data Types are extended versions of records, which permit several type shapes for a given record type. 

Examples:
```prolog
.type T = A { x: number }
        | B { x: symbol }

.type Nat = S {x : Nat}
          | Zero {}

.type Expression = Number { x : number }
                 | Variable { v : symbol}
                 | Add {e_1 : Expression, e_2 :Expression}
                 | Minus {e_1 : Expression, e_2 : Expression}
                 | Mult {e_1 : Expression, e_2 : Expression}
                 | Divide {e_1 : Expression, e_2 : Expression}

.type Tree = Empty {}
           | Node {t1: Tree, val: unsigned, t2: Tree}
```
Branches are initialized by `$branch_constructor(args...)`, and in case of a branch without arguments `$branch_constructor`.

For example, shows how to use branches in rules:

```prolog
.type Expression = Number { x : number }
                 | Variable { v : symbol}
                 | Add {e_1 : Expression, e_2 :Expression}
                 | Imaginary {}

.decl A(x:Expression)
A($Number(10)).
A($Add($Number(10),$Imaginary)).
A($Add($Number(10), $Variable("x"))).
A($Number(x+1)) :- A($Number(x)), x < 20.

.output A
```

Note that a constructor can only be used for an ADT once. For example, 
the following type definition is illegal:
```prolog
.type A = Number { x:number }
        | Symbol { v:symbol }
.type B = Number { x:number }
        | Symbol { v:symbol }
```
Since the constructors `Number` and `Symbol` show up twice in ADT `A` and ADT `B`. 

## Type Conversion
Soufflé permits type conversion with the as functor that takes as a first argument an functor expression and as a second argument the new type of the expression. 

For example, 
```prolog
.type Variable <: symbol
.type StackIndex <: symbol
.type VariableOrStackIndex = Variable | StackIndex

.decl A(a: VariableOrStackIndex)

.decl B(a: Variable)

B(as(a, Variable)) :- A(a).
```

Converts the expression `as(a, Variable)` to an expression of type `Variable` although `a` is of type `VariableOrStackIndex`. 

## Syntax 

In the following, we define type declarations in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced using [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Type Declaration 
 
A type declaration binds a name with a new type. The type is either a subtype, an equivalence/union type, a record type, of an ADT.

![Type Declaration](https://souffle-lang.github.io/img/type_decl.svg)
```
type_decl ::= TYPE IDENT ("<:" type_name | "=" ( type_name ( "|" type_name )* | record_list | adt_branch ( "|" adt_branch )* ))
```

### Type Name 

Souffle has pre-defined types such as `number`, `symbol`, `unsigned`, and `float`. Used-defined types have a name. If a type has been defined in a component, the type can be still accessed outside the component using a qualified name. 

![Type Name](https://souffle-lang.github.io/img/type_name.svg)
```
type_name ::=  "number" | "symbol" |"unsigned" | "float"  | IDENT ("." IDENT )*
```

### Record Declaration 

A record declaration consists of a list of attributes. Record declarations can be recursive.

![Record Declaration](https://souffle-lang.github.io/img/record_list.svg)
```
record_list ::= "[" attribute ( "," attribute)* "]"
```

### ADT Declaration

An ADT consists of a several ADT branches. An ADT branch consists of a list of attributes which 
is associated with a branch identifier. The branch identifier must be unique in Souffle.

![ADT Branch](https://souffle-lang.github.io/img/adt_branch.svg)
```
adt_branch ::= IDENT "{" attribute ( "," attribute)* "}"
```

### Attribute Declaration

An attribute binds a name with a type. 

![Attribute](https://souffle-lang.github.io/img/attribute.svg)

```
attribute ::= IDENT ":" type_name 
```

### Legacy Syntax

The syntax of Souffle changed over time. Older code bases can be still used with 
modern versions of Souffle.  In older versions of Soufflé we used
```prolog
.number_type Even
.symbol_type Place
.type Town
```
to define base types. These definitions should be rewritten to the new syntax listed below,
```
.type Even <: number
.type Place <: symbol
.type Town <: symbol
```
You can enable the old legacy syntax using the command-line flag `--legacy`, but you will receive a warning that this legacy syntax is deprecated.

# Implementation Details 

# Symbols 
TODO: Symbol Table 

# Records 
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

For the above example, an ``IntList`` contains a reference to the next element, which is an ``IntList`` itself.
Internally, this is represented as follows:
<img src="img/record_recursive.png" alt="Record table with recursion example">

# ADT 
TODO: Encoding
