# Health Middleware

Provides a zero-config health check endpoint for load balancer probes and uptime monitoring.

## Usage

### Basic Usage

```go
app.Use(middleware.Health("/health"))
```

### With Custom Response

```go
app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
    Path: "/healthz",
    Handler: func(c *marten.Ctx) error {
        return c.OK(marten.M{
            "status":  "ok",
            "version": "1.2.3",
        })
    },
}))
```

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `Path` | `string` | `"/health"` | URL path that triggers the health response |
| `Handler` | `func(*Ctx) error` | `200 {"status":"ok"}` | Custom response handler |

## Behavior

- Only intercepts `GET` requests to the configured path
- All other requests (including `POST /health`) pass through to the next handler
- Returns immediately — no downstream middleware or handlers run for health requests

## Default Response

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"status":"ok"}
```

## Examples

### Load Balancer Probe

Place health middleware first so probes are answered before logging, auth, or rate limiting:

```go
app.Use(middleware.Health("/health"))  // Answer probe immediately
app.Use(middleware.Logger)             // Everything else gets logged
app.Use(middleware.RateLimit(cfg))     // Health checks bypass rate limit
```

### Kubernetes Liveness / Readiness

```go
// Simple liveness
app.Use(middleware.Health("/healthz/live"))

// Readiness with dependency checks
app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
    Path: "/healthz/ready",
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

### Version Endpoint

```go
app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
    Path: "/health",
    Handler: func(c *marten.Ctx) error {
        return c.OK(marten.M{
            "status":  "ok",
            "version": version,
            "uptime":  time.Since(startTime).String(),
        })
    },
}))
```

### Multiple Health Paths

Register multiple health middleware for different probes:

```go
app.Use(middleware.Health("/ping"))                    // Simple ping
app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
    Path: "/ready",
    Handler: readinessHandler,
}))
```

## Skipping Logs for Health Checks

Combine with Logger's `Skip` option to suppress health check noise in logs:

```go
app.Use(middleware.Health("/health"))
app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    Skip: func(c *marten.Ctx) bool {
        return c.Path() == "/health"
    },
}))
```

Alternatively, place `Health` before `Logger` — since health requests never reach `Logger`, they are never logged.

## Best Practices

1. **Register health middleware first** so probes bypass all other middleware overhead
2. **Keep the default probe fast** — avoid database calls in liveness checks
3. **Use readiness checks for dependencies** — check database, cache, etc. only in readiness probes
4. **Use a dedicated path** — avoid `/` or paths shared with your API
