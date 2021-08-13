---
title: Components
permalink: /components
sidebar: docs_sidebar
folder: docs
---

A component is a modular part in a program that may encapsulate component elements 
including relation declarations, type declarations, rules, facts, directives, and other components. 
A component has component declarations and a component instantiations.

A component declaration starts with the keyword `.comp` followed by the name of the component and a block between `{` and `}`
containing the component elements of the component. 

For example, 
```
.comp MyComponent {
    .type myType = number
    .decl TheAnswer(x:myType)    // component relation
    TheAnswer(42).               // component fact
}
```
declares a new component with the name `MyComponent`. The component elements are a type `myType`, 
a relation `TheAnswer` and a fact. 

We instantiate a component using the `.init` keyword followed by a unique name of the instance, 
the `=` symbol, and the name of the component declaration. 

In this example,
```
.init myInstance1 = MyComponent
.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x).
.output Test
```
we instantiate the component `MyComponent` with the name `myInstance1` and the name `myInstance2`. 
Internally, Souffle flattens the component instantiation
creating for each component its own namespace. The names of the component elements
are extended using the instantiation name as a prefix. 

For the above example, Souffle internally expands the component instantiation as follows:
```
.type myInstance1.myType = number
.decl myInstance1.TheAnswer(x:myType)    // relation of myInstance1
myInstance1.TheAnswer(42).               // fact of myInstance1
.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x).
.output Test
```

In this example, we have two component instantiation of `MyComponent`:
```
.init myInstance1 = MyComponent
.init myInstance2 = MyComponent
myInstance2.TheAnswer(33).

.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x). 
Test(x) :- myInstance2.TheAnswer(x). 
.output Test
```
producing internally the following logic program: 
```
.type myInstance1.myType = number
.decl myInstance1.TheAnswer(x:myType)    // relation of myInstance1
myInstance1.TheAnswer(42).               // fact of myInstance1
.type myInstance2.myType = number
.decl myInstance2.TheAnswer(x:myType)    // relation of myInstance1
myInstance2.TheAnswer(42).               // fact of myInstance1
myInstance2.TheAnswer(33).

.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x).
Test(x) :- myInstance2.TheAnswer(x).
.output Test
```
Note that the two relations `TheAnswer` of both component instantiations are disambiguated by the prefix `myInstance1` and `myInstance2` to avoid name clashes in the program.

## Type Parametrization

Components can be parametrized by unqualified type names. 

```
.comp ParamComponent<myType> {
    .decl TheAnswer(x:myType)    // component relation
    TheAnswer(42).               // component fact
}
.init floatInstance = ParamComponent<number>
.init unsignedInstance = ParamComponent<unsigned>
.init floatInstance = ParamComponent<float>
```


If the parameter is meant to be another component, it can be instantiated:

```
.comp Reachability<TGraph> {
    .init graph = TGraph
    // ...
    reach(u, v) :- graph.edge(u, v).
}
```

When initializing parametrized component, we must provide actual parameters:

```
.init g = Graph<number>
.init reach = Reachability<MyGraph>
```

Note that the following is not possible:

```
.init reach = Reachability<Graph<number>>
```

This limitation can be overcome with inheritance:

```
.comp NumberGraph : Graph<number> {}
.init reach = Reachability<NumberGraph>
```

Note that instantiation of `TGraph` inside `Reachability`
will create new copies of `TGraph` relations (specifically relations
of whatever is used as actual parameter for `TGraph`).

## Inheritance
One component can inherit from another. Inheritance in this case is purely
injection of rules from the base component into its subcomponent. The syntax is as follows:

```
.comp Base {
    .decl TheAnswer(x:number)
    TheAnswer(42).
}

.comp Child : Base {
    .decl WhatIsTheAnswer(n:number)
    WhatIsTheAnswer(n) :- TheAnswer(n).
}
```


## Type Parametrization and Inheritance
Type parameters can passed around when inheriting. Example:

```
.comp A<T> { .... }
.comp B<K> : A<K> { ... }
```

Although it is not recommended, the type parameter can be used as the base class:

```
.comp A<T> : T { ... }
```

## Overridable Relations
If a relation, declared in a super component is overridable, it would be possible to override its associated rules in the sub-component while inheriting the rest of the relations from the super-component.
A relation in a sub-component with the same signature as a relation in the super component can override the super component's relation if it has `overridable` keyword.

```
.comp A {
    .decl Rel(x:number) overridable
    Rel(1).
}
```
Since relation `Rel(x:number)` in the component `A` has `overridable` keyword, any sub-component of `A` can override this relation using `.override` keyword.

```
.comp B : A {
    .override Rel
    Rel(2).
}
```
The override semantic replaces the clauses of all super-components by clauses of the sub-component. We omit the actual relation declaration in the sub-component because changing the signature of the relation would cause serious havoc with the use of the relation outside the scope of the sub-component.

As another example, we can use override semantics to implement a more precise version of an existing analysis by overriding some relations:

```
.comp AbstractPointsto{
    .decl HeapAllocationMerge(heap,mergeHeap) overridable
    HeapAllocationMerge(heap,"<<string-constant>>") :-
        StringConstant(heap).
    // ...
}

.comp PrecisePointsto : AbstractPointsto{
    .override HeapAllocationMerge
    HeapAllocationMerge(heap,"<<string-constant>>") :-
        StringConstant(heap),
        !ClassNameStringConstant(heap),
        !SimpleNameStringConstant(heap).
}

.init precise_pointsto = PrecisePointsto
```
In this example, PrecisePointsto inherits all the relations from AbstractPointsto, but only implements the HeapAllocationMerge relation differently. This feature avoids code duplications when we need several implementations of a generic analysis with small variations.


## Syntax 
In the following, we define the component model more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Component Declaration
A component declaration may consist of [type declarations](types), [relation declarations](relations), [rules](rules), [facts](facts), [directives](directives), override directives, [component declarations and initialisations](components).

![Component Declaration](https://souffle-lang.github.io/img/component_decl.svg)

```ebnf
component_decl ::= 
  '.comp' component_type ( ( ':' | ',' ) component_type )* 
    '{' 
        ( type_decl | relation_decl | rule | fact | directive | '.override' IDENT | component_init | component_decl )* 
    '}'
```

### Component Initialisation
A component initialisation has a name and the component declaration name with its parameters.  

![Component Initialisation](https://souffle-lang.github.io/img/component_init.svg)

```ebnf
component_init ::= '.init' IDENT '=' component_type
```

### Component Type

A component type is a signature of a component either used for initialisation, super-component references, or declaration of a component.

![Component Type](https://souffle-lang.github.io/img/component_type.svg)

```ebnf
component_type ::= IDENT ( '<' IDENT ( ',' IDENT )* '>' )?
```

{% include links.html %}
