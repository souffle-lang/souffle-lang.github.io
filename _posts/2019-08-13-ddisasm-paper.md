---
sidebar: news_sidebar
title: 'Datalog Disassembly'
date: 2019-08-13
author: b-scholz
categories: [related-work]
permalink: ddiasm.html
layout: post
---

A fast disassembler/binary translator for x86-binaries have 
been developed by GrammarTech and open-sourced under the 
GNU Affero General Public License v3.0.
The source code can be found on [github](https://github.com/GrammaTech/ddisasm).

The disassembler parses ELF files and translates the binary 
to a symbolic representation exposing control-flow and data-flow dependencies. 
Antonio Flores-Montoya and Eric Schulte have written a technical 
report summarising the work and have published the work on 
Arxiv.org. The abstract and link are below,

[Datalog Disassembly, Antonio Flores-Montoya, Eric Schulte](https://arxiv.org/abs/1906.03969)

<pre>
Disassembly is fundamental to binary analysis and rewriting. We present a novel disassembly technique that takes a
stripped binary and produces reassembleable assembly code. The resulting assembly code has accurate symbolic information
providing cross-references for analysis and enabling adjustment of code and data pointers to accommodate rewriting. Our
technique features multiple static analyses and heuristics in a combined Datalog implementation. We argue that Datalog's
inference process is particularly well suited for disassembly and the required analyses. Our implementation and
experiments supports this claim. We have implemented our approach into an open-source tool called Ddisasm. In extensive
experiments in which we rewrite thousands of x64 binaries we find Ddisasm is both faster and more accurate than the
current state-of-the-art binary reassembling tool, Ramblr. *
</pre>
