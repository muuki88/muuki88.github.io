---
author: wp_admin
comments: false
date: 2012-10-25 18:00:04+00:00
layout: post
link: http://mukis.de/pages/simple-junit-tests-with-tycho-and-surefire/
slug: simple-junit-tests-with-tycho-and-surefire
title: Simple JUnit Tests with Tycho and Surefire
wordpress_id: 263
categories:
- Allgemein
- News
tags:
- Bundle
- Eclipse
- JUnit
- maven
- Plugin
- Surefire
- Test
- tycho
---

[Eclipse Tych](http://eclipse.org/tycho/)o requires a special packaging type for test bundles, [**eclipse-test-plugin**](http://wiki.eclipse.org/Tycho/Reference_Card#Test_bundles). This is okay, when you have your own eclipse based project with all the modularity you want. However sometimes you have legacy libraries or want to keep your source code and test code close to each other and don't want to create another plugin to run the tests, like in [this](https://github.com/muuki88/jama-osgi) project.

Tycho got a [sure-fire plugin](http://www.eclipse.org/tycho/sitedocs/tycho-surefire/tycho-surefire-plugin/plugin-info.html) which doesn't cover this case. So you need to configure good old [maven surefire plugin](http://maven.apache.org/plugins/maven-surefire-plugin/) for your needs. Before explaining, this is what the important part of the pom.xml looks like:

    
    <!-- plain surefire tests without tycho -->
    <testSourceDirectory>src/test/java</testSourceDirectory>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.12.4</version>
        <executions>
          <execution>
            <id>test</id>
            <phase>test</phase>
            <configuration>
              <includes>
                <include>**/*Test.java</include>
              </includes>
            </configuration>
            <goals>
              <goal>test</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>2.5.1</version>
        <executions>
          <execution>
            <id>compiletests</id>
            <phase>test-compile</phase>
            <goals>
              <goal>testCompile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>





	
  1. You must specifiy the test directory (**src/test/java**).

	
  2. You have to bind the maven compiler plugin to the test-compile phase so this directory gets compiled

	
  3. Activate the maven sure-fire-plugin


Thanks to this [mailing-list post](http://software.2206966.n2.nabble.com/Run-non-eclipse-junit-tests-td5089299.html).
