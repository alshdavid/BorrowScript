## Simple Web Server

_Note the standard library hasn't been designed yet so this is demonstrational_

```typescript
import { Server, Headers, StatusCode, ContentTypes, Request, Response, HandlerFunc } from '@std/http'

async function main() {
  const server = new http.Server()

  server.requests.subscribe(function(req: Request, res: Response) {
    res.setStatus(StatusCode.Ok)
    res.setHeader(Headers.ContentType, ContentTypes.PlainText)
    res.setBody('Hello World!')
    res.send()
  })

  await server.listen([127, 0, 0, 1], 3000)
}
```
