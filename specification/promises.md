# Promises

Operate in the same way that they do in TypeScript. Operate similarly to Rust `Future`s. 

Promises execute synchronously and do not *need* to wrap concurrent behaviours (though there is no reason to use them in that like that). 

```typescript
interface IPromise<T> {
  then<U>(onResolution: (value: T) => U): Promise<U>
  catch<U>(onRejection: (error: unknown) => U): Promise<U>
}
```

## Example

```typescript
import console from '@std/console'
import { Promise } from '@std/sync'

async function main() {
  const result = await new Promise<string>(function(resolve) {
    resolve("Job's done")
  })

  console.log(read result)
}
```

## Example with Queues

A `Promise` itself does not start an concurrent action, but instead is used to manage concurrent actions. You must embed a task within a `Promise` to take advantage of multi-threading.

```typescript
import console from '@std/console'
import { thread0 } from '@std/process'
import { Queue, Promise } from '@std/sync'

async function main() {
  const queue = new Queue([thread0])

  const result = await new Promise<string>(resolve => {
    queue.schedule(() => {
      // Do work
      resolve("Job's Done!")
    })
  })

  console.log(read result)
}
```
