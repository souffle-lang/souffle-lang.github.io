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
Internally, a symbol is represented by an ordinal number. 
The `ord` functor determines the ordinal number of a symbol, e.g., 
`ord("hello")` gives the ordinal number for the symbol `"hello"`
which can change from a run to another one.

### Number Type
The number type consists of the universe of all signed interger numbers.
The accepted range of numbers is given by the two's complement
scheme of the word size. The default range is fixed to the `int32_t` type covering the range [ -2^31 .. 2^31 -1 ]. However, the wordsize can be altered by a configuration flag in the configure script to `int64_t`.

### Unsigned Type
The unsigned type consists of the universe of all non-negative integer numbers.
The range is given by the word size. The default range is fixed to the `uint32_t` type covering the range [ 0 .. 2^32 ]. However, the wordsize can be altered by a configuration flag in the configure script to `uint64_t`.

### Float Type
The float type consists of the universe of floating point numbers in the [IEEE 754
Standard for Floating-Point Arithmetic](https://en.wikipedia.org/wiki/IEEE_754).
The precision of the floating point number type is given by the word size, i.e., 32 bit or 64 bit.

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
You can define equivalence types in Soufflé using the directive `.type <new-type> = <other-type>`
defining a user-defined type `<new-type>` that is equivalent to type `<other-type>`.  For example,
```prolog
.type myNumber = number
```
introduces the user-defined type `myNumber` that is equivalent to the primitive type `number`.
The types `myNumber` and `number` can be synomiously be used. 

## Subtypes 
You can define subtypes in Soufflé using the directive `.type <new-type> <: <other-type>`
defining a user-defined type `<new-type>` that is equivalent to type `<other-type>`. 
If type `<new-type>` is a subtype of `<other-type>`, any term/expression of type `<new-type>` 
can be safely used in  context, where a term of type `<other-type>` is expected, 
but not vice versa. For example,
```prolog
.type myEvenNumber <: number
```
introduces the user-defined type `myEvenNumber` that is a subtype of the primitive type `number`. You can define subtypes of subtypes, e.g., 
```prolog
.type myEvenNumber <: number
.type myMultiplesOfFour <: myEvenNumber
```
In Soufflé's type system, we assume that the a subtype is a strict subset of its supertype. For example, in the above example we assume that `myEvenNumber` is a strict subset of all values of type `number` (i.e. it is not equivalent), and we assume that `myMultipleOfFour` is a strict subset of type `myEvenNumber`.

## Ontologies with Base and Union Types
The sole usage of primitive types (and equivalent types) is
hazardous for large projects. Binding wrong attributes via variable name bindings in rules is a common mistake in writing large logic code bases. Soufflé uses base types in the form of subtypes and union types to avoid wrong bindings. 
Base types are subtypes, and union types permit merging several base types or other union types. With base and union types, partial orders over subsets allow the formation of
type ontologies.

### Base Type
Base types are subtypes of primitive types. For example,
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
The Union type unifies types including base and other union types. We assume that all unified types are derived from the same primitive type. The union type declaration is given below
```prolog
.type <ident> = <ident-1> | <ident-2> | ... | <ident-k>
```
where `<ident>` is the name of the user-defined union type and `<ident-...>` refers to the base and other union types used in the union type.  For example, the union type `Place`
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

In the following example, a union type is ill-defined,
```prolog
.type Weekdays <: symbol
.type Dates <: number
.type Days = Weekdays | Dates // error: unified types origin from different primitive types
```
because `Weekdays` and `Dates` do not orginiate from the same primitive types.

![Place as a union of the location types in the universe of symbols](https://souffle-lang.github.io/img/universe_symbol_place.svg)

In the following example, we show the use of a union type for a relation,
```prolog
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
.type Place = City | Town | Village
.decl Data(c:City, t:Town, v:Village)
Data("Sydney", "Ballina", "Glenrowan").

.decl Location(p:Place)
.output Location
Location(p) :- Data(p,_,_); Data(_,p,_); Data(_,_,p).
```
where the set `Location` can contain values from the subtypes `City`,`Town`, and `Village`.

There can be some pitfalls. As an example,  consider the following code fragment:
```prolog
.type even = number
.type odd = number
.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X). // silent error 
```
In this example, the types `even` and `odd` are defined as equivalence types for `number`. Thus, they are effectively equivalent to type `number`, and no typing error will result from the rule stated for relation `A`, which copies odd numbers into an even domain.

When the aim of defining the two types `even` and `odd` was to enforce that those are two disjoint sets of integer values, the rule stated for `A` should trigger a type clash.

The above example will correctly produce a type clash 
```prolog
.type even <: number
.type odd <: number

.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X). // error: type clash
```
In this example, `even` and `odd` are defined as two disjoint sub-types of the type number, each exhibiting its own domain of values. From this definition Soufflé can deduce that the users intention was to define two disjoint (non-overlapping) sets and will report an error for the rule in the last line. For logic programmers it is crucial to accurately model the categories of values in form of a type system -- as has been illustrated by the example above.

## Record Types

With a large code-base and/or a complex problem, it is convenient having more complex datatypes, including linked lists, trees, etc.  Soufflé provides such an abstraction, which are typed records. The intuition of records is similar to 
Pascal/C. In logical languages it is also known as a functor/term; and a constructor in functional languages:

A record-type is defined as follows,
```prolog
.type <new-record> = [ <name_1>: <type_1>, ..., <name_k>: <type_k> ]
```
where `<new-record>` is the name of the newly defined record. The record combines several values into a 
structured value.  Records permit to hold complex data in tuple elements. A record type may rely on other 
record types and allow recursive type declarations. 

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
Thus, a record may contain (directly or indirectly) fields of its own type. Every record type has the `nil` value. As a base case we can use the `nil` value to construct lists, e.g.,
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
Note that in rules, we use the bracket notation to describe a record as an argument. Consider another record example below,
```prolog
.type IntList = [next: IntList, x: number]
.decl L(l: IntList)
L([nil,10]).
L([r1,x+10]) :- L(r1), r1=[r2,x], x < 30.
.decl Flatten(x: number)
Flatten(x) :- L([_,x]).
.output Flatten
```
which transfers the lists into a set called `Flatten`. 

Note that record types are supported by the I/O system. You can use the ```.input``` and ```.output``` directives
to print tuples of relations with record elements. If a records contains another record, 
the whole record with all its composed records will be either read or written. 
Note that union types involving records and sub-typing of record types are currently not supported in Soufflé.

##  Algebraic Data Types (ADT)

Alebraic Data Types are extended versions of records, which permit several type shapes for an ADT. Each shape in the ADT we call a branch. A branch is defined similar to a record, however, it has a unique branch identifier. 

A record-type is defined as follows,
```prolog
.type <new-adt> = <branch-id> { <name_1>: <type_1>, ..., <name_k>: <type_k> } | ...
```
where `<new-adt>` is the name of the ADT. Each branch has a unique identifier `<branch-id>` followed by the fields in that branch. The fields of a branch are defined in curly braces. 

Below is given an example for an ADT definition,
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
Note that `nil` cannot be used for ADTs and termination branches for recursive definition must be expressed explicitly by the programmer. 

In rules, branches can be constructed using the dollar sign followed by the branch identifier, i.e.,  `$branch_constructor(args...)`.
The following example demonstrate how branch constructors can be used in rules:

```prolog
.type Expression = Number { x : number }
                 | Variable { v : symbol}
                 | Add {e_1 : Expression, e_2 :Expression}
                 | Imaginary {}

.decl A(x:Expression)
A($Number(10)).
A($Add($Number(10),$Imaginary())).
A($Add($Number(10), $Variable("x"))).
A($Number(x+1)) :- A($Number(x)), x < 20.

.output A
```

Note that a constructor can only be used for an ADT once. For example, 
the following type definition is illegal:
```prolog
.type A = Number { x:number }
        | Symbol { v:symbol }
.type B = Number { x:number }  // error: reuse of branch identifier
        | Symbol { v:symbol }  // error: reuse of branch identifier
```
Since the branch identifier `Number` and `Symbol` are reused in the ADT `B`. One way to work around this is to use [Components](components):

```prolog
.type OuterType = Number { n: number }
                | Symbol { s: symbol }
.comp MyComponent {
	.type InnerType = Number { n: number }
                    | Symbol { s: symbol }
}
.init component = MyComponent
.decl Relation(x:component.InnerType)
Relation($component.Symbol("x")).
```

## Type Conversion
Soufflé permits type conversion of an argument using a functor notation. The type of argument `<expr>` is converted to a new type `<new-type>` using the type cast `as(<expr>, <new-type>)`. The correctness of the cast is left to the user. 

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
Note that type casts between numbers and symbols are particular dangerous because strings for certain ordinal numbers may not exist. E.g., the fact `A(as(1034234234, symbol).` most likely will cause troubles in conjunction with an output directive since a symbol with ordinal number 1034234234 may not exist. 

Type conversions are cast the value to a new type without converting the value. For converting, a number stored as a string and vice veras we have the functors `to_number`, `to_string`, `to_unsigned`, and `to_float`.

## Syntax 

In the following, we define type declarations in Soufflé more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Type Declaration 
 
A type declaration binds a name with a new type. The type is either a subtype, an equivalence/union type, a record type, of an ADT.

![Type Declaration](https://souffle-lang.github.io/img/type_decl.svg)
```ebnf
type_decl ::= TYPE IDENT ("<:" type_name | "=" ( type_name ( "|" type_name )* | record_list | adt_branch ( "|" adt_branch )* ))
```

### Type Name 

Soufflé has pre-defined types such as `number`, `symbol`, `unsigned`, and `float`. Used-defined types have a name. If a type has been defined in a component, the type can be still accessed outside the component using a qualified name. 

![Type Name](https://souffle-lang.github.io/img/type_name.svg)
```ebnf
type_name ::=  "number" | "symbol" |"unsigned" | "float"  | IDENT ("." IDENT )*
```

### Record Declaration 

A record declaration consists of a list of attributes. Record declarations can be recursive.

![Record Declaration](https://souffle-lang.github.io/img/record_list.svg)
```ebnf
record_list ::= "[" attribute ( "," attribute)* "]"
```

### ADT Declaration

An ADT consists of a several ADT branches. An ADT branch consists of a list of attributes which 
is associated with a branch identifier. The branch identifier must be unique in Soufflé.

![ADT Branch](https://souffle-lang.github.io/img/adt_branch.svg)
```ebnf
adt_branch ::= IDENT "{" (attribute ( "," attribute)*)? "}"
```

### Attribute Declaration

An attribute binds a name with a type. 

![Attribute](https://souffle-lang.github.io/img/attribute.svg)

```ebnf
attribute ::= IDENT ":" type_name 
```

### Legacy Syntax

The syntax of Soufflé changed over time. Older code bases can be still used with 
modern versions of Soufflé.  In older versions of Soufflé we used
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

{% include links.html %}
