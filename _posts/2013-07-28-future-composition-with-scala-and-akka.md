---
date: 2013-07-28 21:17:32+00:00
title: Future Composition with Scala and Akka
excerpt: Scala is functional and object-oriented language, which runs on the JVM. For concurrent and/or parallel programming it is a suitable choice along with the Akka framework, which provides a rich toolset for all kind of concurrent tasks. In this post I want to show a little example how to schedule a logfile-search job on multiple files/servers with Fitires amd Actors.
categories:
- Allgemein
tags:
- Actor
- Akka
- Concurrent
- Future
- Future Composition
- Futures
- scala
---

[Scala](http://www.scala-lang.org/) is functional and object-oriented language, which runs on the JVM. For concurrent and/or parallel programming it is a suitable choice along with the [Akka framework](http://akka.io/), which provides a rich toolset for all kind of concurrent tasks. In this post I want to show a little example how to schedule a logfile-search job on multiple files/servers with [Futures](http://docs.scala-lang.org/overviews/core/futures.html) and [Actors](http://doc.akka.io/docs/akka/2.2.0/scala/actors.html).


## Setup


I created my setup with the Typesafe Activator Hello-Akka template. This results in a `build.sbt` file with the following content:

```scala
name := """hello-akka"""

version := "1.0"

scalaVersion := "2.10.2"

libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % "2.2.0",
  "com.typesafe.akka" %% "akka-testkit" % "2.2.0",
  "com.google.guava" % "guava" % "14.0.1",
  "org.scalatest" % "scalatest_2.10" % "1.9.1" % "test",
  "junit" % "junit" % "4.11" % "test",
  "com.novocode" % "junit-interface" % "0.7" % "test->default"
)

testOptions += Tests.Argument(TestFrameworks.JUnit, "-v")
```

## Scala build-in Futures


Scala has already a build-in support for Futures. The implementation is based on _java.util.concurrent_.
Let’s implement a Future which runs our log search.

```scala
import scala.concurrent._
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits._

object LogSearch extends App {

println("Starting log search")

val searchFuture = future {
  Thread sleep 1000
  "Found something"
}

println("Blocking for results")
  val result = Await result (searchFuture, 5 seconds)
  println(s"Found $result")
}
```

This is all we need to run our task in another thread. The implicit import from _ExecutionContext_ provides a default ExecutionContext which handles the threads the future is running on. After creating the future we wait with a blocking call **Await result** for our results. So far nothing too fancy.


## Future composition


There are a lot of examples where the
[for-yield](http://docs.scala-lang.org/overviews/core/futures.html#functional_composition_and_forcomprehensions) syntax
is used to compose future results. In our case we have a dynamic list of futures: the log search results from each server.

For testing future capabilities we will create a list of futures from a list of _ints_ which represent the time the task
will run. Types are just for clarification.

```scala
val tasks = List(3000, 1200, 1800, 600, 250, 1000, 1100, 8000, 550)
val taskFutures: List[Future[String]] = tasks map { ms =>
  future {
    Thread sleep ms
    s"Task with $ms ms"
  }
}
```

In the end, we want a_ List[String]_ as a result. This is done with the Futures companion object.

```scala
val searchFuture: Future[List[String]] = Future sequence taskFutures
```

And finally we can wait for our results with

```scala
val result = Await result (searchFuture, 2 seconds)
```

However this will throw a _TimeoutException_, as some of our tasks run more than 2 seconds. Of course we could increase
the timeout, but there error could always happen again, when a server is down. Another approach would be to handle the
exception and return an error. However all other results would be lost.


## Future - Timeout fallback

No problem, we generate a fallback, which will return a default value if the operation takes, too long. A very naive implementation for our fallback could look like this

```scala
def fallback[A](default: A, timeout: Duration): Future[A] = future {
  Thread sleep timeout.toMillis
  default
}
```

The fallback future will return after the executing thread has sleeped for the timeout duration. The calling code now
looks like this.

```scala
val timeout = 2 seconds
val tasks = List(3000, 1200, 1800, 600, 250, 1000, 1100, 8000, 550)
val taskFutures: List[Future[String]] = tasks map { ms =>
val search = future {
  Thread sleep ms
  s"Task with $ms ms"
}

Future firstCompletedOf Seq(search,
  fallback(s"timeout $ms", timeout))
}

val searchFuture: Future[List[String]] = Future sequence taskFutures

println("Blocking for results")
val result = Await result (searchFuture, timeout * tasks.length)
println(s"Found $result")
```

The important call here is `Future firstCompletedOf Seq(..)` which produces a future returning the result of the
**first** finished future.

This implementation is very bad as discussed
[here](http://stackoverflow.com/questions/17672786/scala-future-sequence-and-timeout-handling). In short: We are wasting
CPU time by putting threads to sleep. Also the blocking call timeout is more or less a guess. With a one-thread
scheduler it can actually take more time.


## Futures and Akka

Now let’s do this more performant and more robust. Our main goal is to get rid of the poor fallback implementation,
which was blocking a complete thread. The idea is now to schedule the fallback feature after a given duration. By this
you have all threads working on real, while the fallback future execution time is almost zero. Java has a
[ScheduledExecutorService](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ScheduledExecutorService.html)
on it’s own or you can use a different implementation, a
[HashedWheelTimer](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/util/HashedWheelTimer.html), by Netty. Akka used
to use the HashWheelTimer, but has now a
[own implementatio](http://doc.akka.io/api/akka/2.2.0/index.html#akka.actor.LightArrayRevolverScheduler)n.

So let’s start with the actor.

```scala
import akka.actor._
import akka.pattern.{ after, ask, pipe }
import akka.util.Timeout

class LogSearchActor extends Actor {

  def receive = {
    case Search(worktimes, timeout) =>
      // Doing all the work in one actor using futures
      val searchFutures = worktimes map { worktime =>
      val searchFuture = search(worktime)
      val fallback = after(timeout, context.system.scheduler) {
          Future successful s"$worktime ms > $timeout"
        }
        Future firstCompletedOf Seq(searchFuture, fallback)
      }

      // Pipe future results to sender
      (Future sequence searchFutures) pipeTo sender
    }

  def search(worktime: Int): Future[String] = future {
      Thread sleep worktime
      s"found something in $worktime ms"
  }
}

case class Search(worktime: List[Int], timeout: FiniteDuration)
```

The important part is the [after](http://doc.akka.io/docs/akka/snapshot/scala/futures.html#After) method call. You give
it a duration after which the future should be executed and as a second parameter the scheduler, which is the default
one of the actor system in our case. The third parameter is the future which should get executed. I use the
`Future.success` companion method to return a single string.

The rest of the code is almost identical.
[PipeTo](http://doc.akka.io/docs/akka/snapshot/scala/actors.html#Ask__Send-And-Receive-Future) is a akka pattern to
return results of a future to the sender. Nothing fancy here.

Now how to call all this. First the code

```scala
object LogSearch extends App {

println("Starting actor system")
val system = ActorSystem("futures")

println("Starting log search")
try {
  // timeout for each search task
  val fallbackTimeout = 2 seconds

  // timeout use with akka.patterns.ask
  implicit val timeout = new Timeout(5 seconds)

  require(fallbackTimeout < timeout.duration)

  // Create SearchActor
  val search = system.actorOf(Props[LogSearchActor])

  // Test worktimes for search
  val worktimes = List(1000, 1500, 1200, 800, 2000, 600, 3500, 8000, 250)

  // Asking for results
  val futureResults = (search ? Search(worktimes, fallbackTimeout))
    // Cast to correct type
    .mapTo[List[String]]
    // In case something went wrong
    .recover {
       case e: TimeoutException => List("timeout")
       case e: Exception => List(e getMessage)
  }
  // Callback (non-blocking)
  .onComplete {
      case Success(results) =>
         println(":: Results ::")
         results foreach (r => println(s" $r"))
         system shutdown ()
      case Failure(t) =>
         t printStackTrace ()
      system shutdown ()
  }

} catch {
  case t: Throwable =>
  t printStackTrace ()
  system shutdown ()
}

  // Await end of programm
  system awaitTermination (20 seconds)
}
```

The comments should explain most of the parts. This example is completly asynchronous and works with callbacks. Of course you can use the **Await result** call as before.


## Links

- [https://gist.github.com/muuki88/6099946](https://gist.github.com/muuki88/6099946)
- [http://doc.akka.io/docs/akka/2.1.0/scala/futures.html](http://doc.akka.io/docs/akka/2.1.0/scala/futures.html)
- [http://stackoverflow.com/questions/17672786/scala-future-sequence-and-timeout-handling](http://stackoverflow.com/questions/17672786/scala-future-sequence-and-timeout-handling)
- [http://stackoverflow.com/questions/16304471/scala-futures-built-in-timeout](http://stackoverflow.com/questions/16304471/scala-futures-built-in-timeout)
