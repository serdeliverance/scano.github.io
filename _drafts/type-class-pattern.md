---
title: Type Class Pattern
excerpt: "What is the Type Class Pattern and Why it is useful"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

`TODO`

## Type Class

Type class is a pattern originated in Haskell to enable ad-hoc polymorphism (aka overloading in OOP). In OOP languages we used to have polimorphysm by subtyping.

A type class is a trait that takes a type and describes the operations that can be applied to that type.

Why can this be useful for us? Because it allow us to extend existing libraries with new functionality, without using inheritance and without altering the original source code.

The type class pattern has three components:

* the type class
* instances for particular types
* interface methods that we expose to users.

### The type class

The type class is a interface or API that represents the functionallity we want to implement. In Scala, It use to be a trait with a type parameter. Example:

`TODO think an example`

``` scala
// our type class, our API
trait JsonWriter[A] {
  def write(value: A): Json
}
```

### Instances

We need `instances` of our `type class` to provide implementations of this type class for the types that we are interesting in. We can define implementations for every type we need, either types from the Scala standard library or types from our domain model.

``` scala
// packaging them in object is a good practice
object JsonWriterInstances {
  // the instances
  implicit val stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json = JsString(value)
    }

  implicit val personWriter: JsonWriter[Person] =
    new JsonWriter[Person] {
      def write(value: Person): Json =
        JsObject(Map(
          "name" -> JsString(value.name),
          "email" -> JsString(value.email)
        ))
    }
}
```

### Interfaces

A type class `interface` is the functionallity that we expose to users. It is a generic method that accepts `instances` of the type class as implicit parameters.

Packaging it in a singleton is a good practice.

``` scala
object Json {
  def toJson[A](value: A)(implicit w: JsonWriter[A]): Json = w.write(value)
}
```

Now, we need to bring into scope the type class instances that we need by importing it.

``` scala
import JsonWriterInstances.{personWriter}

Json.toJson(Person("John Doe", "jdoe@gmail.com"))
```

The compiler looks for an implicit value of type `JsonWriter` of the required type (in these case: `Person`) to inject to our `toJson` method and perform the call.

``` scala
// what the compiler does
Json.toJson(Person("John Doe", "jdoe@gmail.com"))(personWriter)   // gets personWriter from the scope and add it to the method invocation, otherwise, compilation fails.
```

These way of specifying the interface is called `Interface Object`

#### Interface Syntax or Syntax or specifying interfaces as extension method

There is another way of specifying interfaces and its called `Interface Syntax` or `Syntax`. In this approach, we can define our interface as an `extension method` to extend the functionallity of an existing type

``` scala
object JsonSyntax {
  implicit class JsonWriterOps[A](value: A) {
    def toJson(implicit w: JsonWriter[A]): Json =
      w.write(value)
  }
}
```

And we used it as follows:

``` scala
import JsonWriterInstances._
import JsonSyntax._

Person("Eddie Torres Jr", "etorresjr@gmail.com").toJson
```

This is what the compiler does:

``` scala
Person("Eddie Torres Jr", "etorresjr@gmail.com").toJson(personWriter)       // again, the compiler looks for a JsonWriter of the required type in the scope
```

In summary, the `syntax` way of specifying a type class `interface` is just a way of exposing the interface to the user. Instead for the interface as a regular method inside a singleton, we provided the interface as an implicit class with an attribue (`A` in this case). The difference is this way we can extend the functionallity of `A` via `extension methods` (a Scala feature), because of that we are able to call the method `toJson` over an instance of `Person`

``` scala
Person("Eddie Torres Jr", "etorresjr@gmail.com").toJson
```

## Best practices

`TODO working with implicits`