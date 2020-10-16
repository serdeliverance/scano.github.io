---
title: Using Akka Streams with Typed Actors on Play Framework 2.8
excerpt: 'How to use Akka Stream with Typed Actors on Play 2.8'
categories:
  - blog
tags:
  - scala
  - akka
  - play
header:
  image: '/images/header.jpg'
---

## Intro

Since `Play Framework 2.8` we can integrate with `typed actors`. So, In this post we are going to see how to run an `Akka Stream` graph which streams the result set of a query to a `typed actor` on `Play Framework`.

## Setup

First, create the project. We are going to use the official seed:

```scala
sbt new playframework/play-scala-seed.g8
```

By the date of this post, I will download a `Play 2.8.2` project.

Add the following dependecies on your `build.sbt`:

`TODO`

```scala
libraryDependencies ++= Seq(
  guice,
  // streams
  "com.lightbend.akka" %% "akka-stream-alpakka-slick" % "2.0.2",
  "com.typesafe.akka" %% "akka-stream-typed" % akkaVersion,
  // test
  "org.scalatestplus.play" %% "scalatestplus-play" % "5.0.0" % Test
)
```

We are using `Alpakka Slick` for streaming the results of the database query. Also, we are going to use `Akka Stream Typed` which is a Akka library that bring us sources and sinks for integrating with typed actors. It is important to note that we don't have to declare `Akka Streams` in our dependencies section, because it will be imported transitively by `Alpakka Slick`.

## Introduction to the use case

Imagine that we are working for a global consulting company which stores information about the software developers they have hired in order the keep track the talent they have. We have a simple `Rest API` that exposes endpoints for performing CRUD operations agains `Developer` resource. It is built on `Play Framework` and a new requirement arrives. It states that we need some batch processing that query all the developers we have stored, perform some calculation in order to build a report and then send an email to some stackholder. We are required to implement that.

Diving into the required report, it needs to calculate: most popular technology, highest salary, hottest location (the location which holds the majority of the developers) and the amount of talent the company has grouped by seniority.

Our developer entity is composed as follows:

```scala
case class Developer(name: String, technology: String, location: String, seniority: String, salary: BigDecimal)
```

## Solution

Our global consultant company has almost 500k employees. Because of that, querying the database and retrieving it results in one sake is not an option. You can query it in a paginated way either but you have started learning streams some weeks ago and then you realized that it will be a perfect fit for that use case.

Because you don't feel so confident with `Akka Streams` yet you decided to use it only for streaming the query results and delegate the report generation to a `Typed Actor`. So, you decided build a simple graph that query the employees from database and sink them to an actor, which olds some mutable state for performing the calculation. The actor also understands the message for sending an email (but we are going to place here a dummy implementation)

`Note` we used a `Typed Actor` here just for tutorial purposes, but you can implement all the report requirements inside the graph using the stream API and not having any actor implemented by you at all or at least having one with less responsabilities.

## Our controller and service layers

First, we have defined the following routes in our `routes` file:

```scala
GET     /healthCheck                           controllers.HealthCheckController.healthCheck()

POST    /batch                                 controllers.BatchController.batch()
```

`Note` we are not going to dive into the `healthCheck` endpoint and controller because it is out of the scope of this post. However, if you want to check it out, the code is on [GitHub](www.google.com).

Having said that, here is how our `BatchController` looks like:

```scala
@Singleton
class BatchController @Inject()(val controllerComponents: ControllerComponents, val batchService: BatchService)
                               (implicit ec: ExecutionContext) extends BaseController {

  def batch() = Action {
    batchService.executeBatchProcessing()
    Ok("batch processing") }
}
```

It is very simple and it returns immediately because its only purpose is to execute a batch process.

`TODO show the BatchService that call to the stream pipeline`

Before creating the stream we are going to add `Akka Typed` support to our `Play` application

## Adding Akka Typed Support

Lets create a package called `global` whose purpose is to hold global configuration and then create a class called `Module` into it with the following content:

`TODO investigate the Module class and give a brief introduction to it purpose`

## Creating the typed actor

## Creating the Stream

## Some refactor to our former Stream

## Conclusion

In this post be have seen how to integrate our `Play Framework 2.8` application with `Typed Actors` and how to using it in a pipeline. The code of this post is available on [GitHub](www.google.com)
