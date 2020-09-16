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

## Functors

A `Functor` is the `Monad` ancestor and it is an abstraction that allows apply a map function over a value inside a context:

``` scala
trait Functor[F[_]] {
    def map(f: A => B): F[B]
}
```

`Functor` is an abstraction for sequencing operations but it has a limitation: it only handles the `happy path` or simple cases. `Functors` do not take complitations into account.

For example: imagine that we are working following methods:

``` scala

def sum(x: Int, y: Int): Int = x + y

def divide(x: Int, y: Int): Int = x / y

def multiply(x: Int, y: Int): Int = x * y

```

Now, let's chain some operations using `Functor`:

``` scala

val num = 3
val functor = Functor[Int](num)

functor
    .map(x => sum(x, 4))
    .map(result1 => divide(result1, 0))         // division by zero => it crashes!
    .map(result2 => multiplyBy2(result2, 2))

```

If we want to deal with failure scenarios, we can wrap the return values into `Option`

``` scala

def sum(x: Int, y: Int): Int = x + y

def divide(x: Int, y: Int): Option[Int] =
    if(x == 0) None else Some(x / y)

```

Now, our operation pipeline would be the following:

``` scala
// TODO
```

We can see that using simply `Functors` when dealing with containarized values which have context specific problems is cumbersome. Imagine if instead of `Int` we have to work with `Future[Something]`. Because of that, `Monads` came into action.


## What is a monad?

A monad is an abstraction for composing and sequencing computations. A monad is a wrapper over a contained value, and give us operations for composing and sequencing computations over that value.

Formally, a `Monad` should define these two operations:

* pure: of type A => Monad[A]
* flatMap: of type (F[A], A => F[B]) => F[B]

`pure` abstracts over constructors and allows to create a Monad from a value.
`flatMap` can be think as a more powerful version of `map` which allows applied a monadic function (a function of type `A => F[B]`) over the value inside the monad and deal with context related problems.

## Monads in Scala

When thinking in monads, try to visualize the following shape:

```scala

trait Monad[A] {
    def pure(value: A): Monad[A]
    def flatMap[B](f: A => Monad[B]): Monad[B]
}
```
 Actually, there isn't a Monad trait in the Scala standard library, but it is helpful for reasoning about what a monad is.
 As you can see, a monad is an abstraction for composing and sequencing computations of some contained value. A monad wrap a value and allow us to operate over it by using `map` and `flatMap`.
 Let's review those methods

### the importancy of flatMap

`flatMap` is the game changer in the `Monad` type class because it allows sequencing computation steps extracting the value from the context a generating a new context in the sequence.

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

## Monads Laws

Last but not least, `Monad` as a type class, should obey the following laws that guarantees the correctness of their operations:

``` scala
// 1) Left identity
pure(a).flatMap(func) == func(a)

// 2) Right identity
m.flatMap(pure) == m

// 3) Associativity
m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))
```

## Conclusion

`TODO`

Answer:

1) What monads are?
2) motivation
3) when to use it