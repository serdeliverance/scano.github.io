---
title: Implicits
excerpt: "What are implicits, Why are they so important and how to start using them?"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post, we are going to see a powerful feature of the `Scala` compiler called `implicit`.

## Implicits

Implicits in Scala refers to the compiler ability to do these two things:

* passing a value to a method automatically
* convert from one type to another automatically

These two features are known as `implicit parameters` and `implicit conversions`, repectively.

## Implicit parameters

Implicits parameters are inserted by the compiler by looking at the `search scope` where the function is called.

Example:

``` scala
def sum(x: Int)(implicit y: Int): Int = x + y

implicit val num = 4

sum(1)    // returns 5    the num parameter was inserted by the compiler by searching in the scope
sum(2)    // returns 6    the num parameter was inserted by the compiler by searching in the scope
```

At a first glance, it not seems to be very useful, but actually it does. It comes in handy when you have a chain of method calls that are passing around the same parameter from one to another. For example, imagine that you are developing a web application which when receives a request, it performs parsing, param validation, authorization and then calls the controller method. In all those steps, the request is passed together with an object of type `HttpContext` which contains server specific configurations:

``` scala

def parse(request: HttpRequest, context: HttpContext): Result = ???

def validate(request: HttpRequest, context: HttpContext): Result = ???

def authorize(request: HttpRequest, context: HttpContext): Result = ???

def handleRequest(request: HttpRequest, context: HttpContext): Result = {
  parse(request, context)
  validate(request, context)
  authorize(request, context)
  controller.action(request, context)
}
```

When using our API it is tedious passing the same parameter all the pipeline along. A part from that, passing `HttpContext` explicitly through all these steps in the chain does not bring useful semantics to the API client (it makes some noise).

However, if we use `implicits parameters` instead:

``` scala
implicit val httpContext: HttpContext = HttpContext(conf)

def parse(request: HttpRequest)(implicit context: HttpContext): Result = ???

def validate(request: HttpRequest)(implicit context: HttpContext): Result = ???

def authorize(request: HttpRequest)(implicit context: HttpContext): Result = ???

def handleRequest(request: HttpRequest): Result = {
  parse(request)
  validate(request)
  authorize(request)
}

val request = HttpRequest("{ name: \"john\"}")

handleRequest(request)  // not even the handle method called, but the method looks more clean
```

That way, we let the compiler do the boilperplate work for us and it makes more clean how to use the API.

Implicit parameters can be:

- val/var
- object
- accessor methods (`defs` with no parenthesis)

And all of them must be inside a class, object or trait in order to be visible by the compiler.

### Implicit scope

`Implicit scope` are the places where the compiler searchs for implicits. The compiler will look for them in the following order:

1) normal scope or LOCAL SCOPE (the local context where the function with implicit params is called)
2) imported scope      (example: ExecutionContext.Implicits.global)
3) companion objects of all types involved in the method signature

Example. Having the following function:

``` scala
def sorted[B >: A](implicit ord: Ordering[B]): List

val list = List(24, 3, 1, 6)

list.sorted
```

The compiler will start lookin at the local scope (the context were the function is called). If it does not succeed, then it will check if there is an implicit value of the required type declared in the imports. If it still fails, it will look at the companion objects of the types involved in the method signature. In this case, it will look at the companions of List, Ordering, A and any supertype of A.

If the compiler finds two candidates for the same type that it wants, it will complain with an `ambiguous implicit values` error.

## Implicit Conversions

If you call `someMethod` over an object `a` of a class `A`, and that class does not supports the method `someMethod`, then the `Scala` compiler will look for an implicit conversion from type `A` to a type `B` that supports the method `someMethod`.

``` scala
"23".map(_.toInt)
```

In this case, `String` does not support the `map` method, so the compiler looks for an `implicit conversion` in order to convert the `String` to a type that supports `map`. In `Predef`, there is implicit conversion (a method called `augmentString`) from `String` to `StringOps` (a class which has the map method defined), so the compiler will use it.

Let's see another example.

``` scala
case class Person(name: String) {
  def sayHello() = s"Hi, I'm $name"
}

implicit def stringToPerson(string: String) = Person(string)

"john".sayHello()
```

In this example, we defined a class `Person` which defines the method `sayHello`. Then we invoke the method `sayHello` over an instance of `String`. Because `String` does not support this method, the compiler looks for an `implicit conversion` from `String` to something that supports this method. Luckly, we have our implicit conversion `stringToPerson` in scope, so the compiler will use it to transform from `String` to `Person` object and then invoke the target method:

``` scala
stringToPerson("john").sayHello()   // what the compiler does
```

In other words, an `implicit conversion` is not more that the compiler ability to look for a conversion, which is actually a function of type `A => B`, being `B` the type that supports the operation that were ask to perform initially over an object of type `A`.

## Conclusion

In this post we've seen an introduction to `implicits` which is a powerful feature of `Scala`. `implicit parameters` and `implicit conversions` are used a lot in the `Scala` ecosystem, and not just there, but even in your domain code when you start feeling comfortably with them. Both features are very important to understand very common concepts in the `Scala` ecosystem, such as `Extension methods`, `Pimp my library pattern`, and even more advanced ones (and most popular too) like `Type Classes`, which are the main gateway to the `Cats` library.