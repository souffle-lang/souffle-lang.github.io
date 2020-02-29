---
title: User-Defined Functors
permalink: /functors
sidebar: docs_sidebar
folder: docs
---

Soufflé is extensible with user-defined functors. Functors are 
introduced via functor declarations and can be used subsequently.
Functors are strongly typed and have a type signature. User-defined
functors are implemented in the form of a shared library, that will 
be loaded at evaluation-time. User-defined functors can be 
used in the interpreter and for the compiled Soufflé program. 

## Functor Declaration 
A functor declaration contains
the name of the functor, the argument types, and the return type 
of the functor. A functor declaration has the following
format:
```
.functor <name>():<type>
```
where the types  ```<type<sub>1</sub>>,...,<type<sub>k</sub>>``` define the types 
of the functor arguments, and ```<type>``` defines the return type. Currently,
the type arguments and result type can only assume the primitive types of Soufflé:
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
User-defined functors have the prefix-notation, i.e., 
```
  <name>(<arg1>,...,<argk>)
```

For example,

```
// introduce new functor f: number -> number
.functor f(number):number


.decl A(x:number) 
.output A
A(1). 
A(@f(i)) :- A(i), @f(i) < 100.
```
declares a user-defined functor with name `f` that has a number as an argument and produces a number as a result. 

## Implementation of User-Defined functors.

User-defined functors can be implemented in any programming language of choice supporting callable external functions. 
There are strong requirements for the implementation of functors. If they are violated, the execution of Datalog program cannot be guaranteed. 

The properties of the functor implementation are the following:

 * The implementation of a functor must return the same return value for the same input arguments. The functor implementation may have state but must follow strict pure functional semantics. For example, a functor cannot produce random numbers, however, f(x) = x + 1 would be a valid functor. 

 * The implementation of a functor must be reentrant. Souffle is highly parallel and several threads may execute the implementation of a user-defined functor in parallel. Pthread synchronisation techniques may be required.

 * By default a single shared-library `libfunctors.so` contains all user-defined functors. Custom libraries may be used by starting souffle with `-l<libraryname>` and `-L<library path>`, e.g. `souffle a.dl -lfunctors -lmorefunctors`. The environment variable `LD_LIBRARY_PATH` can also be used to specify library paths. Note that if you use souffle to compile a standalone executable, the path for dynamic dlls may still be required at execution time, so `LD_LIBRARY_PATH` must be specified for the executable to run.
 
 * The name of the user-defined functor must be a C-linkable name (not a C++ linkable name). For example, if the user-defined functor f is declared, the shared-library must have a function f in the shared library with a C style argument passing mechanism. 

For example, we can implement the user-defined functor in C++ with the following code:

```
#include <cstdint>
extern "C" {

int32_t f(int32_t x) {
    return x + 1;
}

const char *g() {
    return "Hello world";
}

}
```

for the functor declarations 
```
.functor f(number):number
.functor g():symbol
```

Note that number types are implmented as ```int32_t``` and symbol types are implemented as ```const char *```. In Linux, a shared library can be generated with the following instructions:
```
g++ functors.cpp -c -fPIC -o functors.o 
g++ -shared -o libfunctors.so functors.o 
export LD_LIBRARY_PATH=LD_LIBRARY_PATH:`pwd`
```
Assuming that the source code of the user-defined functors is stored in the source file ```functors.cpp```. The export command ensures that either the Soufflé interpreter or the generated executable can find the shared library.
