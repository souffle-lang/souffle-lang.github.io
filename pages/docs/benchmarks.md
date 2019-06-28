---
title: Benchmarks
permalink: /benchmarks
sidebar: docs_sidebar
folder: docs
---

There are some tools available under `benchmarks` that make it easy for you to compare the performance of your branch of souffle to another.

## Available Benchmark Programs

Under `souffle-lang/benchmarks/benchmarks`, you can use `tc, accessX, cprogX, family, poX, tak, tc, ...`. You can `cd` into the program directory and run `./gen_facts.sh` to produce a set of facts. This script will create the facts directory in your current working directory and the facts are randomised. There is an example script in `benchmarks/small_benchmark.sh`.
You can also find a collection of [doop](http://doop.program-analysis.org/benchmarks.html) programs and facts under `souffle-lang/benchmarks/benchmarks/2-object-sensitive+heap`. The directories with names `souffle-X` will contain a `2-object-sensitive+heap.dl` file and a `facts` directory.
You can use either of these options to create scripts that exploit this file structure to run a series of customised benchmarks.

## Available Scripts

### `big_benchmark.sh`in

Use this script if you are interested in testing the performance of your souffle instance on a prechosen suite of doop programs that are available in `benchmarks/benchmarks/2-object-sensitive+heap`.

**Synopsis:** `./big_benchmark.sh --outdir=<OUTPUT DIRECTORY> --instance=<SOUFFLE EXECUTABLE>`

**Description:** `./big_benchmark.sh` executes `./timer.sh` for a small collection of doop benchmarks under `benchmarks/2-object-sensitive+heap`. If `outdir` does not exist yet, it is silently created and `.csv` files are created for each doop program in `antlr, xalan, eclipse`. If `outdir` already exists and has been used in the `big_benchmark.sh` command previously, the rows produced by `timer.sh` will be appended to each existing `.csv` file. If you wish to change which doop programs you want to make part of your own benchmark, just change the `programs` array in the `big_benchmark.sh` script.

**Requirements:** You need to have `cd`'d into your `benchmarks/benchmarks` folder in order for the script to run. If the script runs unsuccessfully since you have not `cd`'d, the output directory and `.csv` files with headers will still be created.

The available options are

| Long  | Short | Description | Example |
| :------------- | :------------- | :------------ | :------------ |
 `--outdir` | `-o` | Specify the output directory for the `.csv` files. | `./big_benchmark.sh --outdir=/home/user/out ...` |
| `--instance` | `-i` | Specify the instance of souffle to run | `./big_benchmark.sh ... --instance=/home/user/souffle/src/souffle` |

### `timer.sh`
Use this script if you are interested in testing the performance of your instance of souffle on a particular datalog program and set of facts.

**Synopsis:** `./timer.sh program.dl --facts=<FACT DIRECTORY> --instance=<SOUFFLE EXECUTABLE> [--pretty]`

**Description:** This utility outputs time and memory performance statistics for `program.dl` in csv row format. Given a particular datalog program, set of facts, and instance of souffle, `timer.sh` will produce a row of data in the following form:

~~~
$ ./timer.sh tc/tc.dl --facts=tc/facts --instance=/home/me/souffle/src/souffle
name,d2creal,d2cuser,d2csys,d2cmem,c2oreal,c2ouser,c2osys,c2omem,runreal,runuser,runsys,runmem
~~~

where `d2c` means Datalog to C++ Souffle Compilation, `c2o` means `gcc`'s C++ to .o Compilation, and `run` means the execution of the produced executable file. `real` is the process's elapsed real time measured in seconds, `user` is the total number of CPU-seconds that the process spent in user mode, and `sys` is the same but for the number of seconds spent in kernel mode. Here is an example of the expected output.

~~~
$ ./timer.sh tc/tc.dl --facts=tc/facts --instance=/home/me/souffle/src/souffle
tc/tc.dl,0.02,0.01,0.00,7276,13.04,12.68,0.35,399780,4.78,4.76,0.00,8520
~~~

When one of these processes exit with a non-zero status, `0,0,0,0` will be given as a dummy result to indicate that something went wrong with this stage of compilation. In the case where all processes fail, an entire dummy row will be produced in the form `program.dl,0,0,0,0,0,0,0,0,0,0,0,0`.

If you prefer, you can pass the optional flag `--pretty` or `-p` to have human-readable output. Here is an example:

~~~
=====tc/tc.dl===============================
	Datalog -> C++
		Real:	0.01 s
		User:	0.01 s
		System:	0.00 s
		Memory usage:	7168 kB
	C++ -> .o
		Real:	10.54 s
		User:	10.24 s
		System:	0.29 s
		Memory usage:	399820 kB
	Running...
		Real:	4.70 s
		User:	4.70 s
		System:	0.00 s
		Memory usage:	8628 kB
~~~

There is also colour but I cannot convey this here unfortunately. You can run this script from any context since it is context-independent, unlike `big_benchmark.sh`. 






{% include links.html %}
