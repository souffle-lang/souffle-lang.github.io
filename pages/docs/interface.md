---
title: C++ Interface
permalink: /interface
sidebar: docs_sidebar
folder: docs
---
## Why do we need C++ interface?
Applications may want to integrate Soufflé programs as a library rather than as a stand-alone tool. 
For this purpose, we have developed a C++ interface to call Soufflé programs. 

With the option `-g <name>.cpp` a C++ class is generated for a Soufflé program that can be directly embedded in another C++ application. 
The generated class may have several instances.
An instance provides interfaces to populate input relations, 
to run the program, 
and to retrieve data from the output relations. 
Note that the generated class is required to be compiled with the flag `__EMBEDDED_SOUFFLE__` to disable the default `main()` function of a Soufflé program. 

## Detailed Usage
A C++ class generated from a Soufflé program can be instantiated and executed as described in this section.

### 0. Including the Soufflé interface

#### `souffle/SouffleInterface.h`
The C++ interface is included by including `souffle/SouffleInterface.h` as shown in the example. For any information not covered by this documentation, please refer to [the comments in this header file](https://github.com/souffle-lang/souffle/blob/master/src/include/souffle/SouffleInterface.h).

*Example*

```c++
#include<souffle/SouffleInterface.h>
```

### 1. Loading a program

#### `souffle::SouffleProgram* souffle::ProgramFactory::newInstance(std::string)`

Creates an instance of a program by name. Returns `nullptr` if the program could not be found.

*Example*

```c++
if (souffle::SouffleProgram *prog = souffle::ProgramFactory::newInstance("<name>")) {
    // Run the program...

    // Clean up
    delete prog;
} else {
    std::cerr << "Failed to create instance\n";
    exit(1);
}

```

### 2. Populating input relations

#### `void souffle::SouffleProgram::loadAll(std::string)`

Loads all input facts from CSV files stored in the given directory.

*Example*

```c++
prog->loadAll("<dir>"); // load facts from CSV files stored in <dir>
```

#### `souffle::Relation* souffle::SouffleProgram::getRelation(std::string)`

Queries a program for a relation by name. The relation can then be populated programmatically. Returns `nullptr` if the relation could not be found.

*Example*

```c++
if (souffle::Relation *rel = prog->getRelation("<in-rel>")) {
    souffle::tuple myTuple(rel); // Create an empty tuple
    myTuple << "Hello" << 10;    // Write symbols and integers to tuple
                                 // (Arity and data-types must match those of <in-rel>)
    rel->insert(myTuple);        // Add the new tuple to the relation
} else {
    std::cerr << "Failed to get input relation\n";
    exit(1);
}
```

### 3. Running the program

#### `void souffle::SouffleProgram::run()`

Executes the program, without any loads or stores.

*Example*

```c++
prog->run();
```

### 4. Reading output relations

#### `void souffle::SouffleProgram::printAll()`

Writes all output relations to their defined destinations as CSV.

*Example*

```
prog->printAll();
```

#### `souffle::Relation* souffle::SouffleProgram::getRelation(std::string)`

As before, queries a program for a relation by name. The relation can then be read programmatically. Returns `nullptr` if the relation could not be found.

*Example*

```c++
if (souffle::Relation *rel = prog->getRelation("<out-rel>")) {
    int myInt;
    std::string mySymbol;
    for (auto &output : *rel) {       // Iterate through the tuples in the output relation
      output >> mySymbol >> myString; // Read symbols and integers from each tuple
                                      // (Data-types must match those of <out-rel>)
    }
} else {
    std::cerr << "Failed to get output relation\n";
    exit(1);
}
```

#### `bool souffle::Relation::contains(const souffle::tuple&)`

Returns `true` if the relation contains a tuple equal to a given tuple and `false` otherwise.

*Example*

```c++
souffle::tuple myTuple(rel);
myTuple << "A" << 123;
if (rel->contains(myTuple)) {
    // ...
}
```

#### `std::string souffle::Relation::getSignature()`

Gets the signature of a relation. The signature is in the form:

```
<<primitive type 1>:<type name 1>,<primitive type 2>:<type name 2>...>
```

for all of the attributes in the relation.

*Example*

```c++
rel->getSignature();
```

#### `std::size_t souffle::Relation::size()`

Returns the number of tuples in a relation.

*Example*

```c++
rel->size();
```

## Complete Example

```c++
if (souffle::SouffleProgram *prog = souffle::ProgramFactory::newInstance("<name>")) {
    prog->loadAll("<dir>");

    if (souffle::Relation *rel = prog->getRelation("<in-rel>")) {
        souffle::tuple myTuple(rel);
        myTuple << "Hello" << souffle::RamSigned(10);
        rel->insert(myTuple);
    } else {
        std::cerr << "Failed to get input relation\n";
        exit(1);
    }

    prog->run();

    if (souffle::Relation *rel = prog->getRelation("<out-rel>")) {
        souffle::RamSigned myInt;
        std::string mySymbol;
        for (auto &output : *rel) {
          output >> mySymbol >> myInt;
        }

        souffle::tuple myTuple(rel);
        myTuple << "A" << souffle::RamSigned(123);
        if (rel->contains(myTuple)) {
        }
    } else {
        std::cerr << "Failed to get output relation\n";
        exit(1);
    }

    prog->printAll();

    delete prog;
} else {
    std::cerr << "Failed to create instance\n";
    exit(1);
}
```
