# Compress Middleware

Gzip compression for responses.

## Usage

### Default Configuration

```go
app.Use(middleware.Compress(middleware.DefaultCompressConfig()))
```

### Custom Configuration

```go
app.Use(middleware.Compress(middleware.CompressConfig{
    Level:        gzip.BestSpeed,
    MinSize:      1024,
    ContentTypes: []string{
        "text/plain",
        "text/html",
        "application/json",
    },
}))
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `Level` | `int` | `gzip.DefaultCompression` | Compression level |
| `MinSize` | `int` | `1024` | Minimum size to compress |
| `ContentTypes` | `[]string` | See below | Content types to compress |

### Default Content Types

- `text/plain`
- `text/html`
- `text/css`
- `text/javascript`
- `application/json`
- `application/javascript`
- `application/xml`

### Compression Levels

| Level | Constant | Description |
|-------|----------|-------------|
| -1 | `gzip.DefaultCompression` | Balance of speed and size |
| 0 | `gzip.NoCompression` | No compression |
| 1 | `gzip.BestSpeed` | Fastest compression |
| 9 | `gzip.BestCompression` | Smallest size |

## Examples

### Fast Compression

```go
app.Use(middleware.Compress(middleware.CompressConfig{
    Level: gzip.BestSpeed,
}))
```

### Best Compression

```go
app.Use(middleware.Compress(middleware.CompressConfig{
    Level: gzip.BestCompression,
}))
```

### Custom Content Types

```go
app.Use(middleware.Compress(middleware.CompressConfig{
    ContentTypes: []string{
        "application/json",
        "text/html",
        "text/css",
        "application/javascript",
    },
}))
```

### Higher Threshold

```go
// Only compress responses > 2KB
app.Use(middleware.Compress(middleware.CompressConfig{
    MinSize: 2048,
}))
```

## How It Works

1. Checks if client accepts gzip (`Accept-Encoding: gzip`)
2. Checks if response content type is compressible
3. Buffers response until `MinSize` is reached
4. If large enough, compresses with gzip
5. Sets `Content-Encoding: gzip` header

## Response Headers

When compression is applied:

```
Content-Encoding: gzip
```

The `Content-Length` header is removed since the compressed size differs.

## Best Practices

1. **Don't compress already compressed content** - Images, videos, etc.
2. **Set appropriate MinSize** - Small responses don't benefit from compression
3. **Consider CPU usage** - Compression uses CPU
4. **Use BestSpeed for high-traffic** - Lower CPU usage
