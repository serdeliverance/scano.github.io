---
title: Reactive Application
excerpt: 'TODO TODO TODO'
categories:
  - blog
tags:
  - scala
header:
  image: '/images/header.jpg'
---

## Intro

A reactive application is non-blocking and event based

Actors are run by a dispatcher - potentially shared - which can also run Futures

Prefer immutable data structures, since they can be shared

Prefer context.become for different states, with data local to the behavior

Do not refer to actor state from code running asynchronously
