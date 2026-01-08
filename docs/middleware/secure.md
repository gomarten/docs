# Secure Middleware

Sets security-related HTTP headers.

## Usage

### Default Configuration

```go
app.Use(middleware.SecureDefault)
```

### Custom Configuration

```go
app.Use(middleware.Secure(middleware.SecureConfig{
    XSSProtection:         "1; mode=block",
    ContentTypeNosniff:    "nosniff",
    XFrameOptions:         "DENY",
    HSTSMaxAge:            31536000,
    HSTSIncludeSubdomains: true,
    ContentSecurityPolicy: "default-src 'self'",
    ReferrerPolicy:        "strict-origin-when-cross-origin",
}))
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `XSSProtection` | `string` | `"1; mode=block"` | X-XSS-Protection header |
| `ContentTypeNosniff` | `string` | `"nosniff"` | X-Content-Type-Options header |
| `XFrameOptions` | `string` | `"SAMEORIGIN"` | X-Frame-Options header |
| `HSTSMaxAge` | `int` | `0` | HSTS max-age in seconds |
| `HSTSIncludeSubdomains` | `bool` | `false` | Include subdomains in HSTS |
| `ContentSecurityPolicy` | `string` | `""` | Content-Security-Policy header |
| `ReferrerPolicy` | `string` | `"strict-origin-when-cross-origin"` | Referrer-Policy header |

## Headers Explained

### X-XSS-Protection

Enables browser's XSS filter:

```
X-XSS-Protection: 1; mode=block
```

### X-Content-Type-Options

Prevents MIME type sniffing:

```
X-Content-Type-Options: nosniff
```

### X-Frame-Options

Controls iframe embedding:

| Value | Description |
|-------|-------------|
| `DENY` | Never allow framing |
| `SAMEORIGIN` | Allow same origin only |

### Strict-Transport-Security (HSTS)

Forces HTTPS:

```go
middleware.Secure(middleware.SecureConfig{
    HSTSMaxAge:            31536000, // 1 year
    HSTSIncludeSubdomains: true,
})
```

Produces:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Content-Security-Policy

Controls resource loading:

```go
middleware.Secure(middleware.SecureConfig{
    ContentSecurityPolicy: "default-src 'self'; script-src 'self' 'unsafe-inline'",
})
```

### Referrer-Policy

Controls referrer information:

| Value | Description |
|-------|-------------|
| `no-referrer` | Never send referrer |
| `same-origin` | Send for same origin only |
| `strict-origin-when-cross-origin` | Full URL for same origin, origin only for cross-origin |

## Examples

### API Security

```go
app.Use(middleware.Secure(middleware.SecureConfig{
    ContentTypeNosniff: "nosniff",
    XFrameOptions:      "DENY",
    ReferrerPolicy:     "no-referrer",
}))
```

### Web App Security

```go
app.Use(middleware.Secure(middleware.SecureConfig{
    XSSProtection:         "1; mode=block",
    ContentTypeNosniff:    "nosniff",
    XFrameOptions:         "SAMEORIGIN",
    HSTSMaxAge:            31536000,
    HSTSIncludeSubdomains: true,
    ContentSecurityPolicy: "default-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline'",
    ReferrerPolicy:        "strict-origin-when-cross-origin",
}))
```

## Best Practices

1. **Enable HSTS** for production HTTPS sites
2. **Use strict CSP** to prevent XSS
3. **Set X-Frame-Options** to prevent clickjacking
4. **Test thoroughly** - strict CSP can break functionality
