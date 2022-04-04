# `string`

The `string` type represents a built-in `String` object. This is a dynamic `char` implementation.

### Details

```typescript
let s0: string = 'Hello World'
let s1 = 'Hello World'
let s2 = new String('Hello World')
```

### Interface

```typescript
abstract class String {
  new(chars: any): this // There will be no char representation, however this could be a byte array
  toString(): String
  push(value: String)
}
```
