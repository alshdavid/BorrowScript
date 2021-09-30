TODO Queues are weird, referencing the main thread

# Concurrency and Parallelism

TSBC includes the capacity to use a single thread directly or use a green thread  tasks queue that spans an allocated thread pool.

This is split specifically into two objects; `Thread` and `Queue`. <br>You construct a `Queue` using a list of `Thread` instances.

This is for more granular control of queues. In UI applications you may want your main thread to be an undisturbed thread dedicated to UI while you have a thread pool managed by a queue for worker tasks. You may even want two queues, one UI queue and one worker queue.

## Thread

Creating an OS thread is done by importing and instantiating the `Thread` constructor from the standard library.

The constructor takes a callback which will serve as the thread's main function.

### Interface

```typescript
interface IThread {
  exec(f: Function): void
  terminate(): void
}
```

### Example

```typescript
import console from '@std/console'
import { Thread } from '@std/thread'
import { second } from '@std/time'

function main() {
  const thread1 = new Thread()

  thread1.exec(() => {
    console.log("Hello from thread")
  })

  // wait for thread to finish
  thread1.join()
}
```

### Main Thread

You can access a reference to the main thread from `@std/process`

```typescript
import { thread0 } from '@std/process'
```

## Queue

A concurrent queue that spans across threads. This queue does not guarantee that tasks will execute in parallel, however code is written as if they are.

A `Queue` must have one or more `Thread` instances allocated. There may me many more concurrent tasks in a Queue than there are allocated Threads (mn) and it will likely be implemented as a task-stealing queue (like Go and Tokio).

### Interface

```typescript
interface IQueue {
  // Add an item to the queue
  spawn(task: () => any): Promise<void>
  // Imprecise sleep that guarantees a
  // wait at least the specified duration
  snooze(duration: time.Duration): Promise<void>
  // Wait for queue to clear before proceeding
  flush(): Promise<void>
  terminate(): void
}
```

### Example

```typescript
import console from '@std/console'
import { Thread } from '@std/thread'
import { thread0 } from '@std/process'
import { Queue } from '@std/sync'

async function main() {
  const thread1 = new Thread()
  const thread2 = new Thread()

  const threadPool: Thread[] = [thread0, thread1, thread2]
  const queue = new Queue(move threadPool)

  queue.spawn(() => {
    console.log("One!")
  })

  queue.spawn(() => {
    console.log("Two!")
  })

  // block current thread, waiting for queue to clear
  queue.flush()
}
```