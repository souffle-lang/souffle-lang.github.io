---
layout: docs
title: A Simple Example
permalink: /docs/simple/
---

We now consider a simple Datalog program, and explore some of the features of Soufflé. This only introduces some of the most basic functionality available, and advanced users are encouraged to look elsewhere.

Say we have a Datalog file `example.dl`, whose contents are as shown.

~~~
.decl edge(x:number, y:number)
.input edge

.decl path(x:number, y:number)
.output path

path(x, y) :- edge(x, y).
path(x, y) :- path(x, z), edge(z, y).
~~~

We see that `edge` is a `.input` relation, and so will be read from disk. Also, `path` is a `.output` relation, and so will be written to disk.

The last two lines say that 1) "there is a path from x to y if there is an edge from x to y", and 2) "there is a path from x to y if there is a path from x to some z, and there is an edge from that z to y".

So if the input `edge` relation is pairs of vertices in a graph, by these two rules the output `path` relation will give us all pairs of vertices x and y for which a path exists in that graph from x to y.

For instance, if the contents of the tab-separated input file `edge.facts` is

~~~
1	2
2	3
~~~

The contents of the output file `path.csv`, after we evaluate this program, will be

~~~
1	2
2	3
1	3
~~~

We can evaluate this program by running

~~~
$ ./src/souffle -F. -D. example.dl
~~~

The `-F` and `-D` options specify the directories for input and output files respectively. So in this case, `edge.facts` is in the current working directory `.`, and `path.csv` will be produced here also.

If instead `edge.facts` was in a subdirectory called `input`, and we wanted to have `paths.csv` in a subdirectory called `output`, we could do

~~~
$ ./src/souffle -F./input -D./output example.dl
~~~

Running Soufflé in this way uses the *interpreter* mode, which means the Datalog program is evaluated immediately. Soufflé also supports a *compiler* mode, which transforms the Datalog program into a C++ program, which is then compiled to produce an executable.

So if we run Soufflé with the `-o` option as shown below.

~~~
$ ./src/souffle -F. -D. -oexample example.dl
~~~

We get an `example.cpp` C++ file and an `example` executable. Note that no `path.csv` has been produced, as the program has not yet been evaluated.

~~~
$ ls
example.dl
edge.facts
example.cpp
example
~~~

To evaluate our program, we have to run the compiled executable

~~~
$ ./example
~~~

Note that the compiled executable supports the `-F` and `-D` options, so running the following is also possible.

~~~
$ ./example -F./input -D./output
~~~

The `-r` option is particularly useful for debugging, as it generates a debug report in html format. Running the following produces an `example.html` file with this report.

~~~
$ ./src/souffle -F. -D. -rexample.html example.dl
~~~

 Soufflé also allows for profiling with the `-p` option. Running the following will create a new file `example.log` with a log of profiling information.

~~~
$ ./src/souffle -F. -D. -pexample.log example.dl
~~~

To analyse the profiling data in `example.log`, we need to open it with the included `souffle-profile` program, done as follows.

~~~
$ ./src/souffle-profile example.log
~~~

The profiler includes a lot of functionality useful for analysing the performance of Datalog programs. It too comes with a help text, which is given by the following command.

~~~
$ ./src/souffle-profile -h
~~~

As noted, there are many other useful options available to Soufflé, so be sure to explore them too!
