---
title: Install Soufflé
permalink: /install
redirect_from: /docs/install
sidebar: docs_sidebar
folder: docs
---

## Binary distributions 📦

If you are using Ubuntu, Fedora or Oracle Linux, get a packaged version from our [github Releases](https://github.com/souffle-lang/souffle/releases).

## From sources 🏗️

You can build and install Soufflé from the sources following the instructions from [Build Soufflé](/build).

## Older releases

⚠️**The PPAs (Personal Package Archives) are not maintained currently.**

### Debian/Ubuntu Systems

For Debian-based systems, the latest release version of Soufflé can be installed from the souffle-lang repository,

```
sudo wget https://souffle-lang.github.io/ppa/souffle-key.public -O /usr/share/keyrings/souffle-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/souffle-archive-keyring.gpg] https://souffle-lang.github.io/ppa/ubuntu/ stable main" | sudo tee /etc/apt/sources.list.d/souffle.list
sudo apt update
sudo apt install souffle
```

These steps install the signing key for the repository as well as adding the repository details to the package manager. The packages are built using Ubuntu 20.04 and are compatible with that and later releases. Older editions of Ubuntu will require building from source.

### Fedora

```
dnf install https://souffle-lang.github.io/ppa/fedora/36/x86_64/souffle.fedora36repo.rpm
dnf install souffle
````
Packages are built using Fedora 36 and are compatible with that and later releases. Older editions of Fedora will require building from source.


### Oracle Linux 8

```
dnf install https://souffle-lang.github.io/ppa/ol/8/x86_64/souffle.ol8repo.rpm
dnf install souffle
````
Packages built using Oracle Linux 8 are likely to be compatible with related OS installs such as CentOS8. If the installation fails, souffle will need to be built from source.


### MAC OS X 

We have a brew formula for MAC OS X using the [brew](http://brew.sh) system. The 
brew formula automates the build and installation on MAC OS X. 
To install Soufflé please type: 

```
brew install --HEAD souffle-lang/souffle/souffle
```

{% include links.html %}
