---
layout: post
title: SBT Remote Cache Recipes
date: 2021-01-08 20:30:00
excerpt: Integration examples for the sbt remote cache feature
categories: [sbt, continous-integration]
---

[SBT 1.4 introduced a remote build cache feature](https://www.scala-sbt.org/1.x/docs/Remote-Caching.html) that
can heavily reduce your local builds or parts of your CI pipeline. This short blog post demonstrates a few
integration possibilites.

- Using [Nexus OSS 3](https://help.sonatype.com/repomanager3)
- Using [Minio](https://min.io/)
- Using Jenkins Pipeline artifacts


# What is the SBT remote cache?

> The idea is for a team of developers and/or a continuous integration (CI) system to share build outputs. If the build is repeatable, the output from one machine can be reused by another machine, which can make the build significantly faster.

[See the SBT remote caching documentation for a bit more information](https://www.scala-sbt.org/1.x/docs/Remote-Caching.html).


# Exampe project

I created a project with

```bash
$ sbt new scala/scala3.g8
```

# Nexus OSS 3

[The Nexus Repository Manager 3](https://de.sonatype.com/nexus/repository-oss) is available as an community (OSS) and Pro
edition. It provides a huge variety of repository formats.

SBT remote cache requires a _raw_ repository with anonymous access.

## Setup a Nexus test instance

Nexus OSS 3 is [available as a docker image](https://hub.docker.com/r/sonatype/nexus3/).

```bash
$ docker run -p 8081:8081 --name nexus sonatype/nexus3
```

The image generates a random password. You can find it by running bash inside the docker
container and reading the `nexus-data/admin.password` file:

```bash
$ docker exec -it nexus /bin/bash
$ cat nexus-data/admin.password
3229f863-1102-411d-8cd6-e9d34e5934cd
``` 

Then go to [localhost:8081](http://localhost:8081), login, set a new password and open the _server administration and configuration_
page (the cog icon at the top).

Now we create a new _raw_ repository

1. Click on _Repositories_
2. Click _Create Repository_ and select _raw (hosted)_
3. Allow _redeploy_

Next we have to grant anonymous write access.

1. Open the _Roles_ page
2. Add a new user role `sbt-build-cache`
3. Add the following priveleges
    1. `nx-repository-view-raw-sbt-build-cache-add`
    2. `nx-repository-view-raw-sbt-build-cache-edit`
    3. `nx-repository-view-raw-sbt-build-cache-read`
4. Go to the _Users_ page
5. Add the `sbt-build-cache` role to the `nx-anonymous` user.

## Configuring SBT

You can get the repository URL from the _Repositories_ page. There's a _copy_ button :)

```scala
ThisBuild / pushRemoteCacheTo := Some("Nexus OSS 3 Remote Cache" at "http://localhost:8081/repository/sbt-build-cache/")
```

# Minio

[Minio](https://min.io/) is an S3 compatible object storage system. It can be easily setup via docker
or a static go binary and provides a nice web UI.

The catch is that this system can only be used internally as public access to the storage bucket is required
if you want don't want to configure anything else in SBT. IMHO this is a reasonable thing to start a single
minio instance internally only for your build cache artifacts.

## Setup a test minio instance

Start a minio server as described [in the mino docs](https://min.io/download).
I changed the data dir to `/tmp/minio-data` for testing purposes.


```bash
$ docker run -p 9000:9000 -e MINIO_ACCESS_KEY=minioadmin \
                          -edocker run -p 9000:9000 -e MINIO_ACCESS_KEY=minioadmin \
                          -e MINIO_SECRET_KEY=minioadmin \
                          -v /tmp/minio-data:/data minio/minio server /data MINIO_SECRET_KEY=minioadmin \
                          -v /tmp/minio-data:/data minio/minio server /data
```

Install the minio client `mc` as described [in the mino client docs](https://docs.min.io/docs/minio-client-quickstart-guide.html).
After that configure an alias for the local minio instance.

```bash
# set an alias 'minio' for your local minio installation
$ mc alias set minio http://localhost:9000 minioadmin minioadmin
.$ mc policy set public minio/sbt-build-cache/
```

## Configuring SBT

```scala
ThisBuild / pushRemoteCacheTo := Some("Minio Remote Cache" at "http://localhost:9000/sbt-build-cache")
```

Now you can push to minio. You can see the artifacts in the web ui at [localhost:9000](http://localhost:9000).
Credentials are `minioadmin` / `minioadmin`.

# Jenkins

I haven't tried this out, but the idea is to

- Create a local build cache in each jenkins job, e.g. in `file("./.build-cache")`
- Record the cache artifacts with the [jenkins `archiveArtifacts` directive](https://www.jenkins.io/doc/pipeline/tour/tests-and-artifacts/)
- Use the _latestSucessfullBuild_ URL as an additional resolver, e.g.
  ```scala
  remoteCacheResolvers += "Jenkins Build Cache".at("https://jenkins.your-company.com/job/my-application/lastSuccessfulBuild")
  ```

# Why do we want to use this?

In our continous deployment pipeline we start up a canary instance of a microservice before deploying it to production.
After the successful deployment we run a bunch of integration test against the internal APIs of this service. If these
pass, we run all the integration tests of services that depend on this service against the canary instance. We call these
the _regression tests_. 

We run these regression tests by checking out the latest successful commit of the service that depends on the service
that should be deployed and run `sbt IntergrationTest / test`. This needs to compile almost the complete application.
With the help of a build cache this can run the test almost immediately!
