---
title: The Ask Pattern
date: 2020-06-05 19:53:52
excerpt: "Ask pattern in Akka"
categories:
  - blog
tags:
  - akka
  - scala
---

## Intro

In this post where are gonna talk about the Ask Pattern. It is an Interaction Pattern that is used when you need some kind a request-response communication between actors. So, it differs from Tell which is used when you just want to send a message to an actor and don't care about the response (Fire and Forget).

## Project Setup

First of all, create an `sbt` project and add the following dependencies in your `build.sbt`:

```scala
val akkaVersion = "2.5.30"

libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % akkaVersion,
  "com.typesafe.akka" %% "akka-testkit" % akkaVersion,
  "org.scalatest" %% "scalatest" % "3.1.0"
)
```
## Ask Pattern

### Introducing the sample Use Case

#### Cinema ticket system

You have a Cinema Ticket Machine which allows users to select a particular movie, and then, if selected the sits are availabe, it makes a reservation.

For these case, we have two components: CinemaTicketMachine and MovieTheater. Of course, both are actors. The first is in charge of receiving the user request for a movie, check if it exists in the catalog, then check if it has available seats by sending a message to the MovieTheater actor, and retrieve the response back.
The MovieTheater holds the seats and knows about their availability.
As you can see, there is a kind a request-response comunication between those components.

For keeping the example simple:
* The CinemaTicketMachine only checks that the film exists in its catalog.
* We have only one MovieTheater instead of several (and it actually has a few seats too)

As an starting point, we have the following actors:

```scala

class CinemaTicketMachine extends Actor {


  val movieCatalog = initializeMovies()

  val movieTheather = context.actorOf(Props[MovieTheater], MOVIE_THEATER_ACTOR_NAME)

  override def receive: Receive = ???

}

class MovieTheater extends Actor {

  // it is not a realistic amount of seats, but it's right for sample purposes
  var seats = List((Seat("A",1), true), (Seat("A",2), true), (Seat("B",1), true), (Seat("B",2), true))

  override def receive: Receive = ???

}
```

Lets see a first approach using `change behavior`.

### First approach: change behaviour

Lets dive into a possible solution based on the use of `context.become`.

```scala
// CinemaTicketMachine  
  def waitingForRequest(): Receive = {
    case MovieRequest(movie) =>
      if (movieCatalog.contains(movie)) {
        movieTheather ! GetAvailableSeats
        context.become(waitingForMovieTheaterResponse(sender()))
      } else sender() ! MovieNotAvailable(MOVIE_NOT_EXISTS)
  }

  def waitingForMovieTheaterResponse(replyTo: ActorRef): Receive = {
    case AvialableSeatsResponse(seats) =>
      if (seats.isEmpty) replyTo ! MovieNotAvailable(NOT_AVAILABLE_SEATS)
      else replyTo ! MovieOK(seats)
      context.become(waitingForRequest())
  }
```

So, the above solution has the following issues:

Using `context.become` that way, we restrict messages cinema ticket can handle in the meantime its collaborator responds. Remember, we are in a distributed environment, handling a lot of requests. So, if we receive a `MovieRequest`, and right after we receive another one, maybe the last message would go straight to dead letters because cinema ticket machine's handler is only allowed to respond `AvailableSeatsResponse` at that time.
Following the previous idea, it is difficult to keep track of the sender we need to respond to. We had to keep the sender in the handler, which is a bad idea because we are closing over the actor state.
You can workaround this by modifying `GetAvailableSeats` to add the sender to respond as a parameter, and retrieving it back from the movie theater response. We would need to modify `AvailableSeatsResponse` too. If we do this, we'll have something like this:

```scala
// CinemaTicketMachine
  def waitingForRequest(): Receive = {
    case MovieRequest(movie) =>
      if (movieCatalog.contains(movie)) {
        movieTheather ! GetAvailableSeats(sender())
        context.become(waitingForMovieTheaterResponse())
      } else sender() ! MovieNotAvailable(MOVIE_NOT_EXISTS)
  }

  def waitingForMovieTheaterResponse(): Receive = {
    case AvialableSeatsResponse(replyTo, seats) =>
      if (seats.isEmpty) replyTo ! MovieNotAvailable(NOT_AVAILABLE_SEATS)
      else replyTo ! MovieOK(seats)
      context.become(waitingForRequest())
  }

// MovieTheater
  override def receive: Receive = {
    case GetAvailableSeats(replyTo) =>
      val availableSeats = seats.filter(_._2).map(_._1)
      sender() ! AvialableSeatsResponse(replyTo, availableSeats)
  }
```

The code is more cumbersome than before and it becomes harder to maintain. Apart from that, we are tightly coupling both actors and adding to movie theater the innecesary responsability of handling the sender and retrieving it back. Another key point to bear in mind is that we've done it because we can, but, if MovieTheater is a component that is out of our control and we have no access to? Maybe, a lot of crazy work arrounds will raise at this point.
Fortunately, we have another way to do that.

### Solution: Ask Pattern

The `ask pattern` consists in handling the interaction between actors by incorporating the use of Futures. For example, we'll have something like this:

``` scala
val future = movieTheather ? GetAvailableSeats 
```

We store the async computation inside a Future which allow us to do some useful operations later. Here, I use the use of the `ask` method vía `?` which is a sintatic sugar. You can also call the ask method on the actor in a classic OOP style as follows: `movieTheather.ask(GetAvailableSeats)`

To use the `ask pattern`, we need to add the following:

* Import the `ask` method:

``` scala
import akka.pattern.ask
```

* add a timeout for the Future responses

``` scala
implicit val timeout: Timeout = Timeout(2 seconds)
```

* add an execution context for the Future

``` scala
implicit val executionContext: ExecutionContext = context.dispatcher
```

Now, lets see how it looks like:

``` scala
class CinemaTicketMachine extends Actor {

  implicit val timeout: Timeout = Timeout(2 seconds)
  implicit val executionContext: ExecutionContext = context.dispatcher

  val movieCatalog = initializeMovies()
  val movieTheather = context.actorOf(Props[MovieTheater], MOVIE_THEATER_ACTOR_NAME)

  override def receive: Receive = {
    case MovieRequest(movie) if movieCatalog.contains(movie) =>
      val replyTo = sender()
      val future = movieTheather ? GetAvailableSeats
      future.onComplete {
        case Success(response) => response match {
          case AvialableSeatsResponse(seats) =>
            if (seats.isEmpty) replyTo ! MovieNotAvailable(NOT_AVAILABLE_SEATS)
            else replyTo ! MovieOK(seats)
        }
      }
  }
}
```
(For simplifying the reading, I added the movie validation to the case sentence)
We stored the original sender in order to use it inside the Future match. See how we are not closing over the actor state. Instead, we have all we need inside the receiver. Future makes easier to handle `Success` or `Failure` results as well.

#### Extra note

Even though we can implement our requirement easily using `ask pattern`, you have to bear in mind that it comes with a performance cost because `Akka` needs to keep track of the timeout, stablish a relation for communication between the actors involved when the task finishes and its even more difficult on a cluster environment. So, keep that in mind and only use it when you must.

## Conclusion

In this post, we have seen what the `ask pattern` is and how we can use it through a simple use case. Also, we compared that solution with another one based on `changing behavior`. Finally, we  in mind that it comes with a performance tradeoff that me need to be aware of and use it only when we really needed. The code is available on [github](https://github.com/serdeliverance/sc-blog-code)