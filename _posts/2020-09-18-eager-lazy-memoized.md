---
title: Eager, Lazy and Memoized
date: 2020-09-18 20:30:00
excerpt: 'Different ways of computating values in Scala'
categories:
  - blog
tags:
  - scala
header:
  image: '/images/header.jpg'
---

## Intro

In this little post we are going to see some notes about `eager`, `lazy` and `memoized` computations in `Scala`.

## Eager, Lazy and Memoized

`Eager` computations happen immediately whereas `lazy` computations happen on access. `Memoized` computations are run once on first access, and then the result is cached.

## val, def and lazy val

Scala `val`s are `eager` and `memoized`. Example:

```scala
val x = {
    println("computing x...")
    math.random
}

// computing x...
// x: Double = 0.654225390459179

x 		// first access
// res0: Double = 0.654225390459179

x // second access
// res1: Double = 0.654225390459179
```

We can see that the value is computed inmediately and then it is cached for future access.

`defs` are `lazy` and `not memoized`. So, the code is computed when accessing it and it is evaluated every time we access it:

```scala

def y = {
    println("computing y...")
    math.random
}

y 		// first access
// computing y...
// res2: Double = 0.8762878254751856

y 		// second access
// computing y...
// res3: Double = 0.7216435660069331
```

`lazy vals` are `lazy` and `memoized`:

```scala
lazy val z = {
    println("Computing Z")
    math.random
}

z               // evaluated when first access is required
// Computing Z
// res4: Double = 0.24849492235046577
z // second access
// res5: Double = 0.24849492235046577

```

## When to use each one?

There is no much mistery about that. `val` and `def` have very known use cases, but `lazy val` is a more special one. `lazy val` is using when a value is expensive to compute and that computation is not needed at first so it makes no sense to compute it eagerly. Because of that, you delay this computation until you really needed it, after that this value is cached for future uses.
