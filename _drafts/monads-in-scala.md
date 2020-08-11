---
title: Monads in Scala
excerpt: "What monads are and how to use them in Scala"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post we are gonna talk about Monads in Scala. We'll try to answer the following questions:

1) What are monads?

2) What is the motivation behind them?

3) When to use them?

## Motivation

`TODO`

## What is a monad?

A monad is an abstraction for composing and sequencing computations. A monad is a wrapper over a contained value, and give us operations for composing and sequencing computations over that value.

## Monads in Scala

When thinking in monads, try to visualize the following shape:

```scala

trait Monad[A] {
    def map[B](f: A => B): Monad[B]
    def flatMap[B](f: A => Monad[B]): Monad[B]
}
```
 Actually, there isn't a Monad trait in the Scala standard library, but it is helpful for reasoning about what a monad is.
 As you can see, a monad is an abstraction for composing and sequencing computations of some contained value. A monad wrap a value and allow us to operate over it by using `map` and `flatMap`.
 Let's review those methods

### map

```scala
def map[B](f: A => B): Monad[B]
```
`TODO review definition of map`
Allow us to operate over the value which is inside de monad by applying a function over it. It returns a monad with the result of that operation (since the function is from A to B, the returned value is a Monad which contains B).
Example:

`TODO example`

### flatMap

```scala
def flatMap[B](f: A => Monad[B]): Monad[B]
```
`flatMap` is similar to `map`, but it receives a monadic function (a function of some type A that returns a Monad of something) as an argument. What flatMap do is flatten the result. For example:

`TODO flatMap example`

`TODO why flatMap is more powerful than map`

## Let implement a Monad

`TODO`

Option is an abstraction that allow us to model the absence of value. If you think about it, it has the shape of a monad:

```scala
sealed trait Option[A] {
    def map[B](f: A => B): Option[B]
    def flatMap[B](f: A => Option[B]): Option[B]
}

case class Some[A](a: A) extends Option[A] {
    def map[B](f: A => B): Option[B] = Some(f(a))
    def flatMap[B](f: A => Option[B]): Option[B] = f(a)
}

case class None[A] extends Option[A] {
    def map[B](f: A => B): Option[B] = None
    def flatMap[B](f: A => Option[B]): Option[B] = None
}

```

## Other monadic types in the Scala standard library

`TODO`

- review List
- review Future

## Conclusion

`TODO`

Answer:

1) What monads are?
2) motivation
3) when to use it