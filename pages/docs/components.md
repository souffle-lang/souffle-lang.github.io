---
title: Components
permalink: /components
sidebar: docs_sidebar
folder: docs
---

A component is a modular part in a program that may encapsulate component elements 
including relation declarations, type declarations, rules, facts, directives, and
other components. A component has component declarations and a component
instantiations.

A component declaration starts with the keyword `.comp` followed by the name of the
component and a block between curly braces (i.e. `{ ... }`) containing the 
component elements. 

For example, 
```
.comp MyComponent {
    .type myType = number
    .decl TheAnswer(x:myType)    // component relation
    TheAnswer(42).               // component fact
}
```
declares a new component with the name `MyComponent`. The component elements are a type `myType`, 
a relation `TheAnswer`, and a fact. 

We instantiate a component using the `.init` keyword followed by a unique name of the instance, 
the `=` symbol, and the name of the component declaration. 

In this example,
```
.init myInstance1 = MyComponent

.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x).
.output Test
```
we instantiate the component `MyComponent` with the name `myInstance1`. 
Internally, Souffle flattens the component instantiation
creating for each component instantiation its own namespace. 
The qualified names of component elements
are preprended using the name of the instantiation.

For the above example, Souffle internally expands the component instantiation as follows:
```
.type myInstance1.myType = number
.decl myInstance1.TheAnswer(x:myType)    // relation of myInstance1
myInstance1.TheAnswer(42).               // fact of myInstance1

.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x).
.output Test
```
You can display the expansion using the specifying the command-line 
option `--show=transformed-datalog`. 

In the following example, we have two component instantiation of 
`MyComponent`:

```
.init myInstance1 = MyComponent
.init myInstance2 = MyComponent
myInstance2.TheAnswer(33).

.decl Test(x:number)
Test(x) :- myInstance1.TheAnswer(x). 
Test(x) :- myInstance2.TheAnswer(x). 
.output Test
```
producing internally the following logic program,
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
Note that the two relations `TheAnswer` of both component instantiations are 
disambiguated by the prefix `myInstance1` and `myInstance2` to avoid a name clash.

If rules/facts are defined in a component for which there are no relation 
declarations, no prefices are added and the resolution is deferred to the actual 
component instatiation (which may contain in another component and so forth). 

For example, in the following example we have facts and rules defined whose 
relations reside outside of the component in which they are defined.
```
.decl Out(x:number) 
.comp A { 
   .decl R(x:number) 
   .comp Count { 
       R(1).                  // fact accessing R outside of Count
       R(x+1):- R(x), x<10.   // rule accessing R outside of Count
   } 
   .init myCount = Count      // instantiate Count
   Out(x) :- R(x).            // rule accessing Out outside of A 
}
.init myA = A
.output Out
```
Souffle expands the code above as follows:
```
.decl Out(x:number) 
.decl myA.R(x:number) 
myA.R(1).
myA.R((x+1)) :- myA.R(x), x < 10.
Out(x) :- 
   myA.R(x).
.output Out
```
Prefices are prepended to relations in rules and facts 
if they are instantiated in the local scope. 

## Type Parametrization

Components can be parametrized by unqualified type names.  In the example,
```
.comp ParamComponent<myType> {
    .decl TheAnswer(x:myType)    // component relation
    TheAnswer(42).               // component fact
    .output TheAnswer            // component output directive
}
.init numberInstance = ParamComponent<number>
.init floatInstance = ParamComponent<float>
```
two different instances are generated whose relation's attribute
are either of type `number` or `float`. Internally, 
Souffle produces the program showing the different attribute types 
of the instantiated relations `TheAnswer`.
```
.decl numberInstance.TheAnswer(x:number)     // relation of numberInstance
numberInstance.TheAnswer(42).                // fact of numberInstance
.output numberInstance.TheAnswer             // output directive of numberInstance

.decl floatInstance.TheAnswer(x:float)       // relation of floatInstance
floatInstance.TheAnswer(42).                 // fact of floatInstance
.output floatInstance.TheAnswer              // output directive of floatInstance
```


Parameters can also be used to parameter a component name in a component instantiation. 
This mechanism is useful to model different behaviours in component since the component 
instantiation is controlled by the user of the component.  For example,
```
.decl R(x:number)
.comp Case<Selector> {
   .comp One { 
     R(1). 
   } 
   .comp Two { 
     R(2).
   } 
   .init selection = Selector // instantiation depending on type parameter "Selector" 
} 
.init myCase = Case<One> 
.output R
```
produces internally the following program 
```
.decl R(x:number) 
R(1). 
.output R
```
since `One` was passed on in the component instantiation of `myCase`. 

By changing the instantiation from `.init myCase = Case<One>` to `.init myCase = Case<Two>`,
the fact `R(2).` would be issued for relation `R`.

## Inheritance

One component can inherit from multiple super components using inheritance. 
The component elements of the super-components are passed on to the 
sub-component. Using inheritance is useful for larger
programs and libraries and avoids code duplication. 

In the following example, 
```
.comp Base1 {
    .type myNumber = number
    .decl TheAnswer(x:myNumber)
    TheAnswer(42).
}
.comp Base2 { 
    TheAnswer(41). 
}
.comp Sub  : Base1, Base2 { // inherit from Base1 and Base2
    .decl WhatIsTheAnswer(n:myNumber)
    WhatIsTheAnswer(n) :- TheAnswer(n).
    .output WhatIsTheAnswer
}
.init mySub = Sub
```
the components `Base1` and `Base2` pass on all component elements to the component `Sub`. 
Souffle produces internally the following code for the component instantiation `mySub`:
```
.type mySub.myNumber = number
.decl mySub.TheAnswer(x:mySub.myNumber) 
.decl mySub.WhatIsTheAnswer(n:mySub.myNumber) 
mySub.TheAnswer(42).
mySub.TheAnswer(41).
mySub.WhatIsTheAnswer(n) :- mySub.TheAnswer(n).
.output mySub.WhatIsTheAnswer
```

## Overridable Relations

If a relation, declared in a super component is declared as `overridable`, 
a sub component may override its associated rules and facts 
from its super component, i.e., the facts and rules of the super component 
are discarded if the sub component uses the directive `override` for this relation. 

For this example, 
```
.comp Base {
    .decl R(x:number) overridable
    R(1).
    R(x+1) :- R(x), x < 5. 
    .output R
}
.comp Sub : Base {
    .override R
    R(2).
    R(x+1) :- R(x), x < 4. 
}
.init mySub = Sub
```
the sub-components discards fact `R(1).` and rule `R(x+1) :- R(x), x < 5.` 
since the relation `R` has been declared with the relation qualifier `overrideable`
and the sub-component has the `.override R` directive, which let Souffle drop the 
facts and rules of its super component. Souffle will produce internally the following
code for the sub-component instantiation `mySub`:
```
.decl mySub.R(x:number)overridable 
mySub.R(2).
mySub.R((x+1)) :- 
   mySub.R(x),
   x < 4.
.output mySub.R
```

In the following, we show a use case for override in 
a more practical setup. For example, a more precise 
version of an existing analysis can be implemented 
by using the  override semantics as follows,
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
In this example, PrecisePointsto inherits all the relations
from AbstractPointsto, but only implements the HeapAllocationMerge 
relation differently. This feature avoids code duplications when 
we need several implementations of a generic analysis with small 
variations and want to overwrite behaviour of the super component.

## Type Parametrization and Inheritance

Type parameters of super-components can be explicitly specified 
in sub-component declarations. For example, 
```
.comp A<T> { .... }
.comp B<K> : A<K> { ... }
```
defines a sub-component `B` with parameter `K` that is used to instantiate 
the super component with parameter `K` for inheriting for `B`.

The type parameter can also be used as the base class
```
.comp A<T> : T { ... }
```
building a selective inheritance based on the type parameter `T`.
The instantiation of `A<T>` defines which super component `A` 
is inheriting from.

With inheritance, complex component instantiatons are
implementable. In the example below
```
.init reach = Reachability<Graph<number>>   // syntax error
```
we would like to us a complex component parameter `Graph<number>`,
but leads to syntax parameter because the parameter is not a 
simple identifier. By introducing a new component, the above
instantiation can be rewritten as followes
```
.comp NumberGraph : Graph<number> { } // NumberGraph inherits  
.init reach = Reachability<NumberGraph>
```

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
