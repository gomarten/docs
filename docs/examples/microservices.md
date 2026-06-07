# Microservices Patterns

Patterns for building microservices with Marten.

## Health Checks

Use `middleware.Health` for liveness probes — it answers before any other middleware runs, keeping probe latency minimal:

```go
// Simple liveness
app.Use(middleware.Health("/health"))

// Readiness with dependency checks
app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
    Path: "/ready",
    Handler: func(c *marten.Ctx) error {
        if err := db.Ping(); err != nil {
            return c.JSON(503, marten.M{
                "status": "unavailable",
                "reason": "database unreachable",
            })
        }
        return c.OK(marten.M{"status": "ok"})
    },
}))
```

## Request Tracing

```go
app.Use(middleware.RequestID)
app.Use(middleware.Logger)

func handler(c *marten.Ctx) error {
    id := c.RequestID()
    log.Printf("[%s] Processing request", id)

    // Propagate to downstream services
    // req.Header.Set("X-Request-ID", id)

    return c.OK(marten.M{"request_id": id})
}
```

## Graceful Shutdown

```go
func main() {
    app := marten.New()

    app.OnShutdown(func() {
        db.Close()
        log.Println("connections closed")
    })

    app.RunGraceful(":8080", 10*time.Second)
}
```

## Rate Limiting

```go
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
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
│   └── api.go
├── middleware/
│   └── auth.go
└── go.mod
```

## Example Service

```go
package main

import (
    "log"
    "time"

    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()

    // Standard microservice middleware stack
    app.Use(
        middleware.Health("/health"),           // Answer probes first
        middleware.RequestID,                   // Assign trace ID
        middleware.Logger,                      // Log all requests
        middleware.Recover,                     // Catch panics
        middleware.Timeout(5*time.Second),      // Enforce SLA
        middleware.RateLimit(middleware.RateLimitConfig{
            Requests: 100,
            Window:   time.Minute,
        }),
    )

    // Readiness probe with dependency check
    app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
        Path: "/ready",
        Handler: func(c *marten.Ctx) error {
            if err := db.Ping(); err != nil {
                return c.JSON(503, marten.M{"status": "unavailable"})
            }
            return c.OK(marten.M{"status": "ok"})
        },
    }))

    // API
    api := app.Group("/api/v1")
    api.GET("/resource", getResource)
    api.POST("/resource", createResource)

    log.Println("Service running on :8080")
    app.RunGraceful(":8080", 10*time.Second)
}
```

## Async Job Pattern

Use `c.Accepted()` (202) when a request is queued for async processing:

```go
func enqueueJob(c *marten.Ctx) error {
    var req JobRequest
    if err := c.Bind(&req); err != nil {
        return c.BadRequest(err.Error())
    }

    jobID := queue.Enqueue(req)

    return c.Accepted(marten.M{
        "job_id": jobID,
        "status": "queued",
    })
}
```

## Key Patterns

- Use `middleware.Health` — not a manual route — so probes bypass rate limiting and logging
- Use `middleware.Health` for liveness, `HealthWithConfig` for readiness (readiness can check dependencies)
- Pass `c.Context()` to database and HTTP calls so timeouts cancel in-flight work
- Use `c.Accepted()` for async job endpoints
- Use `OnShutdown` to close connections cleanly
- Use request IDs for distributed tracing
