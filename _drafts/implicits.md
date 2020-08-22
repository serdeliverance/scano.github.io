---
title: Implicits
excerpt: "What are implicits and how we can use them?"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

`TODO`

## Implicits

Implicits in Scala refers to two this:

* a value that can be passed automatically by the compiler
* a conversion from one type to another that is made by the compiler automatically.

## Implicit methods

## Implicit parameters

Implicits parameters are inserted by the compiler by looking at the search scope where the function is called.

Implicit parameters can be:

- val/var
- object
- accessor methods = defs with no parenthesis

And all of them must be inside a class, object or trait in order to be visible by the compiler.


### Implicit scope

`Implicit scope` are the places where the compiler searchs for implicits.

Implicit scope
- normal scope = LOCAL SCOPE      (highest priority)
- imported scope      (example ExecutionContext.Implicits.global)
- companion objects of all types involved in the method signature
    Example: the method

    ``` scala
    def sorted[B >: A](implicit ord: Ordering[B]): List
    ```

    (companion object) order of search:
    - List companion
    - Ordering companion
    - all the supertype involved = A or any supertype

If the compiler finds two candidates for the same type that it wants, it will fail with an `ambiguous implicit values` error.

## Implicit Conversions

If you call `someMethod` over an object `a` of a class `A`, and that class does not supports the method `someMethod`, then the `Scala` compiler will look for an implicit conversion

## Best practices

### implicit val

1)

* if there is a single possible value for it
* and you can edit the code for the type (for example: it is not on a third party package)

then define the `implicit` in the companion.

2)

* if there are many possible values
* but a single `good one`
* and you can edit the code for the type

then define the `good` implicit in the companion

3) 

* if there are many possible values

then package them in different objects, in order to be easier for you to select which one to import

## Some examples

* Futures

## Conclusion

`TODO`