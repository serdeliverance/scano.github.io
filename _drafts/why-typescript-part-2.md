---
title: Why Typescript? Part 2
excerpt: "TODO TODO TODO"
categories:
  - blog
tags:
  - typescript
header:
  image: "/images/header.jpg"
---

## Intro

* interfaces
* tuples
* type alias
* typed arrays

## Interfaces

Interface is a way to define a new type, describing the property names and value types of an object. So, when definind interfaces we are creating new types.

## Tuples

Some discorauge the use of tuple, because we can have the same functionallity use object and with more meaning at a glance

``` typescript
const carSpecs: [number, number] = [400, 3354]

const carsStats = {
	horsepower: 400,
	weight: 3354
}
```

## Type alias

``` typescript
type Drink = [string, boolean, number]

const coca: Drink = ['brown', true, 40]
const sprite: Drink = ['clear', true, 50]
```
## Typed arrays

help with methods 'map', 'foreach', 'reduce'

``` typescript
carMarkers.map((car: string): string => {
  return car // because we indicate the type, typescript help us with method autocompletion
})
```

## Classes with syntactic sugar

``` typescript
class Vehicle {
  color: string

  constructor(color: string) {
    this.color = color
  }

  drive(): void {
    console.log('driving...')
  }
}
```

A syntactic sugar for reducing code on the constructor is removing the variable declaration and adding the `public` modifier to the constructor parameter:

``` typescript
class Vehicle {
  constructor(public color: string) {}

  drive(): void {
    console.log('driving...')
  }
}
```