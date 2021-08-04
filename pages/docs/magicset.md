---
title: Magic-Set
permalink: /magicset
sidebar: docs_sidebar
folder: docs
---

The magic set transformation is a technique used to reduce the number of irrelevant tuples computed during the evaluation of a Souffl&eacute; program. Irrelevant tuples are avoided by specialising program clauses based on constraints that appear in the body and the bodies of their dependencies. An example of rule specialisation is transforming the following program snippet

```
a(x,y) :- b(x,z), c(y,z).
query(x) :- a("foo", x), !g(x).
```

into

```
a("foo", y) :- b("foo", z), c(y,z).
query(x) :- a("foo", x), !g(x).
```

The magic set transformation generalises this concept of specialisation by assessing the flow of constraints between rules and their dependencies using a process called adornment. The program is altered based on the computed adornment in a sound and complete manner to produce new specialised rules, called *magic rules*, that are added to existing clauses. The final effect is an equivalent program with a potentailly significantly reduced number of intermediate tuples.

## Using the Transformation
To enable the magic set transformation on a given program *foo.dl* for a subset of the program relations *A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>n</sub>*, use the `magic-transform` option as follows:

<code> souffle --magic-transform="A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>n</sub>" foo.dl
</code>

To enable the transformation on the full program, use

<code> souffle --magic-transform=* foo.dl
</code>

Magic set can also be enabled within the program using pragma directives:

<code> .pragma "magic-transform" "A<sub>1</sub>, ..., A<sub>n</sub>" </code>

The complete magic set transformation process can be observed by generating the debug report when the `magic-transform` option is enabled.

The option `magic-transform-exclude` can be used in the same way to disable magic set transformation changes on the given relations. Note that this option also implies `inline-exclude` for the specified relations.

## The Algorithm
The algorithm used was based on the work presented in [this paper](http://www.sciencedirect.com/science/article/pii/074310669190030S). The paper details an algorithm for positive datalog programs, and a second for normal programs (containing negations). Only the algorithm for positive datalog has been implemented in the Souffl&eacute; compiler at this point. Relations with negations in their body or the body of a dependency will not be transformed.

The algorithm consists of three main stages:
### Constraint Normalisation
Constraint normalisation is an independent transformation occurring at the start of the process. Constants used in atoms are replaced with variables that are constrained. For example, the following clause:

```
a(x) :- b("foo", x).
```

becomes:

```
a(x) :- b(var, x), var = "foo".
```

### Adornment
The adornment stage consists of an analysis of the program relations in a top-down approach that provides each atom in each clause an adornment. An adornment for an atom *P(a<sub>1</sub>,...,a<sub>n</sub>)* is a length n string consisting of *b*'s (for '*bound*') and *f*'s (for '*free*'), where the character at index *i* is a '*b*' if and only if the *i*<sup>th</sup> argument of the atom is bound by a constraint.

Adornment analysis begins with the output query, say <code> query(x<sub>1</sub>,...,x<sub>n</sub>) </code>, with an adornment '*f...f*' (as all arguments of the output query begin free). A body atom is then selected to be adorned first, marking each argument with a *b* if its value is constrained, or with an *f* otherwise. Once the adornment of a particular atom is computed, all its arguments are now considered bound when adorning the remaining atoms. A second atom is then chosen to be adorned, and so on, until all body atoms are adorned. If a predicate *P(x<sub>1</sub>,...,x<sub>n</sub>)* is seen with adornment *a* for the first time, then the adornment process is repeated for all clauses with *P* as the head, with each argument of the head atom set as bound or free based on the adornment *a*.

The order in which the body atoms are adorned sets the evaluation order when the program is compiled. The technique used to choose the next atom to adorn is called the *Sideways Information-Passing Strategy* (SIPS). The SIPS chosen is fundamental to the effectiveness of the magic set transformation, as it determines the flow of binding patterns between sibling atoms in a given clause body; to minimise the computation of irrelevant tuples, we generally prefer SIPS that maximise the number of bound arguments. The current implementation is very basic and always selects the left-most atom with at least one bound argument. If none exist, then the left-most EDB atom is selected. If none exist, then the left-most IDB atom is selected.

The adornment is calculated for each output query separately.

As an example, consider the following output query:

```
query(x) :- a(x,y,z), c(z,y), y = "foo".
```

As stated above, we begin with a default adornment of *f* for the output query, indicating that the variable *x* is free. Our SIPS described above chooses the first available atom with a bound argument, which is `a(x, y, z)` in this case. The corresponding adornment for `a(x,y,z)` is hence *fbf*, as the variable *y* is bound by a constraint, while the remaining arguments are free. The variables *x* and *z* are now bound by this atom, however. Only one atom remains, `c(z,y)`, which has adornment *bb*. The final adorned predicate is hence:

```
query_f(x) :- a_fbf(x,y,z), c_bb(z,y), y = "foo".
```

If *a* is not an EDB predicate and we have not seen the adornment *fbf* applied to it yet, then we repeat the adornment process for all the clauses with *a* as the clause head, using *fbf* as the base adornment for its arguments. The same applies to *c* with adornment *bb*.

### The Magic Set Transformation
The final stage of the process is the actual transformation step based on the adorned predicates generated.

The transformation involves generating a set of specialising rules, called *magic rules*, that are then added to the program. Magic rules propagate constraints upwards from a clause to its dependencies, allowing for irrelevant tuples to be skipped in a manner similar to the example in the introduction.

The algorithm for the final transformation step is as follows: <br>

```
For all adorned predicates C = A_a :- A1_a1, ...,An_an:
		For each IDB literal Ai in the body:
			Add the clause 'magic(Ai_ai) :- magic(A_a), A1_a1, ..., A(i-1)_a(i-1)' to the program.
		Replace C = 'A_a :- T' with 'A_a :- magic(A_a), T'.
```

Here, `magic(A_a)` denotes the magic identifier corresponding to the predicate *A* with adornment *a*. Suppose *a* denotes an adornment with *k* bound arguments. Then, `magic(A_a)` is a predicate of arity *k*, where each argument corresponds to a bound variable in the adornment.

The final program is the union of all the transformed adorned predicates with the generated magic rules, as computed above.

## Limitations
The magic set implementation currently skips transforming all relations containing negations, functors, or aggregates in their body or the body of a dependency. An algorithm exists in the [paper referenced above](http://www.sciencedirect.com/science/article/pii/074310669190030S) that details an algorithm for datalog programs containing negations.

The current SIPS used is also very basic at this point, but will soon be updated.
