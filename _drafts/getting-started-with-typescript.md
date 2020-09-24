---
title: Getting started with Typescript and Node
excerpt: "How to setting up a Node project to start using Typescript"
categories:
  - blog
tags:
  - typescript
  - node
header:
  image: "/images/header.jpg"
---

## Intro

In this post we are going to see how to setting up a project to start using `Typescript`.

## Prerequisites

* Nodejs

## Set up

### Install Typescript globally

First of all, we need `Typescript` install globally on our OS.

``` typescript
npm install -g typescript
```

To check that everything is OK, just run a `tsc --version` on the console.

### Project initialization

Create the project directory

``` typescript
mkdir getting-started-typescript
```

The go into that directory and run.

### Initialize it as a Node project

``` typescript
npm init
```

It will create the `package.json` which handles our dependecies and scripts.

### Adding Typescript Compiler settings

The following command will create our `tsconfig.js` which allow us to configure different options for the typescript compiler to apply to our project (such as src and output dirs, etc)

``` typescript
tsc --init
```

### Configs for development

We are going to install

### Typescript support for Standard Node libs

`npm install --save @types/node`

Wherever you need to use some Node standard library, such as `fs`, you will need to run this command. This command downloads the `Type Definition Files` which are files that tell typescript the input and output parameters types of all the methods in a specific library (in this case, `Node`), because remember, `Typescript` relies on types, so it needs the definition of all the methods and variables involved in your application.

### TODO

* config tsc options on tsconfig.ts
* install nodemon
* install concurrently
* config scripts
* typescript support for standard node libs
