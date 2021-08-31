# Observables

For a stream of events, the is the `IObservable` interface.

```typescript
interface IObservable<T> {
  subscribe(onNext: (
    value: T,
    error: unknown,
    isComplete: boolean,
  ) => any): () => void
}
```

## Example

```typescript
import { thread0 } from '@std/process'
import { Queue, Observable } from '@std/sync'
import { second } from '@std/time'

function createCounter(q: Queue): Observable<number> {
  return new Observable<number>((next, error, complete) => {
    // Setup on subscribe
    let i = 0

    // Using queue, schedule tasks on an interval
    const dispose = q.scheduleRepeatAfter(() => {
      next(copy i)
      i++  
    }, second)

    // Teardown
    return () => {
      dispose()
    }
  })
}

function main() {
  const queue = new Queue([thread0])

  // Create counter
  const counter = createCounter(queue)

  // Move number into console.log
  counter.subscribe(console.log)
}
```
