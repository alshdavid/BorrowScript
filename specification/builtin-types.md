# Built-in Types

One of the objectives of this project is to offer a language that simplifies Rust. One method to do that is to introduce opinionated built-in types that offer dynamic behaviours.

One way to think of this is that these are equivalent:

```typescript
const myString: string = "Hello World"
const myArray: number[] = []
```

To their Rust counterparts:
```rust
let myString: String = String::from("Hello World");
let myArray: Vec<f32> = Vec::new();
```

## The `.toString()` method

In order to represent values using the print function, we must know what their string representation looks like. Any object that implements the `IString` interface is capable of being printed.

All built in types expose a `.toString()` method.

```typescript
interface IString {
  read toString(): string
}
```
On the other side, `console.log()` will consume any object that satisfies the `IString` interface:
```typescript
interface Console {
  log(read ...args: IString[]): void
}
```

Where the usage would be:
```typescript
const foo = 1337

console.log(read foo.toString())  // 'Hello World'
console.log(read foo)  // 'Hello World'
console.log(`${read foo}`)  // 'Hello World'
```

Omitting `read` when the signatures match is in discussion so this could possibly be simplified to:
```typescript
console.log(foo.toString())  // 'Hello World'
console.log(foo)  // 'Hello World'
console.log(`${foo}`)  // 'Hello World'
```


## `string`

The `string` type represents a built-in `String` object. This is a dynamic `char` implementation.

### Details

```typescript
let foo: string = 'Hello World'
let bar = 'Hello World'
```

Variable assignment represents a reference to the respective `string` object in memory. The object can be mutated or reassigned.

```typescript
foo.push('More text')
foo = 'Reassigned value'
```
