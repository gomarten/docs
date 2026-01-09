# Routing

Marten uses a radix tree router for fast path matching with support for static routes, path parameters, and wildcards.

## Basic Routes

Register routes using HTTP method helpers:

```go
app.GET("/users", listUsers)
app.POST("/users", createUser)
app.PUT("/users/:id", updateUser)
app.DELETE("/users/:id", deleteUser)
app.PATCH("/users/:id", patchUser)
```

Or use the generic `Handle` method:

```go
app.Handle("GET", "/users", listUsers)
```

All HTTP methods are supported:

```go
app.GET("/resource", handler)
app.POST("/resource", handler)
app.PUT("/resource", handler)
app.DELETE("/resource", handler)
app.PATCH("/resource", handler)
app.HEAD("/resource", handler)
app.OPTIONS("/resource", handler)
```

## Path Parameters

Capture dynamic segments with `:param`:

```go
app.GET("/users/:id", func(c *marten.Ctx) error {
    id := c.Param("id")
    return c.OK(marten.M{"id": id})
})

// GET /users/123 -> {"id": "123"}
```

### Multiple Parameters

```go
app.GET("/users/:userId/posts/:postId", func(c *marten.Ctx) error {
    userId := c.Param("userId")
    postId := c.Param("postId")
    return c.OK(marten.M{
        "user_id": userId,
        "post_id": postId,
    })
})

// GET /users/42/posts/7 -> {"user_id": "42", "post_id": "7"}
```

### Typed Parameters

```go
app.GET("/users/:id", func(c *marten.Ctx) error {
    // String
    id := c.Param("id")
    
    // Integer (returns 0 if invalid)
    idInt := c.ParamInt("id")
    
    // Int64 (for large IDs)
    idInt64 := c.ParamInt64("id")
    
    return c.OK(marten.M{"id": idInt})
})
```

## Wildcard Routes

Capture the rest of the path with `*param`:

```go
app.GET("/files/*filepath", func(c *marten.Ctx) error {
    filepath := c.Param("filepath")
    return c.Text(200, "File: "+filepath)
})

// GET /files/images/photo.png -> "File: images/photo.png"
// GET /files/docs/api/v1.md  -> "File: docs/api/v1.md"
```

### Use Cases

=== "Static Files"

    ```go
    app.GET("/static/*path", func(c *marten.Ctx) error {
        path := c.Param("path")
        return serveFile("./public/" + path)
    })
    ```

=== "SPA Fallback"

    ```go
    app.NotFound(func(c *marten.Ctx) error {
        // Serve index.html for client-side routing
        return serveFile("./public/index.html")
    })
    ```

=== "Proxy"

    ```go
    app.GET("/api/*path", func(c *marten.Ctx) error {
        path := c.Param("path")
        return proxyTo("http://backend/" + path)
    })
    ```

## Route Priority

Routes are matched in this order:

1. **Static routes** (exact match)
2. **Parameter routes** (`:param`)
3. **Wildcard routes** (`*param`)

```go
app.GET("/users/me", getCurrentUser)     // 1. Static
app.GET("/users/:id", getUser)           // 2. Parameter
app.GET("/files/*path", serveFiles)      // 3. Wildcard

// GET /users/me    -> getCurrentUser (static match)
// GET /users/123   -> getUser (param match)
// GET /files/a/b/c -> serveFiles (wildcard match)
```

## Route Groups

Organize routes with shared prefixes:

```go
api := app.Group("/api/v1")
{
    api.GET("/users", listUsers)      // GET /api/v1/users
    api.POST("/users", createUser)    // POST /api/v1/users
    api.GET("/users/:id", getUser)    // GET /api/v1/users/:id
}
```

### Nested Groups

```go
api := app.Group("/api")
{
    v1 := api.Group("/v1")
    {
        v1.GET("/users", listUsersV1)
    }
    
    v2 := api.Group("/v2")
    {
        v2.GET("/users", listUsersV2)
    }
}
```

See [Route Groups](groups.md) for more details.

## Route-Specific Middleware

Add middleware to specific routes:

```go
app.GET("/admin", adminHandler, authMiddleware, adminMiddleware)
```

Or to groups:

```go
admin := app.Group("/admin")
admin.Use(authMiddleware)
admin.GET("/dashboard", dashboard)
admin.GET("/users", adminUsers)
```

## Custom 404 Handler

```go
app.NotFound(func(c *marten.Ctx) error {
    return c.JSON(404, marten.M{
        "error": "not found",
        "path":  c.Path(),
    })
})
```

## 405 Method Not Allowed

When a path exists but the HTTP method doesn't match, Marten returns 405:

```go
app.GET("/users", listUsers)
app.POST("/users", createUser)

// DELETE /users returns:
// HTTP/1.1 405 Method Not Allowed
// Allow: GET, POST
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

Marten detects conflicting parameter routes at registration:

```go
app.GET("/users/:id", getUser)
app.GET("/users/:name", getUserByName) // Panics!
```

## Query Parameters

Query parameters are accessed via the context, not the router:

```go
app.GET("/search", func(c *marten.Ctx) error {
    q := c.Query("q")
    page := c.QueryInt("page")
    limit := c.QueryInt("limit")
    
    return c.OK(marten.M{
        "query": q,
        "page":  page,
        "limit": limit,
    })
})

// GET /search?q=golang&page=1&limit=10
```

## Best Practices

### 1. Use Consistent Naming

```go
// Good - consistent plural nouns
app.GET("/users", listUsers)
app.GET("/users/:id", getUser)
app.GET("/posts", listPosts)
app.GET("/posts/:id", getPost)

// Avoid - inconsistent
app.GET("/user", listUsers)
app.GET("/getUser/:id", getUser)
```

### 2. Use HTTP Methods Correctly

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET | Retrieve resource | Yes |
| POST | Create resource | No |
| PUT | Replace resource | Yes |
| PATCH | Partial update | No |
| DELETE | Remove resource | Yes |

### 3. Version Your API

```go
v1 := app.Group("/api/v1")
v2 := app.Group("/api/v2")
```

### 4. Keep Routes Organized

```go
// Group related routes
func registerUserRoutes(app *marten.App) {
    users := app.Group("/users")
    users.GET("", listUsers)
    users.POST("", createUser)
    users.GET("/:id", getUser)
    users.PUT("/:id", updateUser)
    users.DELETE("/:id", deleteUser)
}
```

## Next Steps

[:octicons-arrow-right-24: Learn about the Context](context.md)
