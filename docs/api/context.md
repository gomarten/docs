# Context (Ctx)

Request context passed to every handler. Wraps the request and response with clean, chainable helpers.

## Properties

```go
c.Request *http.Request
c.Writer  http.ResponseWriter
```

## Response Methods

### JSON

```go
c.JSON(code int, v any) error
c.OK(v any) error        // 200
c.Created(v any) error   // 201
c.Accepted(v any) error  // 202  — async / queued operations
```

### Text

```go
c.Text(code int, text string) error
```

### HTML

```go
c.HTML(code int, html string) error
```

### Binary

```go
c.Blob(code int, contentType string, data []byte) error
```

### Stream

```go
c.Stream(code int, contentType string, r io.Reader) error
```

Passing `nil` as the reader writes headers only without a body.

### No Content

```go
c.NoContent() error  // 204
```

### Error Responses

All error helpers return `{"error": "message"}` as JSON.

```go
c.BadRequest(message string) error    // 400
c.Unauthorized(message string) error  // 401
c.Forbidden(message string) error     // 403
c.NotFound(message string) error      // 404
c.ServerError(message string) error   // 500
```

### Status & Headers

```go
c.Status(code int) *Ctx           // Set status, return *Ctx for chaining
c.StatusCode() int                // Read current status (0 if not written)
c.Header(key, value string) *Ctx  // Set response header, chainable
c.GetHeader(key string) string    // Read request header
c.Written() bool                  // True if response has been written
c.Redirect(code int, url string) error
```

## Request Data

### Path Parameters

```go
c.Param(name string) string
c.ParamInt(name string) int     // 0 if invalid or non-numeric
c.ParamInt64(name string) int64 // 0 if invalid
```

### Query Parameters

```go
c.Query(name string) string
c.QueryInt(name string) int          // 0 if invalid
c.QueryInt64(name string) int64      // 0 if invalid
c.QueryFloat64(name string) float64  // 0 if invalid
c.QueryBool(name string) bool        // false if invalid
c.QueryDefault(name, def string) string
c.QueryValues(name string) []string  // all values for repeated keys
c.QueryParams() url.Values           // full query string as url.Values
```

### Body Binding

```go
c.Bind(v any) error
c.BindValid(v any, validate func() error) error
```

`Bind()` detects the content type automatically:

| Content-Type | Behaviour |
|---|---|
| `application/json` | Decodes JSON body |
| `application/x-www-form-urlencoded` | Parses form fields |
| `multipart/form-data` | Parses multipart form |
| _(anything else)_ | Falls back to JSON |

Returns `*BindError` for empty bodies, invalid JSON, or failed form parsing.

### Form & Files

```go
c.FormValue(name string) string
c.File(name string) (*multipart.FileHeader, error)
```

### Cookies

```go
c.Cookie(name string) string
c.SetCookie(cookie *http.Cookie)
```

## Request Helpers

```go
c.Method() string       // "GET", "POST", …
c.Path() string         // "/users/123"
c.ClientIP() string     // Respects X-Forwarded-For and X-Real-IP
c.Bearer() string       // Token from "Authorization: Bearer <token>"
c.RequestID() string    // X-Request-ID header or auto-generated 16-char hex
c.IsJSON() bool         // Content-Type starts with application/json
c.IsAJAX() bool         // X-Requested-With: XMLHttpRequest
c.Context() context.Context
```

## Request-Scoped Storage

Store values for the lifetime of a request. Typically set in middleware, read in handlers.

```go
c.Set(key string, value any)
c.Get(key string) any

// Typed getters — return zero value if key missing or wrong type
c.GetString(key string) string
c.GetInt(key string) int
c.GetBool(key string) bool
c.GetFloat64(key string) float64
```

## Convenience Types

```go
// map[string]any shorthand
marten.M{"name": "Alice", "age": 30}

// {"error": "message"} shorthand
marten.E("invalid input")
```

## Creating a Bare Context

```go
// Useful in custom middleware that needs an isolated Ctx
c := marten.NewCtx(w, r)
```

## Examples

### Standard JSON Handler

```go
func getUser(c *marten.Ctx) error {
    id := c.ParamInt("id")
    user, err := db.FindUser(id)
    if err != nil {
        return c.NotFound("user not found")
    }
    return c.OK(user)
}
```

### Typed Query Params

```go
func listProducts(c *marten.Ctx) error {
    page     := c.QueryInt("page")
    maxPrice := c.QueryFloat64("max_price")
    inStock  := c.QueryBool("in_stock")
    sort     := c.QueryDefault("sort", "created_at")

    products, err := db.SearchProducts(page, maxPrice, inStock, sort)
    if err != nil {
        return c.ServerError(err.Error())
    }
    return c.OK(products)
}
```

### 202 Accepted for Async Jobs

```go
func enqueueExport(c *marten.Ctx) error {
    var req ExportRequest
    if err := c.Bind(&req); err != nil {
        return c.BadRequest(err.Error())
    }

    jobID := queue.Enqueue(req)
    return c.Accepted(marten.M{
        "job_id": jobID,
        "status": "queued",
    })
}
```

### Context Store in Middleware + Handler

```go
func AuthMiddleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        claims, err := validateToken(c.Bearer())
        if err != nil {
            return c.Unauthorized("invalid token")
        }
        c.Set("user_id", claims.UserID)
        c.Set("role", claims.Role)
        c.Set("score", claims.TrustScore)  // float64
        return next(c)
    }
}

func handler(c *marten.Ctx) error {
    userID := c.GetString("user_id")
    role   := c.GetString("role")
    score  := c.GetFloat64("score")
    return c.OK(marten.M{"user": userID, "role": role, "score": score})
}
```

### Streaming Response

```go
func downloadFile(c *marten.Ctx) error {
    f, err := os.Open("large-file.zip")
    if err != nil {
        return c.NotFound("file not found")
    }
    defer f.Close()

    c.Header("Content-Disposition", "attachment; filename=file.zip")
    return c.Stream(200, "application/octet-stream", f)
}
```

### Request Validation with BindValid

```go
func createUser(c *marten.Ctx) error {
    var input struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }

    if err := c.BindValid(&input, func() error {
        if input.Name == "" {
            return &marten.BindError{Message: "name required"}
        }
        if input.Email == "" {
            return &marten.BindError{Message: "email required"}
        }
        return nil
    }); err != nil {
        return c.BadRequest(err.Error())
    }

    return c.Created(input)
}
```
