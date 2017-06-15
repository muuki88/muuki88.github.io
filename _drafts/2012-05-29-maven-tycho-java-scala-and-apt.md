---
author: wp_admin
comments: false
date: 2012-05-29 10:13:58+00:00
layout: post
link: http://mukis.de/pages/maven-tycho-java-scala-and-apt/
slug: maven-tycho-java-scala-and-apt
title: Maven - Tycho, Java, Scala and APT
wordpress_id: 180
categories:
- Tutorials
tags:
- build
- Eclipse
- Java
- m2e
- maven
- maven3
- scala
- tycho
---

This tutorial shows a small project which is build with maven-tycho and the following requirements:



	
  * Mixed Java / Scala project

	
  * Eclipse plugin deployment

	
  * Eclipse Annotation Processing (APT)

	
  * Manifest-first approach

	
  * Java 7 / Scala 2.9.2




That doesn't sound too hard. In fact it isn't, if you are familiar with maven and how tycho works. 







### **Setting up maven**




First download [maven 3](http://maven.apache.org/download.html) and [configure](http://maven.apache.org/guides/mini/guide-configuring-maven.html) it.







I created two profiles in my _settings.xml_ and added some repositories.




My two profiles are **tycho-build** and **scala-build** which are activated with




the corresponding property present.






    
    <settings>
     <profiles>
      <profile>
       <id>tycho</id>
       <activation>
        <activeByDefault>false</activeByDefault>
        <property>
         <name>tycho-build</name>
        </property>
      </activation>
      <repositories>
       <repository>
        <id>eclipse-indigo</id>
        <layout>p2</layout>
        <url>http://download.eclipse.org/releases/indigo</url>
       </repository>
       <repository>
        <id>eclipse-sapphire</id>
        <layout>p2</layout>
        <url>http://download.eclipse.org/sapphire/0.4.1/repository</url>
       </repository>
       <repository>
        <id>eclipse-scala-ide</id>
        <layout>p2</layout>
       <url>http://download.scala-ide.org/releases-29/milestone/site</url>
      </repository>
      <repository>
       <id>eclipse-gemini-dbaccess</id>
       <layout>p2</layout>
       <url>http://download.eclipse.org/gemini/dbaccess/updates/1.0</url>
       </repository>
      </repositories>
     </profile>
    
     <profile>
      <id>scala</id>
      <activation>
       <activeByDefault>false</activeByDefault>
        <property>
         <name>scala-build</name>
        </property>
       </activation>
      <repositories>
       <repository>
        <id>scala-tools.org</id>
        <name>Scala-tools Maven2 Repository</name>
        <url>http://scala-tools.org/repo-releases</url>
       </repository>
       <repository>
        <id>typesafe</id>
        <name>Typesafe Repository</name>
        <url>http://repo.typesafe.com/typesafe/releases/</url>
       </repository>
      </repositories>
     <pluginRepositories>
      <pluginRepository>
        <id>scala-tools.org</id>
        <name>Scala-tools Maven2 Repository</name>
        <url>http://scala-tools.org/repo-releases</url>
       </pluginRepository>
      </pluginRepositories>
     </profile>
    </profiles>
    </settings>







### Setting up the project - The tycho build




For my project I just used two simple plugins. Nothing fancy here.











	
  1. Create plugin-project

	
  2. Add some dependencies

	
  3. Write some classes in Java




I recommend the following project structure



    
    root-project/
     plugin.core
     plugin.ui
     plugin.xy




go to your _root-project_ folder in your favorite console and use the following command to generate _pom.xml_ with tycho.






    
    mvn org.sonatype.tycho:maven-tycho-plugin:generate-poms -DgroupId=de.mukis -Dtycho.targetPlatform=path/to/target/platform/







which generates a first project for you. A few things to "tweak" as I saw it as a best-practice in most of the other tutorials:











	
  * Replace all concrete version numbers with property placeholders, e.g 0.12.0 with ${tycho.version}

	
  * Remove all _groupId_ and version tags in the pom.xml. The parent _pom.xml_ will generate these.

	
  * Check your folder structure. Tycho infers AND **changes** your source directory according to your **build.properties**.







Next add the p2 repositories needed to resolve all dependencies. This is done via the _[<repository>](http://wiki.eclipse.org/Tycho/Reference_Card#Repository_providing_the_context_of_the_build)_ tag. The full pom.xml is at the end.




Sometimes you have existing OSGi bundles but no p2 repository you can use it. Eclipse PDE has a nice extra feature for you. _[Features and bundles publisher application](http://wiki.eclipse.org/Equinox/p2/Publisher#Features_And_Bundles_Publisher_Application). _Note: It's very important that your repository folder has two folder **plugins** and **features**.







Now you can run your maven build with






    
    mvn clean package







and you will get a nice packaged osgi bundle.










### Setting up the project - The scala build







So now we want to add some Scala classes. Create new source folder src/main/scala and create some classes. Don't forget to import Scala packages. So your **MANIFEST.MF** contains something like:






    
    Import-Package: org.osgi.framework;version="1.6.0",
     scala;version="[2.9.0.1,2.9.3.0]",
     scala.collection;version="[2.9.0.1,2.9.3.0]",
     scala.collection.generic;version="[2.9.0.1,2.9.3.0]",
     scala.collection.immutable;version="[2.9.0.1,2.9.3.0]",
     scala.collection.interfaces;version="[2.9.0.1,2.9.3.0]",
     scala.collection.mutable;version="[2.9.0.1,2.9.3.0]",
     scala.collection.parallel;version="[2.9.0.1,2.9.3.0]",
     scala.collection.parallel.immutable;version="[2.9.0.1,2.9.3.0]",
     scala.collection.parallel.mutable;version="[2.9.0.1,2.9.3.0]",
     scala.concurrent;version="[2.9.0.1,2.9.3.0]",
     scala.concurrent.forkjoin;version="[2.9.0.1,2.9.3.0]",
     scala.io;version="[2.9.0.1,2.9.3.0]",
     scala.math;version="[2.9.0.1,2.9.3.0]",
     scala.parallel;version="[2.9.0.1,2.9.3.0]",
     scala.ref;version="[2.9.0.1,2.9.3.0]",
     scala.reflect,
     scala.reflect.generic;version="[2.9.0.1,2.9.3.0]",
     scala.runtime;version="[2.9.0.1,2.9.3.0]",
     scala.text;version="[2.9.0.1,2.9.3.0]",
     scala.util;version="[2.9.0.1,2.9.3.0]",







No there are, too alternatives to build. I choose to add the source folder in my build.properties and exclude the .scala files in my maven pom. The alternative is described [here](https://bugs.eclipse.org/bugs/show_bug.cgi?id=380766).







We need the maven scala plugin. Add the repository









    
    ...
     <repository>
      <id>scala-tools.org</id>
      <name>Scala-tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
     </repository>
    ...
     <pluginRepository>
      <id>scala-tools.org</id>
      <name>Scala-tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
     </pluginRepository>







and to our root pom.xml we add the maven-scala-plugin






    
    <plugin>
     <groupId>org.scala-tools</groupId>
     <artifactId>maven-scala-plugin</artifactId>
     <version>2.15.0</version>
     <executions>
      <execution>
       <id>compile</id>
       <goals>
        <goal>compile</goal>
       </goals>
       <phase>compile</phase>
      </execution>
    
      <execution>
       <id>test-compile</id>
       <goals>
        <goal>testCompile</goal>
       </goals>
       <phase>test-compile</phase>
      </execution>
    
      <execution>
       <phase>process-resources</phase>
       <goals>
        <goal>compile</goal>
       </goals>
      </execution>
     </executions>
    </plugin>







There is actually an [easier version](Compiling circular dependent java-scala classes), but which doesn't work with [circular dependencies](Compiling circular dependent java-scala classes).







If you have added the src/main/scala folder in your build.properties, than you have to add another plugin, to prevent tycho from exporting all scala source files.






    
    <plugin>
     <groupId>org.eclipse.tycho</groupId>
     <artifactId>tycho-compiler-plugin</artifactId>
     <version>${tycho.version}</version>
     <configuration>
      <excludeResources>
       <excludeResource>**/*.scala</excludeResource>
      </excludeResources>
     </configuration>
    </plugin>







Now the build should work with scala, too.







### **Setting up the project - APT code generation with Eclipse Sapphire**




I'm creating some models with [Eclipse Sapphire](http://www.eclipse.org/sapphire/) which uses [Java Annotation Processing (APT)](http://docs.oracle.com/javase/6/docs/technotes/guides/apt/index.html) to generate the models. [Apt-maven-plugin](http://mojo.codehaus.org/apt-maven-plugin/) is a maven allows us to trigger a processing factory during the build process. The current version alpha-04 has a [bug](http://jira.codehaus.org/browse/MOJO-1702?page=com.atlassian.jira.plugin.system.issuetabpanels:changehistory-tabpanel) which leads to an error with java 7. So, before we can use this plugin you have to[ checkout the source code](http://mojo.codehaus.org/source-repository.html) and build the latest alpha-05 version as it's not released at the moment. Install it in your local maven repository.







Now you can add the [apt-maven-plugin](http://mojo.codehaus.org/apt-maven-plugin/) to your plugin which needs apt. This could look like






    
    <?xml version="1.0" encoding="UTF-8"?>
    <project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
     <groupId>de.lmu.ifi.dbs.knowing</groupId>
     <artifactId>Knowing</artifactId>
     <version>0.1.4-SNAPSHOT</version>
    </parent>
    
    <artifactId>de.lmu.ifi.dbs.knowing.core</artifactId>
    <packaging>eclipse-plugin</packaging>
    
    <build>
     <plugins>
      <plugin>
       <groupId>org.codehaus.mojo</groupId>
       <artifactId>apt-maven-plugin</artifactId>
       <version>1.0-alpha-5-SNAPSHOT</version>
       <executions>
        <execution>
         <goals>
          <goal>process</goal>
         </goals>
        </execution>
       </executions>
       <configuration>
      <factory>org.eclipse.sapphire.sdk.build.processor.internal.APFactory</factory>
       </configuration>
      </plugin>
     </plugins>
    </build>
    </project>







At last you have  to add the factory as optional dependencies to your MANIFEST.MF of your plugin using apt.






    
    org.eclipse.sapphire.sdk;bundle-version="[0.4.0,0.5.0)";resolution:=optional,
    org.eclipse.sapphire.sdk.build.processor;bundle-version="[0.4.0,0.5.0)";resolution:=optional







I you trigger the build, you will see that your apt sources are generated in **target/generated-sources/apt**. However the files are not compiled. At first I tried the [maven-build-helper](http://mojo.codehaus.org/build-helper-maven-plugin/), but tycho seems to override these settings. So i added target/generated-sources/apt to the _build.properties_ of the plugin using apt, which seems for my as a bad work-around. However it works fine.




### 




### Source Code


You can find the code in [my github repository](https://github.com/muuki88/tycho).


### Conclusion


For a beginner it was not that easy to avoid all little traps with tycho, scala, maven apt. But in the end I hope to safe a lot of time when building and testing.


### Things to add




The tutorial doesn't include any testing.







#### Links


[https://github.com/muuki88/tycho
](https://github.com/muuki88/tycho)[http://wiki.eclipse.org/Tycho/Reference_Card
](http://wiki.eclipse.org/Tycho/Reference_Card)[http://mattiasholmqvist.se/2010/02/building-with-tycho-part-1-osgi-bundles/
](http://mattiasholmqvist.se/2010/02/building-with-tycho-part-1-osgi-bundles/)[https://github.com/misto/Scala-Hello-World-Plug-in
](https://github.com/misto/Scala-Hello-World-Plug-in)[Compiling circular dependent java-scala classes
](http://stuq.nl/weblog/2008-11-26/4-steps-to-add-scala-to-your-maven-java-projects)[Eclipse sapphire and tycho
](http://wiki.eclipse.org/Sapphire_Modeling_Tycho_build_support)[compile generated sources
](http://stackoverflow.com/questions/2916790/maven-doesnt-compile-target-hibernate3-generated-sources )[http://mojo.codehaus.org/apt-maven-plugin/
](http://mojo.codehaus.org/apt-maven-plugin/)[APT M2E Connector
](http://marketplace.eclipse.org/content/apt-m2e-connector)[Publish pre-compiled bundles in p2 repository](http://wiki.eclipse.org/Equinox/p2/Publisher#Features_And_Bundles_Publisher_Application)
