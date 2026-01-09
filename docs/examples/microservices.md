# Microservices Patterns

Patterns for building microservices with Marten.

## Health Checks

```go
app.GET("/health", func(c *marten.Ctx) error {
    return c.OK(marten.M{"status": "healthy"})
})

app.GET("/ready", func(c *marten.Ctx) error {
    // Check dependencies
    if !dbConnected {
        return c.ServerError("database not ready")
    }
    return c.OK(marten.M{"status": "ready"})
})
```

## Request Tracing

```go
app.Use(middleware.RequestID)
app.Use(middleware.Logger)

// Access request ID in handlers
func handler(c *marten.Ctx) error {
    id := c.RequestID()
    log.Printf("[%s] Processing request", id)
    
    // Pass to downstream services
    // req.Header.Set("X-Request-ID", id)
    
    return c.OK(marten.M{"request_id": id})
}
```

## Graceful Shutdown

```go
func main() {
    app := marten.New()
    
    // Setup routes...
    
    // Graceful shutdown with 10s timeout
    app.RunGraceful(":8080", 10*time.Second)
}
```

## Rate Limiting

```go
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Max:    100,
    Window: time.Minute,
}))
```

## Timeout Protection

```go
app.Use(middleware.Timeout(5 * time.Second))
```

## Service Structure

```
myservice/
├── main.go
├── handlers/
│   ├── health.go
│   └── api.go
├── middleware/
│   └── auth.go
└── go.mod
```

## Example Service

```go
package main

import (
    "time"
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()

    // Standard middleware stack
    app.Use(
        middleware.RequestID,
        middleware.Logger,
        middleware.Recover,
        middleware.Timeout(5*time.Second),
        middleware.RateLimit(middleware.RateLimitConfig{
            Max:    100,
            Window: time.Minute,
        }),
    )

    // Health endpoints
    app.GET("/health", healthHandler)
    app.GET("/ready", readyHandler)

    // API
    api := app.Group("/api/v1")
    api.GET("/resource", getResource)
    api.POST("/resource", createResource)

    app.RunGraceful(":8080", 10*time.Second)
}
```

## Key Patterns

- Always include health/ready endpoints
- Use request IDs for tracing
- Implement graceful shutdown
- Add rate limiting and timeouts
- Use route groups for versioning
