# Recover Middleware

Catches panics and returns a 500 Internal Server Error instead of crashing the server.

## Usage

```go
app.Use(middleware.Recover)
```

## Behavior

When a panic occurs:

1. The panic is caught
2. The error is logged
3. A 500 response is returned
4. The server continues running

## Example

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    
    app.Use(middleware.Recover)
    
    app.GET("/panic", func(c *marten.Ctx) error {
        panic("something went wrong!")
        return nil
    })
    
    app.Run(":3000")
}
```

Request to `/panic` returns:

```
HTTP/1.1 500 Internal Server Error
Content-Type: text/plain; charset=utf-8

Internal Server Error
```

## Log Output

```
2024/01/15 10:30:00 panic recovered: something went wrong!
```

## Custom Recovery

For custom panic handling:

```go
func CustomRecover(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) (err error) {
        defer func() {
            if r := recover(); r != nil {
                // Log with stack trace
                log.Printf("panic: %v\n%s", r, debug.Stack())
                
                // Custom response
                err = c.JSON(500, marten.M{
                    "error":      "internal error",
                    "request_id": c.RequestID(),
                })
            }
        }()
        return next(c)
    }
}
```

## Best Practices

1. **Always use Recover** - Prevents server crashes
2. **Place early in middleware chain** - Catches panics from all handlers
3. **Log panics** - For debugging
4. **Don't expose panic details** - Security risk
