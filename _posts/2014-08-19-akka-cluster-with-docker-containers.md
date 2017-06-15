---
date: 2014-08-19 21:00:36+00:00
layout: post
title: Akka cluster with docker containers
categories:
- Allgemein
tags:
- Akka
- docker
- sbt
- sbt-native-packager
- scala
---

This article will show you how to build docker images that contain a single akka cluster application. You will be able
to run multiple seed nodes and multiple cluster nodes. 

The code can be found on [Github](https://github.com/muuki88/activator-akka-docker) and will be available as a
[Typesafe Activator](http://typesafe.com/activator/templates#filter:docker).


## If you don't know docker or akka


[Docker](https://www.docker.com/) is the new shiny star in the devops world. It lets you easily deploy images to any OS running docker, while providing an isolated environment for the applications running inside the container image.

[Akka](http://akka.io/) is a framework to build concurrent, resilient, distributed and scalable software systems. The cluster feature lets you distribute your Actors across multiple machines to achieve load balancing, fail-over and the ability to scale up and out.

<!-- more -->


## The big picture


This is what the running application will look like. No matter where your docker containers will run at the end of the day. The numbers at the top left describe the starting order of the containers.

[![akka-docker-bigpicture](http://mukis.de/pages/wp-content/uploads/2014/08/akka-docker-bigpicture.png)](http://mukis.de/pages/wp-content/uploads/2014/08/akka-docker-bigpicture.png)

First you have to start your seed nodes, which will "glue" the cluster together. After the first node is started all following seed-nodes have to know the ip address of the initial seed node in order to build up a single cluster. The approach describe in this article is very simple, but easily configurable so you can use it with other provision technologies like chef, puppet or zookeeper.

All following nodes that get started need at least one seed-node-ip in order to join the cluster.


## The application configuration


We will deploy a small akka application which only logs cluster events. The entrypoint is fairly simple:

```scala
object Main extends App {

  val nodeConfig = NodeConfig parse args

  // If a config could be parsed - start the system
  nodeConfig map { c =>
    val system = ActorSystem(c.clusterName, c.config)

    // Register a monitor actor for demo purposes
    system.actorOf(Props[MonitorActor], "cluster-monitor")

    system.log info s"ActorSystem ${system.name} started successfully"
  }

}
```


The tricky part is the configuration. First the _akka.remote.netty.tcp.hostname_ configuration needs to be set to the
docker ip address. The port configuration is unimportant as we have unique ip address thanks  to docker. You can read
more about [docker networking here](https://docs.docker.com/articles/networking/). Second the seed nodes should add
themselves to the _akka.cluster.seed-nodes_ list. And at last everything should be configurable through system
properties and environment variables. Thanks to the [Typesafe Config Library](https://github.com/typesafehub/config)
this is achievable (even with some sweat and tears).


1. Generate a small commandline parser with [scopt](https://github.com/scopt/scopt) and the following two parameters:
   `--seed` flag which determines if  this node starting should act as a seed node `([ip]:[port])...` unbounded list of _[ip]:[port]_ which represent the seed nodes
2. Split the configuration in three files
  1. `application.conf` which contains the common configuration
  2. `node.cluster.conf` contains only  the node specific configuration
  3. `node.seed.conf` contains only the seed-node specific configuration
3. A class `NodeConfig` which orchestrates all settings and cli parameters in the right order and builds a Typesafe Config object.


Take a closer look at the [NodeConfig](https://github.com/muuki88/activator-akka-docker/blob/master/src/main/scala/com/example/NodeConfig.scala)  class. The core part is this

```scala
// seed nodes as generated string from cli
(ConfigFactory parseString seedNodesString)
  // the hostname
  .withValue("clustering.ip", ipValue)
  // node.cluster.conf or node.seed.conf
  .withFallback(ConfigFactory parseResources configPath)
  // default ConfigFactory.load but unresolved
  .withFallback(config)
  // try to resolve all placeholders (clustering.ip and clustering.port)
  .resolve
```

The part to resolve the IP address is a bit hacky, but should work in default docker environments. First the _eth0_ interfaces is searched and then the first _isSiteLocalAddress_ is being returned. IP adresses in the following ranges are _local_: _172.16.xxx.xxx, 172.31.xxx.xxx , 192.168.xxx.xxx, 10.xxx.xxx.xxx._

The main cluster configuration is done inside the _clustering_ section of the `application.conf`

```config
clustering {
  # ip = "127.0.0.1" # will be set from the outside or automatically
  port = 2551
  cluster.name = "application"
}
```


The ip adress will be filled by the algorithm describe above if nothing else is set. You can easily override all settings with system properties.
E.g if you want to run a seed node and a cluster node inside your IDE without docker start both like this:

```bash
# the seed node
-Dclustering.port=2551 -Dclustering.ip=127.0.0.1 --seed
# the cluster node
-Dclustering.port=2552 -Dclustering.ip=127.0.0.1 127.0.0.1:2551
```


For sbt this looks like this

```bash
# the seed node
sbt runSeed
# the cluster node
sbt runNode
```




## The build


Next we build our docker image. The [sbt-native-packager plugin](https://github.com/sbt/sbt-native-packager) recently added experimental [docker support](http://www.scala-sbt.org/sbt-native-packager/DetailedTopics/docker.html), so we only need to  configure our build to be docker-ready. First add the plugin to your `plugins.sbt`.

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "0.7.4")
```

Now we add a few required settings to our `build.sbt`. You should use sbt 0.13.5 or higher.

```
// adds start script and jar mappings
packageArchetype.java_application

// the docker maintainer. You could scope this to "in Docker"
maintainer := "Nepomuk Seiler"

// Short package description
packageSummary := s"Akka ${version.value} Server"
```

And now we are set. Start sbt and run `docker:publishLocal` and a docker image will be created for you. The _Dockerfile_ is in _target/docker_ if you want to take a closer look what's created.


## Running the cluster


Now it's time to run our containers. The image name is by default `name:version`. For the our activator it's `akka-docker:2.3.4`. The seed ip adresses may vary. You can read it out of the console output of your seed nodes.

```bash
docker run -i -t -p 2551:2551 akka-docker:2.3.4 --seed
docker run -i -t -p 2551:2551 akka-docker:2.3.4 --seed 176.16.0.18:2551
docker run -i -t -p 2551:2551 akka-docker:2.3.4 176.16.0.18:2551 176.16.0.19:2551
docker run -i -t -p 2551:2551 akka-docker:2.3.4 176.16.0.18:2551 176.16.0.19:2551
```

## What about linking?


This [blog entry](http://blog.michaelhamrah.com/2014/03/running-an-akka-cluster-with-docker-containers/) describes a
different approach to build an akka cluster with docker. I used some of the ideas, but the basic concept is build ontop
of linking the docker contains. This allows you to get the ip and port information of the running seed nodes. While this
is approach is suitable for single host machines, it
[seems to get more messy](http://stackoverflow.com/questions/21283517/how-to-link-docker-services-across-hosts) when
working with multiple docker machines.

The setup in this blog requires only one thing: A central way of assigning host ips. If your seed nodes don't change their IP adresses you can basically configure almost everything already in your application.conf.


# Further Reading


* [Effective Akka](http://www.amazon.de/gp/product/1449360076/ref=as_li_tl?ie=UTF8&camp=1638&creative=6742&creativeASIN=1449360076&linkCode=as2&tag=mukis-21&linkId=DNMEDEUHKN32NL6F)![](http://ir-de.amazon-adsystem.com/e/ir?t=mukis-21&l=as2&o=3&a=1449360076)
* [Scala for the Impatient](http://www.amazon.de/gp/product/B007JWDMIE/ref=as_li_tl?ie=UTF8&camp=1638&creative=6742&creativeASIN=B007JWDMIE&linkCode=as2&tag=mukis-21&linkId=3IOQTWQC5IQ52XMG)![](http://ir-de.amazon-adsystem.com/e/ir?t=mukis-21&l=as2&o=3&a=B007JWDMIE)
* [Programming in Scala](http://www.amazon.de/gp/product/0981531644/ref=as_li_tl?ie=UTF8&camp=1638&creative=6742&creativeASIN=0981531644&linkCode=as2&tag=mukis-21&linkId=D7754ZGCFIEXE5SZ)![](http://ir-de.amazon-adsystem.com/e/ir?t=mukis-21&l=as2&o=3&a=0981531644)
