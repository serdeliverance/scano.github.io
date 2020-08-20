---
title: Currying
excerpt: "What is Currying and How to use it with real world examples"
categories:
  - blog
tags:
  - scala
header:
  image: "/images/header.jpg"
---

## Intro

## Motivation

`TODO`

## Currying

Example use case. I'm trying to implement some validation for a port field. I know the port can has a value between 1 to 65535. I can create a validateRange method that checks that a value match inside a range, and use it passing mi value and 1 and 65355 as an argument, and It would be ok. But I have the feeling of loosing some semantics if I do that way.

Why not to define a validateRange function with the range curryfied and define then a validPort function which use this previous function passing the appropiate range and use the last function on my validation pipeline?

Example

``` scala
// defining my generic function (with currified arguments)
def validateToRange(value: Int)(min: Int, max: Int): ValidatedNel[String, Unit] =
Validated.condNel(
  min to max contains value,
  (),
  s"$value is not between $min to $max"
)

// defining my particular function that its based on the previous one
def validPortValue(value: Int): ValidatedNel[String, Unit] = validateToRange(value)(MIN_PORT_VALUE, MAX_PORT_VALUE)

// using the function in the validation pipelin
def validPort(port: String, param: String): ValidatedNel[CirceValidationError, Unit] =
	isDigit(p, param) |+|
    validPortValue(Try(p.toInt).toOption).getOrElse(-1))
       .fold(errors => invalidNel(CirceValidationError("invalid_param", param, errors.head)), validNel)
}
```

## Conclusion
