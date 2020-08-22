---
title: Variance
excerpt: "An introduction to Variance"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post...

It is a very important topic when you deal when generic collections or even when you start working with type classes, either implementing them manually or using a functional programming library like `Cats`.

## Variance

In a few words: `Variance` is the ability to substitute one value for another.

``` scala
// Example: given the following trait

sealed trait Animal
case class Dog(name: String) extends Animal

// we can to something like this
val dogs = List(Dog("firulais"))
val animals: List[Animal] = dogs
```

`TODO variance types`

`TODO variance positions of parameters`

* variance of the type class

* variance annotations affect the compiler's ability to select instances during implicit resolutione

### Covariance

Co and contravariance annotations appears when working with type constructors:

``` scala
trait F[+A]     // the "+" means covariant
```

Covariance means that `F[B]` is a subtype of `F[A]` if `B` is a subtype of `A`.

Another common exaple of this is the trait `List[+T]` from the standard library. It is covariant and, because of that: `List[Nothing]` is a subtype of `List[Int]` for example (remember: `Nothing` is subtype of all the types in the scala `Type System`)

### Contravariance

``` scala
trait F[-A]     // the symnol "-" states that is contravariant
```

Contravariance means that the type `F[B]` is a subtype of `F[A]` if `A` is a subtype of `B`

## Conclusion
