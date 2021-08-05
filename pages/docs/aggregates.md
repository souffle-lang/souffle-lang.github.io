---
title: Aggregates and Generative Functors
permalink: /aggregates
sidebar: docs_sidebar
folder: docs
toc: false
---

## Aggregates 

Aggregate functions `min`, `max`, `sum`, and `count` are available in souffle. Here are syntactically correct uses of each of these:

```
B(y) :- y = max z : A("A", z).
B(z) :- z = min x+y : { A(x), A(y), C(y) }.
B(s, c) :- W(s), c = count : { C(s, _) }.
B(s, count : { C(s, _) }) :- W(s).
B(n, m) :- A(n, m), B(m, s), 2 * s = 2 * sum s : { A(n, s) } + 2.
C(n) :- D(n), B(n, max p : { A(n, p) }).
```

These can be helpful for computing statistics. As an example, if `Prime` contained the first 60 prime integers, you could find the sum of all primes in this table with the rule

```
.decl S(x:number)

S(n) :- n = sum y : Prime(y).
```
`S` will be a table with a single value, the sum of the first 60 prime integers.

You can place any numeric expression in place of the `y`. For example, `x+y`, `x+1`, or `c` where `c` is any constant. Of course, any variable in the numeric expression needs to be grounded by the aggregate body or the outer scope.

An aggregate can also have an unbound result when the aggregate body cannot be satisfied. For example, consider

```
.decl a(s:symbol, x:number)
.decl b(x:number)

a("B", 1).
a("B", 2).

b(n) :- n = min Z : a("A",Z).
```
Here, the expression `min Z : a("A",Z)` computes the smallest `Z` such that the value ["A",Z] is present in relation `a`. There is no such element, so the result is unbound and the containing body will not be satisfied.

You can also use wild cards to collect statistics over a subset of columns of a table. For example, in the term 

```
min Z : a(_,Z).
```

the overall smallest value of the second attribute is obtained, independently of the first attribute. However, the operation will *fail* if `a` is empty. 

If the aggregate body contains more than one literal, you need to use curly brackets as in

```
sum y : { A(y), !B(y) }.
```

In all of the previous examples, the result of the aggregate function is a single, fixed value. If your aggregate computation relies on a variable that is grounded from the outside scope, you will have an aggregate function result for every valid assignment of this variable. For example,

```
res(X,Y) :- a(X,_), Y = min Z : a(X,Z).
```
if `a` contains only the tuples `("B", 1), ("C", 2)`, then valid assignments of `X` are `"B"` and `"C"`. `res` maps each of these elements of the first column of `a` to its minimal corresponding value in the second column.

Here is another example, where we run through valid assignments of `n` in order to bound the values included in a sum.
```
B(x) :- A(n), x = sum y : { C(y), y < n }. 

```
Here is an example of an aggregate where the sum of two values is minimised.
```
min Z+Y : { a(A,Z), a(B,Y), A!=B }.
```
The given term computes the smallest sum of values assigned to two different elements in `a`.

### The Witness Problem

You cannot export information from within the body of an aggregate. This means that you cannot ground a variable from within the scope of the aggregate body and expect this grounding to transfer to the outer scope. In this example, `y` is grounded by the inner scope of the aggregate and we attempt to discover its value within the head `a(x, y)`. 

```
a(x,y) :- b(x), x = sum z : { c(y, z) }.
```
Here is another example where we try to find the name associated with the minimum value in a table.

```
.decl family(name:symbol, age:number)
.decl youngest(name:symbol, age:number)

family("Alissa", 10).
family("Maria", 46).
family("Mark", 50).

youngest(p, n) :- n = min x : family(p, x).

```
This will raise a Witness Problem Error because we attempt to export the assignment of `p` that gives the minimum age.

### Nested Aggregates Are Disallowed

You cannot nest aggregates within aggregates. For example, 

```
D(s) :- s = sum y : { Prime(n), y = sum z : { Prime(z), z < n }, y < 10 }.
```
This rule attempts to sum together all the increasing sums of prime numbers that are less than 10, starting from the first prime number. This is currently disallowed within the souffle language.

## Generative Functors


Souffle supporst generative range functors that produce values within a numeric range. 
The format of the functors are 
```
range(bgn, endExcl, step = sign(endExcl - bgn))
```
This functor is intended to roughly match Python's slice indexing semantics.

For example, 
 - `range(0, 5)` produces the sequence `0, 1, 2, 3, 4`.
 - `range(5, 3.75, -0.5)` produces the sequence `5, 4.5, 4`.
 - `range(5, x, 0)` produces the sequence `5` iff `x != 5` (i.e. non-empty range).

The program
```
.decl A(x:number)
A(x) :- x = range(1,5,1). 
.output A
```
produces the values one to four in relation A. 

## Syntax 
In the following, we define rule declarations in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Aggregator

![Aggregator](https://souffle-lang.github.io/img/aggregator.svg)

```ebnf
aggregator  ::= (( ( 'max' | 'mean' | 'min' | 'sum' ) argument | 'count' ) ':' ( '{' disjunction '}' | atom )) | 
                'range' '(' argument ',' argument (',' argument)? ')'
```

{% include links.html %}
