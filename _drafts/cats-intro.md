---
title: Getting started with Cats
excerpt: "An introduction to the popular Cats library"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

`Cats` is a `Functional Programming` library for `Scala` which is based on `Cathegory Theory` and which has a great ecosystem of `modules` around it for doing almost everything. In this post we are going to see how start playing around with some `Type Classes` provided by `Cats`.

`Disclaimer:` this post is based on the great book `Scala with Cats` by Noel Welsh and Dave Gurnell and published by [Underscore](https://underscore.io). You can get the pdf version for free at [Underscore.io](https://underscore.io/books/scala-with-cats/).

## Prerequisites

* Basic Scala knowledge

* some Type Classes knowledge is needed (check this [previous post](https://serdeliverance.github.io/blog/blog/type-class-pattern/) for an intro to this topic)

## Project Setup

Create an `sbt` project with the following dependecy:

``` scala
scalaVersion := "2.13.3"

val catsVersion = "2.0.0"

libraryDependencies ++= Seq(
  "org.typelevel" %% "cats-core" % catsVersion
)
```

## Type class components

Remember, a type class is composed by these three components:

* type class: the trait that contains the functionallity we are interested in
* instances for particular types: the implementations of the type class for the types that we need
* interface methods: the glue that holds together type type class with the instance that we are interested in. It is the API exposed to users.

## First sample: Show type class

Let's start our tour into `Cats` by using the type class `Show`. This type class defines the following method:

``` scala
trait Show[T] {             // just for illustrative purposes
    def show(t: T): String
}
```

First, we need to import the type class.

``` scala
import cats.Show
```

## Second sample: Eq type class


## Conclusion
