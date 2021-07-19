---
title: Datalog
permalink: /datalog
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
A Horn clause is a disjunction of literals with at most one positive literal
that can be written in implication form as,

**u** ⇐  **p** ∧ **q** ∧ ... ∧ **t**

The Horn clause can be read as: predicate **u** holds if the conditions
**p** ∧ **q** ∧ ... ∧ **t** hold.
We refer to predicate **u** as the head (aka. consequent), and
the condition **p** ∧ **q** ∧ ... ∧ **t**  as body (aka. antecedent),
repectively.

There are three different types of Horn clause:
 * Facts are of the form, **u** ⇐  **true**
 * Definite clauses have a predicate as a head, and one or more predicates in the body, as above.
 * Goal clauses are horn clauses of the form **false** ⇐ **p** ∧ **q** ∧ ... ∧ **t**

The literals are predicates (i.e. logical functions with arguments that
are either constants or variables), negated predicates, or constraints.
All variables of a clause must be *grounded*, i.e., a variable must occur
at least once as an argument of a positive predicate in the body of
a clause.  Soufflé permits arithmetic and string functors assuming
that their arguments are grounded.

A Soufflé program has no interactive mode and queries are reduced
to whole relations. Goals are whole logical relations
of the form **false** ⇐ **p** where **p** is a logical relation.
However, for one logical program several goals may be specified.

Note that Soufflé's language permits the extension of the domain
while evaluting the logic program. Hence Soufflé goes beyond
the standard semantics of Datalog (i.e., finite domains, stratified
negations, and totally ordered domains), and becomes Turing equivalent.
As a consequence, some programs may not terminate because of this
extension.

## Language

 ```A``` and ```B``` with two number attributes.
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
Relation ```B``` has two rules, i.e., ```B(x,y) :- A(x,y).``` and ```B(x,y) :- A(x,y), B(y,z).```.  The directive ```.output B``` queries the relation ```B``` at the end of the execution. Several queries may show up in the program. For example, by adding the directive ```.output A``` the relation ```A``` is queried at the end of execution as well. The locations of the facts, rules, and queries (i.e., output directives) in the source code are irrelevant.

## Relations

Soufflé requires the declaration of relations. A relation is a set
of ordered tuples (x<sub>1</sub>, ..., x<sub>k</sub>) where each
element x<sub>i</sub> is a member of a data domain denoted
by an attribute type. In the previous example, the declaration

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

# Types

Souffle utilises a typed Datalog dialect to conduct static checks enabling the early detection of errors in Datalog query specifications. Each attribute of involved relations has to be typed. Based on those, Souffle attempts to deduce a type for all terms within all the Horn clauses within a given Datalog program. In case no type can be deduced for some terms, a typing error is reported -- indicating a likely inconsistency in the query specification.

## Type Definitions

There are four primitive types in Souffle: (1) `symbol`, (2) `number`, (3) `unsigned`, and (4) `float`. Those types have the following properties:

* **Numbers** are signed integer numbers and are fixed to a fixed range. The range is fixed to the `int32_t` type covering the range [ -2^31 .. 2^31 -1 ]. However, the wordsize can be altered by a configuration flag in the configure script. 

* **Symbols** are reference values that have a symbolic representation as well. They are currently represented as strings. Thus, string based operations, e.g. string concatenation, may introduce new values, not present in any input set, to the stable result set.

* **Unsigned** are unsigned integer numbers and are fixed to a fixed range. The range is fixed to the `uint32_t` type covering the range [ 0 .. 2^32 ]. However, the wordsize can be altered by a configuration flag in the configure script.

* **Float** are reference values that have a symbolic representation as well. They are currently represented as strings. Thus, string based operations, e.g. string concatenation, may introduce new values, not present in any input set, to the stable result set.

Note that there is no distinction made between symbols and their string representation. As a consequence, any symbol may be interpreted as a string for string operations. This would, for instance, enable the creation of a new city "BrisbaneSydney" through an expression `cat(X,Y)` where the variables `X` and `Y` are bound correspondingly. It is the user's responsibility to use `cat` only in sound contexts. Future development may resolve this shortcoming in the type system.

User-defined types can be specified as sub-types of primitive types, and unions of these types can be formed:

* **Base Types** are independent, user-defined types covering a subset of one of the primitive types (symbols, numbers, unsigned, and float). Base types are defined by constructs of the form:
```
.type Position <: number     // creates a user-defined number type
.type weight <: float        // creates a user-defined float type
.type PostCode <: usigned    // creates a user-defined unsigned type
.type Village <: symbol      // creates a user-defined symbol type
```
Operators applicable to the corresponding base type (`number` or `symbol`) can also be applied on the corresponding user-defined primitive type. For instance, arithmetic operations can be applied on values of the `weight` type above -- thus, weights might be added or multiplied.

* **Union Types** are combining multiple types into a new type. Union types are defined similar to
```
.type Place = Village | City
```
where `Place` on the left side is the name of the new type and on the right side there is a list of one or more types to be aggregated -- in this case `Village` and `City`. Any village or city value will be a valid Place. More details are provided on the next page.

The following sub-sections will provide more in-depth details on the semantics of types in Souffle's Datalog dialect.

* **Record Types** are compositions of other values of different types. They permit recursive type definition as well. For example, a recursive list type could be defined as follows:
```
.type List = [next:List, value:number]
```

# Relations

Relations are declared using the directive .decl followed by a parenthesis with its attribute names and types. For example,
```
.decl A(x:number, y:number).
```

defines the relation A with two columns of type number.

# Clauses

Clauses are conditional logic statements. A clause starts with a head followed by a body. For example,
```
A(x,y) :- B(x,y).
```
a pair (x,y) is in A, if it is in B.

Clauses have qualifiers which direct the query planner for better query execution. The qualifier ".strict" forces the query planner to use the order of the specified clause. The qualifer ".plan" permits the programmer to specify a schedule for each version of the clause. Several versions of a clause may occur if the clause is evaluated in a fixpoint.


Clauses can have multiple heads:
```
A(x,y), C(x,y) :- B(x,y). 
```
which is syntactic sugar for
```
A(x,y) :- B(x,y). 
C(x,y) :- B(x,y). 
```

# Negation

A clause of the form
```
CanRenovate(person, building) :- Owner(person, building), !Heritage(building).
```
expresses the rule that an owner can renovate a building with the condition the building is not classified as heritage. Thus the literal "Heritage(building)" is negated (via "!") in the body of the clause.

But negation needs to be used with care. For example,
```
A(x) :- ! B(x).
B(x) :- ! A(x).
```
is a circular definition. One cannot determine if anything belongs to the relation "A" without determining if it belongs to relation "B". But to determine if it is a "B" one needs to determine if the item belongs to "A". Such circular definitions are forbidden. Technically, rules involving negation must be stratifiable.

Negated literals do not bind variables. For example,
```
A(x,y) :- R(x), !S(y).
```
is not valid as the set of values that "y" can take is not clear. This can be rewritten as,
```
A(x,y) :- R(x), Scope(y), !S(y).
```
where the relation "Scope" defines the set of values that "y" can take.

# Disjunction

A clause of the form
```
LivesAt(person, building) :-
    Owner(owner, building),
    ( person=owner ; Housemate(owner, person) ).
```
expresses the rule that a person lives in a building if they are the owner, or a housemate of the owner. Thus the conditions `person=owner` and `Housemate(owner, person)` are joined by `;` to indicate that either must hold.

# Comments and Pre-Processing
Soufflé utilises the same comment syntax as C/C++. Furthermore, all souffle programs are passed through the C pre-processor. As a consequence, e.g. `#include` pragmas may be utilized to organize Datalog input queries into several files and reuse common constructs within several programs. Also, constants may be defined utilizing the `#define` pragma.

# Identifier Naming Rules
Soufflé Identifiers follow the C++ naming convention, except that a question mark may appear anywhere.
- The identifier can only be composed of letters (lower or upper case), numbers, or the question mark and underscore characters. That means the name cannot contain whitespace, or any symbols other than underscores or question marks.
- The identifier must begin with a letter (lower or upper case), an underscore, or a question mark. It can not start with a number.

# Syntax 

The whole syntax of Soufflé can be found as a (railroad grammar)(http://souffle-lang.github.io/syntax.xhtml).


{% include links.html %}
