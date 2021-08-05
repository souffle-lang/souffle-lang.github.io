---
title: Pragmas
permalink: /pragmas
sidebar: docs_sidebar
folder: docs
---

Pragmas permit to set command-line flags and configurations directly in the source code. 

For example,
```prolog
.pragma "legacy" 
.decl A(x:number) output
A(1).
```
will enable the `--legacy` flag in the source without specifying when invoking souffle. 

There are also some configurations that cannot be set by command-line flags including `RamSIPS` choosing a static heuristic for [query plans](tuning). 

## Syntax 
In the following, we define pragmas more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced using [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Pragma
A pragma is followed by a parameter string. An additional string for the value of the parameter is optional. 

![Pragma](https://souffle-lang.github.io/img/pragma.svg)

```ebnf
pragma   ::= '.pragma' STRING STRING?
```

         
{% include links.html %}
