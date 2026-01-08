# Middleware Reference

Marten includes 12 production-ready middleware components.

## Overview

| Middleware | Purpose | Import |
|------------|---------|--------|
| [Logger](logger.md) | Request logging | `middleware.Logger` |
| [Recover](recover.md) | Panic recovery | `middleware.Recover` |
| [CORS](cors.md) | Cross-origin requests | `middleware.CORS(cfg)` |
| [RateLimit](ratelimit.md) | Rate limiting | `middleware.RateLimit(cfg)` |
| [BasicAuth](basicauth.md) | Basic authentication | `middleware.BasicAuth(cfg)` |
| [Timeout](timeout.md) | Request timeouts | `middleware.Timeout(duration)` |
| [Secure](secure.md) | Security headers | `middleware.Secure(cfg)` |
| [BodyLimit](bodylimit.md) | Request size limits | `middleware.BodyLimit(size)` |
| [Compress](compress.md) | Gzip compression | `middleware.Compress(cfg)` |
| [ETag](etag.md) | Response caching | `middleware.ETag` |
| [RequestID](requestid.md) | Request tracking | `middleware.RequestID` |
| NoCache | Cache prevention | `middleware.NoCache` |

## Quick Start

```go
import (
    "github.com/gomarten/marten/marten"
    "github.com/gomarten/marten/marten/middleware"
)

func main() {
    app := marten.New()
    
    // Recommended middleware stack
    app.Use(
        middleware.RequestID,                           // Track requests
        middleware.Logger,                              // Log requests
        middleware.Recover,                             // Catch panics
        middleware.Secure(middleware.DefaultSecureConfig()), // Security headers
        middleware.CORS(middleware.DefaultCORSConfig()),     // CORS
        middleware.BodyLimit(10 * middleware.MB),       // Limit body size
    )
    
    // ...
}
```

## Recommended Order

```go
app.Use(
    middleware.RequestID,  // 1. Assign ID first for tracking
    middleware.Logger,     // 2. Log with request ID
    middleware.Recover,    // 3. Catch panics
    middleware.Secure,     // 4. Set security headers
    middleware.CORS,       // 5. Handle CORS
    middleware.RateLimit,  // 6. Reject excess requests early
    middleware.BodyLimit,  // 7. Reject large requests early
    middleware.Compress,   // 8. Compress responses
    middleware.Timeout,    // 9. Enforce timeouts
)
```

## Creating Custom Middleware

See the [Middleware Guide](../guide/middleware.md) for details on creating custom middleware.
