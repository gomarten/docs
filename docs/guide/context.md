# Context

The context (`*marten.Ctx`) is passed to every handler and provides access to the request, response, and helpful utilities.

## Request Data

### Path Parameters

```go
app.GET("/users/:id/posts/:postId", func(c *marten.Ctx) error {
    // String
    id := c.Param("id")
    
    // Integer (0 if invalid)
    idInt := c.ParamInt("id")
    
    // Int64 (for large IDs)
    idInt64 := c.ParamInt64("id")
    
    // Wildcard
    filepath := c.Param("filepath") // for /files/*filepath
    
    return c.OK(marten.M{"id": idInt})
})
```

### Query Parameters

```go
app.GET("/search", func(c *marten.Ctx) error {
    // String
    q := c.Query("q")
    
    // Integer
    page := c.QueryInt("page")
    
    // Int64
    cursor := c.QueryInt64("cursor")
    
    // Boolean
    active := c.QueryBool("active")
    
    // With default
    sort := c.QueryDefault("sort", "created_at")
    
    // Multiple values (?tag=a&tag=b)
    tags := c.QueryValues("tag") // []string{"a", "b"}
    
    return c.OK(marten.M{
        "query":  q,
        "page":   page,
        "sort":   sort,
        "tags":   tags,
        "active": active,
    })
})
```

### Request Info

```go
func handler(c *marten.Ctx) error {
    method := c.Method()           // "GET", "POST", etc.
    path := c.Path()               // "/users/123"
    clientIP := c.ClientIP()       // Smart IP extraction
    requestID := c.RequestID()     // Unique request ID
    ctx := c.Context()             // context.Context
    
    return c.OK(marten.M{
        "method":     method,
        "path":       path,
        "client_ip":  clientIP,
        "request_id": requestID,
    })
}
```

### Headers

```go
func handler(c *marten.Ctx) error {
    // Get request header
    auth := c.Request.Header.Get("Authorization")
    
    // Extract Bearer token
    token := c.Bearer() // Returns token from "Bearer <token>"
    
    // Check content type
    if c.IsJSON() {
        // Content-Type is application/json
    }
    
    // Check AJAX
    if c.IsAJAX() {
        // X-Requested-With is XMLHttpRequest
    }
    
    return c.OK(nil)
}
```

### Cookies

```go
func handler(c *marten.Ctx) error {
    // Get cookie
    session := c.Cookie("session")
    
    // Set cookie
    c.SetCookie(&http.Cookie{
        Name:     "session",
        Value:    "abc123",
        Path:     "/",
        HttpOnly: true,
        Secure:   true,
        MaxAge:   86400,
    })
    
    return c.OK(nil)
}
```

### Form Data

```go
func handler(c *marten.Ctx) error {
    // Form value
    name := c.FormValue("name")
    
    // File upload
    file, err := c.File("avatar")
    if err != nil {
        return c.BadRequest("no file uploaded")
    }
    
    // Process file...
    
    return c.OK(marten.M{"filename": file.Filename})
}
```

## JSON Binding

### Basic Binding

```go
func createUser(c *marten.Ctx) error {
    var input struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    
    if err := c.Bind(&input); err != nil {
        return c.BadRequest(err.Error())
    }
    
    return c.Created(input)
}
```

### Binding with Validation

```go
func createUser(c *marten.Ctx) error {
    var input struct {
        Name  string `json:"name"`
        Email string `json:"email"`
        Age   int    `json:"age"`
    }
    
    err := c.BindValid(&input, func() error {
        if input.Name == "" {
            return &marten.BindError{Message: "name is required"}
        }
        if input.Email == "" {
            return &marten.BindError{Message: "email is required"}
        }
        if input.Age < 0 || input.Age > 150 {
            return &marten.BindError{Message: "invalid age"}
        }
        return nil
    })
    
    if err != nil {
        return c.BadRequest(err.Error())
    }
    
    return c.Created(input)
}
```

## Responses

### JSON Responses

```go
// Generic JSON
c.JSON(200, data)

// Success helpers
c.OK(data)           // 200
c.Created(data)      // 201
c.NoContent()        // 204

// Error helpers
c.BadRequest("message")    // 400
c.Unauthorized("message")  // 401
c.Forbidden("message")     // 403
c.NotFound("message")      // 404
c.ServerError("message")   // 500
```

Error helpers return JSON:
```json
{"error": "message"}
```

### Text Response

```go
c.Text(200, "Hello, World!")
```

### Redirect

```go
c.Redirect(302, "/new-location")
c.Redirect(301, "/permanent-location")
```

### Custom Status

```go
c.Status(202) // Just set status, no body
```

### Headers

```go
// Set response header
c.Header("X-Custom", "value")

// Chain headers
c.Header("X-One", "1").Header("X-Two", "2")
```

## Request-Scoped Storage

Store data for the duration of a request:

```go
// In middleware
func AuthMiddleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        user := validateToken(c.Bearer())
        c.Set("user", user)
        c.Set("user_id", user.ID)
        c.Set("is_admin", user.IsAdmin)
        return next(c)
    }
}

// In handler
func handler(c *marten.Ctx) error {
    // Get any type
    user := c.Get("user").(User)
    
    // Get string
    userID := c.GetString("user_id")
    
    // Get int
    count := c.GetInt("count")
    
    // Get bool
    isAdmin := c.GetBool("is_admin")
    
    return c.OK(user)
}
```

## Convenience Types

### marten.M

Shorthand for `map[string]any`:

```go
// Instead of
c.JSON(200, map[string]any{
    "name": "Alice",
    "age":  30,
})

// Use
c.OK(marten.M{
    "name": "Alice",
    "age":  30,
})
```

### marten.E

Quick error response:

```go
// Returns {"error": "message"}
c.JSON(400, marten.E("invalid input"))
```

## Raw Access

Access the underlying request and response:

```go
func handler(c *marten.Ctx) error {
    // Raw request
    r := c.Request
    
    // Raw response writer
    w := c.Writer
    
    // Use standard library
    http.ServeFile(w, r, "file.txt")
    
    return nil
}
```

## Context Lifecycle

The context is pooled and reused between requests. Don't store references to it outside the handler:

```go
// Bad - context will be reused
var savedCtx *marten.Ctx
func handler(c *marten.Ctx) error {
    savedCtx = c // Don't do this!
    return c.OK(nil)
}

// Good - copy what you need
func handler(c *marten.Ctx) error {
    userID := c.Param("id") // Copy the value
    go processAsync(userID) // Use the copy
    return c.OK(nil)
}
```

## Next Steps

[:octicons-arrow-right-24: Learn about Middleware](middleware.md)
