<br>
<img align="left" height="50px" src="./assets/borrow-script.svg">
<br>
<p> &nbsp</p>

<img align="left" height="30px" src="./assets/borrow-check.svg">
<p>Rust inspired Borrow Checker, TypeScript inspired Syntax, Go inspired Concurrency</p>

<img align="left" height="30px" src="./assets/static-binary.svg">
<p>Statically Compiled Binary</p>

<img align="left" height="30px" src="./assets/multi-threaded.svg">
<p>Multi Threaded & Concurrent</p>

<img align="left" height="30px" src="./assets/race-conditions.svg">
<p>Compiler Protection from Race Conditions</p>

<img align="left" height="30px" src="./assets/fast.svg">
<p>Memory Safe without Garbage Collection</p>

<br>

<i>Please contribute your thoughts to the design of this language specification!</i>

## Hello World

```typescript
import console from '@std/console'

function main() {
  const text = 'Hello World' // Dynamic string
  console.log(read text) // Handing out a "read" borrow
}
```

## CLI Usage

```shell
bsc --os linux --arch arm64 -o main main.bs
```

## Summary

BorrowScript aims to be a language that offers a Rust inspired borrow checker with a relatively simplified syntax.

It aims to do this by offering higher level builtin types, builtin concurrency and more explanatory keywords for the borrow checker. 

It's hoped that this will make using/learning a borrow checker more accessible while also offering a higher level language well suited to writing applications like desktop applications, web servers and web applications (through web assembly).

BorrowScript does not expected to match the performance of Rust, but it aims to be competitive with languages like Go - offering more consist performance, smaller binaries and a sensible language to target client programs.

# Language Design

## Variable Declaration

To declare a variable you can use the keywords `const` and `let`, which describe immutable and mutable bindings.

```typescript
let foo = 'foo' // mutable
const bar = 'bar' // immutable
```

## Function Declarations

Functions can be defined in full or using shorthand lambda expressions

```typescript
function foo() {} // immutable declaration

// Shorthand
const foo = () => {}
let bar = () => {}
```

## Types

BorrowScript contains opinionated builtin types. Where Rust would use something like:

```rust
let myString: String = String::from("Hello World");
```

BorrowScript uses:

```typescript
const myString: string = "Hello World"
const myString = "Hello World" // type inference
```

All types are references to objects and can be mutated or reassigned if permitted. The types are as follows:

```typescript
const s: string = ""
const n: number = 0
const b: boolean = true
const z: null = null // null is an object and not a null pointer
const a: Array<string> = [] 
const m: Map<string, string> = new Map()
const s: Set<string> = new Set()
```

Nullable types are described as:<br>

```typescript
let foo: string | null = 'foo'
let bar: string | null = null
```

Mutability is defined in the binding and affects the _entire_ value (deeply). 

It's essentially a guarantee that any value assigned to the container will abide by the mutability rules defined on the binding.

Reassignment to another binding will allow values to be changed from mutable/immutable as the binding defines the mutability rules.

```typescript
const foo: string = 'Hello' // immutable string assignment
let bar = foo // move the value from immutable "foo" into mutable "bar"
bar.push(' World')

// console.log(foo) <-- Not possible because foo has been moved into bar and thus ownership transferred
```

## Ownership

Following after Rust, variables are owned by the scope they are declared in. 

```typescript
function main() {
  const foo = 'Hello World' // "foo" is owned by main
  // <-- at the end of main's block, the value in "foo" is released 
}
```

## Ownership Operators

### `read`/`write`

Ownership can be loaned out to another scope as `read` or `write`. There can either be be one scope with `write` access or unlimited scopes with `read` access. 

```typescript
function readFoo(read foo: string) {}
function writeFoo(write foo: string) {}

function main() {
  let foo = "Hello World" // "foo" is owned by main

  readFoo(read foo) // "foo" has 1/infinite read borrow
  // "foo" has 0/infinite read borrow

  writeFoo(write foo) // "foo" has 1/1 read borrow
  // "foo" has 0/1 read borrow

  // <-- at the end of main's block, the value in "foo" is released 
}
```

A scope with `write` has `read`/`write`. <br>
A scope with `read` has `read` only. <br>
A scope can only lend out to another scope a permission equal or lower than the current held permission.

*Note that the ownership operator can be omitted when passing a value into a function*

```typescript
function readFoo(read foo: string) {}

const foo = "Hello World"

readFoo(read foo)
readFoo(foo)  // Infer "read" from function's call signature
```

### `move`

The `move` operator allows for transferal of a variable's ownership from one scope to another. This literally removes a variable from the current scope and makes it unavailable.

```typescript
function moveFooDefault(foo: string) {} // Defaults to move[const]
function moveFooImmutable(move[const] foo: string) {}
function moveFooMutable(move[let] foo: string) {}

function main() {
  const foo = "Hello World"

  moveFooDefault(move foo)
  // moveFooDefault(foo) can be omitted
}
```

### `copy`

This is syntax sugar specifically for BorrowScript. It invokes the `.copy()` method on an object.

The use case for this is to simplify the transferal of types through "ownership gates" which we will discuss further below

```typescript
const foo = "foo"
let bar = copy foo // same as foo.copy()
bar.push('bar')
```

## Rust Examples of Ownership Operators

|Operator|BorrowScript|Rust|
|-|-|-|
|`read`|<pre lang="typescript">function readFoo(read foo: string) {&#13;  console.log(foo)&#13;}</pre>|<pre lang="rust">fn read_foo(foo: &String) {&#13;  print!("{}", foo);&#13;}</pre>|
|`write`|<pre lang="typescript">function writeFoo(write foo: string) {&#13;  foo.push('bar')&#13;  console.log(foo)&#13;}</pre>|<pre lang="rust">fn write_foo(foo: &mut String) {&#13;  foo.push_str("bar");&#13;  print!("{}", foo);&#13;}</pre>|
|`move[const]`|<pre lang="typescript">function moveFoo(move[const] foo: string) {&#13;  console.log(foo)&#13;}&#13;&#13;function moveFoo(foo: string) {&#13;  console.log(foo)&#13;}</pre>|<pre lang="rust">fn move_foo(foo: String) {&#13;  print!("{}", foo);&#13;}</pre>|
|`move[let]`|<pre lang="typescript">function moveFoo(move[let] foo: string) {&#13;  foo.push('bar')&#13;  console.log(foo)&#13;}</pre>|<pre lang="rust">fn move_foo(mut foo: String) {&#13;  foo.push(String::from("bar"));&#13;  print!("{}", foo);&#13;}</pre>|


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

setTimeout(()[move message] => { // "move" can be omitted as it's the default action
  console.log(message)
})
```

The complete syntax looks like:
```typescript
function name<T>(params)[gate]: void { }
```

You can also copy a value into a child scope using ownership gates. Using copy will create a shadow variable with the same name within the child scope.

This is used for moving mutexes into concurrent contexts.

```typescript
setTimeout(()[copy message] => { })
```
Which is equivalent to:
```typescript
const message = "Hello World"

const messageCopy = message.copy()
setTimeout(()[move messageCopy] => {
  const message = messageCopy
  console.log(message)
})
```

## Lifetimes

Lifetimes are parameters described using the `lifeof` type operator within a generic definition.

*This syntax needs a bit of work probably*

```typescript
function longestNumber<lifeof A>(read[A] x: number, read[A] y: number): A<number> {
  // return the longer number
}
```
What this essentially tells the compiler is to only work when both the `x` and `y` variables share the same lifespan. If one drops out of scope before the other (clearing it from memory) then the life times of the variables are not the same.

## Concurrency

BorrowScript will use Go-like co-routines to manage task queues across multiple processors. The control flow of these concurrent tasks will be managed with the concept of a `Channel` 

_Note, BorrowScript uses `async`/`yield` differently to JavaScript. Think of `async` like the `go` keyword in the Go programming language_

### Starting a concurrent task with `async`

A function is invoked concurrently by prefixing it with `async`

```typescript
function concurrentFunc() {
  console.log('Hello World')
}

async concurrentFunc
```

You can do this with functions defined inline
```typescript
async () => {
  console.log('Hello World')
}
```

### Channels and `yield`

A channel will `yield` a single value or values until it's closed. 

Calling `yield` on a channel will provide the first value emitted on it.


```typescript
const channel = new Channel<string>()

async ()[copy channel] => {
  const message = yield channel
  console.log(message) // 'Hello'
}

channel.emit('Hello')
```

Using a channel in a loop will loop over each emitted event until the channel is closed or the loop broken.

```typescript
const channel = new Channel<string>()

async ()[copy channel] => {
  for (const message of channel) {
    console.log(message) // 'Hello', 'World'
  }
}

channel.emit('Hello')
channel.emit('World')
```

## Mutex

To manage variables that need to be written to from multiple threads we use a Mutex which holds a state and allows us to lock/unlock access to it, ensuring no one can get the value when it's being used.

```typescript
const counterRef = new Mutex(0)

async ()[copy counterRef] => {
  let counter = counterRef.unlock()
  counter.increment()
  // <-- counterRef.lock() automatically invoked
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
const [ value, error ] = Number.fromString("Not a number")
if (error != null) {
  // handle
}
```

#### Consuming external libraries

Would be done similarly to [Deno's FFI implementation](https://deno.land/manual/runtime/ffi_api)

# Application Examples

## Hello World

```typescript
import console from '@std/console'

function main() {
  const text: string = 'Hello World'
  console.log(text)
}
```

We create an immutable reference to a `string` object. We then give read-only access to the `console.log` method, where the value is consumed.

#### Notes
- An application begins execution at the `main` function.
- When `main` exits, it returns a status code `0` as default
- Imports starting with `@std/*` target the standard library
- Using the `read` operator, a variable is lent for reading

## Simple HTTP Server

HTTP server that is multi-threaded and the handler functions are concurrent and possibly multi-threaded.

_The API for the http library has not been finalized, this is an approximation_

```typescript
import { Server, HeaderType, ContentType } from '@std/http'

async function main() {
  const server = new Server()

  server.handle((req, res) => {
    res.setHeader(HeaderType.ContentType, ContentType.Text)
    res.send('Hello World')
  })

  server.listen(3000)
}
```

## HTTP Server with State

HTTP server that increments a counter stored in a mutex. On each request the counter value will be incremented and the value sent back in the response.
The HTTP server is multi-threaded and the handler function is scheduled on one of the available threads.

_This is dependant on the design decision describing how ownership of values is passed into nested closures and not final_

```typescript
import { Server, HeaderType, ContentType } from '@std/http'
import { Mutex } from '@std/sync'
import { sleep, Duration } from '@std/time'

async function main() {
  const server = new Server()
  const counterRef = new Mutex(0)

  // Increment counter every second
  async ()[copy counterRef] => {
    while (true) {
      let value = counterRef.lock()
      value.increment()
      sleep(Duration.Second)
    }
  }

  // Send the current value of the counter on the next request
  server.handle((req, res)[copy counterRef] => {
    const value = counterRef.lock()
    res.setHeader(HeaderType.ContentType, ContentType.Text)
    res.send(value)
  })

  server.listen(3000)
}
```
