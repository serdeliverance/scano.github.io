---
title: Type Classes
excerpt: "Notes about Type Classes"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

`TODO`

## Type Classes

Type classes are a tool in functional programming to enable ad-hoc polymorphism (aka overloading in OOP). In OOP languages we used to have polimorphysm by subtyping.

A type class is a trait that takes a type and describes the operations that can be applied to that type.

# Monoids intro

# Type classes vs Subtyping

`TODO ver cats sample`

# implicit Derivation

Using type class, for example, with Monoids, allows the compiler to do type derivation automatically via implicits.

The following example (taken from `Cats` doc) shows how we can derive Monoid[A] and Monoid[B] into a Monoid[Pair[A, B]]

``` scala
object Demo { // needed for tut, irrelevant to demonstration
  final case class Pair[A, B](first: A, second: B)

  object Pair {
    implicit def tuple2Instance[A, B](implicit A: Monoid[A], B: Monoid[B]): Monoid[Pair[A, B]] =
      new Monoid[Pair[A, B]] {
        def empty: Pair[A, B] = Pair(A.empty, B.empty)

        def combine(x: Pair[A, B], y: Pair[A, B]): Pair[A, B] =
          Pair(A.combine(x.first, y.first), B.combine(x.second, y.second))
      }
  }
}
```

`TODO more investigation`
`TODO fully understand of the cat sample`

# Laws

Type classes come with laws which are constraints implementation for a given type.

# Semigroup

A `Semigroup` of type `A` is a typeclass which supports the operation `combine`. This method allows us to combine two elements of the same type and its result is an element which also of the same type.

``` scala
trait Semigroup[A] {
  def combine(x: A, y: A): A
}
```

We can see that this is a asociative operation.

``` python
  combine(x combine(y, z)) == combine(combine(x, y), z)
```

A common example of associative operations is sum `+` on `Int`. Because of that, we can say that the type `Int` together with the operation `+` conforms a `Semigroup`.

We can define a `Semigroup` the following way:

``` scala
import cats.Semigroup

implicit val intAdditionSemigroup: Semigroup[Int] = new Semigroup[Int] {
  def combine(x: Int, y: Int): Int = x + y
}

val x = 1
val y = 2
val z = 3

Semigroup[Int].combine(x, y)

Semigroup[Int].combine(x, Semigroup[Int].combine(y, z))

Semigroup[Int].combine(Semigroup[Int].combine(x, y), z)
```

For types that have a `Semigroup` instance we can use the following operator provided by cats:

``` scala
import cats.implicits._

1 |+| 2
```

I think that is useful to associate the `Semigroups` concept with `asociativity`.

This associativity is very important because it allow us to split a collection of elements for parallel computation, for example, and joining the results at the end and we can be confident that the final result won't be diferent.

However, when we want to generalize some operation as `fold`, the `Semigroup` typeclass is not enough. So we need to incorporate a new operation. 

Now its time talk about `Monoids`

# Monoid

`Monoid` is a type class which extends `Semigroup` and provides the `empty` method.

``` scala
trait Semigroup[A] {
  def combine(x: A, y: A): A
}

trait Monoid[A] extends Semigroup[A] {
  def empty: A
}
```

This `empty` plays the role of identity for the combine operation. It means that for every x of type A, the following is true:

``` python
combine(x, empty) == combine(empty, x) == x
```

Examples of `Monoids`:

* `Int` type with the sum `+` and `0` as empty
* `String` type with the sum `+` and `""` as empty

``` scala
implicit cats.Monoid

implicit val intAdditionMonoid: Monoid[Int] = new Monoid[Int] {
  def empty: Int = 0
  def combine(x: Int, y: Int): Int = x + y
}

val x = 1
Monoid[Int].combine(x, Monoid[Int].empty) // 1

Monoid[Int].combine(x, Monoid[Int].empty) // 1
```

## Example

With `Monoids` we can define a function `combineAll`:

``` scala
def combineAll[A: Monoid](as: List[A]): A =
  as.foldLeft(Monoid[A].empty)(Monoid[A].combine)
```

Importing `cats.implicits._` we can use this function for any type that has a `Monoid` instance. For example:

``` scala
import cats.implicits._

combineAll(List(1, 2, 3))   // 6

combineAll("hello", " ", "cats") // hello cats
```

The function is provided by `Cats` in `Monoid.combineAll`.

# The option Monoid

When we have a `Semigroup` that is not a `Monoid`, how we can perform operations like collapse? One strategy is to lift the Semigroup to an Option, and have a Monoid[Option] that can perform the combine operation.

`TODO continue with the Option monoid in Monoid section of Cats doc`

Lifting and combining `Semigroups` into `Option` is provided by `Cats` with `Semigroup.combineAllOption`

# Applicative and Traversable Functors

See the following powerful function providing by future:

``` scala
import scala.concurrent.{ExecutionContext, Future}

def traverseFuture[A, B](as: List[A])(f: A => Future[B])(implicit ec: ExecutionContext): Future[List[B]] =
  Future.traverse(as)(f)
```

This functions applies de `f` function to any element of the list, and collects its results as it goes.

This idea can be abstracted to be applied to any type and to be used in different contexts (ex: validation, Option, Either, or handling State)

## Functor

`Functor` is a type class that abstract over type the `map` operation. In other words, it has the form of a trait that has the `map` method:

``` scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

Examples of functors are `List`, `Option`, `Future` and wherever type that has a `map` operation.

`TODO think another example... because the following was taken from Cats`
``` scala
// Example implementation for Option
implicit val functorForOption: Functor[Option] = new Functor[Option] {
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa match {
    case None    => None
    case Some(a) => Some(f(a))
  }
}
```

`Functor` must obey the following laws:

* Composition: mapping with `f` and then with `g` is the same as mapping once with the composition of `f` and `g`

``` scala
fa.map(f).map(g) == fa.map(f.andThen(g))
```

* Identity: mapping a value with the identity function, returns the same value. 

``` scala
fa.map(x => x) == x
```

Functors allows lifting pure functions `A => B` into an effectful function.

Example: given the `Functor[F]` and a function `f: A => B` it can convert (lift) that into `F[A] => F[B]`

``` scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]

  def lift[A, B](f: A => B): F[A] => F[B] =
    fa => map(fa)(f)
}
```

### Effect management

The `F` in Functor is referred as `effect` or `computational context`. So, a `Functor` allows to apply a pure function (some `f: A => B`) to a value which is inside a computational context and returning a result preserving its original context (`F`)

## Applicative

`Applicative` is a type class that extends `Functor` and adds two methods: `ap` and `pure`:

``` scala
trait Applicative[F[_]] extends Functor[F] {
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]

  def pure[A](a :A): F[A]

  def map[A, B](fa: F[A])(f: A => B): F[B] = ap(pure(f)) (fa)
}
```

`pure` transforms the value into a the type constructor.

`TODO laws of Applicatives`

`Applicative` type class must obey the following rules:

* Associativity

* Left identity

* Right identity

`TODO Pros and Cons`

While `Functor` can deal with one single effect, `Applicative` allows working with multiple independent effects and then compose them. We can work with N number of independent effects using `Applicatives`.

## Conclusions