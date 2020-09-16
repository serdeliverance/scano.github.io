---
title: Typescript Best Practices
excerpt: "TODO TODO TODO"
categories:
  - blog
tags:
  - typescript
header:
  image: "/images/header.jpg"
---

## Intro

## Best Practices

* anytime we have a file whose primary purpose is to create and export a class, usually has to be name with Capital letter. Example: User.ts

* no use `default exports` because they are a source of confusion. 

* user modifiers to hide attributes and restrict access when needed. Example:

``` typescript
export class CustomMap {
  private googleMap: google.maps.Map

  constructor(divId: string) {
    this.googleMap = new google.maps.Map(document.getElementById(divId), {
      zoom: 1,
      center: {
        lat: 0,
        lng: 0
      }
    })
  }
}
```

In the previous example we hide googleMap object which contains lots of functionallity that we won't like the user to have access to.

* avoid duplicate code using interfaces

``` typescript
// duplicated code
addUserMarker(user: User): void {
    new google.maps.Marker({
      map: this.googleMap,
      position: {
        lat: user.location.lat,
        lng: user.location.lng
      }
    })
  }

// duplicated code
addCompanyMarker(company: Company): void {
	new google.maps.Marker({
	  map: this.googleMap,
	  position: {
	    lat: company.location.lat,
	    lng: company.location.lng
	  }
})
}
```

* use `implements` clauses with interfaces. It forces your class to implement the `interface` correctly, makes easier catch errors early and make the code clearer for other developers.

First, add `export` to your `interface`

``` typescript
export interface Mappable {
  location: {
    lat: number
    lng: number
  }
  markerContent(): string
}
```

Import and use it when you need

``` typescript
import { Mappable } from './CustomMap'

export class User implements Mappable {
  constructor() {
    this.name = faker.name.firstName()
    this.location = {
      lat: parseFloat(faker.address.latitude()),
      lng: parseFloat(faker.address.longitude())
    }
  }
```

Now, if you modified your `Mappable interface` and `User` not satifies which it states, you will detect this inconscistency more easyly.

It helps you being sure you are implementing the `interface` correctly.

* a Typical Typescript file should contain the following (and in this order):

** Interface definitions working with this class

** Class definition