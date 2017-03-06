---
layout: docs
title: Command Line Options
permalink: /docs/execute/
---

Soufflé has various command line options to control the execution mode and parameterise the Soufflé input program.

# Include Directory

The flag ```-I <directory>``` specifies the include directory for Soufflé programs. Soufflé uses a C-preprocessor to preprocess the source code. The flag specifies the include directory such that include files included by```#include``` can be found in the directory ```<directory>```.

# Input

A Soufflé program may load the facts of a relation (aka. as EDB) from various input sources.
The input source is specified by the ```.input``` directive of a relation.
The default input source is a tab-separated file for a relation where
each row in the tab-separated file represents a fact in the relation. 

For example, 
```
.decl A(a:number,b:number)
.input A 
```
defines an input relation ```A``` with two number attributes. 
The input directive ```.input``` make the EDB read from 
a tab-separated file ```A.facts``` in either the current directory (if no ```-F``` flag was specified) or it expects the file ```A.facts``` in the directory ```<fact-dir>``` with the option ```-F <fact-dir>```. 
Note that there is an exception if the filename is either changed in the [input directive](/docs/io) or the relation is a result of a component instantiation [component](components). 

# Output
The output relations of a Datalog program are written to a tab separated file name ```<relation name>.csv``` in the current directory. If the parameter ```-D<output-dir>``` is given then the default output directory will be changed to that given. ```-D-``` can be used to redirect all file output to stdout.

For example, the relation  
```
.decl result(a:number, b:number, c:symbol)
.output result
```
has three number columns that are written either to the file ```result.csv``` in the directory ```<output-dir>``` using the flag ```-D <output-dir>```  or to standard output using the flag ```-D-```. More options for specifying output parameters, such as a specific location or compression, can be found on the [IO directive page](/docs/io).

# Execution Modes

Soufflé has several modes of execution available. Souffle provides an interpreter, a compiler that synthesis C++ from Datalog, and a feedback-directed compilation infrastructure. 
The execution mode is determined by the argument parameters of the souffle command.

## Interpreter

The interpreter is the default option when invoking ```souffle``` as a command line tool. When souffle is invoked in interpreter mode, the parser translates the Datalog program to a RAM program, and executes the RAM program on-the-fly. The compiled mode can execute faster, but has an overhead for the initial compilation. For computationally intensive Soufflé programs, the interpretation mode is slower than the compilation to C++.

## Compiler 

The compiler mode of souffle synthesizes a C++ program from an input program. The compiler mode is enabled using either the flag ```-c```, the flag ```-o <exec>```, or the flag ```-g <class>.cpp```.  

The difference between the flag ```-c``` and ```-o``` (or its long version ```--dl-program```) is whether the program is compiled and immediately executed with the former option or whether an executable is generated with the latter option. If compiled with option ```-o <exec>```, the executable is a stand-alone program whose options can be queried with flag ```-h```. The following message would be produced,

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

The defaults are taken from the compiler invocation, which may be overwritten with user defined parameters of the stand-alone executable. 

The option ```-g <class>.cpp``` synthesizes a C++ class from a Soufflé input program. The C++ can be embedded in other tools. Ensure that the Soufflé include paths are enabled, and the flag ```__EMBEDDED_SOUFFLE__``` is set. More information about the C++ interface can be read in the [interface section](/docs/interface/).

## Feedback-Directed Compilation

Soufflé facilitates feedback-directed compilation, i.e., the interpreter executes a Datalog program. While executing the Datalog program on-the-fly, the runtime behavior (relation sizes, etc.) is monitored, and used for producing a compiled executable of the input program. The feedback-directed compilation mode is enabled using the flags
```
souffle --auto-schedule -o <exec> <prog>.dl
```
where ```<exec>``` is the executable generated by the input program ```<prog>.dl``` using feedback-directed compilation. 
At the moment the query planner is still very experimental and does not produce good query plans. Optimizing queries by hand give better performance. More details can be found in the section for [performance tuning](tuning).  

# Profiling 

Soufflé has a profiler. The profile log for profiler is generated by enabling 
the option```-p <log-file>```. Profiling is available for the interpreter and
the compiler.  The profiler is described in the [profiler](/docs/profiler) section of the manual. 

# Warning Options

Warnings produced by Soufflé can be disabled with the option ```-w```.

# Parallel Execution

All execution modes of Soufflé provide parallel evaluation. The parallel evaluation is enabled with the option ```-j <num>```. 

# Verbose message output
More verbose output can be produced by using the verbose flag ```-v```. 

# Debugging 
A debug report in HTML format is generated using the flag ```--debug-report=<report.html>```. Note that for the debug report, it is essential that graphviz is installed to render the dependency graphs. 
