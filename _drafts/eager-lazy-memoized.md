---
title: Eager, Lazy and Memoized
excerpt: "How values are computed when accessing them?"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post we are going to see notes abouts `eager`, `lazy` and `memoized` computations.

## Eager, Lazy and Memoized

`Eager` computations happen immediately whereas `lazy` computations happen on access. `Memoized` computations are run once on first access, and then the result is cached.

Scala `vals` are `eager` and `memoized`. Example:

``` scala
val x = {
    println("Computing x")
    math.random
}

// Computing X
// x: Double = 0.874225390459179

x // first access
// res0: Double = 0.874225390459179

x // second access
// res1: Double = 0.874225390459179
```

We can see that the value is computed inmediately and then it is cached for future access.

`defs` are `lazy` and `not memoized`. So, the code is computed when accessing it and it is evaluated every time we access it.

``` scala

def y = {
    println("Computing Y")
    math.random
}
// y: Double
y // first access
// Computing Y
// res2: Double = 0.8762878254751856
y // second access
// Computing Y
// res3: Double = 0.7216435660069331
```

`lazy vals` are `lazy` and `memoized`

``` scala
lazy val z = {
    println("Computing Z")
    math.random
}

// z: Double = <lazy>

z               // evaluated when first access is required
// Computing Z
// res4: Double = 0.24849492235046577
z // second access
// res5: Double = 0.24849492235046577

```

## When to use each one?

## Conclusion

`TODO`
