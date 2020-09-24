---
title: Node REST API without using third party libraries
date: 2020-09-24 20:35:00
excerpt: 'Create a minimalist and powerful REST API using just Node built-in APIs'
categories:
  - blog
tags:
  - node
  - javascript
header:
  image: '/images/header.jpg'
---

## Intro

Some weeks ago, I needed to implement a minimal REST API using Nodejs. The purpose of that API was to expose and endpoint to run another Node process and another endpoint for retrieving a file. So, It acted more like an interface with the file system. At first, I thought the same as everyone attempt to think: "just install http-server or express, write the endpoints and go back home", but thinking a litle bit more I realized that just using some built-in `Node modules` was enough. Another interesting aspect of that API is that needed be run on a docker container with an old `Node` version installed (with old I mean a legacy 0.x.x version) which can lead with incompatibility issues between our legacy Node and our third party library (in case of using one).

In this article, I'm going to use a more recent `Nodejs` version together with modern javascript syntax.

## Prerequisites

- Node v14.x.x

## Setting up our base server

Let's setting up our base server.

```javascript
const http = require('http')

const host = 'localhost'
const port = 8080

const requestHandler = (req, res) => {
  res.writeHead(200)
  res.end()
}

const server = http.createServer(requestHandler)

server.listen(port, host, () =>
  console.log(`Server running on http://${host}:${port}`)
)
```

As you can see, the code is very simple: just importing the `http` module, creating a handler function (`resquestHandler`) attend each request, creating a server using `http.createServer` passing to it the handler function and finally calling the `listen` method is enough to have a working server. Let's continue adding some functionallity.

## An endpoint for calling another node process

Now, we need an endpoint whose purpose is execute another `node` script. We don't care about the script's final execution status. We just need to run it in a 'fire and forget' fashion, so, it will be a simple async call to run it and then returning immediately a response to the user without caring about the execution result of the script.

For running a file system command from a Node js app, we'll use a builtin module called `child_process`, which allow us to spawn child process from our current one.

```javascript
const http = require('http')
const exec = require('child_process').exec

const requestHandler = (req, res) => {
  exec('bash factorialCalculator.sh', (error, stdout, stderr) => {
    if (error) {
      console.error(`Error running script: ${error}`)
      return
    }
    console.log(stdout)
  })

  res.writeHead(200)
  res.end()
}
```

Addionally, we require this endpoint to be a `POST`. So, putting all these things together, the code will look like this:

```javascript
const http = require('http')
const exec = require('child_process').exec

const requestHandler = (req, res) => {

    if (req.method === 'POST') {
        exec('bash factorialCalculator.sh', (error, stdout, stderr) => {
        if (error) {
            console.error(`Error running script: ${error}`)
            return
          }
          console.log(stdout)
        })

        res.writeHead(200)
        res.end()
    } else {
        res.writeHead(404)
        res.end()
    }
```

One aspect that I don't like about this code is that the main logic is buried inside the conditional statement. Later, we are going to refactor that.

## More requirements

A part from our first endpoint, we need to expose another one for retrieving files from the file system. We need to add another built module for doing that: `fs`.

```javascript
const fs = require('fs')

const filesDir = 'files'

const requestHandler = (req, res) => {
  if (req.method === 'GET') {
    console.log('Retrieving file...')

    let fileName = req.path

    let filePath = `${filesDir}/${fileName}`

    fs.readFile(filePath, (err, data) => {
      if (err) return {
          res.writeHead(404)
          res.end(JSON.stringify(err))
      }

      res.writeHead(200)
      res.end(data)
    })
  }
}
```

Additionally, we were required to add and endpoint for cleaning up the `files` directory. We are going to associate this endpoint with the `DELETE` operation.

```javascript
const fs = require('path')

const requestHandler = (req, res) => {
  if (req.method === 'DELETE') {
    fs.readdir(filesDir, (err, files) {

        if (err) throw err

        files.forEach(file =>
            fs.unlink(path.join(filesDir, file), (err) => if (err) throw err)
        )
    })
  }
}
```

We used `path`, another built-in module, for building paths.

## Refactor and final code

Now, our code looks like this:

```javascript
const http = require('http')
const exec = require('child_process').exec
const fs = require('fs')
const path = require('path')

const host = 'localhost'
const port = 8080

const filesDir = 'files'

const requestHandler = (req, res) => {

    if (req.method === 'GET') {
        console.log('Retrieving file...')

        let fileName = req.path

        let filePath = `${filesDir}/${fileName}`

        fs.readFile(filePath, (err, data) => {
            if (err) return {
                res.writeHead(404)
                res.end(JSON.stringify(err))
            }

            res.writeHead(200)
            res.end(data)
        })
    }

    if (req.method === 'POST') {
        console.log('Calling script...')
        exec('bash factorialCalculator.sh', (error, stdout, stderr) => {
        if (error) {
            console.error(`Error running script: ${error}`)
            return
          }
          console.log(stdout)
        })

        res.writeHead(200)
        res.end()
    }

    if (req.method === 'DELETE') {
        console.log('cleaning directory')
        fs.readdir(filesDir, (err, files) {

            if (err) throw err

            files.forEach(file =>
                fs.unlink(path.join(filesDir, file), (err) => if (err) throw err)
            )
        })
    }
}

const server = http.createServer(requestHandler)

server.listen(port, host, () => console.log(`Server running on http://${host}:${port}`))
```

We have defined various endpoints an its OK and maybe this API won't grow in a future, but this sequence of `if`s does not seems so clear. We can apply a little refactor.

First, we are going to extract the logic of executing the command, retrieving the file and cleaning directory as separated functions:

```javascript
const runScript = (res) => {
    console.log('Retrieving file...')

    let fileName = req.path

    let filePath = `${filesDir}/${fileName}`

    fs.readFile(filePath, (err, data) => {
        if (err) return {
            res.writeHead(404)
            res.end(JSON.stringify(err))
        }

        res.writeHead(200)
        res.end(data)
    })
}

const getFile = (res) => {
  console.log('Retrieving file...')

    let fileName = req.path

    let filePath = `${filesDir}/${fileName}`

    fs.readFile(filePath, (err, data) => {
        if (err) return {
            res.writeHead(404)
            res.end(JSON.stringify(err))
        }

        res.writeHead(200)
        res.end(data)
    })
}

const cleanDirectory = (res) => {
    console.log('cleaning directory')

    fs.readdir(filesDir, (err, files) {

        if (err) throw err

        files.forEach(file =>
            fs.unlink(path.join(filesDir, file), (err) => if (err) throw err)
        )
    })

    res.writeHead(200)
    res.end()
}
```

Then, we can use a `switch` on our `requestHandler` to centralize all our `HTTP methods` and forward to the appropiate function:

```javascript
const requestHandler = (req, res) => {
  switch (req.method) {
    case 'POST':
      runScript(res)
      break
    case 'GET':
      getFile(res)
      break
    case 'DELETE':
      cleanDirectory(res)
      break
  }
}
```

The final code looks like this:

```javascript
const http = require('http')
const exec = require('child_process').exec
const fs = require('fs')
const path = require('path')

const host = 'localhost'
const port = 8080


const runScript = (res) => {
    console.log('Retrieving file...')

    let fileName = req.path

    let filePath = `${filesDir}/${fileName}`

    fs.readFile(filePath, (err, data) => {
        if (err) return {
            res.writeHead(404)
            res.end(JSON.stringify(err))
        }

        res.writeHead(200)
        res.end(data)
    })
}

const getFile = (res) => {
  console.log('Retrieving file...')

    let fileName = req.path

    let filePath = `${filesDir}/${fileName}`

    fs.readFile(filePath, (err, data) => {
        if (err) return {
            res.writeHead(404)
            res.end(JSON.stringify(err))
        }

        res.writeHead(200)
        res.end(data)
    })
}

const cleanDirectory = (res) => {
    console.log('cleaning directory')

    fs.readdir(filesDir, (err, files) {

        if (err) throw err

        files.forEach(file =>
            fs.unlink(path.join(filesDir, file), (err) => if (err) throw err)
        )
    })

    res.writeHead(200)
    res.end()
}

const requestHandler = (req, res) => {
  switch (req.method) {
    case 'POST':
      runScript(res)
      break
    case 'GET':
      getFile(res)
      break
    case 'DELETE':
      cleanDirectory(res)
      break
  }
}

const server = http.createServer(requestHandler)

server.listen(port, host, () => console.log(`Server running on http://${host}:${port}`))
```

## Conclusion

In this article, we saw how using the built-in modules that comes with `Node` is enough to do simple and powerful things. I believe that the power of `Node` not only lays on its vast library ecosystem, but in its simplicity. Furtermore, using it in conjunction with modern javascript idioms, make it a great and enjoyable platform to write code in.

I think it would be great to write a post about implementing similar stuff using the younger, type safer and still less popular Nodejs's brother: `Deno`. So, I'll pick that idea for a future post.
