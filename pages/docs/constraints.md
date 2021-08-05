---
title: Constraints
permalink: /constraints
sidebar: docs_sidebar
folder: docs
---
Constraints are predicates in the body of the rule that produces true and false values. Constraints can be equalities, inequalities, and string checks such as containment and string matching. 

Constraint **contains(*string1*, *string2*)** is used to check if the latter string contains the former string.

```prolog
.decl stringTable(t:symbol)
.decl substringTable(t:symbol)
.decl outputData(substr:symbol, str:symbol)
.output outputData
outputData(x,y) :- substringTable(x), stringTable(y), contains(x,y).
stringTable("aaaa").
stringTable("abba").
stringTable("bcab").
stringTable("bdab").
substringTable("a").
substringTable("ab").
substringTable("cab").
```
The output would be:
```
a	aaaa
a	abba
a	bcab
a	bdab
ab	abba
ab	bcab
ab	bdab
cab	bcab
```

Constraint **match** is used to check if the latter string matches a wildcard pattern specified in the former string.
```prolog
.decl inputData(t:symbol)
.decl outputData(t:symbol)
.output outputData
outputData(x) :- inputData(x), match("a.*",x).
inputData("aaaa").
inputData("abba").
inputData("bcab").
inputData("bdab").
```
The output would be:
```
aaaa
abba
```

Souffle supports inequalities and equalities, i.e., **&#62;**, **&#60;**, **&#61;**, **&#33;&#61;**, **&#62;&#61;** and **&#60;&#61;**. Examples of this are given below.
```
A(a,c) :- a > c.
B(a,c) :- a < c.
C(a,c) :- a = c.
D(a,c) :- a != c.
E(a,c) :- a <= c.
F(a,c) :- a >= c.
```

## Syntax 
In the following, we define constraints and argument values in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Constraint

A constraint is a predicate in the body of a rule. It either produces a true or a false value. Constraints can be inequalities and equalities for primitive types, and there are string constraints for matching and containment ship. 

![constraint](https://souffle-lang.github.io/img/constraint.svg)

```ebnf
constraint ::= argument ( '<' | '>' | '<=' | '>=' | '=' | '!=' ) argument
           | ( 'match' | 'contains' ) '(' argument ',' argument ')'
           | 'true'
           | 'false'
```


{% include links.html %}
