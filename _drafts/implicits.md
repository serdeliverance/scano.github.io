---
title: Implicits
excerpt: "How to handle state change using pure functions"
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

### Implicit methods

### Implicit parameters

Implicit parameters can be:

- val/var
- object
- accessor methods = defs with no parenthesis

And all of them must be inside a class, object or trait in order to be visible by the compiler.

* Note: implicit parameters are not the same as default args. Implicits parameters are inserted by the compiler by looking at the search scope where the function is called.

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

We have seen how we can refactor an API which modifies data in place to become a more pure one just using plain `Scala`. We modeled our state change as pure functions using `state actions`. In fact, there is more power in this kind of functions. Together with `combinators` (which are HOF) you can create `fluent APIs` to express your data change pipeline in a chainable way. 