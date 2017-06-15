---
date: 2014-09-25 17:52:29+00:00
title: SBT native packager multi-module-build, assembly and custom formats
layout: post
categories:
- Allgemein
- Tutorials
tags:
- debian
- playframework
- rpm
- sbt
- sbt-native-packager
- scala
- tutorial
---

Lately the questions on [Stackoverflow](http://stackoverflow.com/questions/tagged/sbt-native-packager) and the Issues [around](https://github.com/sbt/sbt-native-packager/issues) sbt-native-packager were often about topics concerning


* [Multi Module Builds](https://github.com/muuki88/sbt-native-packager-examples/tree/master/multi-module-build)
Aggregating multiple projects into a single native package
* [SBT-Assembly jar](https://github.com/muuki88/sbt-native-packager-examples/tree/master/assembly-one-jar)
Aggregate everything into a  fat-jar and package this instead of each single jarfile
 [Change Mappings](https://github.com/muuki88/sbt-native-packager-examples/tree/master/linux-mappings)
Changing default mappings, remove ones you don't need or add new ones
* [Custom Formats](https://github.com/muuki88/sbt-native-packager-examples/tree/master/custom-package-format)
Creating your own packaging type. The SBT part will only take you minutes.

Within the next 0.7.x and 0.8.x release we will update the docs as well, but until then you can checkout the [sbt-native-packager-examples](https://github.com/muuki88/sbt-native-packager-examples) on github.
