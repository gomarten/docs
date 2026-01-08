# Rate Limit Middleware

Limits the number of requests per client within a time window.

## Usage

```go
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
}))
```

## Configuration

| Option | Type | Description |
|--------|------|-------------|
| `Requests` | `int` | Max requests per window |
| `Window` | `time.Duration` | Time window |
| `KeyFunc` | `func(*Ctx) string` | Function to identify clients |

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

## Response

When rate limit is exceeded:

```
HTTP/1.1 429 Too Many Requests
Content-Type: application/json

{"error": "rate limit exceeded"}
```

## Different Limits for Different Routes

```go
// Strict limit for auth endpoints
auth := app.Group("/auth")
auth.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 5,
    Window:   time.Minute,
}))
auth.POST("/login", login)

// Relaxed limit for API
api := app.Group("/api")
api.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
}))
api.GET("/users", listUsers)
```

## Best Practices

1. **Set appropriate limits** - Too strict frustrates users, too relaxed allows abuse
2. **Use different limits** for different endpoints
3. **Consider authenticated vs anonymous** users
4. **Monitor rate limit hits** to adjust limits
