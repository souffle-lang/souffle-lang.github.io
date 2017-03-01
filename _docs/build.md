---
layout: docs
title: Build Soufflé
permalink: /docs/build/
---
## Pre-Build Packages

Soufflé has pre-built packages for Debian and MAC OS X. You find the pre-built packages [here](http://github.com/souffle-lang/souffle/releases/latest).

## Software Requirements

To build and install Souffle, the following software must be installed:

* Make, Autoconf tools, GNU G++ supporting C++11 and OpenMP (from version 4.8), Bison (from version 3.0.2), Flex, DoxyGen, Java JDK

CLANG++ with OpenMP support can be used as an alternative for G++.

### Ubuntu/Debian Build

On a Ubuntu/Debian system, following command installs the necessary developer tools to compile and build Soufflé:

```
sudo apt-get install autoconf automake bison build-essential clang doxygen flex g++ git make openjdk-8-jdk python
```

Note that the Ubuntu/Debian version needs to be recent such that G++ 4.8 is part of the standard distribution.

The Soufflé project follows automake/autoconf conventions for configuring, installing and building software. To configure, build, test, and install the project, type:
```
 cd souffle
 sh ./bootstrap
 ./configure
 make
```


### MAC OS X Build

MAC OS X does not have OpenMP/C++ nor a bison version 3.0.2 or higher installed. We recommend [brew](http://brew.sh) to install the required tools to build Soufflé. Run the following commands prior to executing `./configure`:
```
brew update
brew install autoconf automake bison libtool
brew reinstall gcc --without-multilib
brew link bison --force
```

Note: Be careful with the search path for bison, so it points to the correct one. By default, macOS includes bison 2.3 at `/usr/bin/bison`, however brew installs the newer version to `/usr/local/bin/bison`. This can be done by prepending this directory to the path, however, this can break other systems - `PATH=/usr/local/bin:$PATH`.

Currently, the `souffle-wave` preprocessor crashes in macOS, and requires to overwrite the executable in `src/` with a symlink to `cpp-X`. Currently on macOSX, brew installs cpp to `/usr/local/Cellar/gcc/6.2.0/bin/cpp-6`. This step must occur after `make`, and will be overwritten each time it is built. In order to run the tests after we overwrite `souffle-wave`, run `make check` in the `tests/` directory **instead of** running `make check` in the the base directory.

In addition, [Java JDK](https://java.com/en/download/) is necessary to build the profiler of Soufflé. 

Soufflé is built by 

```
cd souffle
export CXX=/usr/local/bin/g++-5
export CXXFLAGS=-fopenmp
export SOUFFLECPP=/usr/local/bin/cpp-5
sh ./bootstrap
./configure
make
```

### Testing Soufflé

With 
```
 make check
```
numerous unit tests and regression tests are performed. This may take up to 45min.
However, this may be sped up with, for instance:
```
TESTSUITEFLAGS=-j8 make check
```
which will run 8 jobs at a time.

### Installing Soufflé 

If you would like to install Soufflé in your system, specify an installation directory with `./configure --prefix=<install-dir>`. The executable, scripts, and header files will be stored in the directory ```<install-dir>```. Use an absolute path for ```<install-dir>```. Type 
```
 make install
```
to install Soufflé. By setting the path variable 
```
 PATH=$PATH:<install-dir>/bin
``` 
the Soufflé commands ```souffle``` and ```souffle-profile``` are available to the users.

