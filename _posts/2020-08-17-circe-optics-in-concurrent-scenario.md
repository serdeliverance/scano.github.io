---
title: Circe Optics in concurrent scenarios
date: 2020-08-17 12:00:00
excerpt: "Just a little test to see how Circe Optics works under concurrent scenarios"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

Some weeks ago I start using `Circe Optics` on a project and I was if `Circe Optics` supports concurrency. Of course, by these moment I had no idea what to answer, so I decided to test [the approach that I used](https://serdeliverance.github.io/blog/blog/circe-optics-with-pf/) under a concurrent scenario in order to get some conclusions. 

## Project Setup

For running the code, you need the following dependencies declared in your `build.sbt`:

```scala
scalaVersion := "2.13.3"

val circeVersion = "0.13.0"

libraryDependencies ++= Seq(
  "io.circe" %% "circe-core" % circeVersion,
  "io.circe" %% "circe-generic" % circeVersion,
  "io.circe" %% "circe-parser" % circeVersion,
  "io.circe" %% "circe-optics" % circeVersion,
  "org.scalatest" %% "scalatest" % "3.2.0" % "test"
)
```
## A sample Json

We'll be using the same JSON as in the [previous post](https://serdeliverance.github.io/blog/blog/circe-optics-with-pf/) but with a new extra field (the `likes` field):

``` json
{
  "id": 3305,
  "active": true,
  "name": "Voyage Dream",
  "hotel_type": "premium",
  "stars": 5,
  "likes": 0,
  "location": {
    "address": "500 East A Street South",
    "zipcode": "69153-3111"
  }
}
```

## The transformations

The transformations are the same that in the previous but we just going add one more (the `like` one) for the purpose of this test. Also, we are going to define a method `updateNameStarsAndLikes` for grouping this three modifications and performing as a pipeline.

Having said that, here is our Json Manipulation API:

``` scala
object JsonManipulationAPI {

  // a type alias used for the transformations
  type JsonAction = Json => Json
  
  def updateNameStarsAndLikes(name: String, stars: Int, likes: Int): JsonAction =
    fold(_)(
      List(
        setName(name),
        setStars(stars),
        setLikes(likes)
      )
    )

  // the transformations we are going to use
  def setName(name: String): JsonAction = root.name.string.set(name)

  def setStars(stars: Int): JsonAction = root.stars.int.set(stars)

  def setLikes(likes: Int): JsonAction = root.likes.int.set(likes)

  // auxiliar method for collapsing a list of transformations
  private def fold(initial: Json)(transformations: List[JsonAction]): Json = {
    @tailrec
    def go(json: Json, transformations: List[JsonAction]): Json = {
      if (transformations.isEmpty) json
      else go(transformations.head(json), transformations.tail)
    }
    go(initial, transformations)
  }
```

## Test case

The test is very simple: we'll a set of attributes concurrently. Those attributes are `name`, `stars` and `likes`. For doing that, we are going to run 2000 threads that modify those attributes using our convenient method `updateNameStarsAndLikes`. These threads are separated in two groups that will perform different modification of those attributes. For example, the first group will modify the `like` field starting from value 1, and the other one that starts from 1001. `modifiedJson1` and `modifiedJson2` will hold the parsed json result of each group of threads. We expects that the final result was correct (for example, that `likes` in `modifiedJsonThread1` will be equals to `1000`, and `2000` for the other group)

Why 2000 threads? Because this amount of concurrent modifications is suitable for my use case.

Here is the test:

``` scala
"multiples transformations run concurrently" should {
  "perform in a thread-safe manner" in {

    var modifiedJson1 = Json.Null
    var modifiedJson2 = Json.Null

    // the original json that will be our starting point for apply modifications
    val json = parse(jsonStr).getOrElse(Json.Null)

    for (i <- 1 to 1000) {

      val thread1 = new Thread(
        () => modifiedJson1 = updateNameStarsAndLikes(s"The Lost Wonder ${i}", 3, i)(json))
      val thread2 =
        new Thread(
          () => modifiedJson2 = updateNameStarsAndLikes(s"The Lost Wonder ${i + 1000}", 5, i + 1000)(json))

      thread1.start()
      thread2.start()
    }

    // just to give threads time enough to finish
    Thread.sleep(3000)

    // extracting results
    val name1 = extractName(modifiedJson1)
    val stars1 = extractStars(modifiedJson1)
    val likes1 = extractLikes(modifiedJson1)

    val name2 = extractName(modifiedJson2)
    val stars2 = extractStars(modifiedJson2)
    val likes2 = extractLikes(modifiedJson2)

    // assertions
    name1 mustBe "The Lost Wonder 1000"
    stars1 mustBe 3
    likes1 mustBe 1000

    name2 mustBe "The Lost Wonder 2000"
    stars2 mustBe 5
    likes2 mustBe 2000
  }
}
```

The `json` used and the extract auxiliar methods are defined in the [companion object](https://github.com/serdeliverance/sc-blog-code/blob/master/circe-optic-compose-demo/src/test/scala/CirceOpticsDemoSpec.scala) of the spec.

If you run it, you can see that the conditions are met and the test passed.

## Conclusions

We have tested `Circe Optics` in a concurrent scenario where an acceptable amount of threads have modified multiples fields of a json object with great results. 
