## Intro

## Traditional multi-thread model

- multiple execution cores within one chip, sharing memory
- vritual cores sharing a single physical execution core

multi-threading: running parts of the same program in parallel

```scala
class BankAccount {

  private var balance = 0

  def deposit(amount: Int): Unit =
    if (amount > 0) balance = balance + amount

  def withdraw(amount: Int): Int =
   if (0 < amount && amount <= balance) {
     balance = balance - amount
     balance
   } else throw new Error("insufficient funds)
}
```

race conditions => synchronize critical sections with locks, mutex, semaphore

Problems:

- possible deadlocks
- code difficult to read

By using locks, we use the implicit lock that exists on objects and composition of synchronized objects is more difficult also.

Problems with blocking

- deadlocks
- bad for CPU utilization
- couples the sender and the receiver

## Actor model

Actor model represents objects and their interactions.

An actor:

- is an object with identity
- has a behaviour
- only interacts using asynchronous message passing

## The Actor trait

```scala
type Receive = PartialFunction[Any, Unit]

trait Actor {
  def receive: Receive
}
```

`receive` is a partial function from Any to Unit and it describe the response of the actor to the message.

```scala
class Counter extends Actor {
  var count = 0
  def receive = {
    case "incr" => count += 1
  }
}
```

Making it Stateful

```scala
class Counter extends Actor {
  var count = 0
  def receive = {
    case "incr" => count += 1
    case "get" => sender ! count
  }
}
```

Changing actor behavior with `context.become`:

```scala
class Counter extends Actor {
  def counter(n: Int): Receive = {
    case "incr" => context.become(counter(n + 1))
    case "get"  => sender ! n
  }

  def receive = counter(0)
}
```

Actors are created by actors.

## Actor encapsulation

- every actor knows its own address (self)
- creating an actor returns its address
- addresses can be sent within messages (sender)

Actors are completely independent

- local execution, no notion of global synchronization
- all actors run concurrently
- message passing primitive is one-way communication

Actors are completed encapsulated and decoupled from other actors

## Acotr-Internal Evaluation Order

An actor is effectively single-threaded:

- messages are received sequentially
- behavior change is effective before processing the next message
- processing one message is the atomic unit of execution

## Message Delivery Guarantees

- all communication is inherently unreliable

- delivery of a emssage requires eventual availability of channel & recipient

at-most-once: sending once delivers [0,1] times

at-least-once: resending until acknowledged delivers [1, inf) times

exactly-once: processing only first reception delivers 1 time

## Reliable Messaging

Messages support reliability

- all messages can be persisted
- can cinlude unique correlation IDs
- delivery can be retried until successful

Reliability can only be ensured by business-level acknowledgement

## Message Ordering

If an actor sends multiple messages to the same destination, they will not arrive out of order (this is Akka-specific)

## Good practices

- Define actor's messages in its companion object

## Extra notes

- Actors are run by a dispatcher - potentially shared - which can also run Futures

## Conclusion
