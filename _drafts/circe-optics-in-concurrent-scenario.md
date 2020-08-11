---
title: Circe Optics on current scenarios
excerpt: "A microbenchmark for testing how Circe Optics works under concurrent scenarios"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

Some weeks ago I start using `Circe Optics` on a project and I was ask to answer a simple question: Does `Circe Optics` supports concurrency? Of course, by these moment I had no idea what to answer, so I decided to test [the approach that I used](http://google.com) under a concurrent scenario in order to arrive to some conclusions. 

## Project Setup

For running the code, you need the following dependencies declared in your `build.sbt`:

```scala
scalaVersion := "2.13.3"

val circeVersion = "0.13.0"

libraryDependencies ++= Seq(
  "io.circe" %% "circe-core" % circeVersion,
  "io.circe" %% "circe-generic" % circeVersion,
  "io.circe" %% "circe-parser" % circeVersion,
  "io.circe" %% "circe-optics" % circeVersion
)
```
## A sample Json

We'll be using the same JSON as in the [previous post](http://google.com):

``` json
{
  "id" : 3305,
  "active" : true,
  "name" : "Voyage Dream",
  "hotel_type" : "premium",
  "stars" : 5,
  "location" : {
    "city" : {
      "id" : 5296
    },
    "address" : "500 East A Street South",
    "zipcode" : "69153-3111",
    "latitude" : 41.11161
  }
}
```

## The transformations

## Test cases

## Conclusions