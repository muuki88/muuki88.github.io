---
date: 2013-02-15 17:05:35+00:00
layout: post
title: Maven Reports in Jenkins
wordpress_id: 332
categories:
- build
- checkstyle
- cobertura
- findbugs
- Jenkins
- JUnit
- maven
- maven3
- report
- Surefire
---

Code quality is a sensitive topic. It affects your maintenance cost as well as your customer satisfaction. Not to mention your developers motivation to work with the code. Who wants to fix ugly code, right?

Discussing code quality always needs hard facts and numbers! So this is a short tutorial how to create some simple reports to analyze some code quality metrics.

# Reports


This section will shorty explain the used reports.


## [Findbugs](http://mojo.codehaus.org/findbugs-maven-plugin/)


> FindBugs looks for bugs in Java programs. It is based on the concept of bug patterns. A bug pattern is a code idiom
> that is often an error


[![FindBugs Analysis](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_027-1024x271.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_027.png) FindBugs Analysis


## [Checkstyle](http://maven.apache.org/plugins/maven-checkstyle-plugin/)


> Checkstyle is a development tool to help programmers write Java code that adheres to a coding standard. It automates
> the process of checking Java code to spare humans of this boring (but important) task. This makes it ideal for
> projects that want to enforce a coding standard.


[![Checkstyle Analysis](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_026-1024x285.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_026.png) Checkstyle Analysis


## [Cobertura Code Coverage](http://mojo.codehaus.org/cobertura-maven-plugin/)


> Cobertura is a free Java tool that calculates the percentage of code accessed by tests. It can be used to identify
> which parts of your Java program are lacking test coverage. It is based on jcoverage.


[![Cobertura Report](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_025-1024x352.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_025.png) Cobertura Report


## [Surefire Test Report](http://maven.apache.org/surefire/maven-surefire-plugin/)


> The Surefire Plugin is used during the test phase of the build lifecycle to execute the unit tests of an application.
> It generates reports...


[![Surefire Testreport](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_028-1024x104.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_028.png) Surefire Testreport


# Basic pom.xml

Starting with a basic pom configuration:


```xml
<project>

  ...
  <properties>
     <findbugs.version>2.5.2</findbugs.version>
     <checkstyle.version>2.9.1</checkstyle.version>
     <surefire.reportplugin.version>2.12.4</surefire.reportplugin.version>
     <cobertura.version>2.5.2</cobertura.version>
  </properties>

  <build>
     <plugins>
        <plugin>
           <groupid>org.codehaus.mojo</groupid>
           <artifactid>findbugs-maven-plugin</artifactid>
           <version>${findbugs.version}</version>
        </plugin>
        <plugin>
           <groupid>org.codehaus.mojo</groupid>
           <artifactid>cobertura-maven-plugin</artifactid>
           <version>${cobertura.version}</version>
           <configuration>
               <formats>
                   <format>xml</format>
               </formats>
           </configuration>
        </plugin>
     </plugins>
  </build>

  <reporting>
     <plugins>
        <plugin>
           <groupid>org.codehaus.mojo</groupid>
           <artifactid>findbugs-maven-plugin</artifactid>
           <version>${findbugs.version}</version>
        </plugin>
        <plugin>
           <groupid>org.apache.maven.plugins</groupid>
           <artifactid>maven-checkstyle-plugin</artifactid>
           <version>${checkstyle.version}</version>
        </plugin>
        <plugin>
           <groupid>org.apache.maven.plugins</groupid>
           <artifactid>maven-surefire-report-plugin</artifactid>
           <version>${surefire.reportplugin.version}</version>
        </plugin>
        <plugin>
           <groupid>org.codehaus.mojo</groupid>
           <artifactid>cobertura-maven-plugin</artifactid>
           <version>${cobertura.version}</version>
           <configuration>
               <formats>
                   <format>xml</format>
               </formats>
           </configuration>
        </plugin>
      </plugins>
   </reporting>
</project>
```


# Jenkins Plugins

You need to install a few jenkins plugins to get a nice integration with your reports.


* [Static Analysis Collector Plug-in](https://wiki.jenkins-ci.org/display/JENKINS/Static+Code+Analysis+Plug-ins)
* Static Analysis Utilities
* Checkstyle Plug-in
* FindBugs Plug-in
* [Jenkins Cobertura Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Cobertura+Plugin)

# Project Configuration

Now you need to configure your project to show the results of your reports.


## Findbugs and Checkstyle

[![FindBugs and Checkstyle](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_029-1024x362.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_029.png) FindBugs and Checkstyle

You can configure them in the "build configuration" tab. There are some limits to set, which influence the representation.

## Cobertura

[![Cobertura Config](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_030-1024x498.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_030.png) Cobertura Config

Cobertura is configured in the "post-build actions". Same configurations as in the findbugs and checkstyle plugin.


# Result

On your main page of your project you have some new graphs and links.

[![Jenkins Trend Graphs](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_031.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_031.png)

Jenkins Trend Graphs

[![Jenkins Navbar](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_032.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_032.png)

Jenkins Navbar
