## Counter

```typescript
class Counter {
  private count: number

  constructor() {
    this.count = 0
  }
  
  public write increment(): void {
    this.count++
  }
  
  public read getValue(): number {
    return this.count
  }
}

function main() {
  let c = new Counter()
  console.log(read c.getValue()) // "0"

  c.increment()
  console.log(read c.getValue()) // "1"
}
```