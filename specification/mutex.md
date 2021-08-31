# Mutex

The standard library provides an implementation for a Mutex that allows for data synchronization

*Note: This is a considerable design challenge and needs more thinking*

```typescript
interface IMutex<T> {
  lock(): T
  unlock(): void
}
```

## Example

```typescript
import { Mutex } from '@std/sync'

function main() {
  let counterValue = 0
  const counterRef = new Mutex<number>(move counterValue)

  increment(read counterRef)
  decrement(read counterRef)
}

function increment(read counterRef: IMutex<number>): void {
  let counterValue = counterRef.lock()
  counterValue = counterValue + 1
  counterRef.unlock()
}

function decrement(read counterRef: IMutex<number>): void {
  let counterValue = counterRef.lock()
  counterValue = counterValue - 1
  counterRef.unlock()
}
```
