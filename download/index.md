---
layout: page
title: Download
---

### [Releases](https://github.com/souffle-lang/souffle/releases/)
The current releases are built automatically for Debian-based systems and MAC OS X.

### Ubuntu
A repository for the latest unstable Ubuntu debs can be set up with
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
sudo add-apt-repository https://dl.bintray.com/souffle-lang/deb-unstable/
```
The stable release repository can be set up with
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
sudo add-apt-repository https://dl.bintray.com/souffle-lang/deb/
```

### Fedora
A repository for the latest unstable Fedora rpms can be set up with
```
sudo dnf install -y https://dl.bintray.com/souffle-lang/rpm-unstable/fedora/27/x86_64/souffle-fedora-repo-1.0.2-1.x86_64.rpm
sudo dnf install souffle
```

### CentOS/Oracle Linux
A repository for the latest unstable Centos rpms can be set up with
```
sudo yum install -y https://dl.bintray.com/souffle-lang/rpm-unstable/centos/7/x86_64/souffle-repo-centos-1.0.2-1.x86_64.rpm
sudo yum install souffle
```

### [Source Code](https://github.com/souffle-lang/souffle)

Follow the [build instructions](/docs/build) to build and install Soufflé locally on your computer. 

The source code can be obtained by git, as a zip-file, or as tar-ball.

* <a href="https://github.com/souffle-lang/souffle" class="btn">View on GitHub</a>
* <a href="https://github.com/souffle-lang/souffle/zipball/master" class="btn">Download .zip</a>
* <a href="https://github.com/souffle-lang/souffle/tarball/master" class="btn">Download .tar.gz</a>

The latest build status for various systems can be found below:

[Jenkins Build System](http://plang1.it.usyd.edu.au/jenkins)
