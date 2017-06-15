---
author: wp_admin
comments: false
date: 2011-04-13 08:46:46+00:00
layout: post
link: http://mukis.de/pages/akka-and-osgi-development-in-eclipse/
slug: akka-and-osgi-development-in-eclipse
title: Akka and OSGi development in Eclipse
wordpress_id: 117
categories:
- Tutorials
---

This short tutorial is about how to run [akka](http://akka.io) in an OSGi environment. I faced
a lot of problems deploying in this in plain eclipse without maven, bnd or sbt.

This example is done with the java-API, however it is also possible with Scala.


### Requirements





	
  * Eclipse Helios 3.6.2 with Scala-Plugin

	
  * akka-1.1-modules distribution




### Configuration


First we have to do some minor changes in some Manifest files in the akka project.



	
  1. Extract akka-modules-1.1.zip, e.g ~/akka

	
  2. go to akka/lib_managed/compile

	
  3. open akka-actor-1.1.jar -> META-INF/MANIFEST.MF

	
  4. delete following line: _private-package: *_

	
  5. Do the same with akka-typed-actor-1.1.jar


Second you have to setup a **target-platform** which is used to run the OSGi environment.



	
  1. Go to windows->Preferences->Plugin development->Target Platform

	
  2. Add target platform, use default

	
  3. Extract your akka-modules-1.1.zip, e.g ~/akka




You need the follow plugins:





	
  1. guice-all-2.0.jar

	
  2. logback-classic-0.9.24.jar

	
  3. logback-core-0.9.24.jar

	
  4. slf4j-api-1.6.0.jar

	
  5. [Aspectwerkz by Jonas Bonér](https://github.com/jboner/aspectwerkz/zipball/v2.2.2)




### The bundle


Create a new plugin project. No contributions to the UI and an activator class.

Copy the following libs into your bundle and add them to your classpath in MANIFEST.MF



	
  * akka-actor-1.1.jar

	
  * akka-typed-actor-1.1.jar

	
  * akka-slf4j-1.1.jar


Create a class **MyActor**

    
    import akka.actor.UntypedActor;
    
    public class MyActor extends UntypedActor {
    
    	@Override
    	public void onReceive(Object msg) throws Exception {
    		System.out.println("Message: " + msg);
    	}
    
    }


Add these lines to your **Activator** class.



    
    import akka.actor.ActorRef;
    import akka.actor.Actors;
    
    //...
    
    	public void start(BundleContext bundleContext) throws Exception {
    		Activator.context = bundleContext;
    		ActorRef actor = Actors.actorOf(MyActor.class).start();
    		actor.sendOneWay("Hello You");
    	}


At last you have to edit the _MANIFEST.MF_.  It should look something like this. (I know
I may have to the smallest set of import scala packages).

    
    Manifest-Version: 1.0
    Bundle-ManifestVersion: 2
    Bundle-Name: Core
    Bundle-SymbolicName: de.lmu.ifi.dbs.knowing.core;singleton:=true
    Bundle-Version: 1.0.0.qualifier
    Bundle-Activator: de.lmu.ifi.dbs.knowing.core.internal.Activator
    Require-Bundle: org.eclipse.core.runtime,
     se.scalablesolutions.akka.actor;bundle-version="1.0.0",
     se.scalablesolutions.akka.stm;bundle-version="1.0.0",
     se.scalablesolutions.akka.typed.actor;bundle-version="1.0.0"
    Bundle-ActivationPolicy: lazy
    Bundle-RequiredExecutionEnvironment: JavaSE-1.6
    Bundle-ClassPath: .
    Import-Package: scala;version="2.9.0.1",
     scala.collection;version="2.9.0.l",
     scala.collection.generic;version="2.9.0.1",
     scala.collection.immutable;version="2.8.1.final",
     scala.collection.interfaces;version="2.8.1.final",
     scala.collection.mutable;version="2.8.1.final",
     scala.compat;version="2.8.1.final",
     scala.concurrent;version="2.8.1.final",
     scala.concurrent.forkjoin;version="2.8.1.final",
     scala.io;version="2.8.1.final",
     scala.math;version="2.8.1.final",
     scala.mobile;version="2.8.1.final",
     scala.ref;version="2.8.1.final",
     scala.reflect;version="2.8.1.final",
     scala.reflect.generic;version="2.8.1.final",
     scala.runtime;version="2.8.1.final",
     scala.text;version="2.8.1.final",
     scala.util;version="2.8.1.final",
     scala.util.automata;version="2.8.1.final",
     scala.util.continuations;version="2.8.1.final",
     scala.util.control;version="2.8.1.final",
     scala.util.grammar;version="2.8.1.final",
     scala.util.matching;version="2.8.1.final",
     scala.util.parsing.ast;version="2.8.1.final",
     scala.util.parsing.combinator;version="2.8.1.final",
     scala.util.parsing.combinator.lexical;version="2.8.1.final",
     scala.util.parsing.combinator.syntactical;version="2.8.1.final",
     scala.util.parsing.combinator.testing;version="2.8.1.final",
     scala.util.parsing.combinator.token;version="2.8.1.final",
     scala.util.parsing.input;version="2.8.1.final",
     scala.util.parsing.json;version="2.8.1.final",
     scala.util.parsing.syntax;version="2.8.1.final",
     scala.util.regexp;version="2.8.1.final"


Now let's run this!


### Launch configuration





	
  1. Open Run->Launch configurtions.

	
  2. Create a new OSGi Launch configuration

	
  3. Add the following bundles

	
    1. org.scala-ide.scala.library (2.8.1) (the akka scala library didn't work for me)

	
    2. se.scalablesolutions.akka.actor

	
    3. se.scalablesolutions.akka.osgi.dependencies.bundle

	
    4. se.scalablesolutions.akka.actor.typed.actor

	
    5. se.scalablesolutions.akka.actor.stm

	
    6. com.google.inject

	
    7. Equinox Runtime Components (e,g eclipse.runtime.core,..)




	
  4. Try to launch




Hope this works for you, too!
