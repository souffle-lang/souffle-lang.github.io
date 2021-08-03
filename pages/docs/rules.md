---
title: Rules
permalink: /rules
sidebar: docs_sidebar
folder: docs
---
A rule is a definite horn clause that has one or more predicates as a head, and one or more 
literals in the body. The literals are predicates, negated 
predicates, or constraints.
All variables of a rule must be *grounded*, i.e., a variable must occur
at least once as an argument of a positive predicate in the body of
a rule.  Soufflé permits arithmetic and string functors assuming
that their arguments are grounded.

 ```A``` and ```B``` with two number attributes.
```prolog
.decl A, B(x:number, y:number)  // declaration of relation B
.input A                     // read A 
B(x,y) :- A(x,y).            // rules of relation B
B(x,z) :- A(x,y), B(y,z).
.output B
```
Relation ```B``` has two rules: ```B(x,y) :- A(x,y).``` and ```B(x,y) :- A(x,y), B(y,z).``` The first rule says, there is a tuple in `B` if this tuple shows up in `A`. The second rule says there is a tuple `(x,z)` in `B`, if there is a tuple in `(x,y)` in `A` and a tuple `(y,z)` in `B`.

## Multiple Heads
Rules can have multiple heads:
```prolog
A(x,y), C(x,y) :- B(x,y). 
```
which is syntactic sugar for
```prolog
A(x,y) :- B(x,y). 
C(x,y) :- B(x,y). 
```

## Negation in Rules
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

## Disjunction
A rule of the form
```prolog
LivesAt(person, building) :-
    Owner(owner, building),
    ( person=owner ; Housemate(owner, person) ).
```
expresses the rule that a person lives in a building if they are the owner, or a housemate of the owner. Thus the conditions `person=owner` and `Housemate(owner, person)` are joined by `;` to indicate that either must hold.

## Query Plan
Rules may have qualifiers, which are used to change the execution behavior / semantics. 
Qualifiers are used to set a query plan for a rule. The qualifer `.plan` let's the programmer chose a query plan for a rule.  

## Syntax 
In the following, we define type declarations in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced using [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Rule

![Rule](https://souffle-lang.github.io/img/rule.svg)

```ebnf
rule ::= atom ( ',' atom )* ':-' disjunction '.' query_plan?
```

### Qualified Name

![Qualifier Name](https://souffle-lang.github.io/img/qualified_name.svg)

A qualified name is a sequence of identifiers separated by `.` to disambiguate relations that are instantiated by components.

```ebnf
qualified_name ::= IDENT ( '.' IDENT )*
```

### Atom

![Atom](https://souffle-lang.github.io/img/atom.svg)

```ebnf
atom ::= qualified_name '(' ( argument ( ',' argument )* )? ')'
```

### Disjunction

![Disjunction](https://souffle-lang.github.io/img/disjunction.svg)

```ebnf
disjunction ::= conjunction ( ';' conjunction )*
```

### Conjunction

![Conjunction](https://souffle-lang.github.io/img/conjunction.svg)

```ebnf
conjunction ::= '!'* ( atom | constraint | '(' disjunction ')' ) ( ',' '!'* ( atom | constraint | '(' disjunction ')' ) )*
```

### Query Plan

![Query Plan](https://souffle-lang.github.io/img/query_plan.svg)

```ebnf
query_plan ::= '.plan' NUMBER ':' '(' ( NUMBER ( ',' NUMBER )* )? ')' ( ',' NUMBER ':' '(' ( NUMBER ( ',' NUMBER )* )? ')' )*
```
         
{% include links.html %}
