---
title: Datalog
permalink: /rules
sidebar: docs_sidebar
folder: docs
---

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

{% include links.html %}
