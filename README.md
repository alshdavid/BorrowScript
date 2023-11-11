<br>
<img align="left" height="50px" src="./assets/borrow-script.svg">
<br>
<p> &nbsp</p>

<img align="left" height="30px" src="./assets/borrow-check.svg">
<p>Rust inspired Borrow Checker, TypeScript inspired Syntax, Go inspired Concurrency</p>

<img align="left" height="30px" src="./assets/race-conditions.svg">
<p>Compiler Protection from Race Conditions</p>

<img align="left" height="30px" src="./assets/fast.svg">
<p>No Runtime, No Garbage Collection, Memory Safe Guarentee</p>

<br>

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
bsc -i main.bs -o main
```

## Summary

BorrowScript aims to be a language that offers a Rust inspired borrow checker within a TypeScript inspired syntax. 

Basically, "what are the minimum changes required to TypeScript for it to support a borrow checker"

It's hoped that this will make using/learning a borrow checker more accessible while also offering a higher level language well suited to writing user-space applications - like desktop applications, web servers and web applications (through web assembly).

BorrowScript does not expect to match the performance of Rust but it aims to be competitive with languages like Go - offering more consistent performance, smaller binaries and a sensible language to target client programs where multi-threading is often under-utilized, dangerous or inaccesible.

# Language Design

<i>Please contribute your thoughts to the language design!</i>

## Variable Declaration

To declare a variable you can use the keywords `const` and `let`, which describe immutable and mutable bindings.

```typescript
let foo = 'foo' // mutable
const bar = 'bar' // immutable
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
const z: null = null
const a: Array<string> = [] 
const m: Map<string, string> = new Map()
const s: Set<string> = new Set()
```

Nullable types are described as:<br>

```typescript
let foo: string | null = 'foo'
let bar: string | null = null
```

### Mutability

Mutability is defined in the binding and affects the _entire_ value (deeply). 

It's essentially a guarantee that any value assigned to the container will abide by the mutability rules defined on the binding.

Reassignment to another binding will allow values to be changed from mutable/immutable as the binding defines the mutability rules.

```typescript
const foo: string = 'Hello' // immutable string assignment
let bar = foo // move the value from immutable "foo" into mutable "bar"
bar.push(' World')
```

## Function Declarations

Functions can be defined in full or using shorthand lambda expressions

```typescript
function foo() {} // immutable declaration

// Shorthand
const foo = () => {}
let bar = () => {}
```

TODO

How to handle `Fn`, `FnOnce`, etc

## Ownership

The BorrowScript compiler will handle memory allocations and de-allocations at compile time, producing a binary that does not require a runtime garbage collector. This ensures consistent and efficient performance of applications written using BorrowScript.

In order for the compiler to know when a value is ready to be released from memory, it needs some hints from the programmer.

Following in Rust's footsteps, BorrowScript uses an ownership tracker to know when a variable is no longer referenced and it's safe to release it from memory.

A secondary benefit is the compiler can know if a value is at risk of being written to from multiple threads - allowing the compiler to avoid compilation if it detects race conditions.

The "owner" of a variable is its declaration scope:

```typescript
function main() {
  const foo = 'Hello World' // "foo" is owned by "main"

  // <-- at the end of main's block, the value in "foo" is released 
  //     avoiding the need for a garbage collector
}
```

Ownership can be loaned out to another scope as `read` or `write`. There can either be be one scope with `write` access or unlimited scopes with `read` access.

An owner can `move` a variable to another scope and doing so will make that value inaccessible in its original scope.

## Ownership Operators

### `read`/`write`

```typescript
function readFoo(read foo: string) {}
function writeFoo(write foo: string) {}

function main() {
  let foo = "Hello World" // "foo" is owned by main

  readFoo(read foo) // "foo" has 1/infinite read borrow
  // "foo" has 0/infinite read borrow

  writeFoo(write foo) // "foo" has 1/1 write borrow
  // "foo" has 0/1 write borrow

  // <-- "foo" is released 
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
```

### `move`

The `move` operator allows for transferal of a variable's ownership from one scope to another. This literally removes a variable from the current scope and makes it unavailable.

```typescript
function moveFooDefault(foo: string) {} // Defaults to move[const]
function moveFooImmutable(move foo: string) {}
function moveFooMutable(move foo: string) {}

function main() {
  const foo = "Hello World"

  moveFooDefault(move foo)
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
|`move`|<pre lang="typescript">function moveFoo(move foo: string) {&#13;  console.log(read foo)&#13;}</pre>|<pre lang="rust">fn move_foo(foo: String) {&#13;  print!("{}", foo);&#13;}</pre>|


### Ownership Gates

In Rust, callback functions do not automatically have access to the variables in their outer scope. In order to gain access to a variable from within a nested scope (callback function), you must explicitly import variables from the parent scope.

Here is a simple example of accessing outer scope in **TypeScript** (not BorrowScript)

```typescript
const message = 'Hello World'

setTimeout(() => {
  console.log(message)
})
```

In Rust, you have to move a value into the callback scope before using it.

```rust
let message = String::from("Hello World");

set_timeout(move || {
  print!("{}", message);
});
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

## Class Declaration

TODO

```typescript
class Foo {
  public value: number

  // Invoked when class is instantiated
  constructor(
    value: number
  ) {
    this.value = value
  }

  // Invoked when class is dropped from scope
  destructor() {
    console.log('Foo has been deallocated')
  }

  // Available to "let" and "const" declarations
  public read printValue(): void {
    console.log(read this.value)
  }

  // Only available to "let" declarations
  public write updateValue(value: move string): void {
    this.value = value
  }
}

function main() {
  let foo = new Foo(42)

  foo.printValue()
  foo.updateValue(4242)
  foo.printValue()
}
```

## Generic Lifetimes

TODO

Notes:

You can see a fantastic video summary by Bogdan Pshonyak here: [Let's get Rusty - The Rust Survival Guide](https://youtu.be/usJDUSrcwqI?si=rxhD7gEio_8o_qDn&t=602)

I am still struggling to explain what Rust lifetimes are - which I take as an indication that I don't understand them well enough yet. This is my attempt:

Rust tracks the "life time" of variables within Rust to determine when a value should be dropped. The compiler cannot statically analyse the lifetime of variables that are passed into abstractions as reference types (like structs and functions) and so the compiler requires assistance from the programmer to help it understand the life time expectation of those references. This is done through a generic type paramater that starts with a `'` character.

For example a function that returns the largest number of two number references

```rust
fn largest<'a>(x: &'a u32, y: &'a u32) -> &'a u32 {
    if x > y {
        return x;
    }
    return y;
}
```

The `'a` type parameter is placed on both the input values of `x` and `y` connecting their life times. The `'a` type parameter is also on the return type, stating that the life time of the returned value is equal to the shortest lifetime of the associated parameters.

Lifetimes can be circumvented by using reference counters, however this moves the burden of tracking to runtime code execution and is essentially GC.

Trying to imagine a TypeScript-y way to describe lifetime annotations is tricky and I am still working it out. Perhaps we could use intersection types like so: 

```typescript
function largest<A extends lifeof>(x: A & number, y: A & number): A & number {
  if (x > y) {
    return x
  }
  return y
}
```

Or perhaps some weird magic generic like so

```typescript
function largest<A extends lifeof>(A(x): number, A(y): number): A(number) {
  if (x > y) {
    return x
  }
  return y
}
```

## Concurrency

TODO

## Dynamically Sized Types

TODO

Notes: Rust requires the `dyn` keyword when a type is supplied where the size is not known at compile time. An example of this is using a trait as a function parameter. Given everything in BorrowScript is expected to be a dynamically sized type, this might not be necessary.

```rust
trait Foo {
  fn foo()
}

fn bar(f: Box<dyn Foo>) {}
```

## Interior Mutability

TODO

## Smart Pointers, Arc, Rc

TODO

Notes: It might be interesting to explore if it's possible to manage smart pointers at the compiler level; where if a variable is analysed as being used within a multi-threaded context it's transparently wrapped in an Arc otherwise an Rc is used. 

## Mutex

_This will be updated pending changes to concurency_

To manage variables that need to be written to from multiple threads we use a Mutex which holds a state and allows us to lock/unlock access to it, ensuring no one can get the value when it's being used.

```typescript
const counterRef = new Mutex(0)

task.spawn(()[copy counterRef] => {
  let counter = counterRef.unlock()
  counter.increment()
  // <-- counterRef.lock() automatically invoked
})
```

The Rust equivalent would look like:

```rust
fn main() {
  let counterRef = Arc::new(Mutex::new(0));
  let counterRef1 = counterRef.clone();

  thread::spawn(move || {
    thread::sleep(Duration::from_sec(1));
    let mut counter = counterRef1.lock().unwrap();
    counter* = counter* + 1;
  });
}
```

## Error handling

We may explore an `Result` class in the future, but for now I aim to use TypeScript's algebreic type system in combination with compiler aware type elimination to use a simple union `Result` type.

```typescript
type Result<Value, Error> = { value: Value } | { error: Error }
```

The compiler can eliminate possible type signatures through static code analysis and fail to compile in situations where the value could be either option.

```typescript
const result: Result<string. FileReadError> = readFile("/path/to/file.txt")

if ('error' in result) {    // Compiler determines that this if statement can only execute if 
  console.log(result.error) // the error type exists which also means the value option cannot exist

                            // Trying to access result.value here will result in a compiler error

  process.exit(1)           // By exiting here, the compiler knows that the error option 
                            // is not possible beyond this point
}

console.log(result.value)   // By process of elimination, the compiler knows that result.value
                            // is the only possibility at this point
```

The advantage of this is:
1) `Result` `Option` classes could be built on top of this as wrappers
2) You don't have to `.unwrap()` everything, reducing the amount of code and developers must read and write

## Panic

TODO

## Consuming External Libraries (FFI)

TODO

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

## Simple HTTP Server

HTTP server that is multi-threaded and the handler functions are concurrent and possibly multi-threaded.

_The API for the http library has not been finalized, this is an approximation_

```typescript
import { Server, HeaderType, ContentType } from '@std/http'

function main() {
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

function main() {
  const server = new Server()
  const counterRef = new Mutex(0)

  // Increment counter every second
  task.spawn(()[copy counterRef] => {
    while (true) {
      let value = counterRef.lock()
      value.increment()
      sleep(Duration.Second)
    }
  })

  // Send the current value of the counter on the next request
  server.handle((req, res)[copy counterRef] => {
    const value = counterRef.lock()
    res.setHeader(HeaderType.ContentType, ContentType.Text)
    res.send(value)
  })

  server.listen(3000)
}
```
