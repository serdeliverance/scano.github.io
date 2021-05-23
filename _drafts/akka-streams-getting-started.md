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

Stream data processing is a way of dealing with data transformation that resembles how computers internally work (for example, low level protocols such as TCP). In a streaming pipeline, elements flow from an origin to a destination, passing through different intermediate transformations.

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

`TODO`

Now, we are ready to write our first graph (remember this term).

# Main components of a stream topology

# It's all about component reutilization

# Materializing a graph

# Materialized value

# Alpakka

So far so good, but what about real world problems?

# A real world example

# Graphs

# Conclusion

Things that were missing in this introduction: logging, error handling, stream lifecycle, testing and some operators. Those topics will be covered in a future post. Also, I want to write a dedicated one about working with graphs.