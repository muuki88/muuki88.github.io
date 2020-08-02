---
layout: post
title: "SBT AutoPlugins Tutorial"
date: 2014-11-01 14:12:48
categories: [sbt, autoplugins]
---

This tutorial will guide you through the process of writing your own sbt plugin. There are several reasons to do this
and it's really simple

* Add customized build steps to your continuous integration process
* Provide default settings for different environments for various projects

Before you start, make sure to have [SBT installed](http://www.scala-sbt.org/) on your machine and accessible via the
commandline. The code is available [on github](https://github.com/muuki88/sbt-autoplugins-tutorial). If you start at the
first commit you can just step-through the tutorial with each commit.

## Setup
The first step is to setup our plugin project. There is only one important setting in your `build.sbt`, the rest is up to you.

```scala
sbtPlugin := true
```

This will mark you project as a sbt-plugin build. For this tutorial I'm using sbt *0.13.7-M3*, which lets you write your
build.sbt as you like. No need for separating lines. The complete `build.sbt` looks like this.


```scala
name := "awesome-os"
organization := "de.mukis"

scalaVersion in Global := "2.10.2"

sbtPlugin := true

// Settings to build a nice looking plugin site
site.settings
com.typesafe.sbt.SbtSite.SiteKeys.siteMappings &lt;+= (baseDirectory) map { dir =&gt;
  val nojekyll = dir / "src" / "site" / ".nojekyll"
  nojekyll -&gt; ".nojekyll"
}
site.sphinxSupport()
site.includeScaladoc()

// enable github pages
ghpages.settings
git.remoteRepo := "git@github.com:muuki88/sbt-autoplugins-tutorial.git"

// Scripted - sbt plugin tests
scriptedSettings
scriptedLaunchOpts <+= version apply { v => "-Dproject.version="+v }
```

The plugins used inside this build are configured in the
[plugins.sbt](https://github.com/muuki88/sbt-autoplugins-tutorial/blob/master/project/plugins.sbt). Nothing special.

# The Plugin

Now we implement a first working version of our plugin and a test project to try it out. What the plugin will actually
do is printing out awesome operating systems. Later we will customize this behavior.

Let's take our look at our plugin code.

```scala
import sbt._
import sbt.Keys.{ streams }

/**
 * This plugin helps you which operating systems are awesome
 */
object AwesomeOSPlugin extends AutoPlugin {

  /**
   * Defines all settings/tasks that get automatically imported,
   * when the plugin is enabled
   */
  object autoImport {
    lazy val awesomeOsPrint = TaskKey[Unit]("awesome-os-print", "Prints all awesome operating systems")
    lazy val awesomeOsList = SettingKey[Seq[String]]("awesome-os-list", "A list of awesome operating systems")
  }

  import autoImport._

  /**
   * Provide default settings
   */
  override lazy val projectSettings = Seq(
    awesomeOsList := Seq(
      "Ubuntu 12.04 LTS","Ubuntu 14.04 LTS","Debian Squeeze",
      "Fedora 20","CentOS 6",
      "Android 4.x",
      "Windows 2000","Windows XP","Windows 7","Windows 8.1",
      "MacOS Maverick","MacOS Yosemite",
      "iOS 6","iOS 7"
    ),
    awesomeOsPrint := {
      awesomeOsList.value foreach (os => streams.value.log.info(os))
    }
  )

}
```

And that's it. We define two keys. *AwesomeOsList* is a *SettingKey*, which means it's set upfront and will only change
if some explicitly sets it to another value or change it,e.g.

```scala
awesomeOsList += "Solaris"
```

*awesomeOsPrint* is a task, which means it gets executed each time you call it.
## Test Project

Let's try the plugin out. For this we create a test project which has a plugin dependency on our awesome os plugin.
We create a **test-project** directory at the root directory of our plugin project. Inside **test-project**
we add a `build.sbt` with the following contents:

```scala
name := "test-project"
version := "1.0"

// enable our now plugin
enablePlugins(AwesomeOSPlugin)
```

However the real trick is done inside the `test-project/project/plugins.sbt`. We create a reference to a project in
the parent directory:

```scala
// build root project
lazy val root = Project("plugins", file(".")) dependsOn(awesomeOS)

// depends on the awesomeOS project
lazy val awesomeOS = file("..").getAbsoluteFile.toURI
```

And that's all. Run sbt inside the test-project and print out awesome operating systems.

```bash
sbt awesomeOsPrint
```

If you change something in your plugin code just call **reload** and your test-project will recompile the changes.
## Add a new task and test it
Next, we add a task which stores the awesomeOsList inside a file. This is something we can automatically test. Testing
sbt-plugins is a bit tedious, but doable with the scripted-plugin.

First we create a folder inside **src/sbt-test. **The directories inside **sbt-test** can be seen as categories where
you put your tests into. I created a **global** folder where I put two test projects. The critical configuration is
again inside the **project/plugins.sbt**

```scala
addSbtPlugin("de.mukis" % "awesome-os" % sys.props("project.version"))
```

The scripted plugin first plublishes the plugin locally and then passes the version number to each started sbt test
build via the system property *project.version*. We added this behaviour in our `build.sbt` earlier:

```scala
scriptedLaunchOpts <+= version apply { v => "-Dproject.version="+v }
```

Each test project contains a file called *test*, which can contain sbt commands and some simple check commands.
you put in some simple checks like *file exists* and do the more sophisticated stuff inside a task defined in the test
project.

The test file for our second test looks like this.

```bash
# Create the another-os.txt file
> awesomeOsStore
$ exists target/another-os.txt
> check-os-list
```


The **check-os-list** task is defined inside the `build.sbt` of the test project
(*/src/sbt-test/global/store-custom-oslist/build.sbt*.

```scala
enablePlugins(AwesomeOSPlugin)

name := "simple-test"

version := "0.1.0"

awesomeOsFileName := "another-os.txt"

// this is the scripted test
TaskKey[Unit]("check-os-list") := {
  val list = IO.read(target.value / awesomeOsFileName.value)
  assert(list contains "Ubuntu", "Ubuntu not present in awesome operating systems: " + list)
}
```

## Separate operating systems per plugin

Our next goal is to customize the operating system list, so users may choose what systems they like most. We do this by
[generating a configuration scope](http://eed3si9n.com/4th-dimension-with-sbt-013)for each operating system category and
a plugin that configures the settings in this scope.

In a real-world plugin you can use this to define different actions in different environments. E.g. develope, staging or
 production. This is a very crucial point of autoplugins as it allows you to enable specific plugins to get a different
 build flavor and/or create different scopes which are configured by different plugins.

The first step is to create three new autoplugins: `AwesomeWindowsPlugin`, `AwesomeMacPlugin` and
`AwesomeLinuxPlugin`. They will all work in the same fashion:


* Scope the projectSettings from [AwesomeOSPlugin to there custom defined configuration scope](http://stackoverflow.com/questions/17437443/how-can-i-make-an-sbt-key-see-settings-for-the-current-configuration/17479710#17479710)
and provide them as setting
* Override specific settings/tasks inside the custom defined configuration scope


The `AwesomeLinuxPlugin` looks like this

```scala
import sbt._

object AwesomeLinuxPlugin extends AutoPlugin{

  object autoImport {
    /** Custom configuration scope */
    lazy val Linux = config("awesomeLinux")
  }

  import AwesomeOSPlugin.autoImport._
  import autoImport._

  /** This plugin requires the AwesomeOSPlugin to be enabled */
  override def requires = AwesomeOSPlugin

  /** If all requirements are met, this plugin will automatically get enabled */
  override def trigger = allRequirements

  /**
   * 1. Use the AwesomeOSPlugin settings as default and scope them to Linux
   * 2. Override the default settings inside the Linux scope
   */
  override lazy val projectSettings = inConfig(Linux)(AwesomeOSPlugin.projectSettings) ++ settings

  /**
   * the linux specific settings
   */
  private lazy val settings: Seq[Setting[_]] = Seq(
    awesomeOsList in Linux := Seq(
      "Ubuntu 12.04 LTS",
      "Ubuntu 14.04 LTS",
      "Debian Squeeze",
      "Fedora 20",
      "CentOS 6",
      "Android 4.x"),
    // add awesome os to the general list
    awesomeOsList ++= (awesomeOsList in Linux).value
  )
}
```

The other plugins are defined in the same way. Let's try things out. Start sbt in your test-project.

```bash
sbt
awesomeOsPrint # will print all operating systems
awesomeWindows:awesomeOsPrint # will only print awesome windows os
awesomeMac:awesomeOsPrint # only mac
awesomeLinux:awesomeOsPrint # only linux
```

SBT already provides some scopes like *Compile*, *Test*, etc. So there's only a small need for creating your very own scopes. Most of the time you will use the already provided and customize these in your plugins.

One more note. You may wonder why the plugins are all getting enabled and we didn't have to change anything in the test-project. That's another benefit from autoplugins. You can specify **requires**, which define dependencies between plugins and **triggers** that specify when your plugin should be enabled.

```scala
// what is required that this plugin can be enabled
override def requires = AwesomeOSPlugin

// when should this plugin be enabled
override def trigger = allRequirements
```

The user of your plugin now doesn't have to care about the order he puts the plugins in his build.sbt, because the
developer defines the requirements upfront and sbt will try to fulfill them.

## Conclusion

SBT Autoplugins make the life of plugin users and developers a lot easier. It lowers the steep learning curve for sbt a bit and creates more readable buildfiles. For sbt-plugin developers the process of migrating isn't very difficult. Replacing <i>sbt.Plugin</i> with *sbt.AutoPlugin* and creating an *autoImport* field.
