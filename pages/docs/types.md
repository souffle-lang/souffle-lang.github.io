---
title: Types
permalink: /types
sidebar: docs_sidebar
folder: docs
---
Soufflé is a statically typed language. The main purpose of Souffle's
type system is the correct usage of relations and records in rules.
The type system defines types for relation attributes and records
and is enforced at compile time. Static types have the advantage
that they do not have runtime overheads while evaluating a
logic program.

Our type system is based on the idea of sets. A type can be
either the universe of all possible values of a primitive type,
or a subset of it; subsets can be composed of other subsets.

## Primitive Types
Soufflé has four primitive types:
* Symbol type: `symbol`
* Signed number type: `number`
* Unsigned number type: `unsigned`
* Float number type: `float`

The word size of a primitive type is 32 bits. The word size can be
changed to 64 bit by appropriately configuring Soufflé using
the configuration option `--enable-64bit-domain` in the `configure` script.

### Symbol Type
The symbol type consists of the universe of all strings.
Internally, the symbol type is represented by an ordinal number.
The ordinal number for a symbol can be determined by using the `ord` functor, e.g.,
`ord("hello")` represents the ordinal number for `"hello"`.

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

### Primitive Type Example
The following is an example of some Soufflé code that uses the primitive types:
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

You can define equivalence types in Soufflé using the directive `.type <name> = <other-type>`
defining a new type `<name>` that is equivalent to type `<other-type>`.  For example,
```
.type myNumber = number
```
introduces the new type `myNumber` that is equivalent to the primitive type `number`.

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
```
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
```
.type <ident> = <ident-1> | <ident-2> | ... | <ident-k>
```
where `<ident>` is the name of the union type and `<ident-...>` refers to the base and other union types used in the union type.  For example, the union type `Place`
```
.type City <: symbol
.type Town <: symbol
.type Village <: symbol
.type Place = City | Town | Village
```
is the union of base types `City`, `Town`, and `Village`.
The union type `PostalCode`,
```
.type PostCodes <: number
.type ZipCodes <: number
.type PostalCode = PostCodes | ZipCodes
```
unifies the base types `PostCodes` and `ZipCodes`.

However, the following union type is not allowed
```
.type Weekdays <: symbol
.type Dates <: number
.type Days = Weekdays | Dates // error
```
and will result in a compile error, because `Weekdays` and `Dates` are not derived from the same primitive types.

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

The above example will produce a type clash
```
.type even <: number
.type odd <: number

.decl A(x:even)
.decl B(x:odd)
A(X) :- B(X) // type clash
```
In this example, `even` and `odd` are defined as two disjoint sub-types of the type number, each exhibiting its own domain of values. From this definition Soufflé can deduce that the users intention was to define two disjoint sets and will report an error for the rule in the last line. For logic programmers it is crucial to accurately model the categories of values in form of a type system -- as has been illustrated by the example above.

## Record Types

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

### I/O 

Record types are supported by the I/O system. You can use ```.input``` and ```.output``` directives
to print tuples of relations with record elements. If a records contains another record, 
the whole record with all its composed records will be either read or written. 

### Implementation
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

### Limitations
In the current verion,  union types involving records and sub-typing of record types are currently not supported.


##  Algebraic Data Types (ADT)

The syntax for defining an ADT is:
```
ADT ::= ".type " IDENT "=" branch (| branch)*
branch ::= IDENT "{" "}"
branch ::= IDENT "{" attr (, attr)* "}"
```
Examples:
```
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

```
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
```
.type A = Number { x:number }
        | Symbol { v:symbol }
.type B = Number { x:number }
        | Symbol { v:symbol }
```
Since the constructors `Number` and `Symbol` show up twice in ADT `A` and ADT `B`. 

## Type Conversion

Soufflé permits type conversion with the as functor that takes as a first argument an functor expression and as a second argument the new type of the expression. 

For example, 
```
.type Variable <: symbol
.type StackIndex <: symbol
.type VariableOrStackIndex = Variable | StackIndex

.decl A(a: VariableOrStackIndex)

.decl B(a: Variable)

B(as(a, Variable)) :- A(a).
```

Converts the expression `as(a, Variable)` to an expression of type `Variable` although `a` is of type `VariableOrStackIndex`. 

## Syntax 

![Type Grammar](https://souffle-lang.github.io/pages/docs/type-grammar.xhtml)

In the following we define the syntax of type declarations in Souffle. The sSyntax is expressed as rail-road grammars and EBNF.

![Type Declaration](https://souffle-lang.github.io/img/type_decl.svg)
```
type_decl ::= TYPE IDENT ("<:" type_name | "=" ( type_name ( "|" type_name )* | record_list | adt_branch ( "|" adt_branch )* ))
```

![Type Name](https://souffle-lang.github.io/img/type_name.svg)
```
type_name ::= IDENT ("." IDENT )* | "unsigned" | "number" | "float" | "symbol" 
```

![Record Declaration](https://souffle-lang.github.io/img/record_decl.svg)
```
record_decl ::= "[" attribute ( "," attribute)* "]"
```

![ADT Branch](https://souffle-lang.github.io/img/adt_branch.svg)
```
adt_branch ::= IDENT "{" attribute ( "," attribute)* "}"
```

![Attribute](https://souffle-lang.github.io/img/attribute.svg)
```
attribute ::= IDENT ":" type_name 
```


      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml" style="font-size: 14px; font-weight:bold"><xhtml:a name="type_name">type_name:</xhtml:a></xhtml:p><svg xmlns="http://www.w3.org/2000/svg" width="197" height="257">
         <defs>
            <style type="text/css">
    @namespace "http://www.w3.org/2000/svg";
    .line                 {fill: none; stroke: #332900; stroke-width: 1;}
    .bold-line            {stroke: #141000; shape-rendering: crispEdges; stroke-width: 2;}
    .thin-line            {stroke: #1F1800; shape-rendering: crispEdges}
    .filled               {fill: #332900; stroke: none;}
    text.terminal         {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #141000;
                            font-weight: bold;
                          }
    text.nonterminal      {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1A1400;
                            font-weight: normal;
                          }
    text.regexp           {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1F1800;
                            font-weight: normal;
                          }
    rect, circle, polygon {fill: #332900; stroke: #332900;}
    rect.terminal         {fill: #FFDB4D; stroke: #332900; stroke-width: 1;}
    rect.nonterminal      {fill: #FFEC9E; stroke: #332900; stroke-width: 1;}
    rect.text             {fill: none; stroke: none;}
    polygon.regexp        {fill: #FFF4C7; stroke: #332900; stroke-width: 1;}
  </style>
         </defs>
         <polygon points="9 61 1 57 1 65"/>
         <polygon points="17 61 9 57 9 65"/><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#IDENT" xlink:title="IDENT">
            <rect x="71" y="47" width="58" height="32"/>
            <rect x="69" y="45" width="58" height="32" class="nonterminal"/>
            <text class="nonterminal" x="79" y="65">IDENT</text></a><rect x="71" y="3" width="24" height="32" rx="10"/>
         <rect x="69" y="1" width="24" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="79" y="21">.</text>
         <rect x="51" y="91" width="82" height="32" rx="10"/>
         <rect x="49" y="89" width="82" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="59" y="109">unsigned</text>
         <rect x="51" y="135" width="72" height="32" rx="10"/>
         <rect x="49" y="133" width="72" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="59" y="153">number</text>
         <rect x="51" y="179" width="50" height="32" rx="10"/>
         <rect x="49" y="177" width="50" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="59" y="197">float</text>
         <rect x="51" y="223" width="68" height="32" rx="10"/>
         <rect x="49" y="221" width="68" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="59" y="241">symbol</text>
         <svg:path xmlns:svg="http://www.w3.org/2000/svg" class="line" d="m17 61 h2 m40 0 h10 m58 0 h10 m-98 0 l20 0 m-1 0 q-9 0 -9 -10 l0 -24 q0 -10 10 -10 m78 44 l20 0 m-20 0 q10 0 10 -10 l0 -24 q0 -10 -10 -10 m-78 0 h10 m24 0 h10 m0 0 h34 m-118 44 h20 m118 0 h20 m-158 0 q10 0 10 10 m138 0 q0 -10 10 -10 m-148 10 v24 m138 0 v-24 m-138 24 q0 10 10 10 m118 0 q10 0 10 -10 m-128 10 h10 m82 0 h10 m0 0 h16 m-128 -10 v20 m138 0 v-20 m-138 20 v24 m138 0 v-24 m-138 24 q0 10 10 10 m118 0 q10 0 10 -10 m-128 10 h10 m72 0 h10 m0 0 h26 m-128 -10 v20 m138 0 v-20 m-138 20 v24 m138 0 v-24 m-138 24 q0 10 10 10 m118 0 q10 0 10 -10 m-128 10 h10 m50 0 h10 m0 0 h48 m-128 -10 v20 m138 0 v-20 m-138 20 v24 m138 0 v-24 m-138 24 q0 10 10 10 m118 0 q10 0 10 -10 m-128 10 h10 m68 0 h10 m0 0 h30 m23 -176 h-3"/>
         <polygon points="187 61 195 57 195 65"/>
         <polygon points="187 61 179 57 179 65"/></svg><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">
         <xhtml:div class="ebnf"><xhtml:code>
               <div><a href="#type_name" title="type_name">type_name</a></div>
               <div>         ::= <a href="#IDENT" title="IDENT">IDENT</a> ( '.' <a href="#IDENT" title="IDENT">IDENT</a> )*</div>
               <div>           | 'unsigned'</div>
               <div>           | 'number'</div>
               <div>           | 'float'</div>
               <div>           | 'symbol'</div></xhtml:code></xhtml:div>
      </xhtml:p>
      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">referenced by:
         <xhtml:ul>
            <xhtml:li><xhtml:a href="#attribute" title="attribute">attribute</xhtml:a></xhtml:li>
            <xhtml:li><xhtml:a href="#type_decl" title="type_decl">type_decl</xhtml:a></xhtml:li>
         </xhtml:ul>
      </xhtml:p><xhtml:br xmlns:xhtml="http://www.w3.org/1999/xhtml" /><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml" style="font-size: 14px; font-weight:bold"><xhtml:a name="type_decl">type_decl:</xhtml:a></xhtml:p><svg xmlns="http://www.w3.org/2000/svg" width="631" height="235">
         <defs>
            <style type="text/css">
    @namespace "http://www.w3.org/2000/svg";
    .line                 {fill: none; stroke: #332900; stroke-width: 1;}
    .bold-line            {stroke: #141000; shape-rendering: crispEdges; stroke-width: 2;}
    .thin-line            {stroke: #1F1800; shape-rendering: crispEdges}
    .filled               {fill: #332900; stroke: none;}
    text.terminal         {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #141000;
                            font-weight: bold;
                          }
    text.nonterminal      {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1A1400;
                            font-weight: normal;
                          }
    text.regexp           {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1F1800;
                            font-weight: normal;
                          }
    rect, circle, polygon {fill: #332900; stroke: #332900;}
    rect.terminal         {fill: #FFDB4D; stroke: #332900; stroke-width: 1;}
    rect.nonterminal      {fill: #FFEC9E; stroke: #332900; stroke-width: 1;}
    rect.text             {fill: none; stroke: none;}
    polygon.regexp        {fill: #FFF4C7; stroke: #332900; stroke-width: 1;}
  </style>
         </defs>
         <polygon points="9 17 1 13 1 21"/>
         <polygon points="17 17 9 13 9 21"/>
         <rect x="31" y="3" width="54" height="32" rx="10"/>
         <rect x="29" y="1" width="54" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="39" y="21">.type</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#IDENT" xlink:title="IDENT">
            <rect x="105" y="3" width="58" height="32"/>
            <rect x="103" y="1" width="58" height="32" class="nonterminal"/>
            <text class="nonterminal" x="113" y="21">IDENT</text></a><rect x="203" y="3" width="34" height="32" rx="10"/>
         <rect x="201" y="1" width="34" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="211" y="21">&lt;:</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#type_name" xlink:title="type_name">
            <rect x="257" y="3" width="92" height="32"/>
            <rect x="255" y="1" width="92" height="32" class="nonterminal"/>
            <text class="nonterminal" x="265" y="21">type_name</text></a><rect x="203" y="69" width="30" height="32" rx="10"/>
         <rect x="201" y="67" width="30" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="211" y="87">=</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#type_name" xlink:title="type_name">
            <rect x="273" y="69" width="92" height="32"/>
            <rect x="271" y="67" width="92" height="32" class="nonterminal"/>
            <text class="nonterminal" x="281" y="87">type_name</text></a><rect x="405" y="69" width="26" height="32" rx="10"/>
         <rect x="403" y="67" width="26" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="413" y="87">|</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#type_name" xlink:title="type_name">
            <rect x="451" y="69" width="92" height="32"/>
            <rect x="449" y="67" width="92" height="32" class="nonterminal"/>
            <text class="nonterminal" x="459" y="87">type_name</text></a><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#record_list" xlink:title="record_list">
            <rect x="273" y="113" width="86" height="32"/>
            <rect x="271" y="111" width="86" height="32" class="nonterminal"/>
            <text class="nonterminal" x="281" y="131">record_list</text></a><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#adt_branch" xlink:title="adt_branch">
            <rect x="293" y="201" width="92" height="32"/>
            <rect x="291" y="199" width="92" height="32" class="nonterminal"/>
            <text class="nonterminal" x="301" y="219">adt_branch</text></a><rect x="293" y="157" width="26" height="32" rx="10"/>
         <rect x="291" y="155" width="26" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="301" y="175">|</text>
         <svg:path xmlns:svg="http://www.w3.org/2000/svg" class="line" d="m17 17 h2 m0 0 h10 m54 0 h10 m0 0 h10 m58 0 h10 m20 0 h10 m34 0 h10 m0 0 h10 m92 0 h10 m0 0 h234 m-420 0 h20 m400 0 h20 m-440 0 q10 0 10 10 m420 0 q0 -10 10 -10 m-430 10 v46 m420 0 v-46 m-420 46 q0 10 10 10 m400 0 q10 0 10 -10 m-410 10 h10 m30 0 h10 m20 0 h10 m92 0 h10 m20 0 h10 m26 0 h10 m0 0 h10 m92 0 h10 m-178 0 l20 0 m-1 0 q-9 0 -9 -10 l0 -12 q0 -10 10 -10 m158 32 l20 0 m-20 0 q10 0 10 -10 l0 -12 q0 -10 -10 -10 m-158 0 h10 m0 0 h148 m-310 32 h20 m310 0 h20 m-350 0 q10 0 10 10 m330 0 q0 -10 10 -10 m-340 10 v24 m330 0 v-24 m-330 24 q0 10 10 10 m310 0 q10 0 10 -10 m-320 10 h10 m86 0 h10 m0 0 h204 m-320 -10 v20 m330 0 v-20 m-330 20 v68 m330 0 v-68 m-330 68 q0 10 10 10 m310 0 q10 0 10 -10 m-300 10 h10 m92 0 h10 m-132 0 l20 0 m-1 0 q-9 0 -9 -10 l0 -24 q0 -10 10 -10 m112 44 l20 0 m-20 0 q10 0 10 -10 l0 -24 q0 -10 -10 -10 m-112 0 h10 m26 0 h10 m0 0 h66 m20 44 h158 m43 -198 h-3"/>
         <polygon points="621 17 629 13 629 21"/>
         <polygon points="621 17 613 13 613 21"/></svg><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">
         <xhtml:div class="ebnf"><xhtml:code>
               <div><a href="#type_decl" title="type_decl">type_decl</a></div>
               <div>         ::= '.type' <a href="#IDENT" title="IDENT">IDENT</a> ( '&lt;:' <a href="#type_name" title="type_name">type_name</a> | '=' ( <a href="#type_name" title="type_name">type_name</a> ( '|' <a href="#type_name" title="type_name">type_name</a> )+ | <a href="#record_list" title="record_list">record_list</a> | <a href="#adt_branch" title="adt_branch">adt_branch</a> ( '|' <a href="#adt_branch" title="adt_branch">adt_branch</a> )* ) )</div></xhtml:code></xhtml:div>
      </xhtml:p>
      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">no references</xhtml:p><xhtml:br xmlns:xhtml="http://www.w3.org/1999/xhtml" /><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml" style="font-size: 14px; font-weight:bold"><xhtml:a name="adt_branch">adt_branch:</xhtml:a></xhtml:p><svg xmlns="http://www.w3.org/2000/svg" width="387" height="97">
         <defs>
            <style type="text/css">
    @namespace "http://www.w3.org/2000/svg";
    .line                 {fill: none; stroke: #332900; stroke-width: 1;}
    .bold-line            {stroke: #141000; shape-rendering: crispEdges; stroke-width: 2;}
    .thin-line            {stroke: #1F1800; shape-rendering: crispEdges}
    .filled               {fill: #332900; stroke: none;}
    text.terminal         {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #141000;
                            font-weight: bold;
                          }
    text.nonterminal      {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1A1400;
                            font-weight: normal;
                          }
    text.regexp           {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1F1800;
                            font-weight: normal;
                          }
    rect, circle, polygon {fill: #332900; stroke: #332900;}
    rect.terminal         {fill: #FFDB4D; stroke: #332900; stroke-width: 1;}
    rect.nonterminal      {fill: #FFEC9E; stroke: #332900; stroke-width: 1;}
    rect.text             {fill: none; stroke: none;}
    polygon.regexp        {fill: #FFF4C7; stroke: #332900; stroke-width: 1;}
  </style>
         </defs>
         <polygon points="9 61 1 57 1 65"/>
         <polygon points="17 61 9 57 9 65"/><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#IDENT" xlink:title="IDENT">
            <rect x="31" y="47" width="58" height="32"/>
            <rect x="29" y="45" width="58" height="32" class="nonterminal"/>
            <text class="nonterminal" x="39" y="65">IDENT</text></a><rect x="109" y="47" width="28" height="32" rx="10"/>
         <rect x="107" y="45" width="28" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="117" y="65">{</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#attribute" xlink:title="attribute">
            <rect x="197" y="47" width="74" height="32"/>
            <rect x="195" y="45" width="74" height="32" class="nonterminal"/>
            <text class="nonterminal" x="205" y="65">attribute</text></a><rect x="197" y="3" width="24" height="32" rx="10"/>
         <rect x="195" y="1" width="24" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="205" y="21">,</text>
         <rect x="331" y="47" width="28" height="32" rx="10"/>
         <rect x="329" y="45" width="28" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="339" y="65">}</text>
         <svg:path xmlns:svg="http://www.w3.org/2000/svg" class="line" d="m17 61 h2 m0 0 h10 m58 0 h10 m0 0 h10 m28 0 h10 m40 0 h10 m74 0 h10 m-114 0 l20 0 m-1 0 q-9 0 -9 -10 l0 -24 q0 -10 10 -10 m94 44 l20 0 m-20 0 q10 0 10 -10 l0 -24 q0 -10 -10 -10 m-94 0 h10 m24 0 h10 m0 0 h50 m-134 44 h20 m134 0 h20 m-174 0 q10 0 10 10 m154 0 q0 -10 10 -10 m-164 10 v14 m154 0 v-14 m-154 14 q0 10 10 10 m134 0 q10 0 10 -10 m-144 10 h10 m0 0 h124 m20 -34 h10 m28 0 h10 m3 0 h-3"/>
         <polygon points="377 61 385 57 385 65"/>
         <polygon points="377 61 369 57 369 65"/></svg><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">
         <xhtml:div class="ebnf"><xhtml:code>
               <div><a href="#adt_branch" title="adt_branch">adt_branch</a></div>
               <div>         ::= <a href="#IDENT" title="IDENT">IDENT</a> '{' ( <a href="#attribute" title="attribute">attribute</a> ( ',' <a href="#attribute" title="attribute">attribute</a> )* )? '}'</div></xhtml:code></xhtml:div>
      </xhtml:p>
      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">referenced by:
         <xhtml:ul>
            <xhtml:li><xhtml:a href="#type_decl" title="type_decl">type_decl</xhtml:a></xhtml:li>
         </xhtml:ul>
      </xhtml:p><xhtml:br xmlns:xhtml="http://www.w3.org/1999/xhtml" /><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml" style="font-size: 14px; font-weight:bold"><xhtml:a name="record_list">record_list:</xhtml:a></xhtml:p><svg xmlns="http://www.w3.org/2000/svg" width="305" height="97">
         <defs>
            <style type="text/css">
    @namespace "http://www.w3.org/2000/svg";
    .line                 {fill: none; stroke: #332900; stroke-width: 1;}
    .bold-line            {stroke: #141000; shape-rendering: crispEdges; stroke-width: 2;}
    .thin-line            {stroke: #1F1800; shape-rendering: crispEdges}
    .filled               {fill: #332900; stroke: none;}
    text.terminal         {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #141000;
                            font-weight: bold;
                          }
    text.nonterminal      {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1A1400;
                            font-weight: normal;
                          }
    text.regexp           {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1F1800;
                            font-weight: normal;
                          }
    rect, circle, polygon {fill: #332900; stroke: #332900;}
    rect.terminal         {fill: #FFDB4D; stroke: #332900; stroke-width: 1;}
    rect.nonterminal      {fill: #FFEC9E; stroke: #332900; stroke-width: 1;}
    rect.text             {fill: none; stroke: none;}
    polygon.regexp        {fill: #FFF4C7; stroke: #332900; stroke-width: 1;}
  </style>
         </defs>
         <polygon points="9 61 1 57 1 65"/>
         <polygon points="17 61 9 57 9 65"/>
         <rect x="31" y="47" width="26" height="32" rx="10"/>
         <rect x="29" y="45" width="26" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="39" y="65">[</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#attribute" xlink:title="attribute">
            <rect x="117" y="47" width="74" height="32"/>
            <rect x="115" y="45" width="74" height="32" class="nonterminal"/>
            <text class="nonterminal" x="125" y="65">attribute</text></a><rect x="117" y="3" width="24" height="32" rx="10"/>
         <rect x="115" y="1" width="24" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="125" y="21">,</text>
         <rect x="251" y="47" width="26" height="32" rx="10"/>
         <rect x="249" y="45" width="26" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="259" y="65">]</text>
         <svg:path xmlns:svg="http://www.w3.org/2000/svg" class="line" d="m17 61 h2 m0 0 h10 m26 0 h10 m40 0 h10 m74 0 h10 m-114 0 l20 0 m-1 0 q-9 0 -9 -10 l0 -24 q0 -10 10 -10 m94 44 l20 0 m-20 0 q10 0 10 -10 l0 -24 q0 -10 -10 -10 m-94 0 h10 m24 0 h10 m0 0 h50 m-134 44 h20 m134 0 h20 m-174 0 q10 0 10 10 m154 0 q0 -10 10 -10 m-164 10 v14 m154 0 v-14 m-154 14 q0 10 10 10 m134 0 q10 0 10 -10 m-144 10 h10 m0 0 h124 m20 -34 h10 m26 0 h10 m3 0 h-3"/>
         <polygon points="295 61 303 57 303 65"/>
         <polygon points="295 61 287 57 287 65"/></svg><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">
         <xhtml:div class="ebnf"><xhtml:code>
               <div><a href="#record_list" title="record_list">record_list</a></div>
               <div>         ::= '[' ( <a href="#attribute" title="attribute">attribute</a> ( ',' <a href="#attribute" title="attribute">attribute</a> )* )? ']'</div></xhtml:code></xhtml:div>
      </xhtml:p>
      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">referenced by:
         <xhtml:ul>
            <xhtml:li><xhtml:a href="#type_decl" title="type_decl">type_decl</xhtml:a></xhtml:li>
         </xhtml:ul>
      </xhtml:p><xhtml:br xmlns:xhtml="http://www.w3.org/1999/xhtml" /><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml" style="font-size: 14px; font-weight:bold"><xhtml:a name="attribute">attribute:</xhtml:a></xhtml:p><svg xmlns="http://www.w3.org/2000/svg" width="273" height="37">
         <defs>
            <style type="text/css">
    @namespace "http://www.w3.org/2000/svg";
    .line                 {fill: none; stroke: #332900; stroke-width: 1;}
    .bold-line            {stroke: #141000; shape-rendering: crispEdges; stroke-width: 2;}
    .thin-line            {stroke: #1F1800; shape-rendering: crispEdges}
    .filled               {fill: #332900; stroke: none;}
    text.terminal         {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #141000;
                            font-weight: bold;
                          }
    text.nonterminal      {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1A1400;
                            font-weight: normal;
                          }
    text.regexp           {font-family: Verdana, Sans-serif;
                            font-size: 12px;
                            fill: #1F1800;
                            font-weight: normal;
                          }
    rect, circle, polygon {fill: #332900; stroke: #332900;}
    rect.terminal         {fill: #FFDB4D; stroke: #332900; stroke-width: 1;}
    rect.nonterminal      {fill: #FFEC9E; stroke: #332900; stroke-width: 1;}
    rect.text             {fill: none; stroke: none;}
    polygon.regexp        {fill: #FFF4C7; stroke: #332900; stroke-width: 1;}
  </style>
         </defs>
         <polygon points="9 17 1 13 1 21"/>
         <polygon points="17 17 9 13 9 21"/><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#IDENT" xlink:title="IDENT">
            <rect x="31" y="3" width="58" height="32"/>
            <rect x="29" y="1" width="58" height="32" class="nonterminal"/>
            <text class="nonterminal" x="39" y="21">IDENT</text></a><rect x="109" y="3" width="24" height="32" rx="10"/>
         <rect x="107" y="1" width="24" height="32" class="terminal" rx="10"/>
         <text class="terminal" x="117" y="21">:</text><a xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#type_name" xlink:title="type_name">
            <rect x="153" y="3" width="92" height="32"/>
            <rect x="151" y="1" width="92" height="32" class="nonterminal"/>
            <text class="nonterminal" x="161" y="21">type_name</text></a><svg:path xmlns:svg="http://www.w3.org/2000/svg" class="line" d="m17 17 h2 m0 0 h10 m58 0 h10 m0 0 h10 m24 0 h10 m0 0 h10 m92 0 h10 m3 0 h-3"/>
         <polygon points="263 17 271 13 271 21"/>
         <polygon points="263 17 255 13 255 21"/></svg><xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">
         <xhtml:div class="ebnf"><xhtml:code>
               <div><a href="#attribute" title="attribute">attribute</a></div>
               <div>         ::= <a href="#IDENT" title="IDENT">IDENT</a> ':' <a href="#type_name" title="type_name">type_name</a></div></xhtml:code></xhtml:div>
      </xhtml:p>
      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">referenced by:
         <xhtml:ul>
            <xhtml:li><xhtml:a href="#adt_branch" title="adt_branch">adt_branch</xhtml:a></xhtml:li>
            <xhtml:li><xhtml:a href="#record_list" title="record_list">record_list</xhtml:a></xhtml:li>
         </xhtml:ul>
      </xhtml:p><xhtml:br xmlns:xhtml="http://www.w3.org/1999/xhtml" /><xhtml:hr xmlns:xhtml="http://www.w3.org/1999/xhtml" />
      <xhtml:p xmlns:xhtml="http://www.w3.org/1999/xhtml">
         <xhtml:table border="0" class="signature">
            <xhtml:tr>
               <xhtml:td style="width: 100%"> </xhtml:td>
               <xhtml:td valign="top">
                  <xhtml:nobr class="signature">... generated by <xhtml:a name="Railroad-Diagram-Generator" class="signature" title="https://www.bottlecaps.de/rr/ui" href="https://www.bottlecaps.de/rr/ui" target="_blank">RR - Railroad Diagram Generator</xhtml:a></xhtml:nobr>
               </xhtml:td>
               <xhtml:td><xhtml:a name="Railroad-Diagram-Generator" title="https://www.bottlecaps.de/rr/ui" href="https://www.bottlecaps.de/rr/ui" target="_blank"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16">
                        <g transform="scale(0.178)">
                           <circle cx="45" cy="45" r="45" style="stroke:none; fill:#FFCC00"/>
                           <circle cx="45" cy="45" r="42" style="stroke:#332900; stroke-width:2px; fill:#FFCC00"/>
                           <line x1="15" y1="15" x2="75" y2="75" stroke="#332900" style="stroke-width:9px;"/>
                           <line x1="15" y1="75" x2="75" y2="15" stroke="#332900" style="stroke-width:9px;"/>
                           <text x="7" y="54" style="font-size:26px; font-family:Arial, Sans-serif; font-weight:bold; fill: #332900">R</text>
                           <text x="64" y="54" style="font-size:26px; font-family:Arial, Sans-serif; font-weight:bold; fill: #332900">R</text>
                        </g></svg></xhtml:a></xhtml:td>
            </xhtml:tr>
         </xhtml:table>
      </xhtml:p>
## Legacy Syntax

In older versions of Soufflé we used
```
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
You can still use this syntax in the current versions of Souffle, but you will receive a warning that this legacy syntax is deprecated.
You can enable the old legacy syntax using the flag `--legacy`.
