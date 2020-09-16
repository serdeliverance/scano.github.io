---
title: Why Typescript?
excerpt: "An introduction to Typescript and the motivation behind it"
categories:
  - blog
tags:
  - typescript
header:
  image: "/images/header.jpg"
---

## Intro

* why type matters? Because otherwise is easier to write buggy code.

Advantages of tyescript:

* Easier catch errors during compilation
* Being typed, helps the compiler to give us better information which makes our life easier (code completion, describe object structure) 

## Example

Example

Communicating with an API. Un vanilla javascript, is easier to have typos that we can detect until runing the code.

``` javascript
axios.get(url).then(response => {
  const todo = response.data

  const ID = todo.ID
  const title = todo.Title
  const finished = todo.finished

  console.log(`
    The Todo with ID: ${ID}
    Has a title of: ${title}
    Is it finished? ${finished}`)
})
```

## Typescript

`TODO`

``` javascript
interface Todo {
  id: number
  title: string
  completed: boolean
}

axios.get(url).then(response => {
  const todo = response.data as Todo

  const id = todo.id
  const title = todo.title
  const completed = todo.completed

  console.log(`
    The Todo with ID: ${id}
    Has a title of: ${title}
    Is it finished? ${completed}`)
})
```

`interfaces` in `Typescript` defines the structure of an object.

## Another example

Passing argument in different order. Having the following method signature:

``` javascript
const logTodo = (id, title, completed) => {
  console.log(`
    The Todo with ID: ${id}
    Has a title of: ${title}
    Is it finished? ${completed}`)
}
```

In vanilla Javascript is easy to call it with arguments in incorrect positions:

``` javascript
const id = todo.id
const title = todo.title
const completed = todo.completed

logTodo(id, completed, title)		// we are passing the arguments in wrong order
```

How to fix that in Typescript? Again, types to the rescue:

``` typescript
const logTodo = (id: number, title: string, completed: boolean) => {
  console.log(`
    The Todo with ID: ${id}
    Has a title of: ${title}
    Is it finished? ${completed}`)
}
```

## Conclusion

The objective of `Typescript` is to help us to catch errors during development.