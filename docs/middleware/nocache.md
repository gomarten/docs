# NoCache

Prevents caching by setting appropriate HTTP headers.

## Usage

```go
app.Use(middleware.NoCache)
```

## Headers Set

```
Cache-Control: no-store, no-cache, must-revalidate, proxy-revalidate
Pragma: no-cache
Expires: 0
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

    // Apply to specific routes
    api := app.Group("/api")
    api.Use(middleware.NoCache)

    api.GET("/user", func(c *marten.Ctx) error {
        return c.OK(marten.M{"user": "data"})
    })

    app.Run(":8080")
}
```

## Use Cases

- API responses that should never be cached
- User-specific data
- Real-time information
- Security-sensitive endpoints
