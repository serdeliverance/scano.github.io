---
title: Persistent Actors and Event Sourcing
date: 2020-06-28 15:35:00
excerpt: "Persistent actors and Event Sourcing in Akka Peristence"
categories:
  - blog
tags:
  - akka
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post we will talk about what event sourcing is and how it can be implemented in Akka using persistent actors.

## What is Event Sourcing?

Most of us are used to work following a traditional CRUD approach. When modeling systems that way, what we have is a photo of the current state of the system. So, image that you are designing an API for people who invest into cryptocurrencies and you have to model the currency entity. Maybe, when you would think about how it will be stored, you would arrived to something like the following:

```
coin_id    | name         | price

1          | "Bitcoin"    | 9650
2          | "Ethereum"   | 230
3          | "Litecoin"   | 42  
4          | "Geekcoin"   | 13
```
And it is OK. But, maybe it could be not enough for this business. What we are getting is the most recent value of our coins. But what if we need to know about what was the Bitcoin value last week? In other words: How can we query for a previous state? Is it possible? What if we need to know how we have arrived to the current state?

Making history can allow us to have some data to analize and support business decisions.

A part from that, traditional CRUD approach doesn't play well with domain semantics (we just have CREAD, READ, UPDATE and DELETE in terms of data operations). We can take a different approach which help us to express better what our domain does (something more DDD like).

How can we do that? The answer is: Event Sourcing

How? Instead of storing the current state, we store events related to our domain model. The application then re apply those events over our domain entity in order to recover its state.

PROS:
- high performance: events are only appended. We can use db that performs well on writing operations
- avoids relational stores and ORM.
- full trace of every state.
- audit logs for free.
- recover the state of an entity at an specific point in the time.
- evaluate historical data.
- plays well with DDD and microservices architecture by letting us writing bounded contexts for our domain.
- if you apply CQRS too, you can have separated models for read and write operations, and each one can be tunned according to its needs.
- ORM independence (we just store the serialized data that we need to apply to our entity to recover its state)

CONS:
- query state is potentially expensive (because of that, it is often used together with [CQRS](https://martinfowler.com/bliki/CQRS.html))
- potential performance issues with long-lived entities
- not the best fit for all business
- requires a mental shift

### Terminology

* Command: an operation sent to a domain object.
* Event: represents something that have happened in the past (ex: `QuoteUpdated`, `OrderCreated`, etc) and that causes our domain entity to change its state. In other words: it event represents an state change of our domain entity.
* State: the state of the domain entity.
* Event store: the place when we store the events of our domain entities.

The flow is the following:

When an entity receives a command, it first creates an event and then persists it to the event store. After this event was successfully persisted, we apply this change to update its state.

So, enough talking about what event sourcing is. Let's take a look at `Persistent Actors` and how they can help us to build event source based applications, but first, we need to configure our project.

## Project Setup

### Sbt project creation and dependencies

First of all, create an `sbt` project and add the following dependencies in your `build.sbt`:

```scala
lazy val akkaVersion = "2.6.6"
lazy val leveldbVersion = "0.7"
lazy val leveldbjniVersion = "1.8"

libraryDependencies ++= Seq(
  "com.typesafe.akka"          %% "akka-persistence" % akkaVersion,
  // local levelDB store
  "org.iq80.leveldb"            % "leveldb"          % leveldbVersion,
  "org.fusesource.leveldbjni"   % "leveldbjni-all"   % leveldbjniVersion
)
```
 Basically, it contains the `Akka Persistence` version and `LevelDB` dependencies for communicating with our local Event Store.

### Local Event Store configuration

For our event store, we are going to use [LevelDB](https://github.com/google/leveldb). We need to configure the `Akka Persistence Journal Plugin`. So, our `src/resources/application.conf` should look like this:

``` scala
# local store journal conf
akka.persistence.journal.plugin = "akka.persistence.journal.leveldb"
akka.persistence.journal.leveldb.dir = "target/akka-pers-demo/journal"
```
We configured our local event store db path on `target/akka-pers-demo` direcory, so be sure to have this directory created before running the code. 

## Use case

Following the use case seen in the intro, imagine that we are designing an API for crypto currencies. Our API clients are operators who hit the API to inform how much a certain currency has raised or down. So, we can start thinking that our domain entity should respond to the following commands:

```scala
// commands
case class UpdateQuote(quotePercentage: Double)
```

`quotePercentage` this value indicates how much the value of the currency has increased or decreased in terms of percentage.

And then we have the following event:

```scala
// events
case class QuoteUpdated(quotePercentage: Double)
```
Of course, as a consequence of firing those events over time, our entity state will be updated. Let's define our state:

``` scala
var quote: Double = 500   // some value
```
And somewhere we will have our event store having an structure similar to the following:

```
event_id    | event         | entity_type | entity_id  | event_data

100         | "Bitcoin"     | "Bitcoin"   |   1        | {...}      // raw json data
101         | "Ethereum"    | "Ethereum"  |   2        | {...}
102         | "Litecoin"    | "Geekcoin"  |   3        | {...}
103         | "Geekcoin"    | "Bitcoin"   |   1        | {...}
```
In `Akka Persistence` our event store can defer a little bit but follows the same idea. In addition to the fields that identifies the entities and events, it is interesting to see how the event data is stored. They are stored as raw json values in this case, but we can apply the serialization mechanism that best fit our needs. It is up to the application to deserialize the `event_data` into a domain event that then will be applied to the specific entity. We are ORM independent.

How can we put all these stuff together? Using `Persistent Actors`.

## Persistent Actors

`Akka Persistence` provide us with `Persistent Actors` in order to implement event sourcing. `Persistent Actors` are similar to regular actors. So, they can send and receive messages (aka commands, in the actor persistence and event source terminology), and keep an internal state. In addition, they have extra capabilities:

- persistent ID: needed to identify the persistent actor related events in the event store.
- persist: method that is used by persistent actor to persist events on the event store.
- recover: method for recovering the state of the persisting actor by querying the event store and re applying all the events asociated with it (this can be possible by using the persistent ID which identifies it)

For creating a Persistent Actor what we need to do is to extend the `PersistentActor` trait:

``` scala
class GeekCoinActor extends PersistentActor {

  override def persistenceId: String = ???

  override def receiveCommand: Receive = ???

  override def receiveRecover: Receive = ???
}
```

When a persistent actor receives a command, it creates an event based on these commands, then persist that event and, after that, update its internal state. Thats the way they work. The events are persisted to the event store asynchronously.

## Implementing our Persistent Actor

First, let's create a companion object for holding our commands and events:

``` scala
object GeekCoinActor {
  
  // commands
  case class UpdateQuote(quotePercentage: Double)
  
  // events
  case class QuoteUpdated(quotePercentage: Double)

  // and a helper function to use later
  def calculateNewQuote(quote: Double, quotePercentage: Double) = quote + quote * quotePercentage / 100
}
```

The state of our entity:

```scala
class GeekCoinActor extends PersistentActor {

  var quote: Double = 100   // our initial state

  // more code below
}
```

Before continue, let's add `ActorLogging` trait in order to have a trace of our actor behavior:

``` scala
class GeekCoinActor extends PersistentActor with ActorLogging {
  // continue below...
}
```

Add a `persistenceId` to identify our entity on the event store:

```scala
class GeekCoinActor extends PersistentActor with ActorLogging {

  var quote: Double = 100

  override def persistenceId: String = "geekcoin"

  override def receiveCommand: Receive = ???

  override def receiveRecover: Receive = ???
}
```

Implement the command handler method:

``` scala
override def receiveCommand: Receive = {
  case UpdateQuote(quotePercentage) =>
    log.info(s"Received UpdateQuote: $quotePercentage")
    // after the event is persisted...
    persist(QuoteUpdated(quotePercentage)) { event =>
      log.info(s"Event persisted: $event")
      // ...we update the state
      quote = calculateNewQuote(quote, quotePercentage)
      log.info(f"State updated. New state -> quote: $quote%1.2f")
    }
}
```

Implement the recover method:

``` scala
override def receiveRecover: Receive = {
  case event @ QuoteUpdated(quotePercentage) =>
    quote = calculateNewQuote(quote, quotePercentage)
    log.info(f"Recovered event: $event. State -> quote: $quote%1.2f")
}
```

Let's test our persistent actor:

``` scala
val system = ActorSystem("PersistentActorDemo")
val geekCoinActor = system.actorOf(Props[GeekCoinActor], "geekCoinActor")

// sending commands to our actor
geekCoinActor ! UpdateQuote(50.0)
geekCoinActor ! UpdateQuote(-15.0)
geekCoinActor ! UpdateQuote(200.0)
geekCoinActor ! UpdateQuote(-45.0)
geekCoinActor ! UpdateQuote(235.0)
```
Running the app, we should see the following output:

```
[INFO] [06/28/2020 17:58:44.360] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Received UpdateQuote: 50.0
[INFO] [06/28/2020 17:58:44.468] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Event persisted: QuoteUpdated(50.0)
[INFO] [06/28/2020 17:58:44.469] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] State updated. New state -> quote: 150,00
[INFO] [06/28/2020 17:58:44.469] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Received UpdateQuote: -15.0
[INFO] [06/28/2020 17:58:44.489] [PersistentActorDemo-akka.actor.default-dispatcher-7] [akka://PersistentActorDemo/user/geekCoinActor] Event persisted: QuoteUpdated(-15.0)
[INFO] [06/28/2020 17:58:44.489] [PersistentActorDemo-akka.actor.default-dispatcher-7] [akka://PersistentActorDemo/user/geekCoinActor] State updated. New state -> quote: 127,50
[INFO] [06/28/2020 17:58:44.489] [PersistentActorDemo-akka.actor.default-dispatcher-7] [akka://PersistentActorDemo/user/geekCoinActor] Received UpdateQuote: 200.0
[INFO] [06/28/2020 17:58:44.511] [PersistentActorDemo-akka.actor.default-dispatcher-7] [akka://PersistentActorDemo/user/geekCoinActor] Event persisted: QuoteUpdated(200.0)
[INFO] [06/28/2020 17:58:44.513] [PersistentActorDemo-akka.actor.default-dispatcher-7] [akka://PersistentActorDemo/user/geekCoinActor] State updated. New state -> quote: 382,50
[INFO] [06/28/2020 17:58:44.513] [PersistentActorDemo-akka.actor.default-dispatcher-7] [akka://PersistentActorDemo/user/geekCoinActor] Received UpdateQuote: -45.0
[INFO] [06/28/2020 17:58:44.534] [PersistentActorDemo-akka.actor.default-dispatcher-12] [akka://PersistentActorDemo/user/geekCoinActor] Event persisted: QuoteUpdated(-45.0)
[INFO] [06/28/2020 17:58:44.534] [PersistentActorDemo-akka.actor.default-dispatcher-12] [akka://PersistentActorDemo/user/geekCoinActor] State updated. New state -> quote: 210,38
[INFO] [06/28/2020 17:58:44.534] [PersistentActorDemo-akka.actor.default-dispatcher-12] [akka://PersistentActorDemo/user/geekCoinActor] Received UpdateQuote: 235.0
[INFO] [06/28/2020 17:58:44.550] [PersistentActorDemo-akka.actor.default-dispatcher-13] [akka://PersistentActorDemo/user/geekCoinActor] Event persisted: QuoteUpdated(235.0)
[INFO] [06/28/2020 17:58:44.550] [PersistentActorDemo-akka.actor.default-dispatcher-13] [akka://PersistentActorDemo/user/geekCoinActor] State updated. New state -> quote: 704,76
```

We re running our app, the actor state is recovered by reading the evento store and you would see something like this:

```
[INFO] [06/28/2020 18:03:03.460] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Recovered event: QuoteUpdated(50.0). State -> quote: 150,00
[INFO] [06/28/2020 18:03:03.461] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Recovered event: QuoteUpdated(-15.0). State -> quote: 127,50
[INFO] [06/28/2020 18:03:03.461] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Recovered event: QuoteUpdated(200.0). State -> quote: 382,50
[INFO] [06/28/2020 18:03:03.461] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Recovered event: QuoteUpdated(-45.0). State -> quote: 210,38
[INFO] [06/28/2020 18:03:03.462] [PersistentActorDemo-akka.actor.default-dispatcher-6] [akka://PersistentActorDemo/user/geekCoinActor] Recovered event: QuoteUpdated(235.0). State -> quote: 704,76
```

## The complete code sample

``` scala
import PersistentActorDemo.GeekCoinActor.{QuoteUpdated, UpdateQuote, calculateNewQuote}
import akka.actor.{ActorLogging, ActorSystem, Props}
import akka.persistence.PersistentActor

object PersistentActorDemo extends App {

  object GeekCoinActor {
    // commands
    case class UpdateQuote(quotePercentage: Double)
    // events
    case class QuoteUpdated(quotePercentage: Double)

    def calculateNewQuote(quote: Double, quotePercentage: Double) = quote + quote * quotePercentage / 100
  }

  class GeekCoinActor extends PersistentActor with ActorLogging {

    var quote: Double = 100

    override def persistenceId: String = "geekcoin"

    override def receiveCommand: Receive = {
      case UpdateQuote(quotePercentage) =>
        log.info(s"Received UpdateQuote: $quotePercentage")
        // after the event is persisted...
        persist(QuoteUpdated(quotePercentage)) { event =>
          log.info(s"Event persisted: $event")
          // ...we update the state
          quote = calculateNewQuote(quote, quotePercentage)
          log.info(f"State updated. New state -> quote: $quote%1.2f")
        }
    }

    override def receiveRecover: Receive = {
      case event @ QuoteUpdated(quotePercentage) =>
        quote = calculateNewQuote(quote, quotePercentage)
        log.info(f"Recovered event: $event. State -> quote: $quote%1.2f")
    }
  }

  // just for testing
  val system = ActorSystem("PersistentActorDemo")
  val geekCoinActor = system.actorOf(Props[GeekCoinActor], "geekCoinActor")

// sending commands to our actor
  geekCoinActor ! UpdateQuote(50.0)
  geekCoinActor ! UpdateQuote(-15.0)
  geekCoinActor ! UpdateQuote(200.0)
  geekCoinActor ! UpdateQuote(-45.0)
  geekCoinActor ! UpdateQuote(235.0)
}
```

## Improving performance

As you can imagine, recovering the state of an entity with lot of events by querying them from the genesys can lead to a performance issue. It that case, you can periodically take snapshots.
Snapshots are a way to store the state of an entity at a given point in time. For example, we can define taking an snapshot every 20 events. It allow us to fast recover the state by re applying the events from the snapshot point in time up to the most recent event. 

## Conclusion

In this post we have seen what `Event Sourcing` is, its pros and cons and how we can implement it using `Persistent Actors` in `Akka`. You have to bear in mind that `Event Sourcing` is a different approach for designing systems and requires a mental shift. It allows you to have a rich domain model with powerfull semantics that express better what your business does. However, it is not a silver bullet (for a simple CRUD system, it could be an inefficient and overkilling solution). In future posts we will talk about Snapshots and more stuff related with `Akka Persistence`.

The code is available on [GitHub](https://github.com/serdeliverance/sc-blog-code/tree/master/akka-persistence)