---
title: CPS concurrency model
excerpt: 'What are Continuations or Callbacks? An introduction to this concurrency model'
categories:
  - blog
tags:
  - scala
header:
  image: '/images/header.jpg'
---

## Intro

- asynchronous allows a better use of resource

Asynchronous execution

- Execution of a computation on another computing unit, without waiting for its termination

- Better resource efficiency

## Concurrency Control of Asynchronoous Programs

What if a program A depends on the result of an asynchronous executed program B?

Example

```scala
def coffeeBreak(): Unit = {
	val coffee = makeCoffee()
	drink(coffee)					// drink depends on the makeCoffee() execution
	chatWithColleagues()
}
```

`drink` depends on `makeCoffee` execution. If we want to express these model asynchronously, we have to make some changes. The first approach is using `callbacks`

## Callback

We call the method `makeCoffee` but passing a function to execute when it finishes. This function is known as `callback`.

def makeCoffee(coffeeDone: Coffee => Unit): Unit = {
// work hard...
// ... and eventually
val coffee = ...
coffeeDone(coffee)
}

def coffeeBreak(): Unit = {
makeCoffee { coffee =>
drink(coffee)}
chatWithColleagues()
}

## From Synchronous to Asynchronous Type Signatures

A synchronous type signature can be turned into an asynchronous type signature by:

- returning `Unit`

- and taking as parameter a `continuation` (or `callback`) defining what to do after the return value has bee computed. (for example: in the example above)

ex:

def program(a: A): B // synchronous signature

def program(a: A, k: B => Unit): Unit // now it is asynchronous

## Combining Asynchronous Programs

Exercise: combine two async computations

```scala
def makeCoffee(coffeeDone: Coffee => Unit): Unit

// my solution

// explanation: chaining two callbacks

def makeTwoCoffees(coffeeDone: (Coffee, Coffee) => Unit): Unit = {
	makeCoffee {
		coffee1 => makeCoffee {
			coffee2 => coffeeDone(coffee1, coffee2)
		}
	}
}
```

// teachers solution

// explanation: creates a unique callback that will be passed to the two methods invocations. This callback
has logic in order to differentiate the invocation time and group the results.

```scala
def makeTwoCoffees(coffeeDone: (Coffee, Coffee) => Unit): Unit = {
	var firstCoffee: Option[Coffee] = None
	val k = { coffee: Coffee ) =>
		firstCoffee match {
			case None 			=> firstCoffee = Some(coffee)
			case Some(coffee2)	=> coffeesDone(cofee, coffee2)
		}
	}
	makeCoffee(k)
	makeCoffee(k)
}
```

The code is cumbersome and error prone

## Callbacks all the way down

def coffeeBreak(breakDone: Unit => Unit): Unit = ???

def workRoutine(workDone: Work => Unit): Unit = {
work { work1 =>
coffeeBreak { \_ =>
work { work2 =>
workDone(work1 + work2)
}
}
}
}

--------------------- async --------------->>>>

## Handling Failures

- In synchronous programs failures are handled with exceptions

- What happens if an asynchronous call fails? we need to progragate the failure.

How can we acchieve this on asynchronous programming? We can signal the result.

Example:

def makeCoffee(coffeeDone: Try[Coffee] => Unit): Unit = ??? // Try is our signal. We handle the failure with Monads in these case

## Summary

Continuations (CPS or Callbacks)

- is a way for sequencing asynchronous computations
- introduces complex signatures
- the code is cumbersome and difficult to read
