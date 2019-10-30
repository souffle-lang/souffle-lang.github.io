---
title: Swig
permalink: /Swig
sidebar: docs_sidebar
folder: docs
---
 
# Using SWIG
[SWIG](http://swig.org/) is a software development tool that connects programs written in C and C++ with a variety of high-level programming languages. Soufflé uses SWIG as a programming interface such that other languages can interface it. This enables a program written in other languages to read in the Datalog input file through the wrapper files and output the corresponding CSV files. These wrapper files are generated through first generating the C++ file for the Datalog file and then compiling it using SWIG. Swig Interface currently supports **Java**  and **Python**.

## SWIG Command Line Option in Souffle
To run the SWIG command line option in Souffle, run:

```./souffle -s <language> <.dl file> ```

Currently, the languages supported are Java and Python.
The language given must be all lowercase.

## Running Souffle in Other Languages
Once the wrapper files have been generated, to use them in your program, you need to import the SwigInterface. Then you may run your program to generate the CSV files.

### Running it in Java
To use the interface in your Java program, add 

```System.loadLibrary("SwigInterface")```

MAC users may need to add this instead

`System.load(System.getProperty("java.library.path")+ "/" + "libSwigInterface.so")`

You are now able to use the SwigInterface to create an instance of the input file to be loaded and run.

`SWIGSouffleProgram p = SwigInterface.newInstance("<name of .dl file without extension>")`

`p.loadAll(".");`

`p.run(); `

`p.printAll(".");`

`p.finalize();   ` 

Compile your program and run it with the -D option to specify the directory of your SwigInterface

`java -Djava.library.path=<path of SwigInterface> <.java>`

### Running it in Python
To use the interface in your Python program, add at the top of the file
`import SwigInterface`

You are now able to use the SwigInterface to create an instance of the input file to be loaded and run.

`p = SwigInterface.newInstance("<name of .dl file without extension>")`

`p.loadAll('.')`

`p.run()`

`p.printAll('.')`

Run your program to see the outputted CSV files.
 
 ## Supported functions
`newInstance("<name of .dl file without extension>")`: creates a new instance from a Datalog file


`loadAll("<input directory>")`: loads all input relations

`run()`: executes the Datalog program

`printAll("<output directory>")`: prints all output relations
