---
date: 2011-12-11 22:09:04+00:00
layout: post
title: Deploying Eclipse RCP - The hard way
categories: [thoughts]
---

Today I had the most painful PDE headless build ever. This post just describes what I had to do to deploy an
Eclipse RCP application.


## The Application and Technology Stack


My application, _Medmon_, contains about 20 bundles. Most of the are written in plain Java (JDK 7). Some of them are
mixed projects with Scala (2.9.1.final). Medmon is a simple
[CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) application with an embedded
[derby db](http://db.apache.org/derby/) for backend. It relies heavily on [OSGi](http://www.osgi.org/Main/HomePage)
services and uses this concept whereever possible. As data persistence layer we use
[Eclipse Gemini JPA](http://www.eclipse.org/gemini/jpa/) and
[Eclipse Gemini DBAccess](http://www.eclipse.org/gemini/dbaccess/) which provide enterprise persistence for OSGi
systems. One plugin uses code generation via the
[Eclipse Annotation Processor](http://www.eclipse.org/jdt/apt/introToAPT.html), because the model is implemented
with [Eclipse Sapphire](http://www.eclipse.org/sapphire/). Finally the Eclipse version we're using ist 3.7.1. So I
summarize the technology stack:


* Eclipse Indigo (3.7.1) with [Scala IDE Plugin](http://www.scala-ide.org/), Sapphire Plugin and Git Plugin
* Persistence Layer: Eclipse Gemini JPA / DBAccess, EclipseLink 2.3.0
* Scala 2.9.1.final
* JDK 7


## The product


At the beginning there is the **product** definition. Defining this for an existing
**Eclipse Application** and your bundles organized in **Features** is really simple. We added a two vm arguments:

```
-DREFRESH_BUNDLES=false
```

Prevents Eclipse Gemini from refresh bundles which causes UI extension points to randomly
disappear.

```
-Dderby.system.home=${system_property:user.home}${system_property:file.separator}".derby"
```

Sets the _derby.system.home_ variable which is used to resolve the embedded db directory if not given absolute. The ${...} is resolved at runtime via Eclipse.

The medmon.derby plugin has a little specialty in its MANIFEST.MF

```
Eclipse-BundleShape: dir
```

which deploys the plugin[ as a directory](http://eclipsesource.com/blogs/2009/01/20/tip-eclipse-bundleshape/) instead of
a JAR file.

Let's deploy it!

Okay. I want to test in on my Ubuntu machine. Using Eclipse Export Product wizard is easy. The wizard generates a repository and an application folder. I'm now writing down every single step I had to do to run this product.


### 1. Redeploy all plugins with Scala code

The PDE headless build [ignores the Scala compiler](https://issues.scala-lang.org/browse/SI-1919) and just compiles all java classes. However it copies all .scala files in your JAR. If you want to or not. You have to redeploy your plugins with the _deploy plugin wizard_ and select **use compiled classes from workspace**. Now you got class plus scala files in your jar. Not good but better.


### 2. One plugin isn't compiled completly


One plain Java plugin didn't have any class classfiles in its deployed jar. I had to copy it manually in the jar, where I found a bug in Ubuntus Archive Manager: If you copy a directory recursive via drag and drop into the jar file only the leaf files are copied. So I have a package **x** and a subpackage **x.y**. But only classes in **x.y** where copied.


### 3. Eclipse-BundleShape: dir - only works deploying meta-repository

If you don't enable the "generate meta repository" option this MANIFEST option will be ignored.


### 4. Placeholder in eclipse.ini aren't translated in deployed products


The VM argument

```
-Dderby.system.home=${system_property:user.home}${system_property:file.separator}".derby"
```

isn't parsed at runtime in a deployed product. Derby tries to create a database relative to the execution folder his folders. Change eclipse.ini.


#### 5. Problems not mentioned or not tested yet

* Startlevel configuration via simpleconfigurator plugin.
* Multiplatform export with DeltaPack.
* [Why use Eclipse-BundleShape](https://bugs.eclipse.org/bugs/show_bug.cgi?id=364748)

### Final notes

I know there's maven. I know there are build servers. I know there's Tycho and I know there's m2scala. Someday I will
migrate this project. What annoys me the most is, that everything fails so silently. I'm a very young programmer and
I'm trying to do my best to get better. What are your experiences with Eclipse PDE headless build?
