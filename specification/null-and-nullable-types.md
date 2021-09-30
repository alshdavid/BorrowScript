# Null and Nullable types

Nullable types will be supported through the use of the built in `null`. 

Unlike JavaScript where there is both `null` and `undefined`, TSBC will combine both into `null`.

`null` represents a built-in object and not a pointer that points to empty memory.

Example:

```typescript
const hasFoo: number | null = 0
const nullFoo: number | null = null

console.log(`hasFool: ${hasFoo != null}`)
console.log(`nullFoo: ${nullFoo != null}`)
```