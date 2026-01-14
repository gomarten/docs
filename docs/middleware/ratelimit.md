# Rate Limit Middleware

Limits the number of requests per client within a time window.

## Usage

### Basic Usage

```go
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
}))
```

### With Cleanup (Recommended)

```go
rl := middleware.NewRateLimiter(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
})
defer rl.Stop() // Clean up goroutine on shutdown

app.Use(rl.Middleware())
```

## Configuration

| Option | Type | Description |
|--------|------|-------------|
| `Requests` | `int` | Max requests per window |
| `Window` | `time.Duration` | Time window |
| `KeyFunc` | `func(*Ctx) string` | Function to identify clients |
| `Skip` | `func(*Ctx) bool` | Skip rate limiting for certain requests |
| `OnLimitReached` | `func(*Ctx) error` | Custom response when limit exceeded |

## Response Headers

The middleware sets these headers on every response:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests allowed |
| `X-RateLimit-Remaining` | Requests remaining in window |
| `X-RateLimit-Reset` | Unix timestamp when window resets |
| `Retry-After` | Seconds until retry (when limited) |

## Default Behavior

By default, clients are identified by IP address using `c.ClientIP()`.

## Examples

### Basic Rate Limiting

```go
// 100 requests per minute per IP
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
}))
```

### Strict Rate Limiting

```go
// 10 requests per second per IP
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 10,
    Window:   time.Second,
}))
```

### Custom Key Function

```go
// Rate limit by API key
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 1000,
    Window:   time.Hour,
    KeyFunc: func(c *marten.Ctx) string {
        return c.Request.Header.Get("X-API-Key")
    },
}))
```

### Skip Certain Requests

```go
rl := middleware.NewRateLimiter(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
    Skip: func(c *marten.Ctx) bool {
        // Skip health checks
        return c.Path() == "/health"
    },
})
defer rl.Stop()

app.Use(rl.Middleware())
```

### Rate Limit by User

```go
// Rate limit authenticated users
api := app.Group("/api")
api.Use(authMiddleware)
api.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
    KeyFunc: func(c *marten.Ctx) string {
        return c.GetString("user_id")
    },
}))
```

### Custom Rate Limit Response

```go
rl := middleware.NewRateLimiter(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
    OnLimitReached: func(c *marten.Ctx) error {
        return c.JSON(429, marten.M{
            "error":   "rate_limit_exceeded",
            "message": "Please slow down and try again later",
            "retry_after": c.Writer.Header().Get("Retry-After"),
        })
    },
})
defer rl.Stop()

app.Use(rl.Middleware())
```

## Response

When rate limit is exceeded:

```
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1704067200
Retry-After: 45

{"error": "rate limit exceeded"}
```

## Different Limits for Different Routes

```go
// Strict limit for auth endpoints
authRL := middleware.NewRateLimiter(middleware.RateLimitConfig{
    Requests: 5,
    Window:   time.Minute,
})
defer authRL.Stop()

auth := app.Group("/auth")
auth.Use(authRL.Middleware())
auth.POST("/login", login)

// Relaxed limit for API
apiRL := middleware.NewRateLimiter(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
})
defer apiRL.Stop()

api := app.Group("/api")
api.Use(apiRL.Middleware())
api.GET("/users", listUsers)
```

## Best Practices

1. **Use `NewRateLimiter`** with `Stop()` for proper cleanup
2. **Set appropriate limits** - Too strict frustrates users, too relaxed allows abuse
3. **Use different limits** for different endpoints
4. **Consider authenticated vs anonymous** users
5. **Monitor rate limit hits** to adjust limits
