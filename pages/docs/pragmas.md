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

There are also some configurations that cannot be set by command-line flags including "`RamSIPS` choosing a static heuristic for query plans. 

## Syntax 


         
{% include links.html %}
