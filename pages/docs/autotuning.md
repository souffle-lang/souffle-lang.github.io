---
title: Auto-Tuning
permalink: /autotuning
sidebar: docs_sidebar
folder: docs
toc: false
---

# Auto-Scheduling 

To achieve high performance in Soufflé, the Auto-Scheduler can be used.

The Auto-Scheduler can be used as follows:

1. ```souffle <program> -p <profile> --index-stats```

2. ```souffle -c <program> --auto-schedule=<profile> -o <binary>```

Stage 1 runs the Soufflé interpreter and gathers statistics about relations during the program's execution, saving them to a profile.

Note that either the interpreter or synthesizer can be used to collect statistics. Both modes of execution will produce the same statistics. 

Stage 2 reads in the profile with statistics, uses these statistics to determine high performance schedules and finally synthesizes a binary that uses the derived schedules.

Note that either the interpreter or synthesizer can be used for auto-scheduling. 
