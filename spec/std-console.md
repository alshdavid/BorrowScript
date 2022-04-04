# Standard Library Console

```typescript
import console from '@std/console'
```

## `log`

`console.log()` will consume any object that satisfies its `IString` interface:

```typescript
interface IString {
  toString(): string
}

interface Console {
  log(read ...args: IString[]): void
}
```

Where the usage would be:
```typescript
const foo = 1337

console.log(read foo.toString())  // 'Hello World'
console.log(read foo)  // 'Hello World'

// read can be omitted
console.log(foo.toString())  // 'Hello World'
console.log(foo)  // 'Hello World'
```
