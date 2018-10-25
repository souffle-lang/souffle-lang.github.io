---
layout: docs
title: External Functors
permalink: /docs/functors/
---

Soufflé is extensible with user-defined functors. Functors are 
introduced via functor declarations and can be used subsequently.
Functors are strongly typed and have a type signature. User-defined
functors are implemented in form of a shared library, that will 
be loaded at evaluation-time. User-defined functors can be 
used in the interpreter and for the compiled Souffle program. 

## Functor Declaration 
A functor declaration contains
the name of the functor, the argument types, and the return type 
of the functor. A functor declaration has the following
format:
```
.functor <name>():<type>
```
where the types  ```<type<sub>1</sub>>,...,<type<sub>k</sub>>``` define the types 
of the arguments and ```<type>``` is the return type. Currently,
the type arguments and result type can only assume the primitive types:
* Symbol type: `symbol`
* Number type: `number`

For example, 
```
.functor f(number):number
```
introduces the functor `f` that has a single number argument and 
returns a number as a result.

## User-Defined Functor 

After declaring the functor, the functor can be used in rules and facts. 
User-defined functors are always represented in functional format, 
```
  <name>(<arg1>,...,<argk>)
```
Note that we don't have an infix representation for user-defined functors
in Souffle's grammar at the moment. 

For example,

```
// introduce new functor f: number -> number
.functor f(number):number


.decl A(x:number) 
.output A
A(1). 
A(f(i)) :- A(i), f(i) < 100.
```

## Implementation of User-Defined functors.

User-defined functors can be impmenented in any programming language of choice supporting callable external functions. 
There are strong requirements for the the implementation of functors. 
If they are violated, the execution of Datalog program cannot be guaranteed. 

The properties of the functor implementation are the following:

 * The implementation of a functor must return the same return value for the same input arguments. The functor implementation may have state but must follow strict pure functional semantics. For example, a functor cannot produce random numbers, however, f(x) = x + 1 would be a valid functor. 

 * The implementation of a functor must be reentrant. Souffle is highly parallel and several threads may execute the implementation of a user-defined functor in parallel. Pthread synchronisation techniques may be required.

 * A single shared-library `libfunctors.so` contains all user-defined functors.    The environment variable `LD_LIBRARY_PATH` must contain the path where the functor library is located so that either the interpreter or the executable Datalog program can load shared-library.

For example, we can implement the user-defined functor in C++ with the following code:

```
#include <cstdint>
extern "C" {

int32_t f(int32_t x) {
    return x + 1;
}

cost char *g() {
    return "Hello world";
}
```

for the functor declarations 
```
.functor f(number):number
.functor g():symbol
```

Note that number types are implmented as ```int32_t``` and symbol types are implemented as ```char const *```. In Linux, a shared library can be generated with
the following instructions:
```
g++ -fPIC -o functors.o 
g++ -shared -o libfunctors.so functors.o 
```
assuming that the source code of the user-defined functors is stored in the source file ```functors.cpp```.
