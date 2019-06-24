---
layout: docs
title: Run Soufflé
permalink: /docs/execute/
---
Soufflé provides an interpreter, a compiler, and a feedback-directed compilation infrastructure for compiling and executing Datalog programs. Profile information can be generated and visualized with a [profiling tool](profiler). The Soufflé execution mode (compile, interpret etc.) is determined by the argument parameters of the souffle command. 

## Input / Output
The Soufflé permits facts to be sourced from tab-separated input files to separate Datalog programs from their data. In Datalog literature the tab-separated input files can be seen as the extensional database (EDB) of the program. The default location of the input files is specified by the parameter ```-F <fact-dir>```. If the flag is not specified, it is assumed that the fact files are stored in the current directory. The default filenames of the input files consists of the name of the input relations with an extension ```.facts```. For example, the declaration 
```
.decl my_relation(a:number,b:number)
.input my_relation
```
defines an input relation ```my_relation``` with two number columns. Note that a relation becomes an input relation with the keyword ```input``` at the end of a declaration.  For the aforementioned example , souffle expects a tab-separated file ```my_relation.facts``` in either the current directory (if no ```-F``` flag was specified) or in the directory ```<fact-dir>``` with the option ```-F <fact-dir>```. Note that there is an exception if a relation is an input relation and is declared in an instantiation of a [component](components). 

If more precise control over file location is needed, more options can be added to the input directive. For example
```
.decl my_relation(a:number,b:number)
.input my_relation(filename="<path to input file>")
```
The output relations of a Datalog program are written to file in the current directory if no command line parameters, directives in the Datalog program, are given. The default output directory can be specified with the flag ```-D <output-dir>``` Output is written to standard output with the parameter ```-D-```. For example, the relation  
```
.decl result(a:number,b:number,c:number)
.output result
```
has three number columns that are written either to the file ```result.csv``` in the directory ```<output-dir>``` using the flag ```-D <output-dir>```  or to standard output using the flag ```-D-```. As with the input directive, more detailed output can be specified via the output directive.
```
.decl result(a:number,b:number,c:number)
.output result(filename="<path to output file")
```

Both input and output file IO can also benefit from the `delimiter` and `compress` options, for example,
```
.decl result(a:number,b:number,c:number)
.output result(filename="<path to output file", delimiter=",", compress=true)
```
This will output using "," as a column delimiter and compress the output using gzip.

Multiple input and output directives may be given in order to read input from, or send output to, multiple locations.

If the directive ```printsize``` is used, the size of the relation is printed to the standard output.
For example, the relation  
```
.decl result(a:number,b:number,c:number)
.printsize
```
is not printed to a file. Instead the number of tuples of relation ```result``` are written to standard output. 

## Interpreter

The interpreter is the default option when invoking the ```souffle``` as a command line tool. When souffle is invoked in interpreter mode, the parser translates the Datalog program to a RAM program, and executes the RAM program on-the-fly. For computationally intensive Datalog programs, the interpretation is slower than the compilation to C++. However, the interpreter has no costs for compiling a RAM program to C++ and invoking the C++ compiler, which is for larger programs expensive (in the order of minutes). 

## Compiler 

The compiler of souffle is enabled using the flag ```-c```, the flag ```-o <exec>```, or the flag ```-g <class.cpp>``` (or the long versions, respectively, ```--compile```, ```--dl-program=<exec>```, and ```--generate=<class.cpp>```). The compiler translates a Datalog program to a C++ program that is compiled to an executable and executed. The performance of a compiled Datalog is superior to the interpreter, however, the compilation of the C++ program may take some time. 

The difference between the flag ```-c``` and ```-o``` is whether the program is compiled and immediately executed with the former option or whether an executable is generated with the latter option. If compiled with option ```-o <exec>```, the executable is a standalone program whose options can be queried with flag ```-h```. The following message would be produced,

```
====================================================================
 Datalog Program: <source file>
 Usage: <exec> [OPTION]

 Options:
    -D <DIR>, --output=<DIR>     -- directory for output relations
                                    (default: <fact-dir>) 
    -F <DIR>, --facts=<DIR>      -- directory for fact files
                                    (default: <output-dir>) 
    -h                           -- Prints this help page.
--------------------------------------------------------------------
 Copyright (c) 2013-15, Oracle and/or its affiliates.
 All rights reserved.
===================================================================
```

The defaults are taken from the compiler invocation, which may be overwritten with user defined parameters of the standalone executable. Note that if the profiling option is enabled, the standalone executable has the additional option ```-p``` (see below). 

## Feedback-Directed Compilation

Soufflé has implemented feedback-directed compilation, i.e., the interpreter executes a Datalog program. Whilst executing the Datalog program on-the-fly, the runtime behaviour (relation sizes, etc.) are collected and used for producing a compiled executable of the input program. The feedback-directed compilation is enabled using the flags
```
souffle --auto-schedule -o <exec> <prog>.dl
```
where ```<exec>``` is the executable generated by the input program ```<prog>.dl``` using feedback-directed compilation. 
At the moment the query planner is still very experimental and does not produce optimal query plans. Optimising queries by hand give better performance. More details can be found in the section for [performance tuning](performance-tuning).  

## Profiling 
As a side-effect of execution, a profile log can be generated. The profile log can be visualized using the souffle-profiler. The option for enabling the profile log is ```-p <log-file>``` that works for the interpreter as well as the compiler. The profiler is described in the [profiler](profiler) section. 

Soufflé has various command line options to control the execution mode and parameterise the Soufflé input program.

# Warning Options

Warnings produced by Soufflé can be disabled with the option ```-w```.

# Parallel Execution

All execution modes of Soufflé provide parallel evaluation. The parallel evaluation is enabled with the option ```-j <num>```. 

# Verbose message output
More verbose output can be produced by using the verbose flag ```-v```. 

# Debugging 
A debug report in HTML format is generated using the flag ```--debug-report=<report.html>```. Note that for the debug report, it is essential that graphviz is installed to render the dependency graphs. 
