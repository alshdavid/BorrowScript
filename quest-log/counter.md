## Counter

```typescript
class Counter {
  private count: Mut<number> // defaults to 0

  constructor() {
    this.count = 0
  }
  
  public increment(): void {
    this.count++
  }
  
  // read access is implied
  public getValue(): number {
    return this.count
  }
}

function main() {
  let c = new Counter()
  // immutable borrow is implied in console.log()
  console.log(c.getValue()) // "0"

  c.increment()
  console.log(c.getValue()) // "1"
}
```
