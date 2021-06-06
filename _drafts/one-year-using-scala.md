---
title: One year using Scala
date: 2021-03-29 23:00:00
excerpt: 'Conclusions and thoughts'
categories:
  - blog
tags:
  - scala
header:
  image: '/images/header.jpg'
---

## Intro

It's been more than a year since I started using Scala. So, I think it's a good time to review my journey and share my thoughts about this awesome language and its ecosystem.

## My journey

My first contact with Scala was through the book `Functional Programming in Scala` (aka `The Red Book`). With that book I learned some functional programming ideas and basic Scala syntax. I must confess that reading it was very challenging for me. The exercises were difficult and makes you think over them for a while until you came up with the solution (If you could).

However, at some point of the reading I thought that It was over complicated and I started considering learning from another resource. So, I came out with some Akka related Udemy courses. I had a great experience learning Akka basics, some Akka Persistence stuff, but learning Akka Streams simply blowed my mind. Writing async pipelines were awesome and I felt in love with Stream processing, in general, and Akka Streams and Alpakka, in particular.

After that, I got my first job as a Scala developer, and I started learning more about `Play Framework` and I had my first contact with `Typelevel libraries`, such as `Cats` and `Circe`.

In my free time I continued reading some books. I read `Scala Essentials`, `Scala with Cats` and I've tried to develop some applications using just the `Typelevel stack` (`Cats`, `Http4s`, `Doobie`, etc) but I've never succeeded. Until now I still think that `Typelevel` ecosystem is very challenging and over complicated. I will not deny that the way of reasoning about effects and developing an app just with pure functions is very interesting and that has its benefits (for example, in testing), but I think the learning curve is very sharp and I don't know if it's really worth it. However, something is true: I'll continue playing around with it. I know that at some point it will make more sense to me.

Enough talking about myself, let's review some points about the Scala language and it's ecosystem.

## Language

`TODO immutability`

`TODO lazy vals`

`TODO case classes`

`TODO pattern matching`

`TODO functions as values`

`TODO type alias`

`TODO companion objects`

`TODO traits`

`TODO implicits`

`TODO the collections API`

`TODO monads`

`TODO for comprehensions`

`TODO async with Futures`

The idea when creating `Scala` was to be a mix between `OOP` and `functional programming`, taking the best of both worlds and taking advantage of the rich set of libraries and frameworks of the `JVM` ecosystem. I think this goal was achieved. The language is very flexible and elegant.

## Community

Having a language that is so flexible comes with different approaches to solve the same problem. Because of that, different `schools` have been emerged in the `Scala` ecosystem.

At a first sight I could identify four `Scala` schools: the Odersky's one, Typelevel's one, ZIO's one and Lil Haoyi's one. Let's review each of them.

### The Odersky's school

Martin Odersky is the creator of the `Scala` language, and this school follows his idea: having a mix between `OOP` and `Functional Programming`. I think his vision was achieved and the language itself is awesome. Things like inmutability, lazy evaluation, case classes, traits, implicits (not abusing of them), the collections API, monads and for comprehensions, and Futures are simply great and you can solve a wide number of problems in a very elegant way using them. Additionally, you have Akka and Play, which gives you more tools for implement high performance services and introduce you to different concurrency models (CPS, Actors, Streamings).

However, given the flexibility of `Scala` plus the rich and solid `JVM` ecosystem, new approaches has been emerged. Many of them are very influenced by `Haskell` and try to implement pure functional programming on the `JVM`.

### Typelevel school

[Typelevel](https://typelevel.org/) takes ideas from `Haskell` and `Type safety` but in another level and you can see that influences in its libraries (`Cats`, `Cats Effect`, `Doobie`, `Http4s`, `Circe`). Those libraries are very concerned about how to deal with `side effects` and I must confess that its approach is very interesting. However, Whenever I try to learn it I end up frustrated. For example, let's see the following code:

``` scala
def allRoles[F[_], Auth](
    pf: PartialFunction[SecuredRequest[F, User, AugmentedJWT[Auth, Long]], F[Response[F]]],
)(implicit F: MonadError[F, Throwable]): TSecAuthService[User, AugmentedJWT[Auth, Long], F] =
  TSecAuthService.withAuthorization(_allRoles[F, AugmentedJWT[Auth, Long]])(pf)
```

I think the approach of abstracting over the effect system and define interpreters is cool, but trying to be ultra type safe is not beginner friendly and engornomic.

Typelevel has a lot of resources to learn and its community is very active. Reading its docummentation is always enriching.

### ZIO

Personally, I have not experimented with [ZIO](https://zio.dev/) yet. The only thing that I know is its community is very active and that ZIO vision is to provide libraries for writting `Pure Functional` and concurrent programs using idiomatic `Scala`. I hope I will try ZIO this year.

### Li Haoyi

Another `Scala` school is the [Li Haoyi's one](https://www.lihaoyi.com/). He takes lots of ideas from `Python` and tries to translate it to the `Scala` world in order to build an [ecosystem of libraries and tools](https://github.com/com-lihaoyi) focused on usability and efficiency. Its libraries are beginner friendly and really useful. I really want to experiment with them during this years and also give [Ammonite](https://github.com/com-lihaoyi/Ammonite) and [Mill](https://github.com/com-lihaoyi/mill) a try. 

Summarizing, I think that having different ways of thinking is also a blessing because it gives you different tools and approaches for solving problems, but it is key to define some king of standarization or styling when working on a team and over big codebase. Also, I feel that pragmatism and cleanest are the two main thinks that every developer should have in mind when developing software.

## Tooling

`TODO sbt`

`TODO sbt plugins`

`TODO intellij`

## Frameworks and Libraries

## Conclusions

Nowadays, I continue learning from conferences, reading tutorials and I have some books on my list of future readings (`Practical Functional in Scala`, `FP for mortals`, `Effects in Scala` and `Hands on Scala`). Additionally, I wan't to give some libraries and tools a try, such as `Quill`, `Finagle Finch`, `Upickle`, `Greyhound`, `Ammonite`, `Request`, `FS2` and `Zio`.