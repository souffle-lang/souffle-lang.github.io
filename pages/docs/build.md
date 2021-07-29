---
title: Build Soufflé
permalink: /build
sidebar: docs_sidebar
folder: docs
---

## Software Requirements

To build and install Soufflé, the following software must be installed:

* cmake (from version 3.15), GNU C++ supporting C++17 and OpenMP (from version 4.8), Bison (from version 3.0.4), Flex, DoxyGen, sqlite3, mcpp

Clang++ (with OpenMP support) can be used as an alternative for C++.

### Ubuntu/Debian Build

On a Ubuntu/Debian system, following command installs the necessary developer tools to compile and build Soufflé:

```
sudo apt-get install cmake bison build-essential clang doxygen flex g++ git libffi-dev libncurses5-dev libsqlite3-dev make mcpp python sqlite zlib1g-dev
```

Support for C++17 is required, which is supported in gnu c++ 7/clang++ 7 on.

The Soufflé project follows cmake conventions for configuring, installing and building software. If not familiar, please refer to the [following document](https://cliutils.gitlab.io/modern-cmake/chapters/intro/running.html) for configuring, building, testing, and installing a project with cmake. You can build Soufflé by typing:
```
cd souffle
cmake -S . -B build
cmake --build build
```
This may take up some time. However, this may be sped up with, for instance:
```
cmake --build build -j8
```
which will run `cmake` with 8 jobs at a time.

### MAC OS X Build

MAC OS X does not have OpenMP/C++ nor a bison version 3.0.2 or higher installed by default. OpenMP is not required, but does improve runtime for large programs by parallelising the execution. 

We recommend [brew](http://brew.sh) to install the required tools for building Soufflé. Run the following commands prior to executing `cmake`:
```
brew update
brew install cmake bison libffi mcpp pkg-config
brew reinstall gcc
brew link bison --force
brew link libffi --force
export PKG_CONFIG_PATH=/usr/local/opt/libffi/lib/pkgconfig/
```

Note: Be careful with the search path for bison, so it points to the correct one. By default, macOS includes bison 2.3 at `/usr/bin/bison`, however brew installs the newer version to `/usr/local/bin/bison`. This can be done by prepending this directory to the path, however, this can break other systems - `PATH=/usr/local/bin:$PATH`. Also note that the version of gcc installed may be different, so 'g++-8' may need to be changed.

Soufflé is built by 

```
cd souffle
cmake -S . -B build
cmake --build build
```
This may take up some time. However, this may be sped up with, for instance:
```
cmake --build build -j8
```
which will run `cmake` with 8 jobs at a time.

### Testing Soufflé

With 
```
  cd build
  ctest
```
numerous unit tests and regression tests are performed. This may take up to 45min.
However, this may be sped up with, for instance:
```
ctest -j8
```
which will run 8 jobs at a time.

For more control, `-R <regex>` allows you to run tests that matching the regular expression;
`-L <regex>` allows you to run tests with _labels_ matching the regular expression, use `--print-labels` to show all available labels;
`--rerun-failed` allows you to run only the tests that failed previously.
For more options please type `ctest --help`.

### Installing Soufflé

If you would like to install Soufflé in your system, specify an installation directory with `./configure --prefix=<install-dir>`. The executable, scripts, and header files will be stored in the directory ```<install-dir>```. Use an absolute path for ```<install-dir>```. Type 
```
 cd souffle
 cmake -S . -B build -DCMAKE_INSTALL_PREFIX=<install-dir>
 cmake --build build --target install
```
to install Soufflé. By setting the path variable 
```
 PATH=$PATH:<install-dir>/bin
``` 
the Soufflé commands ```souffle``` and ```souffle-profile``` are available to the users.


### Configuring Soufflé

To specify building options, run `cmake -S . -B build -D<option>=<value>`. See below some of the options:

 - `SOUFFLE_DOMAIN_64BIT` Enable(=`ON`)/disable(=`OFF`) 64-bit number values in Datalog tuples; default is `OFF`, i.e., domain size is 32 bit.
 - `SOUFFLE_FAST_DEBUG` Enable(=`ON`)/disable(=`OFF`) 'fast debug' mode; default is `ON`.
 - `SOUFFLE_NDEBUG` Enable(=`ON`)/disable(=`OFF`) runtime checks even in release mode; default is `OFF`.
 - `SOUFFLE_FAST_RELEASE` Enable(=`ON`)/disable(=`OFF`) fast release mode; default is `ON`.
 - `SOUFFLE_SANITISE_MEMORY` Enable(=`ON`)/disable(=`OFF`) memory sanitiser; default is `OFF`.
 - `SOUFFLE_SANITISE_THREAD` Enable(=`ON`)/disable(=`OFF`) thread sanitiser; default is `OFF`.
 - `SOUFFLE_SWIG` Enable(=`ON`)/disable(=`OFF`) SWIG language interface for various languages; default is `OFF`.
 - `SOUFFLE_SWIG_PYTHON` Enable(=`ON`)/disable(=`OFF`) Python SWIG interface; default is `OFF`.
 - `SOUFFLE_SWIG_JAVA` Enable(=`ON`)/disable(=`OFF`) Java SWIG interface; default is `OFF`.
 - `SOUFFLE_USE_CURSES`  Enable(=`ON`)/disable(=`OFF`) ncurses-based provenance display"; default is `ON`.
 - `SOUFFLE_USE_ZLIB` Enable(=`ON`)/disable(=`OFF`) use of libz file compression in I/O directives (i.e. `.input`/`.output`); default is `ON`.
 - `SOUFFLE_USE_SQLITE` Enable(=`ON`)/disable use of sqlite3 in I/O directives (i.e. `.input`/`.output`);  default is `ON`. 

The avaiable options are documented in `souffle/CMakeLists.txt`. Alternatively, run `ccmake build` that invokes cmake with a GUI interface and configure the options there.

## IDE/Editor
With CMake, you can easily integrate the Souffle project with your IDE/editor.
For example, if you are using clangd, simply run `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1` to generate the configuration file.


{% include links.html %}
