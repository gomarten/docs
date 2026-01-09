# File Server Example

Serve static files using wildcard routes.

## Basic Setup

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

    // Serve static files
    app.GET("/static/*filepath", serveStatic("./public"))

    // Serve uploads
    app.GET("/uploads/*filepath", serveStatic("./uploads"))

    app.Run(":3000")
}
```

## Static File Handler

```go
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

## SPA Fallback

For single-page applications, serve `index.html` for unmatched routes:

```go
app.NotFound(func(c *marten.Ctx) error {
    // API routes return 404
    if strings.HasPrefix(c.Path(), "/api/") {
        return c.NotFound("endpoint not found")
    }

    // SPA fallback
    return serveFile(c, "./public/index.html")
})
```

## Download Handler

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

## Key Features

| Feature | Usage |
|---------|-------|
| Wildcard routes | `*filepath` captures remaining path |
| `c.Param("filepath")` | Access captured path |
| Security | Check for `..` to prevent traversal |
| Content-Type | Auto-detect from file extension |
