---
title: Futures
excerpt: 'How to handle not blocking computations and combining them using Futures'
categories:
  - blog
tags:
  - scala
header:
  image: '/images/header.jpg'
---

## Intro

In this post we are going to see what `Futures` are, where they come from, and what useful methods them provide for working with asynchronous computations.

## Previous

In a [previous post](http://google.com) about `CPS` we haven seen that a `synchronous computation` can be turned into an asynchronous one by passing as a parameter a function called `callback` which be executed after the program finishes:

`TODO change sample`

```scala

def program(a: A): B        // synchronous

def program(a: A, k: B => Unit): Unit   // asynchronous

```

The `callback` is a function that takes the result of the program (in this case: `B`) and perform some action.

This approach has some disadvantages:

- the method signature is more complex and harder to understand

- the return of the method is `Unit` and it does not express our domain semantics faily (in this case, a computation that computes `A` and returns `B`)

- because of returning `Unit`, composing computations following this model would not be as straigforward as using its sycnrhonous imperative counterpart.

We can improve that a litle bit if we think a new way to express our computation signature. So, Why not to use something more close to the synchronous style signature? Why not to wrap the result type with another one which indicates that it is an asynchronous computation? Thats why `Futures` came into action:

```scala
def program(a: A): Future[B]
```

Now it easier to see that `B` is the result of an asynchronous computation.

## Future

`TODO`

- Futures are the functional way of composing not blocking computations that returns at some point of times

## From CPS to Future

Before getting deeper into `Future`, lets review the reasoning behind it. Lets review how, using functional programming semantics and types, we can refactor our original `CPS` into a `Future`.

```scala
// our CPS computation
def program(a: A, k: B => Unit): Unit

// first, curryin the continuation parameter
def program(a: A): (B => Unit) => Unit

// introducing a type alias
type Future[+T] = (T => Unit) => Unit
def program(a: A): Future[B]

// bonus: improvement by adding failure handling with Try
type Future[+T] = (Try[T] => Unit) => Unit
```

This it the first approach. If we continuing refactoring it:

```scala
// extracting the future to a trait with a proper apply method
trait Future[+T] extends ((Try[T] => Unit) => Unit) {
  def apply(k: Try[T] => Unit): Unit
}

// renaming apply to onComplete
trait Future[+T] {
  def onComplete(k: Try[T] => Unit): Unit
}
```

Using our new `Future` API:

```scala
def makeCoffee(): Future[Coffee] = ???

def coffeeBreak(): Unit = {
  val eventuallyCoffee = makeCoffee()
  eventuallyCoffee.onComplete { tryCoffee =>
    // do something
  }
  chatWithColleagues()
}
```

We can handle errors too:

```scala
def makeCoffee(): Future[Coffee] = ???

def coffeeBreak(): Unit = {
  val eventuallyCoffee = makeCoffee()
  eventuallyCoffee.onComplete { tryCoffee =>
    case Success(coffee) => drink(coffee)
    case Failure(reason) => ...
  }
  chatWithColleagues()
}
```

`Future` in the standard library is more complex than that, but that is the main idea behind it. So, we realized that `Future` follows the `CPS` essence but it doesn't stop there. The power of `Futures` lays on high-level transformation operations that it provides, which allow us to chain asynchronous computations easyly.

## Transformation operations

Operations for transforming successful results:

```scala
def map[B](f: A => B): Future[B]
def flatMap[B](f: A => Future[B]): Future[B]
def zip[B](fb: Future[B]): Future[(A, B)]
```

Operations for transforming failures

```scala
def recover(f: Exception => A): Future[A]
def recoverWith(f: Exception => Future[A]): Future[A]
```

### Trasforming successful results

In this section we will see the most used future operations:

#### map

```scala
trait Future[+A] {
  def map[B](f: A => B): Future[B]
}
```

Transforms a successful `Future[A]` into a `Future[B]` by applying a function `f: A => B` after `Future[A]` has completed.

Progragates the failure in case of `Future[A]` (or another former future operation, if we are chaining) to the resulting `Future[B]`.

```scala
def grindBeans(): Future[GroundCoffee]
def brew(groundCoffee: GroundCoffee): Coffee

def makeCoffee(): Future[Coffee] =
  grindBeans().map(groundCoffee => brew(groundCoffee))

```

We can think in a function `A => B` as a synchronous function which is called when future completes successully. What if instead of passing a function `f: A => B`, we want to pass a function: `f: A => Future[B]`? In other words, we want to chain an asynchronous computation. In this case, using map we will get a `Future[Future[B]]` as a result, which not seems to be too easy to working with (in fact, it is a limitation inherited to functor operations like [map](www.linktofunctorpost.com)).

So, for chaining this king of computations we need something more powerfull. We need a `flatMap`.

#### flatMap

```scala
trait Future[+A] {
  def flatMap[B](f: A => Future[B]): Future[B]
}
```

- Transforms a successful `Future[A]` into a `Future[B]` by applying a function `f: A => Future[B]` after the `Future[A]` has completed.

- it propagates failure results the same way as `map`

```scala
def grindBeans(): Future[GroundCoffee]
def brew(groundCoffee: GroundCoffee): Future[Coffee]

def makeCoffee(): Future[Coffee] =
  grindBeans().flatMap(groundCoffee => brew(groundCoffee))
```

`flatMap` is a monadic operation. If you want to learn more about `Monads`, in a previous blog I covered this topic.

`Important`: the purpose of `flatMap` is introducing sequencially.

If we chain multiple `flatMap` operations we will get code similar to working with `CPS`:

```scala
def work(): Future[Work] = ???
def coffeeBreak(): Future[Unit] = ...

def workRoutine(): Future[Work] = {
  work().flatMap { work1 =>
    coffeeBreak().flatMap { _ =>
      work().map { work2 =>
        work1 + work2
      }
    }
  }
}
```

In these case, we got sequenciallity but we are still losing in readibility. Luckyly, in `Scala` we have `for-comprehensions`:

```scala
def workRoutine(): Future[Work] = {
  for {
    work1 <- work()
    _     <- coffeeBreak()
    work2 <- work()
  } yield work1 + work2
}
```

It is an widely used way of expressing sequencially operations on monadic types like `Future` (types which have `map` and `flatMap` operations) in an imperative way (remember: imperative is not evil, it just a way to describe operations in this case).

#### zip

```scala
trait Future[+A] {
  def zip[B](other: Future[B]): Future[(A,B)]
}
```

- Joins two successful `Future[A]` and `Future[B]` values into a single successful `Future[(A, B)]` value
- returns a failure if any of the two `Future` values failed
- Does not create any dependency between the two `Future` values.

```scala
def makeTwoCoffees(): Future[(Coffee, Coffee)] =
  makeCoffee() zip makeCoffee()
```

### Transforming failure results/fallback

```scala
trait Future[+A] {
  def recover[B >: A](pf: PartialFunction[Throwable, B]): Future[B]
  def recoverWith[B >: A](pf: PartialFunction[Throwable, Future[B]]): Future[B]
}
```

#### recover

takes a partial function that converts the `Throwable` in a sucessfull result of type `B`

#### recoverWith

The same as `recover`, excepts that the partial function is asynchronous (because of `Future[B]`s)

## Execution Context

`ExecutionContext` is the place were continuation are executed. `ExecutionContext` is a fixed thread pool were continuations wew be executed and we can define it accord to our needs. It can be `single thread` also, but in that case we lost parallelism. Usually, we define `ExecutionContexts` with as many threads as the machine where they are executed (actually, that is what `scala.concurrent.ExecutionContext.Implicits.global` does).

How can be pass `ExecutionContext` to our continuated computations? Using implicit parameters:

```scala
trait Future[+A] {
  def onComplete(k: Try[A] => Unit)(implicit ec: ExecutionContext): Unit
}
```

## Ble

`TODO`

```scala
val mark = SocialNetwork.fetchProfile("fb.id.1-zuck")
mark.onComplete {
case Success(markProfile) =>
    val bill = SocialNetwork.fetchBestFriend(markProfile)
    bill.onComplete {
    case Success(billProfile) => markProfile.poke(billProfile)
    case Failure(e) => e.printStackTrace()
    }
case Failure(ex) => ex.printStackTrace()
}
```

## Conclusion

`TODO notes to organize`

- `onComplete` suffers the same composability issues as `callbacks`

- Actually, when working with futures is normal chaining operations as we asume that all of them will be success and delayin the failure handling to a later point.

Summary:

- Future is an equivalent alternative to traditional `CPS`
- offers transformation and failure recovering operations.
- introduces map and flatMap for sequencing operations
