# Simple WASM Application

When the following code compiled to web assembly is faster than the equivalent TypeScript application compiled optimally to JavaScript.

*The document tree structure might need to be reevaluated due to the method parent/child references are accessed*

*We will need to wait for Web Assembly to be a little more mature, though something like this can be approximated with the current generation of Web Assembly features*

```typescript
// TypeScript Static
import { window, HTMLDivElement } from '@std/dom'

function main() {
  // Mutable reference to a div element
  let div: HTMLDivElement = window.document.createElement('div')
  div.innerHTML = 'Hello World'

  // Move ownership to the body
  window.document.body.appendChild(move div)
}
```

Compared to the following source TypeScript compiled to JavaScript

```typescript
// TypeScript

void function main() {
  const div: HTMLDivElement = window.document.createElement('div')
  div.innerHTML = 'Hello World'

  window.document.body.appendChild(div)
}()
```