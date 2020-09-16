---
title: Listing files from a FTP Server with Akka Streams
excerpt: 'A practical example of Akka Streams'
categories:
  - blog
tags:
  - scala
  - akka
  - alpakka
header:
  image: '/images/header.jpg'
---

## Intro

## Prerequisites

- JDK 8
- Sbt
- Docker

## Project Setup

Create an `sbt` project with the following dependecy:

```scala
scalaVersion := "2.13.3"

val AkkaVersion = "2.5.31"

libraryDependencies ++= Seq(
  "com.lightbend.akka" %% "akka-stream-alpakka-ftp" % "2.0.1",
  "com.typesafe.akka" %% "akka-stream" % AkkaVersion
)
```

## Running a FTP server using Docker

Download the Docker image:

```docker
docker pull stilliard/pure-ftpd
```

Run the image:

```
docker run --rm -d --name ftpd_server -p 21:21 -p 30000-30009:30000-30009 stilliard/pure-ftpd bash /run.sh -c 30 -C 10 -l puredb:/etc/pure-ftpd/pureftpd.pdb -E -j -R -P localhost -p 30000:30059
```

## Conclusion

- The type classes are generic traits that are define in the `cats` package.
- Each type has a companion object an apply method for materializing instance, one or more construction methods for creating instances, and other helper methods.
- Default instances for standard library types are provided in the package `cats.instances`
- Many type classes have `syntax` provided in `cats.syntax` package.
