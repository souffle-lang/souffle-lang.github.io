---
layout: docs
title: IO Directives
permalink: /docs/io/
---

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
a tab-separated file ```A.facts```.
A more detailed specification for each relation can be defined in the input directive. For example,
```
.decl A(a:number,b:number)
.input A(IO=file, filename="path/to/file", columns="4:7", delimiter=",")
```
This will load the relation data from file (the default), using the filename "path/to/file" and a comma as a delimiter, and will use the fourth and seventh columns for input data.
```
.decl A(a:number,b:number)
.input A(IO=stdin, columns="4:7", delimiter=",")
```
Data can also be read from stdin, and again the delimiter and columns to read can be specified.
```
.decl A(a:number,b:number)
.input A(IO=sqlite, dbname="path/to/sqlite3db")
```
Data will no be read from an sqlite3 database at the given path.
The data is expected to be stored in the table matching the relation name.

# Output
The output relations of a Datalog program are by default written to a tab separated file name ```<relation name>.csv``` in the current directory. If the parameter ```-D<output-dir>``` is given then the default output directory will be changed to that given. ```-D-``` can be used to redirect all output to stdout.

For example, the relation  
```
.decl result(a:number, b:number, c:symbol)
.output result
```
has three number columns that are written to the file ```result.csv``` in the current directory, or ```<output-dir>``` when using the flag ```-D <output-dir>```.
