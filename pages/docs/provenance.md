---
title: Provenance
permalink: /provenance
sidebar: docs_sidebar
folder: docs
---

Provenance is a way to explain the execution of a Soufflé program, potentially useful for debugging. These explanations come in the form of a proof tree. In Soufflé, for any tuple, these proof trees are of *minimal height* for that tuple.

## Using Provenance
To enable the provenance system, the Soufflé command line option is `-t`
- Use `-t none` for provenance transformer without any explain interface
- Use `-t explain` for provenance with a command line interface to `stdout`
- Use `-t explore` for provenance with an interface in `ncurses` - this allows scrolling around a large proof tree

For example, given a program *foo.dl*, to enable the `stdout` interface for provenance use:
```
souffle -t explain foo.dl
```

### The `explain` interface
To illustrate the interface with an example, assume we have the following simple Soufflé program
```
.decl edge, path(x:number, y:number)
edge(1, 2).
edge(2, 3).
edge(3, 4).
edge(4, 5).

path(x, y) :- edge(x, y).
path(x, z) :- edge(x, y), path(y, z).
```

The explain interface consists of a number of commands:
- `explain path(1, 3)` prints the proof tree for the tuple `path(1, 3)`, showing all input and intermediate tuples required to generate the query tuple:   
```
> explain path(1, 3)
           edge(2, 3)
           -------(R1)
edge(1, 2) path(2, 3)
-------------------(R2)
      path(1, 3)
```   
For usage with strings, enclose values with quotation marks: `explain ancestor("john", "mary")`
- `setdepth 3` sets the maximum height of produced proof trees to be 3. Any parts of the proof tree above this height are replaced with a *subproof* label:   
```
> setdepth 3
Depth is now 3
> explain path(1, 4)
           edge(2, 3) subproof path(0)
           ------------------------(R2)
edge(1, 2)          path(2, 4)
------------------------------------(R2)
               path(1, 4)
```   
This is particularly useful in the case of large programs, where a full derivation tree is too unwieldy to be understood. The default value for this option is 4.   
- `subproof path(0)` prints the remaining part of the proof tree marked by the *subproof* label, generated when the depth limit is reached.   
```
> subproof path(0)
edge(3, 4)
-------(R1)
path(3, 4)
```   
- `explainnegation path(1, 6)` starts an interactive process to explain why the tuple `path(1, 6)` *does not exist* in the result. The user must provide a rule number, and values for any free variables in order for the system to produce a partial proof tree:   
```
> explainnegation path(1, 6)
1: path(x,y) :-
   edge(x,y).
2: path(x,z) :-
   edge(x,y),
   path(y,z).
Pick a rule number: 2
Pick a value for y: 2
====
edge(1, 2) ✓ path(2, 6) x
------------------------(R2)
         path(1,6)
```   
The approach here is required as it is not technically feasible to automatically generate explanations for non-existence, and a bit of user guidance is required.   
- Other commands allow to control the output (`output <filename>`, `format <json|proof>`), print rules of the program (`rule <relation> <rulenumber>`), and exit (`exit`, `quit`, or `q`)

## Internals of Provenance
The approach for provenance in Soufflé is a lazy one, where no proof trees are computed until the user queries for them.

However, to answer provenance queries efficiently, the provenance system requires to keep track of some extra information during evaluation.

In particular, for each tuple, the system tracks the rule producing the tuple, and the *height* of a *minimal height* proof tree for the tuple (we denote this extra information *annotations*). For example, if a tuple `path(1, 3)` is generated using rule number 2, and we know there is a proof tree of height 4 for the tuple, then internally this is represented as:
```
path(1, 3, 2, 4)
```

To compute the height annotation for a tuple, we can take the `max` over tuples in the body of the rule, and add 1 (the `_` denotes that we don't care about which rule generates each body tuple):
```
path(x, z, @rule, max(@height1, @height2) + 1) :-
    edge(x, y, _, @height1), path(y, z, _, @height2).
```

During the proof tree construction phase, the annotations are used to guide a backwards search through all the tuples computed by the program. The result of this backwards search is one level of a proof tree, and applying this recursively, we generate the full proof tree!
