---
title: Hand-Tuning
permalink: /handtuning
sidebar: docs_sidebar
folder: docs
toc: false
---

# Performance

To achieve high performance in Soufflé, the [Auto-Scheduler](autotuning) can be used.

However, attaining optimal performance is challenging and requires a deep understanding of the engine internals.

It will take time and experience to become effective with performance tuning for Soufflé programs. If you want to study 
highly-tuned Datalog code, I suggest that you study the Soufflé code of DOOP.

Soufflé has a profiler which is the main instrument to guide you with performance tuning. The process of performance 
tuning starts with using a suitable input for your Soufflé program. The runtime of the your program with the chosen input should be acceptable so that the tuning cycle is short. The shorter the cycle is the faster you will make progress in
your tuning effort.

The tuning cycle consists of:
 
 1) Run the Soufflé program with the profile option (`-p <file>`)
 
 2) Run the profiler (`souffleprof <file> -o <html>`)
 
 3) Identify the most dominant performance bottlenecks 
 
 4) Eliminate the most dominant performance bottlenecks 
 
 5) Go to step 1 until the performance is satisfactory for the current input 
 
 6) Choose a larger input for your program with acceptable performance
 
 7) Go to step 1 until the performance of the program becomes acceptable
 

# Relational Schemas 

Performance tuning most likely will focus on relational schemas and rules. Both have an impact on 
the performance. The relational schemas determine how much data is stored for computing your 
problem and hence will influence the runtime of the rules. Also, the access to the data (i.e. how 
they are stored) and the number of relations will determine the overall performance of your
program. 

It is worthwhile to note that there are infinitely many ways to express your relational 
schemas that give you the same result. However, some are more effective than others 
and it is the art of the logic programmer to choose the right relational schemas. 

Sometimes, the concept  of  (database normilization)[https://en.wikipedia.org/wiki/Database_normalization] that 
exploits functional dependencies (cf. Edgar F. Codd) helps to improve the performance.
Whether normalization is fruitful dependens on the case. However, the 
concept of normalization is a powerful tool understanding how relational schemas 
can be rewritten. 


We came across various programs in the past, that used cross-products in their modelling. 
Cross-products are particularly bad because they let the data-explode rather than condense.
For example,
```
#define N 10000
.decl A,B(x:number)
A(1).
A(x+1) :- A(x), x < N.
B(x) :- A(x). 

.decl C(x:number, y:number) 
C(x, y) :- A(x), B(y). 

.decl D(x:number, y:number) 
D(x,x) :- C(x,x).
.printsize D
```
introduces the relation `A`, `B`, `C`, and `D` where relation `C` is a cross-product of relation `A` and `B`. 

The effort of computing and storing relation `C` is quadratic in the size of the inputs of `A` and `B`. 
Your storage and computation complexity jumped from *O(N)* to *O(N^2)*.

This is expensive especially if you don't need all the data of your cross-product. In our example,
the cross-product is later filtered in relation `D`. 
A short fix to this problem is to add relational qualifier `inline` to relation `C` 
```
...
.decl C(x:number, y:number) inline 
...
```
that will substitute the occurrences of predicate C by the right-hand side of its rules. 
This is also called resolution (but only for one relation). After adding the relational
qualifier `inline`, relation `C` will not physically exist while evaluating the program. 

This transformation can also be done by hand and the result is as follows:
```
#define N 10000
.decl A,B(x:number)
A(1).
A(x+1) :- A(x), x < N.
B(x) :- A(x). 


.decl D(x:number, y:number) 
D(x,x) :-  A(x), B(x).
.printsize D
``` 
The runtime complexity of this program is substantially reduced because it will be linear in the size of N. 

# Rule Rewriting 

Rules can be written in many ways to express the same computation. For example, consider a simple transitive closure of a graph:
```
#define N 10000
.decl G,T(x:number)
G(1,2).
G(N,1).
G(x,x+1) :- G(x,_), x < N.

T(x,y) :-  G(x,y).
T(x,z) :-  T(x,y), T(y,z).
.printsize D
```

Although it computes the transitive closure, it is a bad formulation of the solution. A better way of writing the transitive closure is the following:
```
T(x,y) :- G(x,y). 
T(x,z) :- T(x,y), G(y,z). 
```

# Rule Scheduling 
Determining a good execution schedule for a given rule requires some insight regarding the internal operation of the Datalog engine. In general, a rule like
```
C(X,Z) :- A(X,Y), B(Y,Z).
```
is translated into a loop nest similar to
```
for( e1 : A ) {
    for( e2 : B ) {
        if ( e1[1] == e2[0] ) {
            C.insert( e1[0], e2[1] );
        }
    }
} 
```
The cost of this loop is mainly dominated by the number of times the innermost loop body, comprising the condition and the insertion operation, needs to be evaluated. The objective of finding a good schedule is to minimize the number of inner-most loop-body executions.

Obviously, a naive conversion like this would be not very efficient. It would require |A| * |B| iterations of the innermost loop body. Fortunately, the second for loop and the condition can be combined into
```
for( e1 : A ) {
    for( e2 : { e in B | e.[0] == e1.[1] } ) {
        C.insert( e1[0], e2[1] );
    }
} 
```
Now, to speed up the execution, an ordered index on B can utilised to more effectively enumerate all the entries in B where the first component is equivalent to the second component of the currently processed tuple in A. This way, the execution cost is reduced to |A| * |fanout of A in B| which in most cases is considerably smaller.

In an extreme case, e.g. given by
```
C(X,Y) :- A(X,Y), B(X,Y).
```
which would be naively converted to
```
for( e1 : A ) {
    for( e2 : B ) {
        if ( e1[0] == e2[0] && e1[1] == e2[1] ) {
            C.insert( e1[0], e1[1] );
        }
    }
} 
```
the operation is contracted to
```
for( e1 : A ) {
    if ( e1 in B ) {
        C.insert( e1[0], e1[1] );
    }
} 
```
eliminating an entire loop, significantly reducing the number of times the loop body is processed.

## Auto-Scheduling

Soufflé's [Auto-Scheduler](autotuning) can be used to automatically derive high quality schedules for a Soufflé program.

## Sideways Information-Passing Strategy

A *Sideways Information-Passing Strategy* (SIPS) is a heuristic used to determine the rule schedule (query plan). A SIPS heuristic is static and does not take advantage of run-time statistics. Therefore, the schedules chosen by any particular SIPS will generally be poor in comparison to those derived by Soufflé's Auto-Scheduler.

The SIPS is applied to any rule without a `.plan` directive. The default SIPS is `all-bound`, you can choose an alternate SIPS by passing `-PSIPS:<strategy>` to Soufflé.

- `strict`: Always choose the left-most atom
- `all-bound`: Prioritise atoms with all arguments bound
- `naive`: Prioritise (1) all bound, then (2) atoms with at least one bound argument, then (3) left-most
- `max-bound`: Prioritise (1) all-bound, then (2) max number of bound vars, then (3) left-most
- `max-ratio`: Prioritise max ratio of bound args
- `least-free`: Choose the atom with the least number of unbound arguments
- `least-free-vars`: Choose the atom with the least amount of unbound variables
- `input`: Goal: Prioritise (1) all-bound, (2) input, then (3) rest

## Datastructure
The datastructure for a relation can be specifically chosen by adding an appropriate attribute to the relation declaration, e.g.,
```
.decl A(x:number, y:number) btree
.decl B(x:number, y:number) brie
.decl C(x:number, y:number) eqrel
```
The default datastructure for most relations in Soufflé is a B-Tree. A Brie structure can improve performance in some cases, and is more memory efficient for particularly large relations.

`eqrel` is a high performance datastructure optimised specifically for equivalence relations. In the example below, eqrel_fast has the same functionality as eqrel_slow, but with greatly improved performance. 
```
.decl rel1, rel2(x:number)
.decl eqrel_fast(x : number, y : number) eqrel
eqrel_fast(a,b) :- rel1(a), rel2(b).

.decl eqrel_slow(x : number, y : number)
eqrel_slow(a,b) :- rel1(a), rel2(b).
eqrel_slow(a,a) :- rel1(a).             // reflexivity
eqrel_slow(b,a) :- eqrel_slow(a,b).     // symmetry
eqrel_slow(a,c) :- eqrel_slow(a,b), eqrel_slow(b, c).   // transitivity
```

## Profiler

The performance impact of the rule order can be measured using the profiling tool of Soufflé. The runtime of a rule, how many tuples are produced by the rule, and the performance behaviour for each iteration of a recursively defined rule can be measured and visualised. In practice, only a few rules are performance critical and need to be considered for performance tuning. 

More detailed description follows. First, Soufflé is executed with the profiler log option enabled on a given Datalog specification and a set of input facts. Then the generated log file is opened with souffle profiler. As described in the user manual section of souffle profiler, the profiler can be made to list the rules in the descending order of the total time consumed by each rule of the given Datalog specification. This list is very useful in the sense that one can compare the time taken by each rule with the number of tuples it generated. The rules which consume more time by generating less tuples are the candidates for optimisation. Typically optimising the top 10 rules in the list is sufficient.

A simple way to optimise a rule is to reorder the relations in its body based on the insight explained in the above section (Best Practices). Note that due to the semi-naive evaluation technique some rules are transformed into multiple rules. They are listed as versions of the rule. A manual query planner may be used to reorder predicates corresponding to a particular version of the rule. An example is given below.

```
A(x) :- B(x), C(x).

.plan 1: (2,1)
```
Assume that the relations B and C depend on A. Then the relations A, B and C are mutually recursive. Then the different version of the above rule will look like

```
Version 0: A(x) :- Δ B(x), C(x)
Version 1: A(x) :- B(x), Δ C(x)
```
Δ B(x) includes only the new tuples of B generated in the previous stage of the semi-naive evaluation.

And the statement starting with .plan swaps the relations in the body of the Version 1 of the rule.
Note that to fully understand the optimisations a good understanding of the semi-naive algorithm is required.
