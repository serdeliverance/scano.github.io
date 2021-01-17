---
title: Play with IO
date: 2021-01-17 12:00:00
excerpt: 'An approach for using the IO monad from Cats Effect on Play Framework'
categories:
  - blog
tags:
  - scala
  - play
  - cats
header:
  image: '/images/header.jpg'
---

## Intro

In this post, we are going to see an approach for using `IO` monad from `Cats Effect` on a `Play` application, in order to take advantage of its benefits for handling effects.

## IO monad

`TODO`

## Dependencies

First, we assume you already have a `Play` application. In case you don't, you can create one or you can checkout the source code of this example which is available on [GitHub](https://github.com/serdeliverance/play-with-io-bc).

Add the following dependencies on your `build.sbt`:

```
libraryDependencies ++= Seq(
  // ...
  // cats
  "org.typelevel" %% "cats-effect" % "2.2.0"
  // ...
)
```

## Our project

## Start using IO

### Define a type for handling operation result

### Define ContextShift as a provider on Module

### Refactor each layer to use our operation result

###

### A unit test using IO

## Conclusion
