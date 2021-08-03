---
title: Relations
permalink: /relations
sidebar: docs_sidebar
folder: docs
---
# Relations
In Souffle, relations declaraitons define the domain of the relations, their constraints, and the execution behaviour. Souffle uses different relation representations internally that can be chosen by the programmer. Inlining, magic-set transformations, and component behaviour is also controlled by
relation declarations. 

## Relation Representation in Souffle
Relations can be represented using different internal data structures in Soufflé, each exhibiting different performance characteristics. By default, the B-tree is used to store tuples of a relation. However, this default selection can be overridden by users, by specifying a relation qualifier. Currently, the possible data structures are B-tree, Brie, and Eqrel (for equivalence relations).

### B-tree Relations 
The B-tree data structure is used by default, however the *direct* flavour (see below) can be forced by adding the `btree` qualifier to a relation declaration:
```prolog
.decl A(x:number, y:symbol) btree
```

The B-tree is a good default data structure that performs reasonably well in general use cases. In Soufflé, two flavours of the B-tree are offered. The first is a *direct* version, where tuple data is stored directly in the B-tree. The second is an *indirect* version, where the B-tree stores pointers to an external table data structure which holds the actual tuple data.

By default, a relation without qualifiers will be *direct* if it has arity ≤ 6, and *indirect* if it has arity > 6. This choice is made because larger arity tuples take more space in the B-tree data structure, so cache coherence can be improved by using pointers instead.

Note, however, that adding the `btree` qualifier to the relation declaration forces the *direct* B-tree.

More details on the Soufflé B-tree can be found in [this paper](https://doi.org/10.1145/3293883.3295719).

### Brie Relations 
The Brie data structure is a specialised data structure designed for *dense* data. It can be used by adding the `brie` qualifier to a relation declaration.
```prolog
.decl A(x:number, y:symbol) brie
```

The Brie data structure is a specialised form of a trie, or prefix tree. The benefits of using a prefix tree-style data structure for dense data can be illustrated as follows.

Consider a 3-arity relation containing tuples
```
(0,0,0), (0,0,1), (0,0,2)
```
In this case, all tuples share the common prefix `0, 0`, and differ only in the final digit. This data can be described as `dense`, since a common prefix reduces redundant stored information in the prefix tree. On the other hand, a relation containing
```
(0,0,0), (1,2,3), (4,5,6)
```
is not dense, and the prefix tree provides no benefit.

The Brie data structure in Soufflé similarly provides a performance benefit for highly dense data. However, it is slower than the B-tree in the average case, and so its use must be considered carefully.

More details on the Brie can be found in [this paper](https://doi.org/10.1145/3303084.3309490).

### Equivalence Relations
An equivalence relation is a special kind of binary relation which exhibits three properties: *reflexivity*, *symmetry*, and *transitivity*. An equivalence relation could be expressed in Datalog as follows:
```prolog
.decl equivalence(x:number, y:number)
equivalence(a, b) :- rel1(a), rel2(b).      // every element of rel1 is equivalent to every element of rel2

equivalence(a, a) :- equivalence(a, _).     // reflexivity
equivalence(a, b) :- equivalence(b, a).     // symmetry
equivalence(a, c) :- equivalence(a, b), equivalence(b, c).  // transitivity
```

However, this expression of an equivalence relation would require a quadratic number of tuples to be stored. In Soufflé, the Eqrel data structure provides a linear representation of equivalence relations, by using a union-find based algorithm. Eqrel can be used by adding the `eqrel` qualifier to a relation declaration.

The following example demonstrates the use of Eqrel, and is semantically equivalent to the example above. However, using Eqrel carries at best a quadratic speed and memory improvement over the above example.
```
.decl eqrel_fast(x : number, y : number) eqrel
eqrel_fast(a,b) :- rel1(a), rel2(b).
```

More details on Eqrel can be found in [this paper](https://doi.org/10.1109/PACT.2019.00015).

### Nullaries
Nullary relations are special relations. Their arity is zero, i.e., they don't have attributes. 
They are defined as 
```prolog
.decl A()
```
These relations are either empty or contain the empty tuple `()`. Internatlly, they are implemented 
as a boolean variable. 

Semantically, they can be seen as a proposition rather than a relation. 

### Auto Selection
In Soufflé, the default data structure is the B-tree, with the *direct* version for relations with arity ≤ 6, or the *indirect* version for relations with arity > 6.

## Magic-Set Transformation
The relation qualifier `magic` enables the magic-set transformation for a relation. A magic set transformation specialies the evaluation for a Datalog program for a given set of output relations. The magic-set transformation does not always lead to better performance. Souffle provides the relation qualifier `magic` to control the magic-set transformation. More information can be found [here](magicset).

## Inlining Relations
Souffl&eacute; offers the ability to manually select one or more program relations to be inlined, i.e., a substitution of the relations are performed. This may lead to performance gains by re-computing results rather than storing them. For example, 

```prolog
.decl R(arg:number) inline
```

replaces every occurrence of predicate ```R``` by its rules, i.e.,  
the `inline` keyword directs Souffl&eacute; to transform the program into an equivalent form that does not contain the marked relations. On a high level, the transformation is essentially achieved by replacing all occurrences of the marked relation with the bodies of each of its rules, while ensuring program semantics are preserved.

Consider the following Souffl&eacute; program,

```prolog
.decl natural_number(x:number)
natural_number(0).
natural_number(x+1) :- natural_number(x), x < 10000.

.decl natural_pairs(x:number, y:number) inline
natural_pairs(x,y) :- natural_number(x), natural_number(y).

.decl query(x:number)
query(x) :- natural_pairs(x,y), x < 5, y < x.
```
which will be transformed into:

```prolog
.decl natural_number(x:number)
natural_number(0).
natural_number(x+1) :- natural_number(x), x < 10000.

.decl query(x:number)
query(x) :- natural_number(x), natural_number(y), x < 5, y < x.
```

The advantage of inlineing is to improve the readability of the original code. 
The new program avoids computing all tuples for the intermediate relation `natural_pairs`, which would contain 100,000,000 entries. 
The result is a signficant improvement in speed, as millions of unnecessary tuples no longer need to be computed,
without having to compromise code quality.

As inlining can potentially bind variables to specific constants, the process may also provide gains 
when used in conjunction with the Magic Set transformation, which specialises rules based on constant propagation.

In general, inlining is most appropriate when used for large relations where it is not beneficial to compute and cache all tuples beforehand.

### Inlining Transformation
Suppose the following relation was marked to be inlined:

```prolog
.decl a(x:number, y:number) inline
a(x,y) :- d(x,x), e(y).
a(x,y) :- f(y,x).
```

and the relation appears in the following rule:

```prolog
b(x) :- c(x,z), b(y), a(y,z).
```

The first stage of inlining is to match the arguments of the body atom, `a(y,z)`, with the heads of each of its rules. Argument-matching is done through a process called unification, where the most general unifier is found. In this case, both rules have the head `a(x,y)`. It is important to note, however, that the variable `y` appearing in the body atom `a(y,z)` is distinct from the `y` appearing in the head atom `a(x,y)`. An &alpha;-reduction must hence be performed to avoid naming conflicts. In this case, we can rename the head atom of the rules to be `a(x0,y0)`, producing the following equivalent rules for `a`:

```prolog
a(x0,y0) :- d(x0,x0), e(y0).
a(x0,y0) :- f(y0,x0).
```

Unification can now proceed. In this case, `a(y,z)` must be unified with `a(x0,y0)` for both rules, which can be done by substituting `x0` with `y`, and `y0` with `z` in both rules. As a result, we must substitute in the following rule bodies to replace `a(y,z)`.

```prolog
d(y,y), e(z).
f(z,y).
```

Note that unification still works in the presence of constants, in which case variables in the body become bound, possibly allowing for further optimisation.

There are now two unified bodies that can replace the original atom. Since both bodies must be considered, two new rules are created to replace the original rule for `b`, hence producing the following final program:

```prolog
b(x) :- c(x,z), b(y), d(y,y), e(z).
b(x) :- c(x,z), b(y), f(z,y).
```

More complex techniques for inlining arise when dealing with negation, records, aggregators, and so on, but they are all centred on removing the relation whilst preserving program equivalence.

### Inlining Limitations
Inlining works in all situations, provided the following conditions are met:
* Relations marked as `input`, `output`, or `printsize` cannot be inlined, as they are semantically necessary.
* There cannot be a cycle in the precedence graph where *every* node in the cycle is inlined.
    * More specifically, let G be the precedence graph, and let G' be the subset of G containing only inlined nodes. Then, G' cannot contain a cycle.
    * The restriction is theoretically necessary, as otherwise no ordering for the inlining process can be imposed such that all relations in the cycle are removed.
* The counter argument, `$`, cannot be used as an argument or in the rule body of an inlined relation.
* At the moment, relations appearing in aggregators cannot be inlined, though this is only a restriction in practice due to the way certain functors are handled.

## Override 
The relation qualifier `override` controlls whether rules in a relation which is defined in a component can be overwritten in a sub-component. The component model of Souffle is described [here](components).

### Choice Domain / Functional-Dependency Constraint

Programmers can impose a functional dependency constraint for a relation to support a notion of non-determinism.
With choice, it is now much easier to express worklist algorithm in Souffle by specifying functional dependencies on the relations.

A functional dependency `x -> y` on relation `R(x:number, y:number)` ensures that
each `x` in `R` will uniquely define a value of `y`.
For example, during the computation, if a set of tuples `{(1,1), (1,2), (1,3)}`
are about to be inserted into `R`, Souffle only chooses arbitrary one of them
(hence the "non-determinism"), since inserting more than one would break the
functional dependency.

Functional dependency can be enforced on relation using the keyword `choice-domain` during relation
declaration in Souffle with the following syntax:

```
<relation-declaration>      ::= ".decl" <relation-name> "(" <attributes> ")" <choice-domain>
<choice-domain>             ::= "" | "choice-domain" <constraint> { "," <constraint>}
<constraint>                ::= <variable> | "(" <variable> { "," <variable> } ")"
```

Note here, for the sake of brevity, our syntax omits the co-domain (i.e. the right hand side of the arrow). 
Therefore, a choice-domain `D` for a relation with attribute set `X` implicitly defines a 
functional dependency of `D -> X \ D`.

Consider the following example,
```
.decl edge(u:symbol, v:symbol)
.decl st(u:symbol, v:symbol) choice-domain v
.decl startNode(x:symbol)

st("root", x) :- startNode(x).
st(u, v) :- st(_, v), edge(u, v).
```
that calculates a spanning tree on a connected component of an undirected graph.
The first rule simply chooses a start node.
The second rule states that, an edge `(u, v)` is in the spanning tree: if `v` is
reachable in the spanning tree and there is an edge between `u` and `v`.
The `choice-domain v` on `st` ensures that the value of `v` is unique, i.e.,
each node can be visited only once in the spanning tree.

Here is another example,
```
.decl student (s:symbol, majr:symbol, year:number)
.decl professor (s:symbol, majr:symbol)
.decl advisor (s:symbol, year:number, p:symbol) choice-domain (s, year)

advisor(s, y, p) :- student(s, y, m), professor(p, m).
```
which allocates an advisor for each student. The `choice-domain (s, year)`
on `advisor` makes sure that the combination of `(student, year)` is unique,
i.e., student from each year is assigned to a professor only once.

In the following example,
```
.decl d(x:symbol)
.decl list(prev:symbol, next:symbol) choice-domain prev, next

list(p, n) :- d(n), list(_, p).
```
the  an unordered set of elements is ordered using a list.
The program effectively computes a total order over the set.
The rule states that, `p` is before `e` if `p` is already assigned into the total order list.
With the help of the two choice-domains `prev` and `next`: 
`prev` makes sure that for each element, there can only be one unique element after
it; similarly, `next` makes sure for each element, there can be only one unique
element before it. Those constraints defines the property of a total order set.


## Syntax 

In the following, we define type declarations in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced using [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Type Name 

Souffle has pre-defined types such as `number`, `symbol`, `unsigned`, and `float`. Used-defined types have a name. If a type has been defined in a component, the type can be still accessed outside the component using a qualified name. 

![Type Name](https://souffle-lang.github.io/img/type_name.svg)
```ebnf
type_name ::=  "number" | "symbol" |"unsigned" | "float"  | IDENT ("." IDENT )*
```

### Attribute Declaration

An attribute binds a name with a type. 

![Attribute](https://souffle-lang.github.io/img/attribute.svg)

```ebnf
attribute ::= IDENT ":" type_name 
```

### Relation Declaration

A relation declaration declares one or more relations. Each relation has a fixed number of attributes.
The definition of attributes is followed by relation qualifiers. 

![Relation Declaration](https://souffle-lang.github.io/img/relation_decl.svg)

```ebnf
relation_decl ::= '.decl' IDENT ( ',' IDENT )* '(' attribute ( ',' attribute )* ')' ( 'override' | 'inline' | 'magic' | 'brie' | 'btree' | 'eqrel' )* choice_domain
```

### Choice-Domain

A choice-domain imposes a functional dependency constraint. The functional dependency constraint is expressed by its domain only. A domain 
can be either a single attribute or a subset of attributes. 

![Choice Domain](https://souffle-lang.github.io/img/choice_domain.svg)

```ebnf
choice_domain ::= ( 'choice-domain' ( IDENT | '(' IDENT ( ',' IDENT )* ')' ) ( ',' ( IDENT | '(' IDENT ( ',' IDENT )* ')' ) )* )?
```

### Legacy Syntax

The syntax of Souffle changed over time. Older code bases can be still used with 
modern versions of Souffle.  In older versions of Soufflé we used
```prolog
.decl A(x:number) input 
.decl B(x:number) output
B(x) :- A(x). 
```
for loading and storing facts. However, we have now I/O directives which permit to change 
the I/O parameters.
```prolog
.decl A(x:number)
.input A
.decl B(x:number) 
.output B
B(x) :- A(x). 
```
You can enable the old legacy syntax using the command-line flag `--legacy`, but you will receive a warning that this legacy syntax is deprecated.

{% include links.html %}
