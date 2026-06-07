# Timeout Middleware

Enforces a maximum duration for request handling.

## Usage

### Basic Usage

```go
app.Use(middleware.Timeout(30 * time.Second))
```

### With Configuration

```go
app.Use(middleware.TimeoutWithConfig(middleware.TimeoutConfig{
    Timeout: 30 * time.Second,
    OnTimeout: func(c *marten.Ctx) error {
        return c.JSON(504, marten.M{
            "error": "request took too long",
            "path":  c.Path(),
        })
    },
}))
```

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `Timeout` | `time.Duration` | — | Maximum request duration (required) |
| `OnTimeout` | `func(*Ctx) error` | `504 {"error":"request timeout"}` | Custom timeout response handler |

## Behavior

The handler always runs in its own goroutine. When the timeout fires:

1. The middleware atomically claims the `ResponseWriter` — all subsequent writes from the handler goroutine are silently dropped (no panic, no error)
2. The middleware waits for the handler goroutine to finish
3. The timeout response is written on the original `ResponseWriter`

This design is **data-race free**: the underlying `ResponseWriter` is never accessed from two goroutines simultaneously.

!!! note "Handlers don't need to check ctx.Done()"
    The timeout middleware suppresses handler writes automatically. Handlers that ignore context cancellation will complete normally — their response is just discarded. However, checking `c.Context().Done()` in long operations is still good practice to avoid wasted work.

## Examples

### Global Timeout

```go
app.Use(middleware.Timeout(30 * time.Second))
```

### Per-Group Timeouts

```go
// Fast endpoints — strict 5s limit
api := app.Group("/api")
api.Use(middleware.Timeout(5 * time.Second))

// Slow endpoints — uploads, reports
slow := app.Group("/exports")
slow.Use(middleware.Timeout(5 * time.Minute))
```

### Custom Timeout Response

```go
app.Use(middleware.TimeoutWithConfig(middleware.TimeoutConfig{
    Timeout: 10 * time.Second,
    OnTimeout: func(c *marten.Ctx) error {
        return c.JSON(504, marten.M{
            "error":      "timeout",
            "message":    "The request took too long. Please try again.",
            "request_id": c.RequestID(),
        })
    },
}))
```

### Cooperative Cancellation

For long-running operations that benefit from early exit when the client disconnects or times out:

```go
func exportReport(c *marten.Ctx) error {
    rows, err := db.QueryContext(c.Context(), "SELECT ...")
    if err != nil {
        if c.Context().Err() != nil {
            // Query was cancelled by timeout — handler goroutine exits cleanly
            return nil
        }
        return c.ServerError(err.Error())
    }
    defer rows.Close()

    // ...
    return c.OK(result)
}
```

## Default Response

When timeout fires:

```
HTTP/1.1 504 Gateway Timeout
Content-Type: application/json; charset=utf-8

{"error": "request timeout"}
```

## Best Practices

1. **Set appropriate timeouts** — too short causes false failures, too long wastes resources
2. **Use per-group timeouts** instead of one global value when endpoints have very different SLAs
3. **Pass context to database/HTTP calls** — `db.QueryContext(c.Context(), ...)` so the database driver can cancel the query early
4. **Log timeouts** — add `OnTimeout` to include the path and request ID for monitoring
