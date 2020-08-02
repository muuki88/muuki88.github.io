---
date: 2012-11-20 22:08:52+00:00
title: Developing OpenSource
categories:
- opensource
---

Developing OpenSource is fun and is supported by a wide range of companies with free infrastructure.
In this articel I want to show how you can build a free development environment for your open source
project. Some stuff is Java (or more general JVM) specific, but some stuff can be applied to any project.


## Source Code Hosting

There are three big hoster I know, which do a real good job: [GitHub](https://github.com/),
[GoogleCode](http://code.google.com/) and [Bitbucket](https://bitbucket.org/).
For my projects I prefer GitHub as it focuses only on git and provide a good integration with some
other tools I use.


#### What GitHub offers


1. Obviously a git repository to host your source code
2. A nice landing page on your repo (_README.md_), which is always nice
3. Optional wiki (awesome for tutorials)
4. Optional issue tracker (just a small set of features, but enough for most projects)
  1. Milestones, Label and schedule support
  2. Eclipse[ Mylyn integration](https://github.com/blog/852-github-mylyn-connector-for-eclipse) with the issue tracker ([Markdown WikiText support](https://bugs.eclipse.org/bugs/show_bug.cgi?id=329528) will be added soon)
5. Pull Request, which is great feature for collaboration with external contributors
6. [GitHub Pages](http://pages.github.com/) (more on that later)
7. Android App


So basically you have everything to get started within GitHub. 300Mb free space (soft limit) for
your projects. You can really code a lot with 300Mb.


## Project Build


Using a build tool should be obligatory for an open source project. Let explain why:

**IDE files**
Of course, it's easier for you to checkout your IDE files inside the repository, because at the beginning it will only be you, coding in your project. However your favorite IDE maybe not [my favorite IDE](http://www.eclipse.org/) and now you must create all the IDE specific files yourself, which can be sometimes a though task.

**Unique build process**
There are no excuses "_my project doesn't_" compile as the build process is describe in a build file and
will be executed the identical way on every machine. Of course this implies you don't hardcode some
paths, which only exists on your machine.

**Continuous Integration**
Ah, such a magic word.  Continuous Integration isn't possible without a build tool, as every build server
needs some hints how to build this project.


###  Maven


[Maven](http://maven.apache.org/) is my favorite build tool. I know there a plenty of other build tools outside like
[sbt](http://www.scala-sbt.org/), [gradle](http://www.gradle.org/), [ant](http://ant.apache.org/) +
[ivy](http://ant.apache.org/ivy/). However as far as I experience some of them, maven is more verbose, but has a huge
ecosystem with a lot of tutorials, plugins and nice features. Some of them are


1. One build file [**pom.xml**](http://maven.apache.org/pom.html)
2. Good IDE integration for most IDEs, but commandline is handsome, too.
3. Writing your [own plugins](http://maven.apache.org/guides/plugin/guide-java-plugin-development.html) is [straightforward](http://wiki.jfrog.org/confluence/display/OSS/Maven+Anno+Mojo)
4. Open source repository server [Nexus OSS](http://www.sonatype.org/nexus/)
5. [Maven Central](http://search.maven.org/) to publish your projects
6. [Site generation](http://maven.apache.org/guides/mini/guide-site.html) for an easy project site.


## Continuous Integration

Pushing your changes to repository doesn't necessary involve that you run all your tests. However
this should be done! For this [travis-ci](https://travis-ci.org/) is great platform. It integrates very smoothly  with github.
You see the build-status of your project on pull-request and can integrate the build-status very
easily in your README.md or on any other website.


## Deploying your stuff

Of course at some point, you want to publish your stuff. Upload your jars is a possible way
and should be done, as nobody wants  to compile your library for himself (yeah, so guys really
love this, but that's not the majority). Maven Central is definitively  the right place to do this.
There is very [good tutorial](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide) 
how to get an account and publish your stuff. The process includes PGP signature generation and getting a bit more
familiar with maven release process, but it's definitely  worth it. And here the circle closes. You can use your
projects now in any other project you create like any other can.


## A small website would be nice


If the github README and the wiki is not enough, github has one more gift for your: [Github Pages](http://pages.github.com/)

As we use maven, we can create a nice site with **mvn site** which can be  customized and some
useful reports such as unitTest, checkstyle and findbugs can be added.


## Mailinglist


[GoogleGroups](https://groups.google.com/) is the first choice. Found nothing better yet.


## Show me a project


My current project I spend time on can be found [here](http://muuki88.github.com/jama-osgi/). Only the mailing list is currently not on
google groups as there is a legacy mailing list.
