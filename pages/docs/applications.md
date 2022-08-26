---
title: Applications
permalink: /applications
sidebar: docs_sidebar
folder: docs
---

The following applications are written in the Soufflé Language. Some are open source and the source code is accessible:

## Doop
Doop is a pointer analysis framework for Java programs. 
Doop implements a range of points-to algorithms, including context insensitive, 
call-site sensitive, and object-sensitive analyses. The source code can be found
[here](https://bitbucket.org/yanniss/doop/).

## Gigahorse
A binary lifter (and related framework) from low-level EVM code to a higher-level function-based three-address
representation, similar to LLVM IR or Jimple. The source code can be found
[here](https://github.com/nevillegrech/gigahorse-toolchain).

## Securify 2.0
Securify 2.0 is a security scanner for the Solidity language. The source code can be found [here](https://github.com/eth-sri/securify2). 

## Insieme Compiler 
Insieme project is a research compiler for automatically optimizing parallel programs for homogeneous and heterogeneous
multi-core architectures. It is a source-to-source compiler. The source code can be found
[here](https://github.com/insieme/insieme). 

## DDISASM
GrammaTech's DDisasm is a disassembler for multiple platforms. The source code can be found [here](https://github.com/GrammaTech/ddisasm). 

## Vandal
Vandal is a static program analysis framework for Ethereum smart contract bytecode, developed at The University of
Sydney. The source code can be found [here](https://github.com/usyd-blockchain/vandal). 

## Prosynth
A synthesis framework that learns Datalog rules from data (appeared POPL'20).

## Haskell Interface 
There is a Haskell/Soufflé interface written by Luc Tielen. You can find the source code [here](https://github.com/luc-tielen/souffle-haskell). 

## Haskell Program Optimisation
GRIN is a Haskell backend that uses Soufflé. The source code can be found
[here](https://github.com/grin-compiler/ghc-grin). 

## Java JDK Security Analysis
Soufflé was initially used to find security vulnerabilities in the Java JDK library. This project was conducted by Oracle Labs, Brisbane. 

## VPN 
Amazon used Soufflé to verify VPN connections in the AWS cloud. 

## cclyzer++ and MATE
[Galois'](https://galois.com/) [MATE](https://github.com/galoisinc/MATE) tool for interactive exploration of potential software vulnerabilities in C/C++ source code uses a context-sensitive pointer analysis [cclyzer++](https://github.com/galoisinc/cclyzerpp) implemented using Soufflé.

## Frank's DDLOG Benchmarks
Frank McSherry has developed three benchmarks for DDLOG. They can be found [here](https://github.com/frankmcsherry/dynamic-datalog/). 

## Soufflé's Benchmark Suite
The souffle team has its own benchmark, which can be found [here](https://github.com/souffle-lang/benchmarks/)

{% include links.html %}
