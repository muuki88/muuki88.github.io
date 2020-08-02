---
date: 2014-06-26 22:07:09+00:00
layout: post
title: How to add a maven-plugin jar as dependency to sbt
excerpt: How to add maven plugins as sbt dependencies
categories:
- maven-plugin
- sbt
- scala
---

I want to use the [jdeb](https://github.com/tcurdt/jdeb) library to integrate in one of my own libraries.
Since it is a maven-plugin it's packaged as a maven plugin. SBT does not resolve the jars if you just add it as a
dependency. This will do the trick:

```scala
"org.vafer" % "jdeb" % "1.2" artifacts (Artifact("jdeb", "jar", "jar"))
```    


This one is for sbt 0.13.5!
