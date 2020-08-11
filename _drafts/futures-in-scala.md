---
title: Futures
excerpt: "How to handle not blocking computations and combining them using Futures"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

`TODO`

## Future

`TODO`

* Futures are the functional way of composing not blocking computations that returns at some point of times

## Chaining futures

`TODO`

``` scala
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

## Functional Compositions

## Fallbacks

## Conclusion