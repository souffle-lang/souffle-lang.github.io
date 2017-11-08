---
layout: docs
title: Inlining for Relations
permalink: /docs/inlining
---

Souffl&eacute; offers the ability to manually select one or more program relations to be inlined, i.e., a substitution of the relations are performed. For example, 

```
.decl R(arg:number) inline
```

replaces every occurrence of predicate ```R``` by its rules, i.e.,  
the `inline` keyword directs Souffl&eacute; to transform the program into an equivalent form that does not contain the marked relations. On a high level, the transformation is essentially achieved by replacing all occurrences of the marked relation with the bodies of each of its rules, while ensuring program semantics are preserved.

# Example

Consider the following Souffl&eacute; program:

```
.decl natural_number(x:number)
natural_number(0).
natural_number(x+1) :- natural_number(x), x < 10000.

.decl natural_pairs(x:number, y:number) inline
natural_pairs(x,y) :- natural_number(x), natural_number(y).

.decl query(x:number)
query(x) :- natural_pairs(x,y), x < 5, y < x.
```

The program will be transformed into:

```
.decl natural_number(x:number)
natural_number(0).
natural_number(x+1) :- natural_number(x), x < 10000.

.decl query(x:number)
query(x) :- natural_number(x), natural_number(y), x < 5, y < x.
```

Note that, without impacting the readability of the original code, the new program avoids computing all tuples for the intermediate relation `natural_pairs`, which would contain 100,000,000 entries. The result is a signficant speed boost, as millions of unnecessary tuples no longer need to be computed, without having to compromise code quality.

As inlining can potentially bind variables to specific constants, the process may also provide gains when used in conjunction with the Magic Set transformation, which specialises rules based on constant propagation.

In general, inlining is most appropriate when used for large relations where it is not beneficial to compute and cache all tuples beforehand.

# Inlining Algorithm
Suppose the following relation was marked to be inlined:

```
.decl a(x:number, y:number) inline
a(x,y) :- d(x,x), e(y).
a(x,y) :- f(y,x).
```

and the relation appears in the following rule:

```
b(x) :- c(x,z), b(y), a(y,z).
```

The first stage of inlining is to match the arguments of the body atom, `a(y,z)`, with the heads of each of its rules. Argument-matching is done through a process called unification, where the most general unifier is found. In this case, both rules have the head `a(x,y)`. It is important to note, however, that the variable `y` appearing in the body atom `a(y,z)` is distinct from the `y` appearing in the head atom `a(x,y)`. An &alpha;-reduction must hence be performed to avoid naming conflicts. In this case, we can rename the head atom of the rules to be `a(x0,y0)`, producing the following equivalent rules for `a`:

```
a(x0,y0) :- d(x0,x0), e(y0).
a(x0,y0) :- f(y0,x0).
```

Unification can now proceed. In this case, `a(y,z)` must be unified with `a(x0,y0)` for both rules, which can be done by substituting `x0` with `y`, and `y0` with `z` in both rules. As a result, we must substitute in the following rule bodies to replace `a(y,z)`.

```
d(y,y), e(z).
f(z,y).
```

Note that unification still works in the presence of constants, in which case variables in the body become bound, possibly allowing for further optimisation.

There are now two unified bodies that can replace the original atom. Since both bodies must be considered, two new rules are created to replace the original rule for `b`, hence producing the following final program:

```
b(x) :- c(x,z), b(y), d(y,y), e(z).
b(x) :- c(x,z), b(y), f(z,y).
```

More complex techniques for inlining arise when dealing with negation, records, aggregators, and so on, but they are all centred on removing the relation whilst preserving program equivalence.

# Limitations
Inlining works in all situations, provided the following conditions are met:
* Relations marked as `input`, `output`, or `printsize` cannot be inlined, as they are semantically necessary.
* There cannot be a cycle in the precedence graph where *every* node in the cycle is inlined.
    * More specifically, let G be the precedence graph, and let G' be the subset of G containing only inlined nodes. Then, G' cannot contain a cycle.
    * The restriction is theoretically necessary, as otherwise no ordering for the inlining process can be imposed such that all relations in the cycle are removed.
* The counter argument, `$`, cannot be used as an argument or in the rule body of an inlined relation.
* At the moment, relations appearing in aggregators cannot be inlined, though this is only a restriction in practice due to the way certain functors are handled.
