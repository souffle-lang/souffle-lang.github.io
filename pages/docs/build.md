---
title: Build Soufflé
permalink: /build
sidebar: docs_sidebar
folder: docs
---

## Software Requirements

To build and install Souffle, the following software must be installed:

* Make, Autoconf tools, GNU G++ supporting C++11 and OpenMP (from version 4.8), Bison (from version 3.0.4), Flex, DoxyGen

Clang++ with OpenMP support can be used as an alternative for G++.

### Ubuntu/Debian Build

On a Ubuntu/Debian system, following command installs the necessary developer tools to compile and build Soufflé:

```
sudo apt-get install autoconf automake bison build-essential clang doxygen flex g++ git libncurses5-dev libtool libsqlite3-dev make mcpp python sqlite zlib1g-dev
```

Support for C++11 is required, which is fully supported in g++ 5.1/clang++ 4.0 on.

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
brew install autoconf automake bison libffi libtool mcpp pkg-config
brew reinstall gcc
brew link bison --force
brew link libffi --force
export PKG_CONFIG_PATH=/usr/local/opt/libffi/lib/pkgconfig/
```

Note: Be careful with the search path for bison, so it points to the correct one. By default, macOS includes bison 2.3 at `/usr/bin/bison`, however brew installs the newer version to `/usr/local/bin/bison`. This can be done by prepending this directory to the path, however, this can break other systems - `PATH=/usr/local/bin:$PATH`.

Soufflé is built by 

```
cd souffle
export CXX=g++-8 #Your version of g++ may vary
export CC=gcc-8  #Your version of gcc may vary
./bootstrap
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


{% include links.html %}
