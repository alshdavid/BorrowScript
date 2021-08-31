## Counter

```typescript
class Counter {
  private _count: number

  constructor() {
    this._count = 0
  }
  
  public write increment(): void {
    this._count++
  }
  
  public read getValue(): number {
    return this._count
  }
}
```