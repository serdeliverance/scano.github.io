---
title: Akka Typed
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

1. typed messages and actors

The messages than an actor can receive will be reflected in the type of the actor itself and vice versa

2. mutable state

3. hierarchy

## Typed messages and actors

```scala
trait ShoppingCartMessage
case class AddItem(item: String) extends ShoppingCartMessage
case class RemoveItem(item: String) extends ShoppingCartMessage
case object ValidateCard extends ShoppingCartMessage
```

In Akka Typed, an actor is defined by its behavior:

```scala
val shoppingRootActor = ActorSystem(
  Behaviors.receiveMessage[ShoppingCartMessage] { message: ShoppingCartMessage =>
    message match {
      case AddItem(item) => println(s"Adding $item to cart")
      case RemoveItem(item) => println(s"Removing $item from cart")
      case ValidateCard => println("The cart is good")
    }

    Behaviors.same
  },
  "simpleShoppingActor"
)
```

When defining an actor system what we are doing is defining the Root Guardian Behavior. The behavior is not more than a function of type `Message => Behavior`.

## Mutable state

In the classic Actor System we handle mutable state by using `context.become()` pattern which changed the receives handler of the actor.

In the new typed API:

```scala
val shoppingRootActor2 = ActorSystem(
  Behaviors.setup[ShoppingCartMessage] { ctx =>

    // local state = mutable
    var items: Set[String] = Set()

    Behaviors.receiveMessage[ShoppingCartMessage] { message: ShoppingCartMessage =>
      message match {
        case AddItem(item) =>
          println(s"Adding $item to cart")
          // mutate variable
          items += item
        case RemoveItem(item) =>
          println(s"Removing $item from cart")
          items -= item
        case ValidateCard => println("The cart is good")
      }
      Behaviors.same
    }
  },
  "simpleShoppingActorMutable"
)
```

`Behavior.setup` is a preliminar `Behavior` factory which takes a type argument `T` which is the type of message that this actor will handle.

This new approach is interesting because it allow us to perform some preliminar actions, such as spawning children, or reading some configuration, set some variable, etc.

It is important to note that we are returning a new `Behavior` each time we respond to a message. So, instead of using `context.become`, we return a new immutable behavior and this way of work is the way the new API defines for handling messages.

It is more easy to create new Behaviors and working with mutable state.

```scala
def shoppingBehavior(items: Set[String]): Behavior[ShoppingCartMessage] =
  Behaviors.receiveMessage[ShoppingCartMessage] {
    case AddItem(item) =>
      println(s"Adding $item to cart")
      // returning a new behavior with the new variable (immutable way)
      shoppingBehavior(items + item)
    case RemoveItem(item) =>
      println(s"Removing $item from cart")
      // returning a new behavior with the new variable (immutable way)
      shoppingBehavior(items - item)
    case ValidateCard =>
      println("The cart is good")
      Behaviors.same
  }
```

## Hierarchy

Instead of using `system.actorOf` was the old way of creating actors. In the new API, the only way we can create actors is by using `spawn` over an actor. The benefit of that is that it forces us to think better in the actor hierarchy. So, we can only spawn child actors from a given actor.

```scala
val rootOnlineStoreActor = ActorSystem(
  Behaviors.setup { ctx =>
    // create children
    ctx.spawn(shoppingBehavior(Set()), "johnShoppingCarg")

    Behaviors.empty
  },
  "onlineStore"
)
```

## TODO

- disadvantages
