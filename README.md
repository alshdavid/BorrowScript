<br>
<img align="left" height="50px" src="./assets/borrow-script.svg">
<br>
<p> &nbsp</p>

<img align="left" height="30px" src="./assets/borrow-check.svg">
<p>Rust inspired Borrow Checker, TypeScript inspired Syntax </p>

<img align="left" height="30px" src="./assets/static-binary.svg">
<p>Statically Compiled Binary</p>

<img align="left" height="30px" src="./assets/multi-threaded.svg">
<p>Multi Threaded & Concurrent</p>

<img align="left" height="30px" src="./assets/fast.svg">
<p>Memory Safe Without Garbage Collection</p>

<br>

<i>Please contribute your thoughts to the design of this language specification!</i>

<h4>Hello World</h4>

```typescript
import console from '@std/console'

function main() {
  const text = 'Hello World' // Dynamic string
  console.log(read text) // Handing out a "read" borrow
}
```

[Skip to Language Design](#language-design)

<h4>CLI Usage (expected)</h4>

The CLI will include a compiler, test runner, formater, linter, documentation generation and module management.


```shell
bsc --os linux --arch arm64 -o main main.bs
```

<h4>Table of Contents</h4>

- [Introduction](#introduction)
- [Why not Rust?](#why-not-rust)
- [Borrow Checker tl:dr](#borrow-checker-tldr)
- [Language Design](#language-design)
  - [Builtin Types](#builtin-types)
  - [Ownership Operators](#ownership-operators)
    - [Declaration location](#declaration-location)
    - [`read`](#read)
    - [`write`](#write)
    - [`move`](#move)
    - [`copy`](#copy)
    - [Ownership Gates](#ownership-gates)
  - [Lifetimes](#lifetimes)
  - [Threads](#threads)
  - [Concurrency, `async` & `await`](#concurrency-async--await)
  - [Mutex](#mutex)
  - [Error handling](#error-handling)
- [Audiences](#audiences)
- [Justification](#justification)
  - [GUI Applications](#gui-applications)
    - [Multi threading](#multi-threading)
    - [Small Binary](#small-binary)
  - [Web Servers:](#web-servers)
    - [Performance (speculation)](#performance-speculation)
    - [Go-like concurrency with Rust-like safety](#go-like-concurrency-with-rust-like-safety)
    - [Small, statically linked binaries are good for containers](#small-statically-linked-binaries-are-good-for-containers)
    - [No GC is good for performance consistency](#no-gc-is-good-for-performance-consistency)
    - [Good candidate for PaaS, FaaS](#good-candidate-for-paas-faas)
- [Language Design](#language-design-1)
  - [Hello World](#hello-world)
      - [Notes](#notes)
  - [Lambdas, Type Inference and Shorthand](#lambdas-type-inference-and-shorthand)
    - [Experimental Syntax](#experimental-syntax)
      - [Shorthand ownership operators](#shorthand-ownership-operators)
      - [Omitting matching ownership operators](#omitting-matching-ownership-operators)
      - [Consuming external libraries (Rust, C++)](#consuming-external-libraries-rust-c)
- [Success Challenges / Quest Log](#success-challenges--quest-log)
- [Examples](#examples)
  - [Simple HTTP Server](#simple-http-server)
  - [HTTP Server with State](#http-server-with-state)

# Introduction

This repository describes the specification for a language based on TypeScript that implements a Rust-like borrow checker.

By writing our code in a way that gives the compiler hints as to how we are using our variables; we can have reliable, crazy fast multi-threaded applications.

The objective of this is to have a language that: 

- Is easy to understand
- Has a quick time-to-productive, competitive with languages like Go
- Encourages good software design with a focus on maintainability 
- Produces the smallest binaries possible
- Supports and encourages multi-threading and concurrency


# Why not Rust?

The immediate question is ***"why not just use Rust?"***. 

Rust is a fantastic language but often you will hear engineers say that it's "too difficult", "I am not smart enough to learn Rust", or "the language takes too long to become productive in".

It's my opinion that this perceived difficulty comes from the granularity of the data types, symbol overload, syntax format and **not because the borrow checker is hard to understand**.

While the borrow checker certainly has a learning curve, the use of simplified descriptive operators combined with opinionated built-in data types allows the concept to be more readably described. 

It's the goal of this project is to provide a language that, at the cost of a little performance, is quick to learn and become productive in while also empowering engineers to fearlessly write crazy fast software.

# Borrow Checker tl:dr

*Feel free to skip this if you already understand the borrow checker.*

Rust uses a compiler driven feature, kind of like a linter, that tracks the "ownership" of variables in a program.

A variable is owned by the scope it was declared in. The owner can then "move" the value to another scope or lend the value out for "reading" or "mutation". 

A variable can only have one owner and the owner can only lend out either one "mutable" borrow, or many "readable" borrows.

This allows the compiler to know when a variable is at risk of a data race (e.g. multiple scopes writing to it). It also allows the compiler to know when to make memory allocations and a memory deallocations at *compile time*.

This results in producing a binary that does not need a garbage collector and a ships a negligible runtime.

For example; we have three functions declared:
```javascript
// Asks for read only access to a string
const readLog = (read value: string) => console.log(read value)

// Asks for write access to a string
const writeLog = (write value: string) => log.push('bar')

// Asks for ownership transferal of a string
const moveLog = (move value: string) => console.log(read value)
```

If we declare a variable we can then give it to one of these functions to complete an operation. 

In the following example we create a variable and hand out one read borrow to the `readLog` function.
```typescript
let foo = 'foo'
readLog(read foo)
```
The read borrow given to `readLog` is temporary and is only present when that function is executing. 

It's helpful to imagine a sort of counter state tracking the borrows for `foo`.

|Owner|top level|
|-|-|
|reads|0|
|writes|0|

When given to `readLog`, the number of `reads` ticks up by one while the function executes, then ticks down when the function completes.

If we then hand a `write` borrow to `writeLog`, we will have one write borrow. Remember, only 1 write borrow **or** infinite read borrows.

```typescript
let foo = 'foo'

readLog(read foo)
writeLog(write foo)
```
In the above we tick up one `read`, tick down one `read`, tick up one `write`, tick down one `write`.

We can also change the owner of `foo` to another scope by using `move`.
```typescript
let foo = 'foo'
moveLog(move foo)
```
This now means that anything on the top level scope can no longer use `foo` as its owner is `moveLog`. This also means that the `foo` variable will be dropped from memory when the `moveLog` function completes.

So you can imagine the following code would fail
```typescript
let foo = 'foo'

moveLog(move foo)
readLog(read foo) // Error "tried to use 'foo' when you don't own 'foo'"
```

Tracking a variable's lifecycle like this means no need for a garbage collector.
Rust's idea is pretty clever, nice work Mozilla!

[Great short and simple video by YouCodeThings explaining the concept](https://www.youtube.com/watch?v=8M0QfLUDaaA)

# Language Design

BorrowScript is a derivative subset of the existing TypeScript language with the addition of keywords and concepts to enable the use of borrow checking.

BorrowScript is not compatible with existing TypeScript code or libraries. BorrowScript aims to co-exist as an independent language that inherits the fantastic type system from TypeScript and applies the borrow checker concept from Rust.

For the most part, you can look at the existing TypeScript language specification and apply the borrow checker additions to it.

## Builtin Types

BorrowScript contains opinionated builtin types. Where Rust would use something like:
```rust
let myString: String = String::from("Hello World");
```
BorrowScript uses:
```typescript
const myString: string = "Hello World"
```
All types are references to objects and can be mutated or reassigned if permitted. The types are as follows:
```typescript
const s: string = ""
const n: number = 0
const b: boolean = true
const z: null = null
const a: Array<string> = [] 
const m: Map<string, string> = new Map()
```
Nullable types are described as. Remember that `null` is an object and not a null pointer:
```typescript
let foo: string | null = 'foo'
let bar: string | null = null
```

## Ownership Operators

Following after Rust, variables are owned by the scope they are declared in. Ownership can be loaned out to another scope as `read` or `write`. There can either be be one scope with `write` access or unlimited scopes with `read` access. 

It's practical to visualize each variable with it's own ownership state table.

|Variable|foo|
|-|-|
|Owner|main()|
|reads|0|
|writes|0|

```typescript
function main() {
  const foo = "Hello World"

  readFoo(read foo) // reads++
  // reads--

  writeFoo(write foo) // writes++
  // writes--
}
```

Ownership privilege is retroactive where an owner has `read`/`write`. A scope with `write` has `read`/`write`. A scope with `read` has `read` only. A scope can only lend out to another scope a permission equal or lower than the current held permission.

### Declaration location

Ownership permissions are defined in function or method parameters.

```typescript
function addWorld(write hello: string): void {
  hello.push(' World')
}
```
From there the caller can supply a variable to this function, providing the scope has the correct ownership permissions.

```typescript
function main() {
  let hello = "Hello" // main() is owner with read/write
  const world = "World" // main() is owner with read

  addWorld(write hello) // main() gives a write borrow to addWorld() 
  addWorld(hello) // "write" can be inferred therefore omitted 
}
```

### `read`

The `read` operator prevents mutation deeply for all types, including arrays and maps. The compiler will fail to compile the application if a mutation is attempted when read access is provided.

```typescript
function readFoo(read foo: string) {
  console.log(foo) // "foo"
}
```

Rust equivalent
```rust
fn read_foo(foo: &String) {
  print!("{}", foo);
}
```

### `write`

The `write` operator allows mutation deeply for all types, including arrays and maps. Only one write can be given out at any time. There can be no `read` borrows while there is a `write`.

```typescript
function writeFoo(read foo: string) {
  foo.push('bar')
  console.log(foo) // "foobar"
}
```

Rust equivalent
```rust
fn write_foo(foo: &mut String) {
  foo.push_str("bar");  // Static string declaration
  print!("{}", foo);
}
```

### `move`

The `move` operator allows for transferal of a variable's ownership from one scope to another. This literally removes a variable from the current scope.

Rust has two variations of moving a value into a function. Moving a value as mutable or immutable. 

It's practical to think of the parameter as a variable declaration `let` or `let mut` (`let` or `const`)
```rust
fn move_foo(foo: String) { // let foo
  print!("{}", foo);
}

fn move_foo(mut foo: String) { // let mut foo
  foo.push(String::from("bar")); // Dynamic string declaration
  print!("{}", foo);
}
```

BorrowScript expresses this using the following syntax *(still in discussion)*:

```typescript
function moveFoo(foo: string) { // if omitted, the compiler will assume an immutable move
  console.log(foo)
}

function moveFoo(move<const> foo: string) { // default if omitted 
  console.log(foo)
}

function moveFoo(move<let> foo: string) {
  foo.push('bar')
  console.log(foo)
}
```

### `copy`

This is syntax sugar specifically for BorrowScript. It invokes the `.copy()` method on an object. The use case for this is to simplify the transferal of types through "ownership gates" which we will discuss further below

```typescript
const foo = "foo"
let bar = copy foo // same as foo.copy()
bar.push('bar')
```

### Ownership Gates

In Rust, callback functions do not automatically have access to the variables in their outer scope. In order to gain access to a variable from within a nested scope (callback function), you must explicitly import variables from the parent scope.

Here is a simple example of this in TypeScript

```typescript
const message = 'Hello World'

setTimeout(() => {
  console.log(message)
})
```

In Rust, you have to move a value into the callback scope before using it.

```rust
let message = String::from("Hello World");

set_timeout(&(move || {
  print!("{}", message);
}));
```
In BorrowScript we describe imports from the parent scope of a callback using ownership gates which are declared as square brackets after the function parameters:

```typescript
const message = "Hello World"

setTimeout(()[move message] => { // "move" can be omitted
  console.log(message)
})
```

The complete syntax looks like:
```typescript
function name<T>(params)[gate]: void { /* code */ }
```

You can also copy a value into a child scope using ownership gates. Using copy will create a shadow variable with the same name within the child scope.

```typescript
setTimeout(()[copy message] => { /* code */ })
```
Which is equivalent to:
```typescript
const message = "Hello World"

const messageCopy = copy message
setTimeout(()[move messageCopy] => {
  const message = messageCopy
  console.log(message)
})
```

## Lifetimes

Lifetimes are parameters described using the `lifeof` type operator within a generic definition.

*This syntax needs a bit of work probably*

```typescript
function longestNumber<lifeof A>(A<read> x: number, A<read> y: number): A<number> {
  // return the longer number
}
```
What this essentially tells the compiler is to only work when both the `x` and `y` variables share the same lifespan. If one drops out of scope before the other (clearing it from memory) then the life times of the variables are not the same.

## Threads

Threads will be available from the standard library via:

```typescript
import { Thread } from '@std/threads'

function main() {
  const thread1 = new Thread()

  thread1.exec(() => {
    console.log('Some work')
  })

  thread1.waitForIdle()
}
```

## Concurrency, `async` & `await`

Concurrency is managed by `Promises` (which are basically analogous to Rust's `Future`s). Control flow is managed using `async` `await`. Like JavaScript, functions are executed in an event loop. Unlike JavaScript, these functions may execute on any one of the allocated threads (like Go).

*This is still in design*

I am experimenting with the idea of having the queue its own object for cases where you want the main thread dedicated to something like UI or IO and the rest of the threads dedicated to work.

```typescript
import { Thread } from '@std/thread'
import { Queue } from '@std/queue'

async function main() {
  const thread1 = new Thread()
  const thread2 = new Thread()

  const queue = new Queue([thread1, thread2])

  queue.task(() => console.log('Hello'))
  queue.task(() => console.log('World'))

  await queue.allSettled()
}
```

Where potentially we use a decorator to use an all-thread concurrency setup

```typescript
import { DefaultQueue, queue } from '@std/queue'

@DefaultQueue()
async function main() {
  queue.task(() => console.log('Hello'))
  queue.task(() => console.log('World'))

  await queue.allSettled()
}
```

## Mutex

To manage variables that need to be written to from multiple threads we use a Mutex which holds a state and allows us to lock/unlock access to it, ensuring no one can get the value when it's being used.

```typescript
import { Mutex } from '@std/sync'

async function main() {
  const counterRef = new Mutex(0)

  setInterval(()[copy counterRef] => {
    let counter = counterRef.unlock()
    counter.increment()
  }, 1000)
}
```

The Rust equivalent same thing in Rust would look like:

```rust
fn main() {
  let counterRef = Arc::new(Mutex::new(0));
  let counterRef1 = counterRef.clone();

  set_timeout(&(move || {
    let mut counter = counterRef1.lock().unwrap();
    counter* = counter* + 1;
  }), 1000);
}
```

## Error handling

At this stage, errors will be return values from tuples

```typescript
const [ value, error ] = parseInt("Not a number")
if (error != null) {
  // handle
}
```

# Audiences

BorrowScript targets the engineering of high level application development, namely:
- Graphical applications
  - Web applications via web assembly
  - Native mobile applications
  - Native desktop applications
- Web servers
- CLI tools
- Embedded applications

# Justification

## GUI Applications

Graphical applications written targeting web, desktop and mobile have seen a lot of innovation and attention. From technologies like React/React Native, Electron/Cordova to Flutter, there is an interest in creating client applications that are portable between platforms, are performant and memory efficient.

While BorrowScript is simply a language and does not describe a framework that manages bindings to the various platform UI APIs - it does allow for a simple, familiar language which addresses the concerns that graphical applications have, making it a great candidate for such a use case.

While its primary focus is targeting the web platform via web assembly, there is no reason we would not be able to see React-Native like platform producing native desktop and native mobile platforms.

### Multi threading

Effective use of multi-threading is critical for making graphical applications feel responsive. 

As an example, web applications are often criticized for their lack of performance when compared to their native counterparts. Native applications however make maximal use of threads and as a result have interfaces that seldom block.

Including a borrow checker into a familiar language used in web development means that concurrency across threads can be used without fear of data races. This will also allow graphical applications to be used on lower end devices as their resources can be utilized more effectively.

### Small Binary

As web assembly requires languages to ship their runtime, it's important for languages to ship as little runtime code as possible. 

A borrow checker allows a compiler to compile the code without the need for a garbage collection implementation or a substantial runtime.

This is valuable for web applications however, while not strictly required, efficient small binaries are also appreciated when distributing native applications and web server applications.

## Web Servers:

Web servers require high and consistent IO performance, maintainable code bases and simple distribution.

There are many languages to choose from in this space making the argument for BorrowScript less compelling on the server side.

There are a few ways that BorrowScript can be competitive in this context, however.

### Performance (speculation)

Rust performance is often compared to C++ and out performs languages like Go and Java. 

While it's too early to guarantee performance, we know that TSCB is a slightly higher level Rust, where the types are replaced with built-ins. 

I would like to see its performance sit between Rust and Go.

### Go-like concurrency with Rust-like safety

Having an application server written in a high level, familiar and maintainable language that promises the concurrently of a language like Go while also ensuring you cannot write data races might be compelling for back-end application developers.

### Small, statically linked binaries are good for containers

While this is an implementation detail of the compiler, entirely self contained binaries that embed their dependencies (unless explicitly specified) are fantastic from a distribution and security perspective.

It allows engineers to distribute containerized applications based on `scratch` images.

### No GC is good for performance consistency

Languages without garbage collection have consistent performance as they avoid locks originating from garbage collection sweeps.

This is especially noticeable in applications with lots of activity - such as chat servers.

### Good candidate for PaaS, FaaS

Lastly; small, self contained, memory and performance optimized binaries make for great candidates in PaaS or FaaS contexts (Lambda, App Engine, Heroku, etc). 

# Language Design

BorrowScript is a derivative subset of the existing TypeScript language with the addition of keywords and concepts to enable the use of borrow checking.

BorrowScript is not compatible with existing TypeScript code or libraries. BorrowScript aims to co-exist as an independent language that inherits the fantastic type system from TypeScript and applies the borrow checker concept from Rust.

For the most part, you can look at the existing TypeScript language specification and apply the borrow checker additions to it.

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

## Lambdas, Type Inference and Shorthand 

In the example code I am using the long hand version of everything. I am including type signatures as well as function definitions.

The language will support lambda functions and TypeScript type inference so it won't be necessary to write the complete type signatures for everything.

### Experimental Syntax

#### Shorthand ownership operators

I am exploring the idea of using shorthand ownership operators and sensible defaults namely;

`move` is the default if omitted. `write` will also accept `w`, `read` or `r`, `copy` or `c`.

Both would work:
```typescript
function foo(read bar: number) { }
function foo(r bar: number) { }
```

#### Omitting matching ownership operators

I am exploring the idea of omitting ownership operators on method parameters when called if the incoming value has matching ownership parameters.

For example the signature for `console.log` describes that it accepts values provided with read access that contain the `toString()` method:

```typescript
interface Stringable {
  read toString(): string
}

console.log(read ...any Stringable[])
```
All built-in types have a `toString` method that only requires `read` access to use.
```typescript
const foo = 1337

console.log(read foo)
console.log(foo) // perhaps this will be fine
```

#### Consuming external libraries (Rust, C++)

I would like to introduce a means to consume external Rust and C++ libraries. The how of this is still in discussion. 

TODO

<h4>More Details</h3>

Below are pages with additional information on the language specification.

- [Borrow Checker](./specification/borrow-checker.md)
- [Built-in Types](./specification/builtin-types.md)
- [Null and Nullable Types](./specification/null-and-nullable-types.md)
- [Concurrency and Parallelism](./specification/concurrency-and-parallelism.md)
- [Mutex](./specification/mutex.md)
- [Observables](./specification/observables.md)
- [Promises](./specification/promises.md)
- [Structural Types](./specification/structural-types.md)
- [Exceptions](./specification/exceptions.md)
- [Classes and Inheritance](./specification/classes-and-inheritance.md)

# Success Challenges / Quest Log

We know this is successful when these example programs are completed and their objectives are met.

*Note that these examples may change as the specification and standard library specification evolves*

- [#1 - Simple Counter](./quest-log/counter.md)
- [#2 - Simple Web Assembly Application](./quest-log/simple-wasm-application.md)
- [#3 - Simple HTTP Server](./quest-log/simple-http-server.md)

# Examples

## Simple HTTP Server

HTTP server that is multi-threaded and the handler function is scheduled on one of the available threads.

_The API for the http library has not been finalized, this is an aproximation_

```typescript
import { Server } from '@std/http'

async function main() {
  const server = new Server()

  server.get('/', (req, res) => {
    res.setBodyString('Hello World')
    res.send()
  })

  await server.listen([127, 0, 0, 1], 3000)
}
```

## HTTP Server with State

HTTP server that increments a counter stored in a mutex. On each request the counter value will be incremented and the value sent back in the response.
The HTTP server is multi-threaded and the handler function is scheduled on one of the available threads.

_This is dependant on the design decision describing how ownership of values is passed into nested closures and not final_

```typescript
import { Server } from '@std/http'
import { Mutex } from '@std/sync'

async function main() {
  const server = new Server()
  const counterRef = new Mutex(0)

  server.get('/', (req, res)[copy counterRef] => {
    let value = counterRef.lock()
    value.increment()
    res.send()
  })

  await server.listen([127, 0, 0, 1], 3000)
}
```





