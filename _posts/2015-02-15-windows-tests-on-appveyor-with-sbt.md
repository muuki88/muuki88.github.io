---
layout: post
title: Windows Tests on AppVeyor with SBT
date: 2015-02-15 12:50:52
excerpt: continous testing on windows with appveyor
categories: [sbt]
tags: [sbt, continous-integration, travis-ci, windows, appveyor]
---

For SBT Native Packager we rely on [travis-ci][travis-ci] to run our tests. However travis has not yet windows support,
so we have to run these tests manually, which is not nice. Lucky there's another free-for-opensource
continous integration server for windows name [AppVeyor][appveyor].

## Setup your appveyor.yml

The tricky part is to install sbt and let it run your tests. There is an awesome blogpost on
[how to do it with maven][maven-appveyor].
I took the script and changed it to work with sbt instead.

```yaml
version: '{build}'
os: Windows Server 2012
install:
  - cmd: choco install sbt -ia "INSTALLDIR=""C:\sbt"""
  - cmd: SET PATH=C:\sbt\bin;%JAVA_HOME%\bin;%PATH%
  - cmd: SET SBT_OPTS=-XX:MaxPermSize=2g -Xmx4g
build_script:
  - sbt clean compile
test_script:
  - sbt validateWindows
cache:
  - C:\sbt\
  - C:\Users\appveyor\.m2
  - C:\Users\appveyor\.ivy2
```


[appveyor]:  http://www.appveyor.com/
[maven-appveyor]: http://www.yegor256.com/2015/01/10/windows-appveyor-maven.html
[travis-ci]: https://travis-ci.org/sbt/sbt-native-packager
