---
title: Handling state with pure functions in Scala
date: 2020-07-04 17:00:32
excerpt: "How to handle state following a pure function approach"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post we'll dive into a technique for handling state in a pure functional way. This post is inspired in the great book `Functional Programming in Scala`.

## The problem

Imagine that we are designing an API for managing developers in your company. By the moment, it should have two methods:

* `work`: return the lines of code delivered by the developer and updates its total lines of code amount.

* `drinkCoffee`: auments the caffeine in blood of a developer to help him to develiver more lines of code.

We just came out with the following first draft:

``` scala
// our domain entity
class Developer(var name:String, var currentLanguage: String, var totalLinesOfCode: Int, var caffeineInBlod: Double)

object Developer {
  def work(dev: Developer): Int = {
    val linesOfCode = calculate(dev)
    dev.totalLinesOfCode += linesOfCode
    linesOfCode
  }

  def drinkCoffee(dev: Developer): DrinkResult = {
    if (dev.caffeineInBlod < 0.6) {
      dev.caffeineInBlod += CAFFEINE_INFUSION_INCREMENT
      Ok("you are ready to rock!")
    }
    else Failed("No more magic for today. Your caffeine level exceeds the maximum")
  }

  // helper method
  def calculate(dev: Developer) = Math.round(AVERAGE_LINES_OF_CODE * (1 + dev.caffeineInBlod)).toInt

  // DrunkResult classes were hidden for easy reading
}
```

When you have to perform some operations, you'll have something like this:

``` scala
val developer = new Developer("pedro", "java", 0, 0)
for(i <- 1 to 5) {
  if (i > 3) drinkCoffee(developer)
  work(developer)
}

println(s"developer: ${developer.name}. total lines of code: ${developer.totalLinesOfCode}. caffeine in blood: ${developer.caffeineInBlod}")

// prints "developer: pedro. total lines of code: 575. caffeine in blood: 0.5"
```

And it works, but the problem is that we are performing the operations as side effects. We are modifying data in place. Effectful APIs are not easy to test, compose, reason about and parallelized (shared mutable state, race conditions, non-deterministic behaviors). Let's see how we can improve that using a more functional approach.

## How to represent state in a functional programming way?

The answer is simple: we can perform the operation and return the result together with the new state. This way, we are not modifying data in place:

``` scala
// inmutable class
case class Developer(name:String, language: String, totalLinesOfCode: Int, caffeineInBlod: Double)

object Developer {
  def work(dev: Developer): (Int, Developer) = {
    val linesOfCode = calculate(dev)
    val newTotalLinesOfCode = dev.totalLinesOfCode + linesOfCode
    (linesOfCode, dev.copy(totalLinesOfCode = newTotalLinesOfCode))
  }

  def drinkCoffee(dev: Developer): (DrinkResult, Developer) = {
    if (dev.caffeineInBlod < 0.6) {
      val newCaffeineInBlood = dev.caffeineInBlod + CAFFEINE_INFUSION_INCREMENT
      (Ok("you are ready to rock!"), dev.copy(caffeineInBlod = newCaffeineInBlood))
    }
    else (Failed("No more magic for today. Your caffeine level exceeds the maximum"), dev)
  }
}
```

Two key points to notice here:
* representing the state change as a function
* inmutable data

## New requirements

Now, suppose that our company is taking care of its developers health, because it is said that they are drinking dangerous amount of coffee in order to reach the deadlines. We need to check how our employees coffee level is and return some alert for further analysis.

``` scala
def coffeeCheck(dev: Developer): (Option[Alert], Developer) = {
    if (dev.caffeineInBlod > ACCEPTED_CAFFEINE_LEVEL) (Option(Alert("Your rockstart's caffeine level is dangerous")), dev)
    else (None, dev)
}
```

Now imagine we need to implement a little piece of the role promotion policy. This process is very important because it allows developers to become team leaders. The complete promotion policy is under development but we need to implement just a piece of that. We need to implement the mechanisim that allow us detect possibly leaders. The rule is very simple: when a dev reaches the million of lines of codes delivered, he/she will become a `Team Lead Candidate`. Let's implement this requirement:

``` scala
def getCandidateForPromotion(dev: Developer): (Option[TeamLeadCandidate], Developer) = {
  if (dev.totalLinesOfCode > MILLION_LINES_OF_CODE) (Some(TeamLeadCandidate(dev.name)), dev)
  else (None, dev)
}
```

Some notes here:

* `Team Lead Candidates` are subject for further evaluation in order to become leaders, but this process is not defined yet, so we don't need to implement it.

* `getCandidateForPromotion` does not change the state, so its not needed to return `Option` and `Developer` but we implemented to follow the API style (and for another reason that we will talk on a next section.)

Up to now, we have the following API:

``` scala
object Developer {

  // some class were hidden for easy reading
  
  def work(dev: Developer): (Int, Developer) = {
    val linesOfCode = calculate(dev)
    val newTotalLinesOfCode = dev.totalLinesOfCode + linesOfCode
    (linesOfCode, dev.copy(totalLinesOfCode = newTotalLinesOfCode))
  }

  def drinkCoffee(dev: Developer): (DrinkResult, Developer) = {
    if (dev.caffeineInBlod < 0.6) {
      val newCaffeineInBlood = dev.caffeineInBlod + CAFFEINE_INFUSION_INCREMENT
      (Ok("you are ready to rock!"), dev.copy(caffeineInBlod = newCaffeineInBlood))
    }
    else (Failed("No more magic for today. Your caffeine level exceeds the maximum"), dev)
  }

  def coffeeCheck(dev: Developer): (Option[Alert], Developer) = {
      if (dev.caffeineInBlod > ACCEPTED_CAFFEINE_LEVEL) (Option(Alert("Your rockstart caffeine'S level is dangerous")), dev)
      else (None, dev)
  }

  def getCandidateForPromotion(dev: Developer): (Option[TeamLeadCandidate], Developer) = {
    if (dev.totalLinesOfCode > MILLION_LINES_OF_CODE) (Some(TeamLeadCandidate(dev.name)), dev)
    else (None, dev)
  }
}
```

We are done for now, but you know that new requirements are comming. Apart from that, we noticed that all these methods follow the same pattern and we start thinking if these abstractions can be improved a little bit.

## Factoring out types to improve our API

So far, we made a big progress making your API by expressing your business as a set of functions that works which inmutable data and avoids side effects. But it is not enough, you know that there is some boilerplate code that could be abstracted to express our API better.

We can see that all of our functions do something very similar: receives a state and returns some return value together with the new state. The type of them can be described as follows:

``` scala
Developer => (A, Developer)
```

These kind of functions are called `state actions`.

With that in mind, we can apply a little refactor to describe our functions better:

``` scala
def work: Developer => (Int, Developer) =
  dev => {
    val linesOfCode = calculate(dev)
    val newTotalLinesOfCode = dev.totalLinesOfCode + linesOfCode
    (linesOfCode, dev.copy(totalLinesOfCode = newTotalLinesOfCode))
  }

def drinkCoffee: Developer => (DrinkResult, Developer) =
  dev =>  {
    if (dev.caffeineInBlod < 0.5) {
      val newCaffeineInBlood = dev.caffeineInBlod + CAFFEINE_INFUSION_INCREMENT
      (Ok("you are ready to rock!"), dev.copy(caffeineInBlod = newCaffeineInBlood))
    }
    else (Failed("No more magic for today. Your caffeine level exceeds the maximum"), dev)
  }

def coffeeCheck: (Developer) => (Option[Alert], Developer) =
  dev => {
    if (dev.caffeineInBlod > ACCEPTED_CAFFEINE_LEVEL) (Option(Alert("Your rockstart caffeine level is dangerous")), dev)
    else (None, dev)
  }

def getCandidateForPromotion: (Developer) => (Option[TeamLeadCandidate], Developer) = 
  dev => {
    if (dev.totalLinesOfCode > MILLION_LINES_OF_CODE) (Some(TeamLeadCandidate(dev.name)), dev)
    else (None, dev)
  }
```

So, we have our `state actions` but we realized that we can typify them to avoid tedious repitition of code. Let's take advantage of `Scala` features and defined a `type alias`:

``` scala
type DeveloperAction[+A] = Developer => (A, Developer)
```

Applying this refactor, thats how our API methods look like:

``` scala
def work: DeveloperAction[Int] =
  dev => {
    val linesOfCode = calculate(dev)
    val newTotalLinesOfCode = dev.totalLinesOfCode + linesOfCode
    (linesOfCode, dev.copy(totalLinesOfCode = newTotalLinesOfCode))
  }

def drinkCoffee: DeveloperAction[DrinkResult] =
  dev =>  {
    if (dev.caffeineInBlod < 0.5) {
      val newCaffeineInBlood = dev.caffeineInBlod + CAFFEINE_INFUSION_INCREMENT
      (Ok("you are ready to rock!"), dev.copy(caffeineInBlod = newCaffeineInBlood))
    }
    else (Failed("No more magic for today. Your caffeine level exceeds the maximum"), dev)
  }

def coffeeCheck: DeveloperAction[Option[Alert]] =
  dev => {
    if (dev.caffeineInBlod > ACCEPTED_CAFFEINE_LEVEL) (Option(Alert("Your rockstart caffeine level is dangerous")), dev)
    else (None, dev)
  }

def getCandidateForPromotion: DeveloperAction[Option[TeamLeadCandidate]] =
  dev => {
    if (dev.totalLinesOfCode > MILLION_LINES_OF_CODE) (Some(TeamLeadCandidate(dev.name)), dev)
    else (None, dev)
  }
```

So, we have `state actions` and we typified them to avoid repitition of code. 

## Getter and Setters in a pure functional way

Let's see how we can model getter and setters following the same approach:

``` scala
def get: DeveloperAction[Developer] = dev => (dev, dev)

def set(dev: Developer): DeveloperAction[Unit] = _ => ((), dev)
```

As we can see, the returning value of `get` operation is the state itself, and it returns the next state (which is the same as the current state, because we are performing a `get`).

The `set` method receives the new state to set as a parameter. As a result, it returns nothing meaningful(`Unit`), however the next state is the same we have passed in the function parameter.

## Conclusion

We have seen how we can refactor an API which modifies data in place to become a more pure one just using plain `Scala`. We modeled our state change as pure functions using `state actions`. In fact, there is more power in this kind of functions. Together with `combinators` (which are HOF) you can create `fluent APIs` to express your data change pipeline in a chainable way. 