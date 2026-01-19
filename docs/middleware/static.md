# Static Middleware

Serves static files from a directory with caching, security, and directory browsing support.

## Import

```go
import "github.com/gomarten/marten/middleware"
```

## Basic Usage

```go
app := marten.New()

// Serve files from ./public directory
app.Use(middleware.Static("./public"))

// Now files are accessible:
// GET /index.html -> ./public/index.html
// GET /css/style.css -> ./public/css/style.css
// GET /images/logo.png -> ./public/images/logo.png
```

## Configuration

```go
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",           // Root directory (required)
    Index:  "index.html",          // Index file for directories
    Browse: false,                 // Enable directory browsing
    MaxAge: 3600,                  // Cache-Control max-age in seconds
    Prefix: "/static",             // URL prefix to strip
    NotFoundHandler: customHandler, // Custom 404 handler
    SkipLogging: false,            // Skip logging for static files
}))
```

## Features

### Automatic Content-Type Detection

Content types are automatically detected based on file extensions:

```go
app.Use(middleware.Static("./public"))

// GET /style.css -> Content-Type: text/css
// GET /script.js -> Content-Type: text/javascript
// GET /image.png -> Content-Type: image/png
// GET /data.json -> Content-Type: application/json
```

### Directory Index Serving

Automatically serves `index.html` for directory requests:

```go
app.Use(middleware.Static("./public"))

// GET / -> serves ./public/index.html
// GET /docs/ -> serves ./public/docs/index.html
```

### Directory Browsing

Enable directory listing when no index file exists:

```go
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    Browse: true,  // Enable directory browsing
}))

// GET /images/ -> shows list of files in ./public/images/
```

### HTTP Caching

Supports `If-Modified-Since` header for efficient caching:

```go
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    MaxAge: 3600,  // Cache for 1 hour
}))

// First request: 200 OK with Last-Modified header
// Subsequent requests: 304 Not Modified (if not changed)
```

### URL Prefix

Strip a URL prefix before looking up files:

```go
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    Prefix: "/static",
}))

// GET /static/style.css -> serves ./public/style.css
// GET /style.css -> not handled by static middleware
```

### Security

Built-in protection against directory traversal attacks:

```go
app.Use(middleware.Static("./public"))

// GET /../etc/passwd -> rejected (404)
// GET /../../secret.txt -> rejected (404)
```

### HEAD Request Support

Properly handles HEAD requests without sending body:

```go
app.Use(middleware.Static("./public"))

// HEAD /file.txt -> returns headers only, no body
```

## Examples

### Basic File Server

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    
    app.Use(middleware.Logger)
    app.Use(middleware.Static("./public"))
    
    app.Run(":8080")
}
```

### With URL Prefix

```go
app := marten.New()

// Serve static files under /static prefix
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    Prefix: "/static",
    MaxAge: 86400, // Cache for 24 hours
}))

// API routes
app.GET("/api/users", listUsers)
app.POST("/api/users", createUser)

app.Run(":8080")
```

### SPA (Single Page Application)

```go
app := marten.New()

// API routes first
api := app.Group("/api")
api.GET("/users", listUsers)
api.POST("/users", createUser)

// Serve static files
app.Use(middleware.Static("./dist"))

// Fallback to index.html for client-side routing
app.GET("/*", func(c *marten.Ctx) error {
    return c.File("./dist/index.html")
})

app.Run(":8080")
```

### With Custom 404 Handler

```go
app := marten.New()

app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root: "./public",
    NotFoundHandler: func(c *marten.Ctx) error {
        return c.JSON(404, marten.M{
            "error": "File not found",
            "path":  c.Path(),
        })
    },
}))

app.Run(":8080")
```

### Multiple Static Directories

```go
app := marten.New()

// Serve uploads from /uploads
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./uploads",
    Prefix: "/uploads",
}))

// Serve assets from /assets
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./assets",
    Prefix: "/assets",
}))

// Serve main site from /
app.Use(middleware.Static("./public"))

app.Run(":8080")
```

### With Directory Browsing

```go
app := marten.New()

app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./files",
    Browse: true,  // Enable directory listing
    Prefix: "/files",
}))

app.Run(":8080")

// GET /files/ -> shows directory listing
// GET /files/documents/ -> shows documents directory listing
```

## Configuration Options

### StaticConfig

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Root` | `string` | (required) | Root directory to serve files from |
| `Index` | `string` | `"index.html"` | Index file for directories |
| `Browse` | `bool` | `false` | Enable directory browsing |
| `MaxAge` | `int` | `0` | Cache-Control max-age in seconds |
| `Prefix` | `string` | `""` | URL prefix to strip |
| `NotFoundHandler` | `Handler` | `nil` | Custom 404 handler |
| `SkipLogging` | `bool` | `false` | Skip logging for static files |

## Best Practices

### 1. Place Static Middleware After API Routes

```go
// ✅ Good: API routes first
app.GET("/api/users", listUsers)
app.Use(middleware.Static("./public"))

// ❌ Bad: Static middleware might catch API routes
app.Use(middleware.Static("./public"))
app.GET("/api/users", listUsers)
```

### 2. Use URL Prefix for Clarity

```go
// ✅ Good: Clear separation
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    Prefix: "/static",
}))

// ❌ Less clear: No prefix
app.Use(middleware.Static("./public"))
```

### 3. Set Appropriate Cache Duration

```go
// ✅ Good: Cache static assets
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    MaxAge: 86400, // 24 hours for static assets
}))

// ❌ Bad: No caching (more server load)
app.Use(middleware.Static("./public"))
```

### 4. Disable Directory Browsing in Production

```go
// ✅ Good: Disable browsing in production
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./public",
    Browse: false, // Don't expose directory structure
}))

// ⚠️ Use with caution: Only enable for file sharing
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./files",
    Browse: true, // Only if intentional
}))
```

## Performance Tips

1. **Use a CDN** for production static assets
2. **Set appropriate MaxAge** to reduce server load
3. **Compress assets** before serving (use build tools)
4. **Use HTTP/2** for better performance with many small files
5. **Consider nginx/Apache** for high-traffic static file serving

## Security Considerations

1. **Directory Traversal**: Automatically prevented by the middleware
2. **Directory Browsing**: Disabled by default, enable only when needed
3. **File Permissions**: Ensure proper file system permissions
4. **Sensitive Files**: Don't place sensitive files in public directories
5. **CORS**: Configure CORS middleware if serving assets cross-origin

## See Also

- [Middleware Guide](../guide/middleware.md)
- [File Server Example](../examples/file-server.md)
- [Compress Middleware](compress.md) - Compress responses
- [NoCache Middleware](nocache.md) - Prevent caching
- [ETag Middleware](etag.md) - Response caching
