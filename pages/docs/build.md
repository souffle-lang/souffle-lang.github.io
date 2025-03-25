---
title: Build Soufflé
permalink: /build
sidebar: docs_sidebar
folder: docs
---

## Build with Docker and install (Linux only)

```
# Build the image that will build souffle from sources
#
# For Fedora 41:
# Other distributions are available too.
docker build . -f ./.github/images/fedora-41/Dockerfile -t souffle-builder-image

# Actually build souffle and create a package
#
# Look for message 'CPack: - package: /souffle/build/souffle-VERSION-OS.EXTENSION'
#
docker run -e DOMAIN_SIZE="64bit" -t souffle-builder-image --name souffle-builder

# Copy the package onto your system
docker cp souffle-builder:/souffle/build/souffle-VERSION-OS.EXTENSION .

# Use your system package manager to install souffle from the package.
#
# For DNF package manager (.rpm package extension):
dnf install souffle-VERSION-OS.rpm
```

## Software Requirements

To build and install Soufflé, the following software must be installed:

* cmake (version 3.15 or greater)
* Python 3
* A C++ compiler supporting C++17: GNU C++, clang++, Microsoft Visual Studio
* OpenMP (optional)
* GNU Bison (version 3.6 or greater)
* flex
* DoxyGen (optional)
* git (optional)
* libffi (optional)
* libncurses (optional)
* libsqlite3 (optional)
* mcpp (optional)
* swig (optional)
* zlib (optional)

## Build

### CMake configuration options

All available options can be found by browsing the source code, see `souffle/CMakeLists.txt`. Below is a table of the most relevant.

To specify build options, run `cmake -S . -B build -D<option>=<value>`. Alternatively, run `cmake build` to start cmake with a terminal GUI interface that lets you configure the options.


| Option | Default | Description
|--------|---------|------------
| `CMAKE_BUILD_TYPE`      | `Release` | `Release`, `Debug`
| `SOUFFLE_DOMAIN_64BIT`  | `OFF`     | Support 64-bit integer and float values in Datalog tuples instead of default's 32-bit
| `SOUFFLE_SWIG`          | `OFF`     | Support all [`SWIG`](swig) builds
| `SOUFFLE_SWIG_JAVA`     | `OFF`     | Support Java `SWIG` build
| `SOUFFLE_SWIG_PYTHON`   | `OFF`     | Support Python `SWIG` build
| `SOUFFLE_USE_CURSES`    | `ON`      | Support terminal-UI based [provenance](provenance) display
| `SOUFFLE_USE_LIBFFI`    | `ON`      | Support [c++ functors](interface) with arbitrary number of arguments. When `OFF` limitations applies on the number of arguments in c++ functors supported by the interpreter engine, see `StatelessFunctorMaxArity` and `CALL_STATEFULL` in source code.
| `SOUFFLE_USE_OPENMP`    | `ON`      | Support parallel evaluation with OpenMP
| `SOUFFLE_USE_SQLITE`    | `ON`      | Support [sqlite I/Os](directives#iosqlite)
| `SOUFFLE_USE_ZLIB`      | `ON`      | Support `compress` [file I/Os](directives#iofile)
| `SOUFFLE_SANITISE_MEMORY` | `OFF`   | Build with memory sanitiser
| `SOUFFLE_SANITISE_THREAD` | `OFF`   | Build with thread sanitiser

### Ubuntu/Debian Dependencies

On a Ubuntu/Debian system the following command installs the necessary dependencies to compile and build Soufflé:

```sh
sudo apt install \
  bison \
  build-essential \
  clang \
  cmake \
  doxygen \
  flex \
  g++ \
  git \
  libffi-dev \
  libncurses5-dev \
  libsqlite3-dev \
  make \
  mcpp \
  python \
  sqlite3 \
  zlib1g-dev
```

### Fedora Dependencies

```sh
sudo dnf install bison flex libffi-devel sqlite-devel zlib-devel
```

### Linux Build

Support for C++17 is required. This support is present in GNU C++ version 7 or greater and Clang++ version 7 or greater.

The Soufflé project follows cmake conventions for configuring, installing and building software. If you are not familiar with cmake, please refer to the [following document](https://cliutils.gitlab.io/modern-cmake/chapters/intro/running.html) for configuring, building, testing, and installing a project.

You can build Soufflé by typing:

```sh
cd souffle
cmake -S . -B build ## example of options: -DCMAKE_BUILD_TYPE=Release -DSOUFFLE_DOMAIN_64BIT=ON
cmake --build build
```
This may take up some time. However, this may be sped up with, for instance:
```sh
cmake --build build -j8
```
which will run `cmake` with 8 jobs at a time.

### MAC OS X Build

MAC OS X does not have OpenMP/C++ nor GNU Bison version 3.0.2 or higher installed by default. OpenMP is not required to build Soufflé, but does improve runtime for large Datalog programs by parallelising the execution.

We recommend [brew](http://brew.sh) to install the required tools for building Soufflé. Run the following commands prior to executing `cmake`:

```sh
brew update
brew install cmake bison libffi mcpp pkg-config
brew reinstall gcc
brew link bison --force
brew link libffi --force
export PKG_CONFIG_PATH=/usr/local/opt/libffi/lib/pkgconfig/
```

Note: Be careful with the search path for bison, so it points to the correct one. By default, macOS includes bison 2.3 at `/usr/bin/bison`, however brew installs the newer version to `/usr/local/bin/bison`. This can be done by prepending this directory to the path, however, this can break other systems - `PATH=/usr/local/bin:$PATH`. Also note that the version of gcc installed may be different, so `g++-8` may need to be changed.

Soufflé is built by 

```sh
cd souffle
cmake -S . -B build
cmake --build build
```
This may take up some time. However, this may be sped up with, for instance:
```sh
cmake --build build -j8
```
which will run `cmake` with 8 jobs at a time.

### Windows & Visual Studio Build

See `.github/workfloww/VS-CI-Tests.yml` for an example of how to build and test on Windows with Visual Studio 2019 using package managers `Chocolatey` and `vcpkg` to resolve dependencies.

## Testing Soufflé

With:
```sh
  cd build
  ctest
```
Numerous unit tests and regression tests are performed. This may take up to 45min.
However, this may be sped up with, for instance:
```sh
ctest -j8
```
which will run 8 jobs at a time.

For more control, `-R <regex>` allows you to run tests that matches the regular expression;
`-L <regex>` allows you to run tests with _labels_ matching the regular expression, use `--print-labels` to show all available labels;
`--rerun-failed` allows you to run only the tests that failed previously.

```sh
cd build
# run tests in interpreted mode (fast)
ctest -L interpreted -VV --output-on-failure
# run tests in compiled mode (slower)
ctest -LE interpreted -VV --output-on-failure
```

For more options please type `ctest --help`.

## Installing Soufflé

If you would like to install Soufflé in your system, specify an installation directory with `-DCMAKE_INSTALL_PREFIX=<install-dir>`. The executable, scripts, and header files will be stored in the directory ```<install-dir>```. Use an absolute path for ```<install-dir>```. Type 
```sh
 cd souffle
 cmake -S . -B build -DCMAKE_INSTALL_PREFIX=<install-dir>
 cmake --build build --target install
```
to install Soufflé. By setting the path variable 
```sh
 PATH=$PATH:<install-dir>/bin
``` 
the Soufflé commands ```souffle``` and ```souffleprof``` are available to the users.


## IDE/Editor
With CMake, you can easily integrate the Soufflé project with your IDE/editor.
For example, if you are using clangd, simply run `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1` to generate the configuration file.


{% include links.html %}
