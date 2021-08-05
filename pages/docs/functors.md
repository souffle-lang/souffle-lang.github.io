---
title: User-Defined Functors
permalink: /functors
sidebar: docs_sidebar
folder: docs
---

Soufflé is extensible with user-defined functors. Functors are 
introduced via functor declarations. Functors are strongly typed 
and have a type signature. User-defined functors are implemented 
in C/C++ and are stored in a shared library, that will 
be loaded at evaluation-time. User-defined functors can be 
used in the interpreter and synthesiser. 

## Functor Declaration 
A functor declaration contains
the name of the functor, the argument types, and the return type 
of the functor. A functor declaration has the following
format:

```
.functor <name>(<name1>:<type1>,...,<name-k>:<type-k>):<type>
```
  
where the types  `<name-1><type-1>,...,<name-k><type-k>` define 
the argument types of the functor, and `<type>` defines the return 
type. For functors that are not stateful, the type arguments and 
result type can only be primitive types (and their subtypes):
* Symbol type: `symbol`
* Number type: `number`
* Float type: `float`

For example, 
```
.functor f(a:anumber):number
```
introduces the user-defined functor `f` that has a single number argument and 
returns a number as a result.

## Invocation of User-Defined Functor 

The user-defined functors have the prefix-notation, i.e., 
```
  @<name>(<arg1>,...,<argk>)
```

For example,
```
// introduce new functor f: number -> number
.functor f(x:number):number


.decl A(x:number) 
.output A
A(1). 
A(@f(i)) :- A(i), @f(i) < 100.
```
declares a user-defined functor with name `f`. The functor has a number as an argument and produces a number as a result. 

## Implementation of User-Defined Functors

User-defined functors can be implemented in C++ (other languages could be used if they support an external C/C++ interface). 
There are strong execution requirements when user-defined functors are implemented. If these requirements are violated, 
the execution of Souffle program cannot be guaranteed. 

The properties of the functor implementation are the following:

 * The implementation of a functor must return the same return value for the same input arguments. The functor implementation may have state but must follow strict pure functional semantics. For example, a functor cannot produce random numbers, however, f(x) = x + 1 would be a valid functor. 

 * The implementation of a functor must be reentrant. Souffle is highly parallel and several threads may execute the implementation of a user-defined functor in parallel. Pthread synchronisation techniques may be required for the implementation.

 * By default a single shared-library `libfunctors.so` in the current folder contains all user-defined functors. Custom libraries may be used by starting souffle with `-l<libraryname>` and `-L<library path>`, e.g. `souffle -lfunctors -lmorefunctors a.dl`. Note that these options must precede the Datalog file in the command line. The environment variable `LD_LIBRARY_PATH` can also be used to specify library paths for the shared functor library. If you use Souffle to compile a standalone executable, the path for dynamic dlls may still be required at execution time, so `LD_LIBRARY_PATH` must be specified for the executable to run.
 
 * The name of the user-defined functor must be a C-linkable name (not a C++ linkable name). For example, if the user-defined functor `f` is declared, the shared-library must have a function `f` in the shared library with a C style argument passing mechanism.

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
.functor f(x:number):number
.functor g():symbol
```

Number types are implemented as ```int32_t``` and symbol types are implemented as ```const char *```. The float types are by default implemented as C `float` (or as C `double` if configured with `--enable-64bit-domain`). In Linux, a shared library can be generated with the following instructions:
```
g++ functors.cpp -c -fPIC -o functors.o 
g++ -shared -o libfunctors.so functors.o 
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH:+$LD_LIBRARY_PATH:}`pwd`
```
Assuming that the source code of the user-defined functors is stored in the source file ```functors.cpp```. The export command ensures that either the Soufflé interpreter or the generated executable can find the shared library.

If you are on the MAC OS X system, you need to create an additional dynamic library. For the creation of the dynamic library, use following instructions: 

```
g++ functors.cpp -c -fPIC -o functors.o 
g++ -shared -o libfunctors.so functors.o 
g++ -dynamiclib -install_name libfunctors.dylib -o libfunctors.dylib functors.o
export DYLD_LIBRARY_PATH=${DYLD_LIBRARY_PATH:+$DYLD_LIBRARY_PATH:}`pwd`
```

## Stateful User-Defined Functors 
Souffle exposes the symbol and record table to a stateful user-defined functor.
A stateful functor can access and manipulate these tables for [symbols and records](types).

For example, the following line
``` 
.functor mycat(a:symbol, b:symbol):symbol stateful
```
declares the stateful user-defined `mycat` functor in the Souffle program. The C++ implementation 
is shown below:

```
extern "C" { 
 souffle::RamDomain mycat(souffle::SymbolTable* symbolTable, souffle::RecordTable* recordTable,
        souffle::RamDomain arg1, souffle::RamDomain arg2) {
    assert(symbolTable && "NULL symbol table");
    assert(recordTable && "NULL record table");
    const std::string& sarg1 = symbolTable->decode(arg1);
    const std::string& sarg2 = symbolTable->decode(arg2);
    std::string result = sarg1 + sarg2;
    return symbolTable->encode(result);
 }
}
```
The first two parameters are pointers to Souffle's symbol and record table. 
Although the two arguments of the functor are symbols, only the ordinal numbers 
are passed on when the functor is called. To implement a concatenation, 
the ordinal numbers must be converted to strings first using 
the `decode()` method. The return value must be an ordinal number as well, 
hence the result of the concatenation is converted to an ordinal number
using the `encode` method. 

### Records
Stateful functors require a similar conversion between records and their ordinal numbers.


```
.functor myappend(x:List):List stateful
 
.type List = [x:number, y:List]
.decl L(x:List)
L([1,nil]).
L(@myappend(l)) :- L(l), l = [x, _l1], x < 10.
.output L
``` 

The C++ implementation has direct access to the record table:

```
souffle::RamDomain _myappend(
        souffle::SymbolTable* symbolTable, souffle::RecordTable* recordTable, souffle::RamDomain arg) {
    assert(symbolTable && "NULL symbol table");
    assert(recordTable && "NULL record table");
 
    if (arg == 0) {
        // Argument is nil
        souffle::RamDomain myTuple[2] = {0, 0};
        // Return [0, nil]
        return recordTable->pack(myTuple, 2);
    } else {
        // Argument is a list element [x, l] where
        // x is a number and l is another list element
        const souffle::RamDomain* myTuple0 = recordTable->unpack(arg, 2);
        souffle::RamDomain myTuple1[2] = {myTuple0[0] + 1, myTuple0[0]};
        // Return [x+1, [x, l]]
        return recordTable->pack(myTuple1, 2);
    }
}
``` 

## Syntax 
In the following, we define user-defined functor declarations more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### User-Defined Functors

![User-Defined Functor](https://souffle-lang.github.io/img/functor_decl.svg)

```ebnf
functor_decl
         ::= '.functor' IDENT '(' ( attribute ( ',' attribute )* )? ')' ':' type_name 'stateful'?

```

### Attribute Declaration

An attribute binds a name with a type. 

![Attribute](https://souffle-lang.github.io/img/attribute.svg)

```ebnf
attribute ::= IDENT ":" type_name 
```

### Type Name 

Souffle has pre-defined types such as `number`, `symbol`, `unsigned`, and `float`. Used-defined types have a name. If a type has been defined in a component, the type can be still accessed outside the component using a qualified name. 

![Type Name](https://souffle-lang.github.io/img/type_name.svg)
```ebnf
type_name ::=  "number" | "symbol" |"unsigned" | "float"  | IDENT ("." IDENT )*
```

{% include links.html %}
