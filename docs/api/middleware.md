# Middleware Reference

All built-in middleware at a glance.

## Middleware Signature

```go
type Middleware func(Handler) Handler
```

## Built-in Middleware

| Middleware | Description |
|------------|-------------|
| `Logger` | Request logging |
| `Recover` | Panic recovery |
| `CORS` | Cross-origin resource sharing |
| `RateLimit` | Request rate limiting |
| `BasicAuth` | HTTP Basic authentication |
| `Health` | Health check endpoint |
| `Timeout` | Request timeout |
| `Secure` | Security headers |
| `BodyLimit` | Request body size limit |
| `Compress` | Gzip compression |
| `ETag` | Automatic ETag caching |
| `RequestID` | Request ID tracking |
| `Static` | Static file serving |
| `NoCache` | Cache prevention |

## Quick Usage

### Simple Middleware (no config)

```go
app.Use(middleware.Logger)
app.Use(middleware.Recover)
app.Use(middleware.RequestID)
app.Use(middleware.ETag)
app.Use(middleware.NoCache)
app.Use(middleware.Health("/health"))
app.Use(middleware.Static("./public"))
```

### Configurable Middleware

```go
// Health check with custom response
app.Use(middleware.HealthWithConfig(middleware.HealthConfig{
    Path: "/healthz",
    Handler: func(c *marten.Ctx) error {
        return c.OK(marten.M{"status": "ok", "version": "1.0.0"})
    },
}))

// CORS
app.Use(middleware.CORS(middleware.CORSConfig{
    AllowOrigins: []string{"https://example.com"},
    AllowMethods: []string{"GET", "POST", "PUT", "DELETE"},
    AllowHeaders: []string{"Content-Type", "Authorization"},
}))

// Rate Limit
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Requests: 100,
    Window:   time.Minute,
}))

// Basic Auth
app.Use(middleware.BasicAuthSimple("admin", "secret"))

// Timeout
app.Use(middleware.Timeout(5 * time.Second))

// Timeout with custom response
app.Use(middleware.TimeoutWithConfig(middleware.TimeoutConfig{
    Timeout: 10 * time.Second,
    OnTimeout: func(c *marten.Ctx) error {
        return c.JSON(504, marten.E("request timed out"))
    },
}))

// Secure headers
app.Use(middleware.Secure(middleware.DefaultSecureConfig()))

// Body limit
app.Use(middleware.BodyLimit(10 * middleware.MB))

// Compression
app.Use(middleware.Compress(middleware.DefaultCompressConfig()))

// Static files with config
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    Prefix: "/static",
    MaxAge: 3600,
    Browse: false,
}))
```

## Creating Custom Middleware

```go
func MyMiddleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        // Before handler
        start := time.Now()

        err := next(c)

        // After handler
        log.Printf("%s %s — %v", c.Method(), c.Path(), time.Since(start))

        return err
    }
}
```

## Middleware Order

Middleware executes in registration order:

```go
app.Use(middleware.Health("/health"))  // 1st: short-circuit probes
app.Use(middleware.RequestID)          // 2nd: assign ID for tracking
app.Use(middleware.Logger)             // 3rd: log with request ID
app.Use(middleware.Recover)            // 4th: catch panics
```

## Chaining

Compose middleware manually using `marten.Chain`:

```go
composed := marten.Chain(
    middleware.Logger,
    middleware.Recover,
)(finalHandler)
```
