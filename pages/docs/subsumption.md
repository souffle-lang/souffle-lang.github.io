---
title: Subsumption
permalink: /subsumption
sidebar: docs_sidebar
folder: docs
---

Subsumption permits to delete more specific tuples by more general tuples. 
A programmer can express this by declaring a partial-order in the form 
of a subsumptive clause.

## Syntax 
In the following, we define rule declarations in Souffle more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Subsumptive Rule
A subsumptive rule has a dominated and a dominating head seperated by `<=` and is followed by its rule body. The rule body is defined as in a standard [rule](/rules).  A query plan is optional for a subsumptive rule.

![Rule](/img/subsumptive_rule.svg)

```ebnf
rule ::= atom '<=' atom ':-' disjunction '.' query_plan?
```

{% include links.html %}
