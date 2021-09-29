<h1>TypeScript BC</h1>
<i>TypeScript with a borrow checker, multi-threading, and tiny binaries.</i><br>
<br>
Born out of the need for safe multi threading in graphical applications (web, native, desktop) while maintaining the high level of productivity we experience with expressive modern high-level languages.

<h4>Sneak Preview - Hello World</h4>


```typescript
import console from '@std/console'

function main() {
  const text: string = 'Hello World'
  console.log(read text)
}
```

<i>Please contribute to the design of this language specification!</i><br><br>
<i>Note that this <b>does not aim to be compatible with TypeScript and JavaScript</b> libraries instead, like Rust and Go, aims to have a strong standard library to facilitate most use cases where it leans on the community to port projects like Preact</i><br>


<h4>Table of Contents</h4>

- [Introduction](#introduction)
- [Borrow Checker tl:dr](#borrow-checker-tldr)
- [Audiences](#audiences)
- [Justification](#justification)
    - [GUI Applications](#gui-applications)
      - [Multi threading](#multi-threading)
      - [Small Binary](#small-binary)
    - [Web Servers:](#web-servers)
      - [Go-like Concurrency with Rust-like safety](#go-like-concurrency-with-rust-like-safety)
      - [Tiny, statically linked binaries](#tiny-statically-linked-binaries)
- [Language Design](#language-design)
  - [Hello World](#hello-world)
      - [Notes](#notes)
- [Success Challenges / Quest Log](#success-challenges--quest-log)
- [Examples](#examples)
  - [Simple HTTP Server](#simple-http-server)
  - [HTTP Server with Sate](#http-server-with-state) 

<h4>Specification Details</h3>

Below are pages with additional information on the language specification.

- [Borrow Checker](./specification/borrow-checker.md)
- [Concurrency and Parallelism](./specification/concurrency-and-parallelism.md)
- [Mutex](./specification/mutex.md)
- [Observables](./specification/observables.md)
- [Promises](./specification/promises.md)
- [Structural Types](./specification/structural-types.md)
- [Exceptions](./specification/exceptions.md)
- [Classes and Inheritance](./specification/classes-and-inheritance.md)

# Introduction

This repository describes the specification for a language based on TypeScript that implements a Rust-like borrow checker.

The objective of this is to have a language that produces the smallest binaries possible, supports multi-threading and offers the ergonomics of the TypeScript language.

The Borrow Checker checker introduced by Rust allows for thread-safe code to be written while also offering the compiler the ability to manage memory allocations and deallocations at compilation time, rather than at runtime through a garbage collector.

The concept of modules in TypeScript allows for efficient static analysis allowing compilers to trim dead code efficiently. 

TypeScript BC aims to be the simplest implementation of a borrow checker and therefore will have built-in types which make assumptions on their implementation. 

For example:

```typescript
const myString: string = "Hello World"
const myArray: number[] = []
```
Would assume:
```rust
let myString: String = String::from("Hello World");
let myArray: Vec<i32> = Vec::new();
```

# Borrow Checker tl:dr

For engineers reading this who are unfamiliar with the concept of Rust's borrow checker. Rust uses a compiler driven feature, kind of like a linter, that tracks the "ownership" of variables in a program.

A variable declared is owned by the scope it was declared in, such as a function body. The owner can then "move" the value to another scope or lend the value out for "reading" or "mutation". 

```javascript
const readLog = (read value) => console.log(value)
const writeLog = (write value) => log.push('bar')
const moveLog = (move value) => console.log(value)

// foo is owned by the current scope
let foo = 'foo'

// The owner gives temporary read access to readLog. 
// Read access expires when the readLog function exits.
readLog(read foo)

// The owner gives temporary write access to writeLog. 
// Due to there being no read borrows, this works.
//
// The value is mutated inside writeLog then the write 
// borrow is released when the function exits.
writeLog(write foo)

// The owner transfers ownership of foo to the scope of 
// moveLog. This means it can no longer be accessed from 
// outside the scope of moveLog and the variable will 
// be removed from memory when moveLog exits
moveLog(move foo)
```

A variable can only have one owner and the owner can only lend out either one "mutable" borrow, or many "readable" borrows.

This allows the compiler to know when a variable is at risk of a data race (e.g. multiple scopes writing to it). It also allows the compiler to know when to make memory allocations and a memory deallocations at *compile time*. 

This results in producing a binary that does not need a garbage collector and a ships a negligible runtime.

Rust's idea is pretty clever, nice work Mozilla!

# Audiences

TypeScript BC targets the engineering of high level application development, namely:
- Graphical applications
  - Web applications via web assembly
  - Native mobile applications
  - Native desktop applications
- Web servers
- CLI tools
- Embedded applications

# Justification

The immediate question is _"why not just use Rust?"_. 

Rust is a fantastic language but often you will hear engineers say that it's "too difficult" or that "I am not smart enough to learn Rust". 

It's my opinion that this perceived difficulty comes from the granularity of the data types, syntax format and not the concepts brought in by the borrow checker. 

While the borrow checker certainly has a learning curve, the use of simplified descriptive operators combined with the simplified control flow and opinionated built-in data types; the concept can be more readably described such that it is more accessible.

Additionally, the high granularity of Rust's type system (e.g. the type signature for a callback) is appropriate for situations where any performance loss is unacceptable. When writing a web application (e.g. Reddit), those details are an unnecessary distraction and for a minimal performance cost the use of high level built in types allows for significant productivity and maintainability gains.

TypeScript BC seeks to make the borrow checker concept accessible to engineers who would otherwise skip Rust due to its low level appearance and systems-level focus.

### GUI Applications

Graphical applications written targeting web, desktop and mobile have seen a lot of innovation and attention. From technologies like React/React Native, Electron/Cordova to Flutter, there is an interest in creating client applications that are portable between platforms, are performant and memory efficient.

While TypeScript BC is simply a language and does not describe a framework that manages bindings to the various platform UI APIs - it does allow for a simple, familiar language which addresses the concerns that graphical applications have, making it a great candidate for such a use case.

While its primary focus is targeting the web platform via web assembly, there is no reason we would not be able to see React-Native like platform producing native desktop and native mobile platforms.

#### Multi threading

Effective use of multi-threading is critical for making graphical applications feel responsive. 

As an example, web applications are often criticized for their lack of performance when compared to their native counterparts. Native applications however make maximal use of threads and as a result have interfaces that seldom block.

Including a borrow checker into a familiar language used in web development means that concurrency across threads can be used without fear of data races. This will also allow graphical applications to be used on lower end devices as their resources can be utilized more effectively.

#### Small Binary

As web assembly requires languages to ship their runtime, it's important for languages to ship as little runtime code as possible. 

A borrow checker allows a compiler to compile the code without the need for a garbage collection implementation or a substantial runtime.

This is valuable for web applications however, while not strictly required, efficient small binaries are also appreciated when distributing native applications and web server applications.

### Web Servers:

Web servers require high and consistent IO performance, maintainable code bases and simple distribution.

There are many languages to choose from in this space making the argument for TypeScript BC less compelling on the server side.

There are a few ways that TSBC can be competitive in this context, however.

#### Go-like Concurrency with Rust-like safety

Having an application server written in a high level, familiar and maintainable language which promises the concurrently of a language like Go while also ensuring you cannot write data races would be compelling.

#### Tiny, statically linked binaries

While this is an implementation detail of the compiler, entirely self contained binaries that embed their dependencies (unless explicitly specified) are fantastic from a distribution and security perspective.

It allows engineers to distribute containerized applications based on `scratch` images.

Languages without garbage collection have consistent performance as they avoid locks originating from garbage collection sweeps.

Lastly; small, self contained, memory and performance optimized binaries make for great candidates in PaaS or FaaS contexts (Lambda, App Engine, Heroku, etc). 

This is further made appealing by the fact that it is accessible though the simplistic and ergonomic language of TypeScript.

# Language Design

TypeScript BC is a derivative subset of the existing TypeScript language with the addition of keywords and concepts to enable the use of borrow checking.

TSBC is not compatible with existing TypeScript code or libraries. TypeScript BC aims to co-exist as an independent language that inherits the fantastic type system from TypeScript.

For the most part, you can look at the existing TypeScript language specification and apply the borrow checker additions below to it.

## Hello World

```typescript
import console from '@std/console'

function main() {
  const text: string = 'Hello World'
  console.log(read text)
}
```

We create an immutable reference to a `string` object. We then give read-only access to the `console.log` method, where the value is consumed.

#### Notes
- An application begins execution at the `main` function.
- When `main` exits, it returns a status code `0` as default
- Imports starting with `@std/*` target the standard library
- Using the `read` operator, a variable is lent for reading

<h4>More Details</h3>

Below are pages with additional information on the language specification.

- [Borrow Checker](./specification/borrow-checker.md)
- [Concurrency and Parallelism](./specification/concurrency-and-parallelism.md)
- [Mutex](./specification/mutex.md)
- [Observables](./specification/observables.md)
- [Promises](./specification/promises.md)
- [Structural Types](./specification/structural-types.md)
- [Exceptions](./specification/exceptions.md)
- [Classes and Inheritance](./specification/classes-and-inheritance.md)

# Success Challenges / Quest Log

We know this is successful when these example programs are completed and their objectives are met.

*Note that these examples make change as the specification and standard library specification evolves*

- [#1 - Simple Counter](./quest-log/counter.md)
- [#2 - Simple Web Assembly Application](./quest-log/simple-wasm-application.md)
- [#3 - Simple HTTP Server](./quest-log/simple-wasm-application.md)

# Examples

## Simple HTTP Server

HTTP server that is multi-threaded and the handler function is scheduled on one of the available threads.

```typescript
import http from '@std/http'
import { Mutex } from '@std/sync'

function main() {
  const server = new http.Server()

  server.handle(function(move c: http.Context): void {
    c.response.setStatus(200)
    c.response.setBodyString(value.toString())
    c.response.send()
  })

  server.listen([127, 0, 0, 1], 3000)
}
```

## HTTP Server with State

HTTP server that increments a counter stored in a mutex. On each request the counter value will be incremented and the value sent back in the response.
The HTTP server is multi-threaded and the handler function is scheduled on one of the available threads.

```typescript
import http from '@std/http'
import { Mutex } from '@std/sync'

function main() {
  const server = new http.Server()
  const counter = new Mutex(0)

  server.handle(function(move c: http.Context)[read counter]: void {
    let value = counter.lock()
    value.increment()

    c.response.setStatus(200)
    c.response.setBodyString(value.toString())
    c.response.send()

    counter.unlock()
  })

  server.listen([127, 0, 0, 1], 3000)
}
```





