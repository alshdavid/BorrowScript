# Structural Types / Duck Typing

I am not sure how to achieve duck typing in a statically compiled language so for now, all objects must be accompanied by a type definition. 

```typescript
// Invalid
const foo = {
  bar: 'a',
}
```

```typescript
// Valid
type Foo = {
  bar: string
}

const foo: Foo = {
  bar: 'a',
}
```