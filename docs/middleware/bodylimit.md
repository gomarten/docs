# Body Limit Middleware

Limits the maximum size of request bodies.

## Usage

```go
app.Use(middleware.BodyLimit(10 * middleware.MB))
```

## Size Constants

| Constant | Value |
|----------|-------|
| `middleware.KB` | 1,024 bytes |
| `middleware.MB` | 1,024 KB |
| `middleware.GB` | 1,024 MB |

## Examples

### Global Limit

```go
// 10 MB limit for all requests
app.Use(middleware.BodyLimit(10 * middleware.MB))
```

### Per-Group Limits

```go
// Strict limit for JSON API
api := app.Group("/api")
api.Use(middleware.BodyLimit(1 * middleware.MB))

// Relaxed limit for file uploads
uploads := app.Group("/uploads")
uploads.Use(middleware.BodyLimit(100 * middleware.MB))
```

### Specific Sizes

```go
middleware.BodyLimit(100 * middleware.KB) // 100 KB
middleware.BodyLimit(5 * middleware.MB)   // 5 MB
middleware.BodyLimit(1 * middleware.GB)   // 1 GB
middleware.BodyLimit(500 * 1024)          // 500 KB (custom)
```

## Response

When the body exceeds the limit:

```
HTTP/1.1 413 Request Entity Too Large
Content-Type: application/json; charset=utf-8

{"error": "request body too large"}
```

## How It Works

1. **Fast path** — if `Content-Length` is present and already exceeds the limit, the request is rejected immediately before reading any bytes
2. **Streaming path** — wraps the request body with a counting reader; if bytes read exceed the limit during the handler's read, the middleware intercepts the error on the way back and responds 413

This covers both normal requests (with `Content-Length`) and chunked-encoded requests (without `Content-Length`).

!!! note "Guaranteed 413 for all transports"
    Prior to v0.1.4, over-limit chunked bodies could fall through to `OnError` and produce a 500. This is fixed — both `Content-Length`-gated and streaming reads now consistently return 413.

## Best Practices

1. **Set appropriate limits** — balance security and usability
2. **Use per-group limits** — different endpoints often have very different needs
3. **Document your limits** — include them in API docs so clients can handle 413 gracefully
4. **Pair with BodyLimit for uploads** — combine with `multipart/form-data` parsing for file upload endpoints

## Common Limits by Use Case

| Use Case | Suggested Limit |
|----------|-----------------|
| JSON API | 1 MB |
| Form submission | 10 MB |
| Avatar / image upload | 5–20 MB |
| Document upload | 50–100 MB |
| Large file upload | Up to 1 GB |
