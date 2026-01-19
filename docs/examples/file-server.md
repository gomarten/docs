# File Server Example

Serve static files using the built-in Static middleware or custom wildcard routes.

## Using Static Middleware (Recommended)

The easiest way to serve static files is using the built-in `middleware.Static()`:

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    app.Use(middleware.Logger, middleware.Recover)

    // Serve static files from ./public
    app.Use(middleware.Static("./public"))

    app.Run(":3000")
}
```

## With Configuration

```go
func main() {
    app := marten.New()
    app.Use(middleware.Logger, middleware.Recover)

    // Serve static files with custom config
    app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
        Root:   "./public",
        Prefix: "/static",
        MaxAge: 86400, // Cache for 24 hours
        Browse: false, // Disable directory browsing
    }))

    app.Run(":3000")
}
```

## Multiple Static Directories

```go
func main() {
    app := marten.New()
    app.Use(middleware.Logger, middleware.Recover)

    // Serve uploads from /uploads
    app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
        Root:   "./uploads",
        Prefix: "/uploads",
    }))

    // Serve assets from /assets
    app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
        Root:   "./assets",
        Prefix: "/assets",
        MaxAge: 31536000, // Cache for 1 year
    }))

    // Serve main site from /
    app.Use(middleware.Static("./public"))

    app.Run(":3000")
}
```

## SPA (Single Page Application)

For single-page applications with client-side routing:

```go
func main() {
    app := marten.New()
    app.Use(middleware.Logger, middleware.Recover)

    // API routes first
    api := app.Group("/api")
    api.GET("/users", listUsers)
    api.POST("/users", createUser)

    // Serve static files
    app.Use(middleware.Static("./dist"))

    // Fallback to index.html for client-side routing
    app.NotFound(func(c *marten.Ctx) error {
        // API routes return 404
        if strings.HasPrefix(c.Path(), "/api/") {
            return c.NotFound("endpoint not found")
        }
        
        // SPA fallback
        return c.File("./dist/index.html")
    })

    app.Run(":3000")
}
```

## Custom Static File Handler (Advanced)

If you need more control, you can implement a custom handler using wildcard routes:

```go
package main

import (
    "io"
    "mime"
    "os"
    "path/filepath"
    "strings"

    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    app.Use(middleware.Logger, middleware.Recover)

    // Custom static file handler
    app.GET("/static/*filepath", serveStatic("./public"))

    // Custom uploads handler
    app.GET("/uploads/*filepath", serveStatic("./uploads"))

    app.Run(":3000")
}

func serveStatic(root string) marten.Handler {
    return func(c *marten.Ctx) error {
        filepath := c.Param("filepath")

        // Security: prevent directory traversal
        if strings.Contains(filepath, "..") {
            return c.BadRequest("invalid path")
        }

        path := root + "/" + filepath
        return serveFile(c, path)
    }
}

func serveFile(c *marten.Ctx, path string) error {
    file, err := os.Open(path)
    if err != nil {
        if os.IsNotExist(err) {
            return c.NotFound("file not found")
        }
        return c.ServerError("failed to open file")
    }
    defer file.Close()

    stat, _ := file.Stat()
    if stat.IsDir() {
        return serveFile(c, filepath.Join(path, "index.html"))
    }

    // Set content type
    ext := filepath.Ext(path)
    contentType := mime.TypeByExtension(ext)
    if contentType == "" {
        contentType = "application/octet-stream"
    }
    c.Header("Content-Type", contentType)

    c.Status(200)
    io.Copy(c.Writer, file)
    return nil
}
```

## Download Handler

Force file downloads with custom headers:

```go
app.GET("/download/:filename", func(c *marten.Ctx) error {
    filename := c.Param("filename")

    // Security check
    if strings.Contains(filename, "..") {
        return c.BadRequest("invalid filename")
    }

    path := filepath.Join("./public", filename)
    file, err := os.Open(path)
    if err != nil {
        return c.NotFound("file not found")
    }
    defer file.Close()

    c.Header("Content-Disposition", "attachment; filename="+filename)
    c.Header("Content-Type", "application/octet-stream")
    c.Status(200)
    io.Copy(c.Writer, file)
    return nil
})
```

## Directory Browsing

Enable directory listing for file sharing:

```go
app.Use(middleware.StaticWithConfig(middleware.StaticConfig{
    Root:   "./files",
    Prefix: "/files",
    Browse: true, // Enable directory browsing
}))
```

## Key Features

### Static Middleware Features

| Feature | Description |
|---------|-------------|
| Content-Type Detection | Automatic based on file extension |
| Directory Index | Serves `index.html` for directories |
| Directory Browsing | Optional HTML directory listing |
| HTTP Caching | If-Modified-Since support (304 responses) |
| Security | Directory traversal prevention |
| URL Prefix | Strip prefix before file lookup |
| HEAD Support | Proper HEAD request handling |

### Custom Handler Features

| Feature | Usage |
|---------|-------|
| Wildcard routes | `*filepath` captures remaining path |
| `c.Param("filepath")` | Access captured path |
| Security | Check for `..` to prevent traversal |
| Content-Type | Auto-detect from file extension |

## Best Practices

1. **Use Static Middleware** for most use cases - it's battle-tested and feature-complete
2. **Set appropriate cache headers** with `MaxAge` for better performance
3. **Disable directory browsing** in production unless intentional
4. **Place API routes before static middleware** to avoid conflicts
5. **Use URL prefixes** for clarity and organization
6. **Consider a CDN** for production static assets

## See Also

- [Static Middleware Documentation](../middleware/static.md)
- [Middleware Guide](../guide/middleware.md)
- [Compress Middleware](../middleware/compress.md) - Compress static files
- [ETag Middleware](../middleware/etag.md) - Response caching
