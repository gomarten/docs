# Router

HTTP routing with radix tree implementation.

## Route Registration

```go
app.GET(path string, h Handler, mw ...Middleware)
app.POST(path string, h Handler, mw ...Middleware)
app.PUT(path string, h Handler, mw ...Middleware)
app.DELETE(path string, h Handler, mw ...Middleware)
app.PATCH(path string, h Handler, mw ...Middleware)
app.HEAD(path string, h Handler, mw ...Middleware)
app.OPTIONS(path string, h Handler, mw ...Middleware)
app.Handle(method, path string, h Handler, mw ...Middleware)
```

## Path Parameters

Named parameters with `:name`:

```go
app.GET("/users/:id", func(c *marten.Ctx) error {
    id := c.Param("id")
    return c.OK(marten.M{"id": id})
})

app.GET("/posts/:year/:month", func(c *marten.Ctx) error {
    year := c.Param("year")
    month := c.Param("month")
    return c.OK(marten.M{"year": year, "month": month})
})
```

## Wildcard Routes

Capture remaining path with `*name`:

```go
app.GET("/files/*filepath", func(c *marten.Ctx) error {
    path := c.Param("filepath")
    // /files/images/logo.png → filepath = "images/logo.png"
    return c.OK(marten.M{"path": path})
})
```

## Route Priority

1. Static segments (exact match)
2. Named parameters (`:id`)
3. Wildcards (`*filepath`)

```go
app.GET("/users/new", newUser)      // Matches first
app.GET("/users/:id", getUser)      // Matches /users/123
app.GET("/files/*path", serveFile)  // Matches /files/any/path
```

## Route Groups

```go
api := app.Group("/api/v1")
api.Use(authMiddleware)

api.GET("/users", listUsers)
api.POST("/users", createUser)
```

## Middleware

### Global Middleware

```go
app.Use(middleware.Logger)
```

### Route-Specific Middleware

```go
app.GET("/admin", adminHandler, adminAuth)
```

### Group Middleware

```go
admin := app.Group("/admin")
admin.Use(adminAuth)
admin.GET("/dashboard", dashboard)
```

## Custom 404

```go
app.NotFound(func(c *marten.Ctx) error {
    return c.JSON(404, marten.M{
        "error": "endpoint not found",
        "path":  c.Path(),
    })
})
```

## 405 Method Not Allowed

When a path exists but the HTTP method doesn't match, Marten returns 405 with an `Allow` header:

```
HTTP/1.1 405 Method Not Allowed
Allow: GET, POST
Content-Type: text/plain

Method Not Allowed
```

## Trailing Slash Handling

Configure how trailing slashes are handled:

```go
// Ignore (default) - /users and /users/ match the same route
app.SetTrailingSlash(marten.TrailingSlashIgnore)

// Redirect - /users/ redirects to /users with 301
app.SetTrailingSlash(marten.TrailingSlashRedirect)

// Strict - /users and /users/ are different routes
app.SetTrailingSlash(marten.TrailingSlashStrict)
```

## Route Conflict Detection

Marten detects conflicting parameter routes at registration time:

```go
app.GET("/users/:id", getUser)
app.GET("/users/:name", getUserByName) // Panics: conflicts with :id
```

Same parameter name is allowed for different methods:

```go
app.GET("/users/:id", getUser)   // OK
app.POST("/users/:id", updateUser) // OK - same param name
```

## List Registered Routes

```go
routes := app.Routes()
for _, r := range routes {
    fmt.Printf("%s %s\n", r.Method, r.Path)
}
```

## Handler Type

```go
type Handler func(*Ctx) error
```

## Middleware Type

```go
type Middleware func(Handler) Handler
```
