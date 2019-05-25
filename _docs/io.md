---
layout: docs
title: IO Directives
permalink: /docs/io/
---
## Input

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
A more detailed specification for each relation can be defined in the input directive. For example,
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
The data is expected to be stored in the table matching the relation name.

## Output
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
Data can also be sent to stdin, and again the delimiter can be specified.
```
.decl A(a:number,b:number)
.output A(IO=sqlite, dbname="path/to/sqlite3db")
```
Data will be written to an sqlite3 database at the given path.
The data will be stored with a view that shows the symbols and numbers contained in the relation.
This is backed by a database table to store the symbol table and a table to store the relations numbers or symbol indices.


# Standard .input options

```
.input <relation id> (IO=[file|stdin|sqlite] [optional parameters])
```

## IO=file

`filename`
Note that if the `-F<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.
 
 `delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

`columns`
Particular columns can be selected from an input file. `columns="2:0"` will use the third input file column for the first attribute of the relation, and the first input file column for the second attribute. The default is to use the first columns of the input file until all attributes are described.

`compress`
Input is assumed to be in gzip compressed format. By default, gzip compressed input is automatically detected,

## IO=stdin
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

## IO=sqlite
`dbname`
The path to the sqlite3 database.

# Standard .output options

```
.output <relation id> (IO=[file|stdout|sqlite] [optional parameters])
```

## IO=file
`filename`
Note that if the `-D<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

`compress`
Output is in gzip compressed format.


## IO=stdout
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

## IO=sqlite
`dbname`
The path to the sqlite3 database.

