<br>
<img align="left" height="50px" src="./assets/borrow-script.svg">
<br>
<p> &nbsp</p>

<img align="left" height="30px" src="./assets/borrow-check.svg">
<p>TypeScript Syntax, Rust Borrow Checker, Go Philosophies</p>

<img align="left" height="30px" src="./assets/race-conditions.svg">
<p>High Performance, Fearless Concurrency, Compile Time Checks</p>

<img align="left" height="30px" src="./assets/fast.svg">
<p>No Garbage Collection, Memory Safety Guarantee</p>

<br>

## Hello World

```typescript
import console from '@std/console'

function main() {
  const text = 'Hello World'
  console.log(read text)
}
```

## CLI Usage

```shell
$ bsc main.bs
$ ./main
> "Hello World"
```

## Summary

BorrowScript aims to be a language that offers a TypeScript inspired syntax with concepts borrowed from Rust and Go. 

Basically; "what are the minimum changes required to TypeScript for it to support a borrow checker"

It's hoped that this will make using/learning a borrow checker more accessible while also offering a higher level language well suited to writing user-space applications - like desktop applications, web servers and web applications (through web assembly).

BorrowScript does not expect to match the performance of Rust but it aims to be competitive with languages like Go - offering more consistent performance, smaller binaries and a sensible language to target client programs where multi-threading is often under-utilized, dangerous or inaccessible.

# Language Design

<i>Please contribute your thoughts to the language design!</i>

## Variable Declaration

To declare a variable you can use the keywords `const` and `let`, which describe immutable and mutable bindings.

```typescript
let foo = 'foo'   // mutable
const bar = 'bar' // immutable
```

## Types

BorrowScript contains opinionated built in types:

```typescript
const myString: string = "Hello World"
const myString = "Hello World" // type inference
```

Where Rust would have multiple types for different use cases:

```rust
let myString: String = String::from("Hello World");
let myString2: &str = "Hello World"
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

## Enums

BorrowScript features Rust-inspired enum types and match statements

```typescript
enum Foobar {
  Foo,
  Bar
}
```

### Mutability

Mutability is defined in the binding and affects the _entire_ value (deeply, unlike TypeScript). 

It's essentially a guarantee that any value assigned to the container will abide by the mutability rules defined on the binding.

Reassignment to another binding will allow values to be changed from mutable/immutable as the binding defines the mutability rules.

```typescript
const foo: string = 'Hello' // immutable string assignment
let bar = foo               // move the value from immutable "foo" into mutable "bar"
                            // "foo" become inaccessible after it has been moved, "bar" can be used
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

## Ownership

The BorrowScript compiler will handle memory allocations and de-allocations at compile time, producing a binary that does not require a runtime garbage collector. 

This is done through the automatic de-allocation of values when they fall out of their owned scope.

```typescript
function main() {
  const foo = 'Hello World' // "foo" is allocated and owned by "main"

  // <-- at the end of main's block, "foo" is de-allocated
  //     avoiding the need for a garbage collector
}
```

### Borrowing 

Ownership can be temporarily loaned out to another scope with `read` or `write` permissions.

```typescript
function readFoo(read foo: string) {}

function main() {
  let foo = "Hello World" // "foo" is owned by main
  readFoo(read foo)       // "foo" is lent to "readFoo" with "read" permission
  // <---------------------- "foo" is still owned by "main" and is de-allocated when "main" completes
}
```

There can only be one owner of the value and either one scope with `write` access or unlimited scopes with `read` access.

```typescript
function main() {
  let foo = "Hello World" // "foo" is owned by main
  readFoo(read foo)       // "foo" loaned to "readFoo" with 1 of infinite read borrows
  // <---------------------- "readFoo" completes decrementing the read borrow to 0 of infinite read borrows
  writeFoo(write foo)     // "foo" loaned to "writeFoo" with 1 of 1 write borrows
  // <---------------------- "writeFoo" completes decrementing the write borrow to 0 of 1 write borrows
  // <---------------------- "foo" is owned by "main" and is de-allocated when "main" completes
}
```

An owner can `move` a variable to another scope and doing so will make that value inaccessible in its original scope.

### Ownership Operators `read` `write` `move` `copy`


```typescript
function moveFoo(let foo: string) { // "foo" is moved into "moveFoo" which consumes it as mutable
                                    // if "let" is omitted, the moved value assumes "const"
  readFoo(read foo)
  writeFoo(write foo)
  // <---------------------- "foo" is owned by "moveFoo" and is de-allocated when "moveFoo" completes
}

function main() {
  let foo = "Hello World" // "foo" is owned by main
  readFoo(read foo)       // "foo" loaned to "readFoo" with 1 of infinite read borrows
  // <---------------------- "readFoo" completes decrementing the read borrow to 0 of infinite read borrows
  moveFoo(foo)            // "foo" moved into "moveFoo"
  // <---------------------- "foo" is no longer available in "main"
  // console.log(foo)     // Attempts to access "foo" in this scope will fail after it has been moved
}
```

A scope with `write` has `read`/`write`. <br>
A scope with `read` has `read` only. <br>
A scope can only lend out to another scope a permission equal or lower than the current held permission.

A value can be copied, creating a new owned value

```typescript
const foo = "foo"
let bar = copy foo        // same as foo.copy()
bar.push('bar')
```

```typescript
function main() {
  let foo = "Hello World" // "foo" is owned by main
  moveFoo(copy foo)       // a new copy of "foo" is moved into "moveFoo"
  console.log(foo)        // "foo" can still be accessed from this scope
}
```

## Rust Examples of Ownership Operators

<table>
<tr><th>Operator</th><th>BorrowScript</th><th>Rust</th></tr>

<tr><td>Read</td><td>

```typescript
function readFoo(read foo: string) {
  console.log(foo)
}
```
</td><td>

```rust
fn read_foo(foo: &String) {
  print!("{}", foo);
}
```

</td></tr>
<tr><td>Write</td><td>

```typescript
function writeFoo(write foo: string) {
  foo.push("bar")
}
```
</td><td>

```rust
fn write_foo(foo: &mut String) {
  foo.push_str("bar")
}
```

</td></tr>
<tr><td>Move (mutable)</td><td>

```typescript
function moveMutFoo(let foo: string) {
  foo.push("bar")
}
```
</td><td>

```rust
fn move_mut_foo(mut foo: String) {
  foo.push_str("bar")
}
```

</td></tr>
<tr><td>Move (immutable)</td><td>

```typescript
function moveFoo(foo: string) {
  console.log(foo)
}
```
</td><td>

```rust
fn move_foo(foo: String) {
  print!("{}", foo);
}
```

</td></tr>
</table>

### Closures / Callbacks

Callbacks in BorrowScript don't automatically have access to variables in their outer scope. In order for a callback to gain access to a variable from an outer scope, it must be explicitly imported from its parent scope.

This is done using "gate" parameters within square brackets.

```typescript
const message = 'Hello World'

setTimeout([message]() => {
  console.log(message)
}, 0)
```

This is required because the compiler must move the value from the parent scope and into the nested callback scope

Once moved, the original value is no longer accessible to the outer scope

```typescript
const message = 'Hello World'

setTimeout([message]() => {
  console.log(message)
}, 0)

// console.log(message)               <- "message" has been moved and can no longer be accessed here
```

This enables the principle of "fearless concurrency" - making race conditions in multi-threaded contexts impossible

```typescript
import thread from "std:thread"

function main() {
  let message = 'Hello World'

  thread.spawn([copy message]() => { // Create a new OS thread and copy "message" into that scope
    console.log(message)
  })

  console.log(message)               // "message" is still available in "main"

  thread.spawn([message]() => {      // Create a new OS thread and move "message" into that scope 
    console.log(message)
  })
}
```

## Multiple Owners & Multiple Mutable References

Unless I can find a way to infer and automatically apply smart pointers and mutexes to shared references, they will need to be explicitly defined.

```typescript
import { Error } from 'std:error'
import thread, { Handle } from 'std:thread'
import { Mutex, Arc } from 'std:sync'

function main(): Result<void, Error> {
  const count = Arc.new(Mutex.new(0))
  let handles: Array<Handle> = []

  for (const i in 0..10) {  
    handles.push(thread.spawn([copy message]() => { // Spawn a thread and copy a reference to the mutex + value 
      let count = count.lock()                      // Unlock the mutex and assign it to a mutable container
      count++                                       // Increment the count ("count" is a "&mut i32")
    }))
  }
  
  for (const handle in handles) handle.join()?      // Wait for threads to complete, propagate the error if a thread failes
                
  console.log(message.lock())                       // Unlock the mutex and print the inner value 
                                                    // Prints "10"
}
```

## Class Declaration

TODO

BorrowScript will have FFI capabilities similar to Rust and will need to have structs to facilitate that.

I personally like classes as a means to visually group methods with an object but if there are classes they will not be extendable.

My preference is composition over inheritance and appreciate Go's ability to embed structs within other structs.

At this stage, my feeling is that classes will not be part of the language unless I can find a way to make them fit naturally within the language

## Structural Types and Type Kung-Fu

TODO: depends on struct syntax

I love the ability in TypeScript to accept values by their shape and want this to be a part of BorrowScript

```typescript
interface IFoo {
  foo(read self): void
}

function acceptFoo(read foo: IFoo) {}        // Function that accepts a type that looks like the IFoo interface

struct Foo {}

impl Foo {
  foo(read self): void {}
}

struct AlsoFoo {}

impl AlsoFoo {
  foo(read self): void {}
}

function main() {
  const foo1 = Foo{}
  const foo1 = AlsoFoo{}

  acceptFoo(read foo1)
  acceptFoo(read foo2)
}
```

In Rust, this is expressed using the `dyn` keyword - however there are cases where the unknown size of a value request a `Box<dyn T>`. 

There are also considerations on if/how interfaces need to be described as satisfied. Rust uses its `trait` system to describe this but this couples the struct to the type it satisfies.

Go can accept structs as interfaces, implicitly inferring that the struct satisfies the type from its shape - however this is only one level deep (meaning interfaces that have methods that return interfaces don't work).

### Type Kung Fu (Algebraic Types)

TODO

I have seen this topic to be misunderstood and polarizing but ultimately it's simply the ability to unify and allow the compiler to discriminate types structurally

```typescript
function main() {
  const stringOrNumber: string | number = 0

  match (typeof stringOrNumber) {
    number(value => console.log(value))
    string(value => console.log(value))
  }
}
```

Personally I quite like this as it can be used to compose types together

```typescript
type Foo = { foo: string }
type Bar = { bar: number }
type Foobar = Foo & Bar
```

And creating new types using TypeScript's `keyof` `in` `extends` keywords can be quite ergonomic.

It is quite difficult to implement though and will need more thinking. I'd like to do as much in the compiler as possible and this may require that union variables live inside a container with that type metadata included.


## Generic Lifetimes

TODO

Notes:

You can see a fantastic video summary by Bogdan Pshonyak here: [Let's get Rusty - The Rust Survival Guide](https://youtu.be/usJDUSrcwqI?si=rxhD7gEio_8o_qDn&t=602)

I haven't thought of a way to apply this to BorrowScript without reaching for Rust's `'` generic type parameter.


## Async, Concurrency, Threads

For BorrowScript to work in many different environments (wasm, napi, iot), it needs to give the developer control of how concurrency works.

There will be the capability to create system threads using OS APIs

```typescript
import thread from 'std:thread'

function main() {
  thread
    .spawn(() => {
      console.log('hello world')
    })
    .join()
    .panicOnError()
}
```

Async will be supported using an async runtime selected by the developer, where the runtimes will be provided by the standard library

```typescript
import asynch from 'std:asynch'

function main() {
  let rt = asynch
    .runtime({ workers: 4 })
    .blockOn(async () => {
      console.log('hello world')
    })
    .panicOnError()
}
```

And simplified with the use of compiler macros

```typescript
import asynch from 'std:asynch'

@asynch.main!()
async function main() {
  console.log('Hello World')
}
```

Where wasm, napi, etc can select the runtime which makes sense for them

```typescript
import asynchWasm from 'std:asynch/wasm'
import asynch from 'std:asynch'

@asynch.main!(asynchWasm.default)
async function main() {
  console.log('Hello World')
}
```

## Error handling

Will use Rust-style `Result` enum and pattern matching and the `?` operator to propagate errors upwards

## Consuming External Libraries (FFI)

TODO

tl;dr yes
