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
| `Timeout` | Request timeout |
| `Secure` | Security headers |
| `BodyLimit` | Request body size limit |
| `Compress` | Gzip compression |
| `ETag` | Automatic ETag caching |
| `RequestID` | Request ID tracking |
| `NoCache` | Cache prevention |

## Quick Usage

### Simple Middleware (no config)

```go
app.Use(middleware.Logger)
app.Use(middleware.Recover)
app.Use(middleware.RequestID)
app.Use(middleware.ETag)
app.Use(middleware.NoCache)
app.Use(middleware.Compress)
```

### Configurable Middleware

```go
// CORS
app.Use(middleware.CORS(middleware.CORSConfig{
    AllowOrigins: []string{"https://example.com"},
    AllowMethods: []string{"GET", "POST"},
}))

// Rate Limit
app.Use(middleware.RateLimit(middleware.RateLimitConfig{
    Max:    100,
    Window: time.Minute,
}))

// Basic Auth
app.Use(middleware.BasicAuth(middleware.BasicAuthConfig{
    Users: map[string]string{"admin": "secret"},
}))

// Timeout
app.Use(middleware.Timeout(5 * time.Second))

// Secure
app.Use(middleware.Secure(middleware.SecureConfig{
    XSSProtection:      true,
    HSTSMaxAge:         31536000,
    ContentSecurityPolicy: "default-src 'self'",
}))

// Body Limit
app.Use(middleware.BodyLimit(middleware.BodyLimitConfig{
    MaxSize: 10 * middleware.MB,
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
        log.Printf("Request took %v", time.Since(start))
        
        return err
    }
}
```

## Middleware Order

Middleware executes in registration order:

```go
app.Use(middleware.RequestID)  // 1st: adds request ID
app.Use(middleware.Logger)     // 2nd: logs with request ID
app.Use(middleware.Recover)    // 3rd: catches panics
```

## Chaining

```go
handler := middleware.Chain(
    middleware.Logger,
    middleware.Recover,
)(finalHandler)
```
