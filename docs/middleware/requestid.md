# Request ID

Adds unique identifiers to each request for tracing and debugging.

## Usage

```go
app.Use(middleware.RequestID)
```

## How It Works

The middleware:

1. Checks for existing `X-Request-ID` header (from upstream proxy)
2. Generates a new ID if none exists
3. Sets `X-Request-ID` response header
4. Makes ID available via `c.RequestID()`

## Example

```go
package main

import (
    "github.com/gomarten/marten/marten"
    "github.com/gomarten/marten/marten/middleware"
)

func main() {
    app := marten.New()
    app.Use(middleware.RequestID)
    app.Use(middleware.Logger)

    app.GET("/", func(c *marten.Ctx) error {
        return c.OK(marten.M{
            "request_id": c.RequestID(),
        })
    })

    app.Run(":8080")
}
```

## Accessing the Request ID

```go
// In any handler
func handler(c *marten.Ctx) error {
    id := c.RequestID()
    
    // Use in logs
    log.Printf("[%s] Processing request", id)
    
    // Include in response
    return c.OK(marten.M{"id": id})
}
```

## ID Format

- 16-character hexadecimal string
- Generated using crypto/rand
- Example: `a1b2c3d4e5f6g7h8`

## Use Cases

- Request tracing across services
- Correlating logs
- Debugging production issues
- Client-side error reporting

## Preserving Upstream IDs

If your app is behind a proxy that sets `X-Request-ID`, the middleware preserves it:

```
Client → Proxy (sets X-Request-ID) → Marten (preserves ID)
```
