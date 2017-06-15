---
date: 2012-12-23 17:10:48+00:00
title: JUnit Benchmarking
categories:
- Allgemein
tags:
- benchmarks
- Java
- JUnit
- maven
- maven3
---

Benchmarks are important if you have a performance critical application. Even if your application is not
performance critical, a better performance is always nice to have.

To check that your application works correctly [unit tests with JUnit](http://www.vogella.com/articles/JUnit/article.html) are the first choice in the Java world.
So why don't use unit tests to check our performance? There is an awesome small library called [JUnitBenchmarks](http://labs.carrotsearch.com/junit-benchmarks.html) which enables you to do that easily. There are some other frameworks like [Google Caliper](http://code.google.com/p/caliper/), but you don't get the entire JUnit comfort.


## Getting started


For the simple start go to the [JUnitBenchmarks Tutorial](http://labs.carrotsearch.com/junit-benchmarks-tutorial.html) page, which is very good. There is no black magic behind the scenes.


## Parameterized Benchmarks


Google Caliper has a nice feature to run the benchmarks with different types of parameters. JUnitBenchmark doesn't need that, as JUnit supports this out of the box. Taking the first very simple benchmark from JUnitBenchmark, a parameterized test could look like this:

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters;

import com.carrotsearch.junitbenchmarks.AbstractBenchmark;
import com.carrotsearch.junitbenchmarks.BenchmarkOptions;

@RunWith(Parameterized.class)
public class MyTest extends AbstractBenchmark {

  private final long sleep;

  public void MyTest(long sleep) {
    this.sleep = sleep;
  }

  @Parameters
  public static Collection<Object[]> data() {
    // The actual parameters
    Object[][] data = new Object[][] { { 20 }, { 50 }, { 100 } };
    return Arrays.asList(data);
  }

  @Test
  public void testSleep() throws Exception {
    Thread.sleep(sleep);
  }
}
```

Now you can run your benchmarks with different settings.


## Links


* [JUnit Benchmarks](http://labs.carrotsearch.com/junit-benchmarks.html)
* [Parameterized Tests with JUnit](http://www.asjava.com/junit/junit-time-and-parameterized-test/)
* [Custom JUnit Runner](http://www.asjava.com/junit/implement-junit-runner/)
