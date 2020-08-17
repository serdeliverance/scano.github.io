---
title: Chaining states changes on json with Circe, Optics and Pure Functions
date: 2020-08-16 21:00:00
excerpt: "Thoughts for working with json using Circe Optics and Pure Functions"
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
  "id": 3305,
  "active": true,
  "name": "Voyage Dream",
  "hotel_type": "premium",
  "stars": 5,
  "location": {
    "address": "500 East A Street South",
    "zipcode": "69153-3111"
  }
}
```

## Using Cursors

The traditional way of traversing and modifying `JSON` in `Circe` is using `Cursors`. Here is an example:

``` scala
val json = parse(jsonStr).getOrElse(Json.Null)

// creating the cursor
val cursor = json.hcursor

val result = cursor
  .downField("name").withFocus(_.mapString(_ => "The Lost Wonder")).top
  .fold("parser error") (_.toString())
```

I think it is acceptable for modifying just one field. Of course, you can see that its some boilerplate, but for a simple modification theres no problem. Now, what if we want to perform multiple modifications?

Lets see a possible approach:

``` scala
val result = cursor
  .downField("name").withFocus(_.mapString(_ => "The Lost Wonder")).top
  .flatMap(json => json.hcursor.downField("hotel_type").withFocus(_.mapString(_ => "premium")).top)
  .flatMap(json => json.hcursor.downField("stars").withFocus(_ => Json.fromInt(5)).top)
  .fold("parser error")(_.toString())
```

There is a hug amount of code. We start thinking if we can do it better. Apart from that, we realized that using that solution we can not reuse singular transformations. For example: what if I want to have name modification in another transformation or pipeline? We have to repeat code and It is not what we want.

We can separate our transformations in independent functions:

``` scala
def updateName(name: String): Json => Option[Json] =
  json => json.hcursor.downField("name")
            .withFocus(_.mapString(_ => name)).top
def updateHotelType(hotelType: String): Json => Option[Json] =
  json => json.hcursor.downField("hotel_type")
            .withFocus(_.mapString(_ => hotelType)).top

def updateStars(stars: Int): Json => Option[Json] =
  json => json.hcursor.downField("stars")
    .withFocus(_ => Json.fromInt(stars)).top
``` 

And use them with `for-comprehensions`:

``` scala
// modifying multiple fields using cursors and for comprehension

val result = for {
  step1 <- updateName("The Lost Wonder")(doc)
  step2 <- updateHotelType("premium") (step1)
  jsonResult <- updateStars(5) (step2)
} yield jsonResult
```

It is great. Now we can reuse specific transformations, but dealing with multiple `Option` in a `for comprehension` sounds like an overkill. There is some boilerplate yet, but it is more elegant too. However, we know we can do it even better. A part from that, having the intermediate n steps inside the comprehension (`step1`, `step2`... `stepN`) is not expresive enough.

Fortunately, we have `Circe Optics` to help us reducing these boilerplate.

## Using Optics

Let's define our transformations using `Optics`. For example, the following are the transformation for modifying the name and hotel_type attributes:

``` scala
def name(name: String): Json => Json = root.name.string.set(name)

def hotelType(hotelType: String): JsonAction = root.hotel_type.string.set(hotelType)

// the rest of the transformations follow the same pattern
```

`root` is a `JsonPath` which brings us methods for traversing the `JSON` structure up to the element that we are interested in. For doing that, it uses a `Scala` feature called `Dynamic` which allows it to call methods that actually don't exists (for example: `root.hotel_type` or `root.name`). As we can see, this is not type safe.

It is interesting to see that all our transformations are actually, functions from `Json => Json`. In fact, they are `state actions`.

We'll define a type alias for reducing repeated code. Lets call it `JsonAction`:

``` scala
type JsonAction = Json => Json
```

Finally, these are our transformations:

``` scala
  // transformations
  def setName(name: String): JsonAction = root.name.string.set(name)

  def setHotelType(hotelType: String): JsonAction = root.hotel_type.string.set(hotelType)

  def setStars(stars: Int): JsonAction = root.stars.int.set(stars)

  def isEnabled(active: Boolean): JsonAction = root.active.boolean.set(active)

  def setLocationAddress(address: String): JsonAction = root.location.address.string.set(address)

  def setLocationZipCode(zipCode: String): JsonAction = root.location.zipcode.string.set(zipCode)
```

## Redefining our API with composable State Actions

Having defined our transformations that way, we can define transformation pipelines by chaining functions that we need. For example:

``` scala
val result: String = json.map(
  setName("The Lost Wonder")
    .andThen(setHotelType("Premium"))
    .andThen(setStars(5))
    .andThen(setLocationAddress("Fake Street 123"))
    .andThen(setLocationZipCode("1212-003"))
    .andThen(_.toString())
).fold(_ => "invalid json", r => r)
```

We can extract this pipeline in its own method:

``` scala
// transformation pipeline
def update(name: String,
            hotelType: String,
            stars: Int,
            address: String,
            zipCode: String): JsonAction =
  setName(name)
    .andThen(setHotelType(hotelType))
    .andThen(setStars(stars))
    .andThen(setLocationAddress(address))
    .andThen(setLocationZipCode(zipCode))
```

Lets use our new update method:

``` scala
val result2: String = json.map(update(
  name = "The Lost Wonder",
  hotelType = "Premium",
  stars = 5,
  address = "Fake Street 123",
  zipCode = "1212-003"
).andThen(_.toString()))
  .fold(_ => "invalid json", r => r)
```

This solution allow us to define different pipelines by reusing those transformations according to our needs.

## Refactor our API using Fold

Chaining functions as we did is a great improvement, but at first glance we realized that the code has some boilerplates and we start thinking if we can refactor it to be more functional and clean.

What have we done so far? We have been defining transformations as functions that be reused and combined together. In the FP world there is a well known function for cases like that, when we want to apply different operations starting from an initial value. This function is called `fold` and we can implement it for our use case:

``` scala
private def fold(initial: Json)(transformations: List[JsonAction]): Json = {

  @tailrec
  def go(json: Json, transformations: List[JsonAction]): Json = {
    if (transformations.isEmpty) json
    else go(transformations.head(json), transformations.tail)
  }

  go(initial, transformations)
}
```

And our update method (just named `updateF` to differentiating it from the previous one) would look like this:

``` scala
def updateF(name: String,
            hotelType: String,
            stars: Int,
            address: String,
            zipCode: String): JsonAction =
  fold(_)(List(
    setName(name),
    setHotelType(hotelType),
    setStars(stars),
    setLocationAddress(address),
    setLocationZipCode(zipCode)
  ))
```

We can see that it is more easy to read and that has less boilerplate code.

## Conclusions

In this post we have seen how to use `Circe Optics` in conjuntion with some FP ideas that can make our code more modular, clean, mantainable, easy to understand and fun when dealing `JSON` modifications.

The code is available on [GitHub](https://github.com/serdeliverance/sc-blog-code/tree/master/circe-optic-compose-demo)