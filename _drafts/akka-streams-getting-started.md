---
title: Akka Streams Getting Started
date: 2021-05-23 16:15:52
excerpt: "What is Akka Streams and how to use it"
categories:
  - blog
tags:
  - akka
  - scala
header:
  image: "/images/header.jpg"
---

# Intro

Hi, everyone. There has been a long time from my last post. In this post, I want to introduce you to the `Akka Streams` library which I considered is an awesome and powerfull tool for writing asynchronous pipelines of data transformation. In this intro, I want to show you what `Akka Stream` is, what problems does it solves, where it can be a good fit, what are its basic building blocks, an intro to Alpakka and a real world example.

Before talking about Akka Streams, it important to know what streams are.

# What are streams?

Imagine you have are working on a method that receives a list of transactions and perform some kind of calculation. If you receive 100 elements there may be no problem. But what if the app growns and you start receiving 10k, 100k or even more transactions each time. Generally, collections are data structures that needs to be fully loaded in memory before processing.

So, there are scenarios when processing the data as a whole is not an option.

Stream data processing is a way of dealing with data transformation that resembles how computers internally work (for example, low level protocols such as TCP). In a streaming pipeline, elements flow from an origin to a destination, emitted one at a time, passing through different intermediate transformations.

# Akka Streams

`Akka Streams` is a library for building asynchronous pipelines of data transformation.

It has many goals but the most important ones are the following:

* to offer an easy way of formulating stream processing pipelines that feels like working with collections.
* component reutilization (stream components can be reused in many other streams because all them are just descriptions)
* bound resource usage through back-pressure (more on this later).

# Why not just using plain Actors?

Of course, you can implement asynchronous streaming processing pipelines just using Actors. In that case, you would have to deal with the following issues:

* actor mailboxes are unbounded by default, so you have to manually control the message flow, otherwise, you can have an overflow. 
* messages can be lost and it is up to you to implement some retransmition mechanism.
* implement error handling by hand.
* lots of boilerplate because of manually implementing previous features.

# Use cases

* Batch processing
* Event Driven Architectures
* Realtime Applications (such as MMO Servers, chat servers, SSE)
* IoT

Or whenever you have to manipulate data as a pipeline or deal with potentially infinite data flows.

# Akka Streams vs Reactive Streams vs Reactive Systems

Quoting [the Reactive Streams documentation](https://www.reactive-streams.org/): `Reactive Streams is an initiative to provide a standard for asynchronous stream processing with non-blocking back pressure`.

So, `Reactive Streams` is an specification of how to deal with asynchronous boundaries without data lost or resource exhaustion.

In contrast, `Akka Streams` is an API that is compliant with that specification, but it offers its own API which is most suitable for end users. Remember, `Akka Streams` is all about building/expressing asynchronous pipelines and component reutilization.

On the other hand, `Reactive Systems` is a more broader topic. It is about building reactive system architectures as a whole, being compliant with a series of postulates that guarantees responsiveness, fault tolerant, loosed-coupled and scalable systems. In comparison, `Akka Streams` could be seen as a tool that can help to build reactive systems.

Now, we are more clear about what `Akka Stream` is, but before diving into its API, we need to talk about `Back pressure`

# Back pressure

Streams are similar to producer-consumer architectures, where elements are emitted by a producer and consumed by consumers. In those arquitectures, we can lead to the following scenarios:

* slow producer and fast consumer
* fast producer and slow consumer

The first scenario is acceptable. There's no problem with having a slow consumer. In that case, at least the consumer will be idle, but there's no buffer overflow risk.

`TODO image slow producer and fast consumer`

However, the second scenario is something we want to be cared about. If we have a fast consumer and a slow consumer, it will lead to a situation where the consumer's buffer got overflowed. `Back-pressure` solves this problem.

`TODO image fast producer and slow consumer`

`Back-pressure` is a flow-control mechanism where the consumer signals the producer with the amount of elements it is able to handle. In order words, it is a comunication from consumer to producer, where the first one signals the demand it can process. That way, producers can adjust their rate of emitting elements.

`TODO image backpressure`

It's important to mention that all this comunication between producers and consumers is asynchronous.

Now, we are ready to write our first graph (remember this term).

# Dependencies and getting started

First of all, add the following dependency to your `build.sbt`

``` scala
val AkkaVersion = "2.6.14"
libraryDependencies += "com.typesafe.akka" %% "akka-stream" % AkkaVersion
```

Lets create a new `object` and place the needed imports and infrastructure for running streams:

``` scala
import akka.actor.ActorSystem
import akka.stream.scaladsl._

object GettingStarted extends App {
  implicit val system = ActorSystem("GettingStarted")
}
```

You may be wondering why we need an `ActorSystem` here. In order to run our streams we need a `Materializer`. This component has the responsility of intialize all the required resources our stream needs in order to run. A `Materializer` is bringed into scope through the `ActorSystem`. Well talk more about this later.

# Main stream components

`Source` is the beginning of the stream. Its purpose is to emit elements (because of that, it just has output).

``` scala
val source: Source[Int, NotUsed] = Source(1 to)
```

The first type parameter refers to the type of value the `Source` emits. The second one refers to the `materialized value type` (more on this later).

`Flow` is an intermediate step that performs transformations over the elements it receives. So, it has input and produces output. For example:

``` scala
val addOne: Flow[Int, Int, NotUsed] = Flow[Int].map(x => x + 1)
```

The first two parameters refer to the input and output types, respectively. The last one is the type of the `materialized value`.


# It's all about component reutilization

# Materialization

# Materialized value

# Alpakka

So far so good, but what about real world problems?

# A real world example

# Graphs

# Conclusion

Things that were missing in this introduction: logging, error handling, stream lifecycle, testing and some operators. Those topics will be covered in a future post. Also, I want to write a dedicated one about working with graphs.