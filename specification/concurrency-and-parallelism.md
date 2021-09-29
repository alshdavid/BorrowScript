# Concurrency and Parallelism

TypeScript BC includes the capacity to use a single thread directly, or queue tasks in a pool of threads similarly to Go, and syntactically similarly to TypeScript.

This is split specifically into two objects; `Thread` and `Queue`. You construct a `Queue` using a list of specified `Thread` instances.

## Thread

Creating a thread is done by importing and instantiating the `Thread` constructor from the standard library.

The constructor takes a callback which will serve as the thread's main function.

### Interface

```typescript
interface IThread {
  exec(fn: () => any): void
  sleep(duration: time.Duration): void
  terminate(): void
}
```

### Example

```typescript
import console from '@std/console'
import { Thread, sleep } from '@std/os'
import { second } from '@std/time'

function main() {
  const thread1 = new Thread()

  thread1.exec(() => {
    console.log("Hello from thread")
  })

  // Crude way to prevent application exiting
  sleep(second * 5)
}
```

### Main Thread

You can access the main thread from `@std/process`

```typescript
import { thread0 } from '@std/process'
```

## Queue

A concurrent queue that spans across threads. This queue is concurrent in that it does not guarantee that tasks will execute in parallel, however they can and code is written as if they are.

A `Queue` must have one or more `Thread` instances allocated. It operates as a task-stealing queue and shares the syntax of Go and TypeScript like concurrency. 

### Interface

```typescript
interface IQueue {
  schedule(task: () => any): void
  scheduleAfter(task: () => any, duration: time.Duration): () => void
  scheduleRepeatAfter(task: () => any, interval: time.Duration): () => void
  flush(): void
  terminate(): void
}
```

### Example

```typescript
import console from '@std/console'
import { Thread } from '@std/os'
import { thread0 } from '@std/process'
import { Queue } from '@std/sync'

function main() {
  const thread1 = new Thread()
  const thread2 = new Thread()

  const threadPool: Thread[] = [thread0, thread1, thread2]
  const queue = new Queue(move threadPool)

  queue.schedule(() => {
    console.log("Hello from queue")
  })

  // block current thread, waiting for queue to clear
  queue.flush()
}
```