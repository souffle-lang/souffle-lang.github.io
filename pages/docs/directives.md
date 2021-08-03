---
title: Directives
permalink: /directives
sidebar: docs_sidebar
folder: docs
---
# Directives 
Souffle has directives controlling the I/O behaviour of relations, and their execution behaviour.
The I/O system for relations supports various data-sources for input and output using various formats.
Soufflé supports terminal output, file I/O, and I/O utilising a SQLite as a database. 
For a relation in a Datalog program, several input and output directives can be issued. 
Souffle supports directies to limit the number of tuples per relation. 

## Input Directive
A Soufflé program may load the facts of a relation (aka. as EDB) from various input sources.
The input source is specified by the `.input` directive of a relation.
The default input source is a tab-separated file for a relation where
each row in the tab-separated file represents a fact in the relation. 

For example, 
```
.decl A(a:number,b:number)
.input A 
```
defines an input relation `A` with two number attributes. 
The input directive `.input` make the EDB read from 
a tab-separated file `A.facts`.


For example,
```
.decl A(a:number,b:number)
.input A(IO=file, filename="path/to/infile", columns="4:7", delimiter=",")
```
This will load the relation data from file (the default if no IO type is specified) using the filename "path/to/infile" and a comma as a delimiter, and will use columns four and seven (zero-indexed) for input data.
```
.decl A(a:number,b:number)
.input A(IO=stdin, columns="4:7", delimiter=",")
```
Data can also be read from stdin, and again the delimiter and columns to read can be specified.
```
.decl A(a:number,b:number)
.input A(IO=sqlite, dbname="path/to/sqlite3db")
```
Data will now be read from an sqlite3 database at the given path.
The data is expected to be stored in a table matching the relation name
prefixed by an underscore and the sqlite3 database is expected to contain a
view matching the relation name. For example, for the relation `edge`, the
sqlite3 database should have a table named `_edge` containing the data and a
view named `edge` that returns the data from `_edge`.

A more detailed specification for each relation can be defined in the input directive. 
```
.input <relation id> (IO=[file|stdin|sqlite] [optional parameters])
```

### IO=file
`filename`
Note that if the `-F<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.
 
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

`columns`
Particular columns can be selected from an input file. `columns="2:0"` will use the third input file column for the first attribute of the relation, and the first input file column for the second attribute. The default is to use the first columns of the input file until all attributes are described.

`compress`
Input is assumed to be in gzip compressed format. By default, gzip compressed input is automatically detected.

`headers`
The header can be enabled by `headers=true` or surpressed by `headers=false` (which is default) for CSV files. 

`rfc4180=[false|true]`
Enables the [rfc4180](https://datatracker.ietf.org/doc/html/rfc4180) compatibility mode. Delimiter is set to `,` by default and double-quotes are only allowed to enclose fields. Two consecutive double-quote characters `""` in a double-quote enclosed field encode for a single double-quote in the actual value. The `"` delimiter is forbidden.

### IO=stdin
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character. The order of evaluation of relations is not fixed. This method is not reliable when reading more than one relation from stdin.

### IO=sqlite
`filename`
The path to the sqlite3 database. Note that if the `-F<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

## Output Directive
The output relations of a Datalog program are, by default, written to a tab separated file with name `<relation name>.csv`, located in the current directory. If the parameter `-D<output-dir>` is given then the default output directory will be changed to that given. `-D-` can be used to redirect all output to stdout.

For example, the relation  
```
.decl result(a:number, b:number, c:symbol)
.output result
```
has three number columns that are written to the file `result.csv` in the current directory, or `<output-dir>` when using the flag `-D <output-dir>`.

As for input more detailed specification for each relation can be defined in the output directive. For example,
```
.decl A(a:number,b:number)
.output A(IO=file, filename="path/to/outfile", delimiter=":")
```
This will store the relation data to file (the default if no IO type is specified) using the filename "path/to/outfile" and a colon as a delimiter.
```
.decl A(a:number,b:number)
.output A(IO=stdout, delimiter=",")
```
Data can also be sent to stdout, and again the delimiter can be specified.
```
.decl A(a:number,b:number)
.output A(IO=sqlite, dbname="path/to/sqlite3db")
```
Data will be written to an sqlite3 database at the given path.
The data will be stored with a view that shows the symbols and numbers contained in the relation.
This is backed by a database table to store the symbol table and a table to store the relations numbers or symbol indices.

The output can be parameterised:

```
.output <relation id> (IO=[file|stdout|sqlite] [optional parameters])
```

### IO=file
`filename`
Note that if the `-D<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

`compress`
Output is in gzip compressed format.

`rfc4180=[false|true]`
Produces an rfc4180 compliant CSV file. Delimiter is set to `,` by default and all fields are double-quote delimited.  The `"` delimiter is forbidden.

### IO=stdout
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

### IO=sqlite
`filename`
The path to the sqlite3 database. Note that if the `-D<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

### I/O Types
Attributes types are signed numbers, unsigned numbers, float, strings, and records. If for primitive types (i.e. numbers, unsigned, and float) wrong input values are provided while reading from a data-source, an error will be issued. 

Records are written in a recursive format for input and output directives. A recursive data-structure is expanded completely
and printed.  For example:
```
.type List = [data:number, next:List]
.decl A(l:List,y:number)
A([1,[2,[3,nil]]],10). 
.output A
```
produces following output:
```
---------------
A
l	y
===============
[1, [2, [3, nil]]]	10
===============
```

To read or write arbitrary complex records and symbols without ambiguity, you may use the `rfc4180=true` I/O option. For example:
```
.type ListOfSymbols = [data:symbol, next:ListOfSymbols]
.decl A(l:ListOfSymbols,y:symbol,z:number)
A(["comma=,",["double-quote=\"",["space= ",["bracket=]",["nil",nil]]]]],"double-quote=\" comma=,",10).
.output A(rfc4180=true,delimiter="\t")
```

Produces following output:

```
---------------
A
l       y       z
===============
"[""comma=,"", [""double-quote=\\"""", [""space= "", [""bracket=]"", [""nil"", nil]]]]]"        "double-quote=\\"" comma=,"     10
===============
```

Compared to (without the `rfc4180` option):

```
---------------
A
l       y       z
===============
[comma=,, [double-quote=\", [space= , [bracket=], [nil, nil]]]]]        double-quote=\" comma=, 10
===============
```



## Limit-Size Directive 
Souffle can stop the fix-point calculation after a certain number of 
tuples have been computed. This number is a soft-limit and should be
used for debugging purposes only.

The main application for the limit-size directive is to debug non-terminating or slow-terminating programs.
After the limit-size directive is set, the program will terminate early and the output of the program
can be inspected.  For example, the program
```
.decl A(x:number)
A(1).
A(x+1) :- A(x), x< 1000.
.output A

.limitsize A(n=47)
```
computes the first 47 numbers in set `A` rather than 1000 numbers. 
This is a consequence of the limitsize directive that stops the fix-point computation when the size of `A` reaches 47 tuples or more. 

Note that if there are other mutual recursive relations in the same stratum, they will be stopped as well. 

## Legacy Syntax
Older versions supported I/O qualifiers in the relation declaration such as 
```
.decl A(x:number, y:symbol) input
.decl B(x:number, y:symbol) output
```
that should be rewritten to 
```
.decl A(x:number, y:symbol)
.input A
.decl B(x:number, y:symbol) 
.output B
```
Current versions of Soufflé still support the legacy syntax, but a warning message will be issued.
