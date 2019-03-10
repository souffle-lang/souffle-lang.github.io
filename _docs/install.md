---
layout: docs
title: Build Soufflé
permalink: /docs/install/
---
## Installation

## Debian/Ubuntu Systems

For Debian systems, the latest development version of Soufflé can be installed from the Bintray PPA,

[Bintray PPA](https://bintray.com/souffle-lang/deb-unstable)


You can update your system with unsupported packages from this untrusted PPA by adding https://dl.bintray.com/souffle-lang/deb-unstable to your system's Software Sources using the following instructions:

```
sudo apt-add-repository https://dl.bintray.com/souffle-lang/deb-unstable
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
sudo apt-get update
```

Then to install souffle itself
```
apt-get install souffle
```

## Centos7

```
yum install https://dl.bintray.com/souffle-lang/rpm-unstable/centos/7/x86_64/souffle-repo-centos-1.0.2-1.x86_64.rpm
yum install souffle
```

## Fedora

```
yum install https://dl.bintray.com/souffle-lang/rpm-unstable/fedora/27/x86_64/souffle-fedora-repo-1.0.2-1.x86_64.rpm
yum install souffle
```

## MAC OS X with [brew](http://brew.sh)

```
brew install souffle-lang/souffle/souffle
```

For the latest updates follow the development head

```
brew install --HEAD souffle-lang/souffle/souffle
```


