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

Let's define our transformations using `Optics`. For example, the following are the transformation for modifying the name and hotel_type attributes:

``` scala
def name(name: String): Json => Json = root.name.string.set(name)

def hotelType(hotelType: String): JsonAction = root.hotel_type.string.set(hotelType)

// the rest of the transformations follows the same pattern
```

`root` is a `JsonPath` which brings us methods for traversing the `JSON` structure up to the element that we are interested in. For doing that, it uses a `Scala` feature called `Dynamic` which allows it to call methods that actually don't exists (for example: `root.hotel_type` or `root.name`). As we can see, this is not type safe.

It is interesting to see that all our transformations are actually, functions from `Json => Json`. In fact, they are `state actions`.

We'll define a type alias for reducing repeated code. We'll call it `JsonAction`:

``` scala
type JsonAction = Json => Json
```

Finally, these are all our transformations:

``` scala
def name(name: String): JsonAction = root.name.string.set(name)

def hotelType(hotelType: String): JsonAction = root.hotel_type.string.set(hotelType)

def stars(stars: Int): JsonAction = root.stars.int.set(stars)

def enabled(active: Boolean): JsonAction = root.active.boolean.set(active)

def locationAddress(address: String): JsonAction = root.location.address.string.set(address)

def locationZipCode(zipCode: String): JsonAction = root.location.zipcode.string.set(zipCode)
```

## Redefining our API with composable State Actions

Having defined our transformations that way, we can define transformation pipelines by chaining functions that we need. For example:

``` scala
// TODO
```

## A more functional approach using Fold

## Conclusions