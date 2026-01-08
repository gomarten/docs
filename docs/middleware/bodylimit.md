# Body Limit Middleware

Limits the maximum size of request bodies.

## Usage

```go
app.Use(middleware.BodyLimit(10 * middleware.MB))
```

## Size Constants

| Constant | Value |
|----------|-------|
| `middleware.KB` | 1024 bytes |
| `middleware.MB` | 1024 KB |
| `middleware.GB` | 1024 MB |

## Examples

### Global Limit

```go
// 10 MB limit for all requests
app.Use(middleware.BodyLimit(10 * middleware.MB))
```

### Different Limits

```go
// Small limit for API
api := app.Group("/api")
api.Use(middleware.BodyLimit(1 * middleware.MB))

// Large limit for uploads
uploads := app.Group("/uploads")
uploads.Use(middleware.BodyLimit(100 * middleware.MB))
```

### Specific Sizes

```go
// 100 KB
middleware.BodyLimit(100 * middleware.KB)

// 5 MB
middleware.BodyLimit(5 * middleware.MB)

// 1 GB
middleware.BodyLimit(1 * middleware.GB)

// Custom (500 KB)
middleware.BodyLimit(500 * 1024)
```

## Response

When body exceeds limit:

```
HTTP/1.1 413 Request Entity Too Large
Content-Type: application/json

{"error": "request body too large"}
```

## How It Works

1. Checks `Content-Length` header first
2. If header exceeds limit, rejects immediately
3. Wraps request body with a limiting reader
4. If body exceeds limit during read, returns error

## Best Practices

1. **Set appropriate limits** - Balance security and usability
2. **Use different limits** for different endpoints
3. **Consider file upload endpoints** - May need larger limits
4. **Document limits** - Let API users know the limits

## Common Limits

| Use Case | Suggested Limit |
|----------|-----------------|
| JSON API | 1 MB |
| Form submission | 10 MB |
| File upload | 50-100 MB |
| Large file upload | 1 GB |
