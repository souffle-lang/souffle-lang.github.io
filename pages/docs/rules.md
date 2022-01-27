---
title: Rules
permalink: /rules
sidebar: docs_sidebar
folder: docs
---
A rule is a horn clause that has one predicate (or more; see below) as a head, and one or more 
literals in the body. The literals can be predicates, negated 
predicates, or constraints.
All variables of a rule must be *grounded*, i.e., a variable must occur
at least once as an argument of a positive predicate in the body of
a rule.  Soufflé permits arithmetic and string functors assuming
that their arguments are grounded.

Consider the following example, that has relation ```A``` and ```B``` with two number attributes.
```prolog
.decl A, B(x:number, y:number)  // declaration of relation B
.input A                     // read A 
B(x,y) :- A(x,y).            // rules of relation B
B(x,z) :- A(x,y), B(y,z).
.output B
```
Relation ```B``` has two rules: ```B(x,y) :- A(x,y).``` and ```B(x,y) :- A(x,y), B(y,z).``` The first rule says, there is a tuple in `B` if this tuple shows up in `A`. The second rule says there is a tuple `(x,z)` in `B`, if there is a tuple in `(x,y)` in `A` and a tuple `(y,z)` in `B`.

For the first rule, variables ```x``` and ```y``` are *grounded*, since they show up as variables in the 
positive predicate ```B(x,y)```. Similiar, the variables ```x```, ```y```, and ```z``` are *grounded* because 
they occur in the positive predicates ``` A(x,y)``` and ```B(y,z)```. 

However, note that the following example has an *ungrounded* variable:
```prolog
.decl fib(idx:number, value:number)
fib(1,1).
fib(2,1).
fib(idx, x + y) :- fib(idx-1, x), fib(idx-2, y), idx <= 10.
.output fib
```

The reason for this is that variable ```idx``` is not bound as an argument of a positive predicate in the body. In the example, variable ```idx``` occurrs in the predicates ```fib(idx-1, x)``` and ```fib(idx-2, y)``` but as arguments of a functor rather than as a direct argument. To make variable ```idx``` bound, we can shift the index by one and obtain a program whose variables are *grounded*:
```prolog
.decl fib(idx:number, value:number)
fib(1,1).
fib(2,1).
fib(idx+1, x + y) :- fib(idx, x), fib(idx-1, y), idx <= 9.
.output fib
```
And the program can produce the following output,
```
---------------
fib
idx	value
===============
1	1
2	1
3	2
4	3
5	5
6	8
7	13
8	21
9	34
10	55
===============
```

### Type System

Variables in a rule have no explicit type declaration. Hence, 
Soufflé's typesystem must automatically deduce the type of variables in a rule, depending on how they are used 
as arguments in predicates, functors, and constraints. Besides deducing their type, its other task is to check 
their appropiate use.  Soufflé's typesystem supports subtypes and union types, e.g., ```.type A <: B``` and ```.type C = A|B```  that makes the semantics of the typesystem more complex. For example,

```prolog
.type A <: number
.type B <: number
.type C = A | B 

.decl P(x:A)
.decl Q(x:B)
.decl R(x:C)

R(x) :- P(x), Q(x).
```
In the above example, the type of variable is of type `C`. The reason is that the right-hand side of the body constraints the variable ```x``` such that ```x``` must be of type `A` / `B` or a supertype of them. The left-hand side constrain the type of variable to type `C` or a subtype of it. We also refer to *grounded* variable occurrences in the body of a rule as `sources` and the variable occurrences in the head of a rule / constraints / negation etc. as `sinks`. The reason for this is that sources may produce a concrete assignment and the the sinks use as concrete assignment. 

### Negation in Rules
A rules of the form
```prolog
CanRenovate(person, building) :- Owner(person, building), !Heritage(building).
```
expresses the rule that an owner can renovate a building with the condition that the building is not classified as heritage. Thus the literal "Heritage(building)" is negated (via "!") in the body of the rule. Not all negations are semantically permissible. For example,
```prolog
A(x) :- ! B(x).
B(x) :- ! A(x).
```
is a circular definition. One cannot determine if anything belongs to the relation "A" without determining if it belongs to relation "B". But to determine if it is a "B" one needs to determine if the item belongs to "A". Such circular definitions are forbidden. Technically, rules involving negation must be stratifiable.

Negated literals do not bind variables. For example,
```prolog
A(x,y) :- R(x), !S(y).
```
is not valid as the set of values that "y" can take is not clear. This can be rewritten as,
```prolog
A(x,y) :- R(x), Scope(y), !S(y).
```
where the relation "Scope" defines the set of values that "y" can take.

### Multiple Heads
Rules can have multiple heads:
```prolog
A(x,y), C(x,y) :- B(x,y). 
```
which is syntactic sugar for
```prolog
A(x,y) :- B(x,y). 
C(x,y) :- B(x,y). 
```

### Disjunction
A rule of the form
```prolog
LivesAt(person, building) :-
    Owner(owner, building),
    ( person=owner ; Housemate(owner, person) ).
```
expresses the rule that a person lives in a building if they are the owner, or a housemate of the owner. Thus the conditions `person=owner` and `Housemate(owner, person)` are joined by `;` to indicate that either must hold.

### Query Plan
Rules may have qualifiers, which are used to change the execution behavior / semantics. 
Qualifiers are used to set a query plan for a rule. The qualifer `.plan` let's the programmer chose a query plan for a rule.  

## Syntax 
In the following, we define rule declarations in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Rule
A rule has one or more heads followed by symbol `:-` and a disjunctive term. A query plan is optional for a rule.

![Rule](https://souffle-lang.github.io/img/rule.svg)

```ebnf
rule ::= atom ( ',' atom )* ':-' disjunction '.' query_plan?
```

### Qualified Name
A qualified name is a sequence of identifiers separated by `.` to disambiguate relations that are instantiated by components.

![Qualifier Name](https://souffle-lang.github.io/img/qualified_name.svg)

```ebnf
qualified_name ::= IDENT ( '.' IDENT )*
```

### Atom
An atom consists of a relation name followed by comma-separated arguments in parenthesis.

![Atom](https://souffle-lang.github.io/img/atom.svg)

```ebnf
atom ::= qualified_name '(' ( argument ( ',' argument )* )? ')'
```

### Disjunction
A disjunction is a list of conjunction separated by ';'.
![Disjunction](https://souffle-lang.github.io/img/disjunction.svg)

```ebnf
disjunction ::= conjunction ( ';' conjunction )*
```

### Conjunction
A conjunctive term is a list of (negated) atoms / constraints / disjunctions separated by ','. 

![Conjunction](https://souffle-lang.github.io/img/conjunction.svg)

```ebnf
conjunction ::= '!'* ( atom | constraint | '(' disjunction ')' ) ( ',' '!'* ( atom | constraint | '(' disjunction ')' ) )*
```

### Query Plan
A query plan gives for each version of a rule a permutation. The permutation dictates the execution order in the loop nest.

![Query Plan](https://souffle-lang.github.io/img/query_plan.svg)

```ebnf
query_plan ::= '.plan' NUMBER ':' '(' ( NUMBER ( ',' NUMBER )* )? ')' ( ',' NUMBER ':' '(' ( NUMBER ( ',' NUMBER )* )? ')' )*
```
         
{% include links.html %}
