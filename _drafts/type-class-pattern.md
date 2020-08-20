---
title: Type Class Pattern
excerpt: "What is the Type Class Pattern and Why is it useful"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

`TODO`

## Type Class

Type class is a pattern originated in Haskell to enable ad-hoc polymorphism (aka overloading in OOP). In OOP languages we used to have polimorphysm by subtyping.

A type class is a trait that takes a type and describes the operations that can be applied to that type.

Why can this be useful for us? Because it allow us to extend existing libraries with new functionality, without using inheritance and without altering the original source code.

The type class pattern has three components:

* the type class
* instance for particular types
* interface methods that we expose to users.