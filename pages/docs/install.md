---
title: Install Soufflé
permalink: /install
redirect_from: /docs/install
sidebar: docs_sidebar
folder: docs
---

You can build and install Soufflé following the instructions from [Build Soufflé](/build).

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

## MAC OS X 

We have a brew formula for MAC OS X using the [brew](http://brew.sh) system. The 
brew formula automates the build and installation on MAC OS X. 
To install Soufflé please type: 

```
brew install --HEAD souffle-lang/souffle/souffle
```

{% include links.html %}
