---
title: Akka Streams Getting Started
date: 2021-06-06 15:20:52
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

Hi, everyone. There has been a long time from my last post. In this post, I want to introduce you to the `Akka Streams` library which I considered is an awesome and powerfull tool for writing asynchronous pipelines of data transformation. In this intro, I want to show you what `Akka Stream` is, what problems it solves, where it can be a good fit, what are its basic building blocks, an intro to the Alpakka project through a real world example.

Before talking about Akka Streams, it is important to know what streams are.

# What are streams?

Imagine you are working on a method that receives a list of transactions and performs some kind of calculation. If you receive 100 elements there may be no problem. But what if the app growns and you start receiving 10k, 100k or even more transactions each time. Generally, collections are data structures that needs to be fully loaded in memory before processing.

So, there are scenarios when processing the data as a whole is not an option.

Stream data processing is a way of dealing with data transformation that resembles how computers internally work (at the end, everything is a flow of bytes that flow from a computer into another, or through different tiers of the same computer). In a streaming pipeline, elements flow from an origin to a destination, emitted one at a time, passing through different intermediate transformations.

![flow sample](/assets/images/akka-streams/01-akka-streams-flow-sample.png "Flow Sample")

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
* implementing error handling by hand.
* lots of boilerplate because of manually implementing the previous features.

# Akka Stream use cases

* Batch processing
* Event Driven Architectures
* Realtime Applications (such as MMO Servers, chat servers, SSE)
* IoT

Or whenever you have to manipulate data as a pipeline or deal with potentially infinite data flows.

# Akka Streams vs Reactive Streams vs Reactive Systems

Quoting [the Reactive Streams documentation](https://www.reactive-streams.org/): `Reactive Streams is an initiative to provide a standard for asynchronous stream processing with non-blocking back pressure`.

So, `Reactive Streams` is an specification of how to deal with asynchronous boundaries without data lost or resource exhaustion.

In contrast, `Akka Streams` is an API that is compliant with that specification, but it offers its own API which is most suitable for end users. Remember, `Akka Streams` is all about building/expressing asynchronous pipelines and component reutilization.

On the other hand, `Reactive Systems` is a more broader topic. It is about building reactive system architectures as a whole, being compliant with a series of postulates that guarantees responsiveness, fault tolerant, loosed-coupled and scalable systems. In comparison, `Akka Streams` could be seen as a tool that can help you to build reactive systems.

Now, we are more clear about what `Akka Stream` is, but before diving into its API, we need to talk about `Back pressure`

# Back pressure

Streams are similar to producer-consumer architectures, where elements are emitted by a producer and consumed by consumers. In those arquitectures, we can lead to the following scenarios:

* slow producer and fast consumer
* fast producer and slow consumer

![slow producer fast consumer](/assets/images/akka-streams/sonic-slow-producer-fast-consumer.png "Slow Producer, Fast Consumer")

The first scenario is acceptable. There's no problem having a slow consumer. In that case, at least the consumer will be idle, but there's no buffer overflow risk.

![fast producer slow consumer](/assets/images/akka-streams/sonic-fast-producer-slow-consumer.png "Fast Producer, Slow Consumer")

However, the second scenario is something we want to be cared about. If we have a fast consumer and a slow consumer, it will lead to a situation where the consumer's buffer got overflowed. `Back-pressure` comes to solve this problem.

`Back-pressure` is a flow-control mechanism where the consumer signals the producer with the amount of elements it is able to handle. In order words, it is a comunication from consumer to producer, where the first one signals the demand it can process. That way, producers can adjust their rate of emitting elements.

![backpressure](/assets/images/akka-streams/sonic-backpressure.png "Backpressure")

It's important to mention that all this comunication between producers and consumers is asynchronous.

In a backpressure scenario, the with the amount of message (demand) the consumer is able to handle. To accomplish that the producer can implement one of the following strategies:
* stopping emitting message (if possible)
* buffering elements until suscriber signals that more elements can be emitted
* dropping elements
* tear down the stream if none of the previous strategies could be applied.

Now, we are ready to write our first graph (remember this term).

# Dependencies and getting started

First of all, the following dependecies should be added to your `build.sbt`:

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

Why we need an `ActorSystem` here? We'll answer that question along this tutorial.

# Main stream components

![source](/assets/images/akka-streams/source.png "Source")

`Source` is the beginning of the stream. Its purpose is to emit elements (because of that, it just has output).

``` scala
val source: Source[Int, NotUsed] = Source(1 to 1000)
```

The first type parameter refers to the type of value the `Source` emits. The second one refers to the `materialized value type` (more on this later).

![flow](/assets/images/akka-streams/flow.png "Flow")

`Flow` is an intermediate step that performs transformations over the elements it receives. So, it has input and produces output. For example:

``` scala
val addOne: Flow[Int, Int, NotUsed] = Flow[Int].map(x => x + 1)
```

The first two parameters refer to the input and output types, respectively. The last one is the type of the `materialized value`.

![sink](/assets/images/akka-streams/sink.png "Sink")

`Sink` is the final piece of your graph. It is the susbcriber to data sent by the `Source`. A Sink just has input data a produce no output: 

``` scala
val sinkPrintln = Sink.foreach(println)
```

When we have all our components connected, we say that we have a `RunnableGraph`, or a graph, for short. A graph is the topology built by connecting a source to a sink, passing through different intermediate transformation/processing steps (flows). It is important to know that a graph is just a description (a blueprint) of the pipeline. Nothing has happend yet (no resources have been assigned or processing has been started).

![graph](/assets/images/akka-streams/graph.png "Graph")


# It's all about component reutilization

Of course, you can write a graph and run it all in the sample place:

``` scala
Source(1 to 100)
  .map(x => x + 1)
  .map(x => x * x * x)
  .map(e => s"num: $e")
  .runWith(Sink.foreach(println))
```

But, you can build reusable pieces and then connect it together to form a graph:

``` scala
val sourceList = Source(1 to 1000)

val addOne = Flow[Int].map(x => x + 1)

val cube = Flow[Int].map(x => x * x * x)

val toLine = Flow[Int].map(number => s"num: $number")

val sinkPrintln = Sink.foreach(println)
```

Then you can connect all these pieces together and running the graph:

``` scala

val graph = sourceList
  .via(addOne)
  .via(cube)
  .via(toLine)
  .to(sinkPrintln)

graph.run()
```

It comes in handy if you, for example, require to build a different graph whose only difference is that it sinks to a filed instead of console.

``` scala
val lineToBytes = Flow[String].map(line => ByteString(s"$line\n"))

val sinkToFile = FileIO.toPath(Paths.get("result.txt"))

val graph2 = sourceList
  .via(addOne)
  .via(cube)
  .via(toLine)
  .via(lineToBytes)
  .to(sinkToFile)

graph2.run()
```

If we see that different sequences of transformations repeats frequently we can also create new reutilizable pieces (source, flow, sinks) by connecting them:

``` scala

val numberConverter = addOne.via(cube).via(toLine)

// combination of linetToBytes (flow) with sinkToFile, creating a new Sink.
val sinkLineToFile: Sink[String, NotUsed] = lineToBytes.to(sinkToFile)

val graph3 = sourceList
  .via(numberConverter)
  .to(sinkLineToFile)

graph3.run()
```

So, you can see that is very easy to define reutilizable pieces. It is up to you to decide how to combine them and what fits better with your use case.

# Materialization

An stream is a description. It is just a blueprint of the processing you want to perform. At that point we are not allocating any resource. The process that takes our stream description and allocates all the resources it needs is called `Stream Materialization`.

`Materializer` is the component that has the responsility of initializing all the required resources our stream needs in order to run. Most of times, it means starting actors (remember, streams are backed by actors), but it can also mean oppening TCP connections, files or allocating another kind of resource.

A `Materializer` is bringed into scope through the `ActorSystem`. So, having an `ActorSystem` in scope is enough to run your graphs.

# Materialized value

What happens inside a stream stays inside it.

What I'm trying to say is that when running a stream there is no observable effect outside. The effectful part happends in the sink, which is the consumer side of the stream, but not in other places of the external world.

However, there are situations when you want the stream to give you some information regarding processing. In that case, you need a `Materialized value`.

A `Materialized value` is an auxiliary value emitted from the stream to the outside world. It is a value that the stream "expose" to us when running.

Every graph component (Source, Flow and Sink) can materialize values. The type of the materialized value a component can emit is indicated by its last type parameter. Example:

``` scala
val sourceList: Source[Int, NotUsed] = Source(1 to 1000)      // emits no value to the external world

val addOne: Flow[Int, Int, NotUsed] = Flow[Int].map(x => x + 1) // emits no value to the external world

val sinkToFile: Sink[ByteString, Future[IOResult]] = FileIO.toPath(Paths.get("result.txt")) // emits a file
```

If every element can emit a materialized value, how can we decide which one to pick? By default, between two contiguous components, the materialized value of the component who is nearest to the source is taken. Example:

``` scala
val result = Source(List("scala", "akka", "streams"))
  .map(word => ByteString(s"$word\n"))
  .runWith(FileIO.toPath(Paths.get("result.txt")))
```

``` scala
val graph = sourceList
  .via(addOne)
  .via(cube)
  .via(toLine)
  .via(lineToBytes)
  .toMat(sinkToFile)(Keep.right)    // we indicate we want to materialize the materialized value emitted by sinkToFile

val result: Future[IOResult] = graph2.run()

```

# Conclusion

In this tutorial we have seen what `Akka Stream` is and what problems it solves. However, some topics were missing, such as logging, error handling, stream lifecycle, testing and some operators. Those will be covered in future posts. Also, I want to write a dedicated one about working with `Graph DSL`, which is a powerful API that `Akka Streams` provide us in order to write no linear asynchronous pipelines.