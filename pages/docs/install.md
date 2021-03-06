---
title: Install Soufflé
permalink: /install
redirect_from: /docs/install
sidebar: docs_sidebar
folder: docs
---
## Installation

Currently the only way to install Soufflé is by building from source. 
Please follow the instructions from [Build Soufflé](https://souffle-lang.github.io/build). 
However, MAC OS X with [brew](http://brew.sh) can automate the build process: 
```
brew install --HEAD souffle-lang/souffle/souffle
```

<!---
## Debian/Ubuntu Systems

For Debian systems, the latest development version of Soufflé can be installed from the Bintray PPA,

To add the repository:
```
echo "deb https://dl.bintray.com/souffle-lang/deb-unstable bionic main" | sudo tee -a /etc/apt/sources.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
sudo apt-get update
```
Packages for Debian buster and Ubuntu focal are also available by replacing 'bionic' with 'buster' or 'focal'.

Then to install souffle itself
```
sudo apt-get install souffle
```

Stable versions do not have the latest features, but if you require a stable version replace 'deb-unstable' with 'deb' in the above instructions.


## Centos7

To add the repository and install souffle:
```
yum install https://dl.bintray.com/souffle-lang/rpm-unstable/centos/7/x86_64/souffle-repo-centos-1.0.2-1.x86_64.rpm
yum install souffle
```

## Fedora

To add the repository and install souffle:
```
yum install https://dl.bintray.com/souffle-lang/rpm-unstable/fedora/27/x86_64/souffle-fedora-repo-1.0.2-1.x86_64.rpm
yum install souffle
```

Stable versions do not have the latest features, but if you require a stable version replace 'rpm-unstable' with 'rpm' in the above instructions.

## MAC OS X with [brew](http://brew.sh)

```
brew install souffle-lang/souffle/souffle
```

For the latest updates follow the development head

```
brew install --HEAD souffle-lang/souffle/souffle
```
-->

{% include links.html %}
