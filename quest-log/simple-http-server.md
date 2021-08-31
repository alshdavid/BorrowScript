

## Web Server

```typescript
import console from '@std/console'
import { Server, Headers, ContentTypes, Request, Response, HandlerFunc } from '@std/http'

function handler(): HandlerFunc {
  return function (request: Request, response: Response) {
    response.setStatus(200)
    response.setHeader(Headers.ContentType, ContentTypes.PlainText)
    response.setBody('Hello World!')
    response.send()
  }
}

async function main() {
  let server = Server.from([127, 0, 0, 1], 3000)

  server.handle(handler())

  try {
    await server.serve()
    console.log('Serving on http://localhost:3000')
  catch (error) {
    console.log(`Server failed to launch with error: ${error}`)
  }
}
```

# Examples 