---
layout: docs
title: Benchmarks
permalink: /docs/benchmarks/
---

There are some tools available under `benchmarks` that make it easy for you to compare the performance of your branch of souffle to another.

### `big_benchmark.sh`

Use this script if you are interested in testing the performance of your souffle instance on a prechosen suite of doop programs.

**Synopsis:** `./big_benchmark.sh --outdir=<OUTPUT DIRECTORY> --instance=<SOUFFLE EXECUTABLE>`

**Description:** `./big_benchmark.sh` executes `./timer.sh` for a small collection of doop benchmarks under `benchmarks/2-object-sensitive+heap`. If `outdir` does not exist yet, it is silently created and `.csv` files are created for each doop program in `antlr, xalan, eclipse`. If `outdir` already exists and has been used in the `big_benchmark.sh` command previously, the rows produced by `timer.sh` will be appended to each existing `.csv` file. If you wish to change which doop programs you want to make part of your own benchmark, just change the `programs` array in the `big_benchmark.sh` script.

**Requirements:** You need to have `cd`'d into your `benchmarks` folder in order for the script to run. If the script runs unsuccessfully since you have not `cd`'d, the output directory and `.csv` files with headers will still be created.

The available options are

| Long  | Short | Description | Example |
| :------------- | :------------- | :------------ | :------------ |
 `--outdir` | `-o` | Specify the output directory for the `.csv` files. | `./big_benchmark.sh --outdir=/home/user/out ...` |
| `--instance` | `-i` | Specify the instance of souffle to run | `./big_benchmark.sh ... --instance=/home/user/souffle/src/souffle` |

### `timer.sh`
Use this script if you are interested in testing the performance of your instance of souffle on a particular datalog program and set of facts.

**Synopsis:** `./timer.sh program.dl --facts=<FACT DIRECTORY> --instance=<SOUFFLE EXECUTABLE> [--pretty]`

**Description:** This utility outputs time and memory performance statistics for `program.dl` in csv row format. To produce these statistics, it runs the `<SOUFFLE EXECUTABLE>` with the `-g` flag in order to produce a `.cpp` file. Then it gives `program.dl, elapsed real time (seconds), Total number of CPU-seconds that the process spent in user mode, Total number of CPU-seconds that the process spent in kernel mode, Maximum resident set size of the process during its lifetime in Kbytes`. Then it outputs the same statistics for the process `souffle-compile result.cpp`, and then for the execution of the final executable `./result`. The columns of the row output have the following meaning:

`name,d2creal,d2cuser,d2csys,d2cmem,c2oreal,c2ouser,c2osys,c2omem,runreal,runuser,runsys,runmem`

where `d2c` means Datalog to C++ Souffle Compilation, `c2o` means `gcc`'s C++ to .o Compilation, and `run` means the execution of the produced executable file.

When one of these processes exit with a non-zero status, `0,0,0,0` will be given as a dummy result to indicate that something went wrong with this stage of compilation. In the case where all processes fail, an entire dummy row will be produced in the form `program.dl,0,0,0,0,0,0,0,0,0,0,0,0`.

If you prefer, you can pass the optional flag `--pretty` or `-p` to have human-readable output.
You can run this script from any context since it is context-independent, unlike `big_benchmark.sh`. 





