---
title: Eta-expansion
excerpt: "TODO TODO TODO"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

In this post we are going to see `eta-expansions` which is a feature of the Scala compiler which converts a method into a `FunctionN`. It is very useful on partially applied functions and in every case we want to use a method as a function.

## Method vs Functions

Methods are member of a class or object and can only be called if we refer to the member that enclose it. So, methods can not be invoked independently:

``` scala
```

Functions or lambdas can be invoked independently because they not depend of any class or object.

## Eta-expansions

`Eta-expansion` is a way to convert a method into a `Function`

## Eta-expansions over partially applied functions

## Conclusion

