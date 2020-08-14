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

When we have a `Semigroup` that is not a `Monoid`, how we can perform operations like collapse? One strategy is to lift the Semigroup.

`TODO continue with the Option monoid in Monoid section of Cats doc`
## Conclusions