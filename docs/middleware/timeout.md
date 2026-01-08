# Timeout Middleware

Enforces a maximum duration for request handling.

## Usage

```go
app.Use(middleware.Timeout(30 * time.Second))
```

## Behavior

1. Starts a timer when request begins
2. If handler completes in time, response is sent normally
3. If timeout expires, returns 504 Gateway Timeout

## Examples

### Global Timeout

```go
app.Use(middleware.Timeout(30 * time.Second))
```

### Different Timeouts

```go
// Quick endpoints
api := app.Group("/api")
api.Use(middleware.Timeout(5 * time.Second))

// Slow endpoints (file uploads, reports)
slow := app.Group("/slow")
slow.Use(middleware.Timeout(5 * time.Minute))
```

## Response

When timeout occurs:

```
HTTP/1.1 504 Gateway Timeout
Content-Type: application/json

{"error": "request timeout"}
```

## Checking Context Cancellation

Handlers should check for context cancellation:

```go
func slowHandler(c *marten.Ctx) error {
    // Long operation
    result, err := longOperation(c.Context())
    if err != nil {
        if c.Context().Err() == context.DeadlineExceeded {
            return c.ServerError("operation timed out")
        }
        return c.ServerError(err.Error())
    }
    
    return c.OK(result)
}

func longOperation(ctx context.Context) (string, error) {
    select {
    case <-time.After(10 * time.Second):
        return "done", nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}
```

## Best Practices

1. **Set appropriate timeouts** - Too short causes failures, too long wastes resources
2. **Check context in long operations** - Allows early termination
3. **Use different timeouts** for different endpoints
4. **Log timeouts** for monitoring
