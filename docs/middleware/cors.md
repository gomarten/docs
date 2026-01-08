# CORS Middleware

Handles Cross-Origin Resource Sharing (CORS) for browser requests.

## Usage

### Default Configuration

```go
app.Use(middleware.CORS(middleware.DefaultCORSConfig()))
```

Default allows:
- All origins (`*`)
- Methods: GET, POST, PUT, DELETE, PATCH, OPTIONS
- Headers: Origin, Content-Type, Accept, Authorization

### Custom Configuration

```go
app.Use(middleware.CORS(middleware.CORSConfig{
    AllowOrigins:     []string{"https://example.com", "https://app.example.com"},
    AllowMethods:     []string{"GET", "POST", "PUT", "DELETE"},
    AllowHeaders:     []string{"Origin", "Content-Type", "Authorization"},
    AllowCredentials: true,
}))
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `AllowOrigins` | `[]string` | Allowed origins. Use `*` for all. |
| `AllowMethods` | `[]string` | Allowed HTTP methods |
| `AllowHeaders` | `[]string` | Allowed request headers |
| `AllowCredentials` | `bool` | Allow credentials (cookies, auth) |

## Examples

### Single Origin

```go
app.Use(middleware.CORS(middleware.CORSConfig{
    AllowOrigins: []string{"https://myapp.com"},
    AllowMethods: []string{"GET", "POST"},
}))
```

### Multiple Origins

```go
app.Use(middleware.CORS(middleware.CORSConfig{
    AllowOrigins: []string{
        "https://app.example.com",
        "https://admin.example.com",
        "http://localhost:3000", // Development
    },
    AllowMethods: []string{"GET", "POST", "PUT", "DELETE"},
    AllowHeaders: []string{"Origin", "Content-Type", "Authorization"},
}))
```

### With Credentials

```go
app.Use(middleware.CORS(middleware.CORSConfig{
    AllowOrigins:     []string{"https://app.example.com"},
    AllowMethods:     []string{"GET", "POST"},
    AllowCredentials: true, // Required for cookies
}))
```

!!! warning "Credentials and Wildcards"
    `AllowCredentials: true` cannot be used with `AllowOrigins: []string{"*"}`. This is a security restriction enforced by browsers.

## Preflight Requests

The middleware automatically handles OPTIONS preflight requests:

```
OPTIONS /api/users HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type

HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Origin, Content-Type, Authorization
```

## Response Headers

The middleware sets these headers:

| Header | Description |
|--------|-------------|
| `Access-Control-Allow-Origin` | Allowed origin |
| `Access-Control-Allow-Methods` | Allowed methods |
| `Access-Control-Allow-Headers` | Allowed headers |
| `Access-Control-Allow-Credentials` | If credentials allowed |

## Security Considerations

1. **Don't use `*` in production** with credentials
2. **Whitelist specific origins** instead of allowing all
3. **Limit allowed methods** to what's needed
4. **Limit allowed headers** to what's needed
