---
title: Modify json as a combination of states changes with Circe, Optics and Pure Functions
date: 2020-07-22 19:53:52
excerpt: "Chain modifications over a json object using Circe, Optics and Pure Functions"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

## Project Setup

First of all, create an `sbt` project and add the following dependencies in your `build.sbt`:

```scala
val akkaVersion = "2.5.30"

libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % akkaVersion,
  "com.typesafe.akka" %% "akka-testkit" % akkaVersion,
  "org.scalatest" %% "scalatest" % "3.1.0"
)
```
## A sample Json

## Using cursors

## Using Optics

## Redefining our API with State Actions

## Compose changes