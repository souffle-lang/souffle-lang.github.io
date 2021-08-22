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

## Debian/Ubuntu Systems

For Debian systems, the latest release version of Soufflé can be installed from the PackageCloudIO,

Execute PackageCloudIO's script
```
curl -s https://packagecloud.io/install/repositories/souffle-lang/souffle/script.deb.sh | sudo bash
```

Then to install souffle itself
```
sudo apt-get install souffle
```
{% include links.html %}
