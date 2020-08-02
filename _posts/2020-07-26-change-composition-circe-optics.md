---
title: Chaining states changes on json with Circe, Optics and Pure Functions
date: 2020-07-22 19:53:52
excerpt: "Chain modifications over a json object using Circe, Optics and pure functions"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In a [previous post](https://serdeliverance.github.io/blog/blog/handling-state-with-pure-functions/), I talked about how to handle state change using pure functions. Now, we're going to see how to apply those principles to a very common use case: modifying a `JSON`. In order to do that, we'll going to use a well know `Scala` library called [Circe](https://circe.github.io/circe/) in conjuntion with [Optics](https://circe.github.io/circe/optics.html). We'll start looking at [Circe cursors](https://circe.github.io/circe/cursors.html), which is the common way to traverse and modify a `JSON` in `Circe` (it traverse the `JSON` as if it were a tree structure). Then, we'll see how to do the same using `Optics`, and finally, how to improve that a little bit with `State Actions`.

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

Let's imagine that we are working in a API for a travel agency and that in some point, we are required to store, retrieve and apply modifications over a `JSON` that represents a hotel entity.

``` json
{
  "id" : 3305,
  "active" : true,
  "name" : "Voyage Dream",
  "hotel_type" : "premium",
  "stars" : 5,
  "location" : {
    "address" : "500 East A Street South",
    "zipcode" : "69153-3111",
  }
}
```

## Using cursors

## Using Optics

## Redefining our API with State Actions

## Compose changes

## A more functional approach using Fold

## Conclusions