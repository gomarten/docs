# ETag

Automatic ETag generation for HTTP caching with conditional request support.

## Usage

```go
app.Use(middleware.ETag)
```

## How It Works

The ETag middleware:

1. Captures response body for GET/HEAD requests
2. Generates SHA-1 hash of the content
3. Sets `ETag` header with the hash
4. Returns `304 Not Modified` if client sends matching `If-None-Match`

```go
// Client sends: If-None-Match: "a1b2c3d4e5f6g7h8"
// If content hash matches, returns 304 with no body
```

## Example

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    app.Use(middleware.ETag)

    app.GET("/data", func(c *marten.Ctx) error {
        return c.JSON(200, marten.M{
            "items": []string{"a", "b", "c"},
        })
    })

    app.Run(":8080")
}
```

## Behavior

| Condition | Result |
|-----------|--------|
| GET/HEAD with 2xx response | ETag header added |
| `If-None-Match` matches | 304 Not Modified |
| POST/PUT/DELETE | Middleware skipped |
| Non-2xx response | No ETag added |

## Benefits

- Reduces bandwidth for unchanged resources
- Improves client-side caching
- Zero configuration required
- Works with any response content

## Notes

- ETag is generated from response body content
- Only applies to successful (2xx) responses
- Automatically handles `If-None-Match` validation
