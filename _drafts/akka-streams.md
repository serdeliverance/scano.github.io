---
title: Akka Streams
excerpt: 'TODO TODO TODO'
categories:
  - blog
tags:
  - scala
  - akka
header:
  image: '/images/header.jpg'
---

## Intro

## Stream processing

- processing a number (possibly infinite of elements)

- such a pipeline is composed of operations that modified the elements

- operations often expressed as DSK similar to Scala collections (map, flatMap, filter)

## Motivation for streaming APIs

- lots of application are about processing data
- data sources can be intermittent or unbounded
- stream proecssing = manipulation fo data whose sources and intermitent and potentially unbounded

Streaming is only a piece of the reactive manifesto.

## Stream processing goals

- composition building-blocks
- handle flow-control through such stream pipeline
- process many, possible inifte elements at optimal rate

## Challenges

- resources efficiency
- flow controlled processing
- failure handling
- separation of business and operational concerns

## Reactive Stream semantics

### Flow-control and Reactive Streams

`flow-control == backpressure`

`Backpressure` is a mechanisim to control the rate at which an upstream publisher is emitting data to a downstream subscriber in proportion to the subscriber receiver capabilities.

Basically, it is the following case:

The upstream produces data at a fast rate
the downstream consumes data at a slow rate

val slowCOnsumer = context.spawn(SlowConsumer)
val fastProducer = context.spawn(FastProducer(slowConsumer))

`backpressure` tries avoids buffer overflow and message lost

### Reactive Streams

It is an initiative that provides a standard for exchanging streams of data.

Reactive Streams is an initiative to provide a standard for asynchronous stream processing with non-blocking back-pressure

asynchronous: producer send message and don't have to wait for response

non-blocking: means that the producer is not forced to wait for the consumer to process the data.

back-pressure: flow of data constrained by consumer demand

### Reactive Streams components

- a set of interaction RUles (the Specification)
- a set of types (the SPI)
- a Technology Compliance (test) Kit (TCK)

## Divers

Akka Streams is one of the many implementations of Reactive Streams
