# What is Node js
Node.js is a runtime for built on top of Chrome's V8. It allows you to develop apps in JavaScript outside of the browser. It's single threaded non blocking and asynchronous
# Installing Node.js
## For Windows
If you're running windows and not using WSL, I recommend you use the official instlaller from the Node.js site. Choose the latest LTS version.

## For everyone else
If you're not on windows or you are using WSL, I recommend using installing Node.js with nvm (node version manager). NVM allows you to install many versions of Node.js at once and switch when you need. Also, NVM installs Node.js in a folder that will not have permission errors that you would otherwise run into with the official installer.

Once you have nvm installed, you need to install a Node version. You can download the latest LTS version with this command.

```nvm install --lts```

# Executing Node
## Node REPL
Before we get into creating programs and apps with Node.js, let's get a feet wet with the Node.js REPL. Just type node in your terminal. This will create a Node.js environment where we can write and execute JavaScript. You'll see that all that JS knowledge you have learned carries over to Node.js! 💯. Although, some things, specifically browser based API's probably won't work. Go ahead, try an alert('hello world') and see what happens. You'll get an error. Although the Node.js runtime uses JS as it's languge, none of the Browser based globals that you are familiar with actually exist in Node.js or they are pollyfilled to prevent runtime errors.

## File Execution
The Node REPL it nice, but you can't build an application with that. We need to be able write our code to a file and tell Node.js to execute that.

Create a file called:index.js

In that file, write some JS. Try out console.log('hello there'). To execute this .js file in the Node.js runtime, we can use the node command with the file name as an argument.

```> node index.js```

# Globals
Like the Browser, Node.js comes with some practical globals for us to use in our applications.

## Common
global Think of this as like window but for Node.js. DON'T ABUSE IT!
__dirname This global is a String value that points the the directory name of the file it's used in.
__filename Like __dirname, it too is relative to the file it's written in. A String value that points the the file name.
process A swiss army knife global. An Object that contains all the context you need about the current program being executed. Things from env vars, to what machine you're on.
exports module require These globals are used for creating and exposing modules throughout your app

# Modules
There is no GUI in Node.js, no HTML or CSS. This also means there aren't any scipt tags to include JS files into our application. Node.js uses modules to to share your JavaScript with other JavaScript in your apps.
## What is a module
A module is a reusable chunk of code that has its own context. That way modules can't interfere with or polute the global scope.

You can think of them like lego blocks that you can create, import, and share.

## Two module types
By default, Node.js uses common js modules. With newer versions of Node.js we can now take advantage of ES6 modules. To opt into using this syntax, you can use the .mjs extension instead of .js

## Module syntax
Now, let's create our first module. The only thing we have to do is expose some code in one for our JavaScript files. We can do that with the export keyword.
```
// utils.js
export const action = () => {

}

export const run = () => {

}
```

```
// app.js

import { action, run } from './utils'
```

Few things happening here. Let's look at the utils.js file. Here we're using the export keyword before the variable declartion. This creates a named export. With the name being whatever the variable name is. In this case, a function called action. Now in app.js, we import the action module from the utils file. The path to the file is relative to the file you're importing from. You also have to prefix your path with a './'. If you don't, Node will think you're trying to import a built in module or npm module. Because this wa a named export, we have to import with brackets { action, run } with the exact name of the modules exported. We don't have to import every module that is exported.

Usually if you only have to expose one bit of code, you should use the default keyword. This allows you to import the module with whatever name you want.

```
// utils.js
default export function () {
  console.log('did action')
}
```

```
// app.js
import whateverIWant from './utils'
```

You can read more about the module syntax on the Node.js [Docs](https://nodejs.org/api/packages.html).

## Internal Modules
Node.js comes with some great internal modules. You can think of them as like the phenonminal global APIs in the browser. Here are some of the most useful ones:

fs - useful for interacting with the file system.
path - lib to assit with manipulating file paths and all their nuiances.
child_process - spawn subprocesses in the background.
http - interact with OS level networking. Useful for creating servers.

# File System
Until Node.js, there wasn't a great way to access the file system on a machine with JavaScript, this is due to secutrity restrictions in most browsers. With Node.js, one can create, edit, remote, read, stream, & more with files. If you've ever used a build tool like webpack or a parser like babel, then you realize just how powerful Node.js can be when manipulating the file system.

## Reading a file
Node.js ships with a powerful module, fs short for file system. There are many methods on the fs module. To read a file, we'll use the readFile method.

Create a simple html file template.html.

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
  <h1>{title}</h1>
  <p>{body}</p>
</body>
</html>
```

This html file will be used as template and has placeholders that we will interpolate later when writing a file.

To read the file:

```
import { readFile } from 'fs/promises'

let template = await readFile(new URL('./template.html', import.meta.url), 'utf-8')
```

The fs module has import for promise based methods. We'll opt to use those as they have a cleaner API and using async + non-blocking methods are preferred. More on that later. Because we're using .mjs files, we don't have access to __dirname or __filename which is what is normally used in combination with the path module to form an appropiate file path for fs. So we have to use the URL global that takes a relative path and a base path and will create a URL object that is accepted by readFile. If you were to log template, you'd see that its just a string.

## Write a file
Writing a file is similar to reading a file, except you need some content to place in the file. Let's interpolate the variables inside our template string and write the final html string back to disk.

```
import { readFile, writeFile } from 'fs/promises'

let template = await readFile(new URL('./test.html', import.meta.url), 'utf-8')

const data = {
  title: 'My new file',
  body: 'I wrote this file to disk using node'
}

for (const [key, val] of Object.entries(data)) {
  template = template.replace(`{${key}}`, val)
}

await writeFile(new URL('./index.html', import.meta.url), template)
```

You should now have a index.html file that is the same as the template.html file but with the h1 and body text substituted with the data object properties. This is some powerful stuff 🔥! Open it in a browser and see it work. At their core, static analysis tools like TypeScript, Babel, Webpack, and Rollup do just this.

# Error Handling
The last thing you want is your entire server crashing because of an error, or, is that exactly what you want 🤷‍♀️? Regardless, you should have th choice. So you better handle those errors. Depending on the type of code (sync, promise, async callback, async await, etc) Node allows us to handle our errors how we see fit.

## Process exiting
When a exception is thrown in Node.js, the current process will exit with a code of 1. This effectively errors out and stops your programing completely. You can manually do this with:

process.exit(1)

Although you shouldn't. This is low level and offers no chance to catch or handle an exception to decide on what to do. Imagine your entire API shutting down and restarting just because a user sent a malformed payload that resulting the DB throwing and exception. That would be terrible.

## Async Errors
When dealing with callbacks that are used for async operations, the standard pattern is:

```
fs.readFile(filePath, (error, result) => {
  if (error) {
    // do something
  } else {
    // yaaay
  }
})
```

Callbacks accept the (error, result) argument signature where error could be null if there is no error.

For promises, you can continue to use the .catch() pattern. Nothing new to see here.

For async / await you should use try / catch.

```
try {
  const result = await asyncAction()
} catch (e) {
  // handle error
}
```

## Sync Errors
For sync errors, try / catch works just fine, just like with async await.

```
try {
  const result = syncAction()
} catch (e) {
  // handle error
}
```

## Catch All
Finally, if you just can't catch those pesky errors for any reason. Maybe some lib is throwing them and you can't catch them. You can use:

```
process.on('uncaughtException', cb)
```

# Packages
The most beautiful part about Node.js is not the JavaScript, it's the thriving community. There are millions of node projects ready to be installed and consumed by your application. These projects are called packages. A package can have several modules and other packages. Node.js has built in support for these packages so you can take advantage of them at any time.

## NPM
### Init
To consume a package, we must first turn our app into a package. We can do this with a simple file called package.json on the root of our app. Writing it by hand is cool, but using a CLI called npm is better. NPM was already installed when you installed Node.js. In a new folder, run: npm init

This will initialze a new package by walking you through a few prompts. Once you're finished, you'll have a package.json file that looks like this:

```
{
  "name": "app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

Now are soon to be app is a package. We'll get into how to distribute and deploy different types of Node.js apps, but for now, this package is staying local. Let's take a look at some of these fields:

"name" - is the name of your package. Can be anything since we're local
"version" - is the Semantic Version Number or semver
"main" - the main entry point into your package
"scripts" - object of custom scripts to be excuted with npm cli

### Commands
NPM has several commands at its disposal that you don't need to know really. There are some important ones that you will use repeatedly.

npm install - installs given module(s) from remote registries or local sources
npm test - runs the test script in your package.json
npm uninstall - will uninstall a give package

### Finding and installing packages
Most modules are hosted on a registry somewhere. The biggest and most used one is, well, the [NPM registry](https://www.npmjs.com/). They don't stand alone though. Github (which owns NPM now) also allows devs to publish packages to their registry. And there are many others. The sweet thing about NPM, is that you can point to any registry, default being NPM.

A good flow to find a package you need that you don't already know by name yet, is to go the the [npm site](https://www.npmjs.com/) or Google and search for what you need. Say you need a lib to convert html to PDF's 🤷, NPM will give you back a list of packages.

Once you click a package, you can see the documentation from the README.md and any links to Github or website. You can also see the author and the last time it was updated. All of this info is great to help with choosing a package to install. You never know what you're going to get. Once you know the package(s) you want to install, you can do so with:

```
npm install package1 package2 package3 --save
```

You can install as many packages with one command as you like. The --save flag is to let NPM know to update the package.json's dependency field with all of these packages. We need this because we don't want to check in the downloaded packages into source code for many reason. So how does anyone else on your team, or even you on another machine know what packages this app needs? Well NPM will save the package names and versions so NPM on another machine can look at that and install from there. Your package.json should have updated.

You'll also notice a new folder on your project's root named node_modules. This is where NPM will install your packages. You should never have to touch this folder. But if you take a peek, you'll see more than the packages you installed. That's because those packages needed other packages, and so on and so on. NPM stores them flat in the node_modules folder. This helps with preventing duplicates and circular dependencies.

# Servers
Node.js has access to OS level functionality, like networking tools. This allows us to build very capable servers. Mixed with the fact that Node.js is single threaded and runs an even loop for async tasks, Node.js is widely used for API's that need to respond fast and don't require heavy CPU intensive work.

## The hard way
Node.js ships with the http module. This module is an abstraction around OS level networking tools. For Node.js, the http module would be considered "low level". Let's create a simple server.


```
import http from 'http'

const host = 'localhost'
const port = 8000

const server = http.createServer((req, res) => {
  if (req.method === 'POST') {
    let body = ''

    req.on('data', chunk => {
      body += chunk.toString()
    })

    req.on('end', () => {
      if (req.headers['content-type'] === 'application/json') {
        body = JSON.parse(body)
      }

      console.log(body)
      res.writeHead(201)
      res.end('ok')
    })
  } else {
    res.writeHead(200)
    res.end('hello from my server')
  }

})

server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`)
})
```

Using the createServer method on the http module, we create a server. Before we start the server, we need to make sure it can handle incoming requests. That's the callback inside of createServer. Next is starting the server. To do that, we need a port and a host. Sites default to port 8080 or 8000 so it's not uncommon to use that when developing locally. The host is going to be your machine, which is localhost or 127.0.0.1.

Using the http module is fine for this small example, but for bulding real world APIs we should utilize the community and install some packages to help up with this task.

## Express
There is an awesome packaged, express, that makes creating servers in Node.js a breeze. We're going to use it now.

npm install express body-parser morgan

express - a framework for building servers
body-parser - a middleware that parses incoming requests
morgan = a middleware for logging incoming requests
With everything installed, we'll create a simple API for a todo app using express.

```
import express from 'express'
import morgan from 'morgan'
import bp from 'body-parser'

const { urlencoded, json } = bp

const db = {
  todos: [],
}

const app = express()

app.use(urlencoded({ extended: true }))
app.use(json())
app.use(morgan('dev'))

app.get('/todo', (req, res) => {
  res.json({ data: db.todos })
})

app.post('/todo', (req, res) => {
  const newTodo = { complete: false, id: Date.now(), text: req.body.text }
  db.todos.push(newTodo)

  res.json({ data: newTodo })
})

app.listen(8000, () => {
  console.log('Server on http://localhost:8000')
})
```

Compared to the native http module, express feels like cheating.

Our todo API has two routes:

GET /todo - get all todos
POST /todo - create a new todo

# Testing
One of the most common usecases for Node.js is writing test for Node.js and Frontend apps. Because Node.js can run outside the browser, it's perfect for CI environments and testing automations.

## Basic unit tests
Unit test will test little chunks of your code in isolation to ensure they behave has intended. Node.js ships with the assert module. This module gives us so many utilities that allow us to create expectations of on our code. When those expectations aren't met, assert will throw an error telling us why. This is perfect for testing!.

First let's create some code to test:

```
// myLib.mjs
export const add = (num1, num2) => num1 * num2
```

Next let's write some assetions on this code

```
// test.mjs
import assert from 'assert'
import { add } from './myLib.mjs'

try {
  console.log('add() should add two numbers ')
  assert.strictEqual(add(2, 5), 7)
  console.log('  ✅ passed')
} catch (e) {
  console.log('  🚫 fail')
  console.error(e)
}
```

Now run the test file with:

```node test.mjs```

Happy coding :)