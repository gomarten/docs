# Route Groups

Route groups help organize routes with shared prefixes and middleware.

## Basic Groups

Create a group with a prefix:

```go
api := app.Group("/api")
{
    api.GET("/users", listUsers)      // GET /api/users
    api.POST("/users", createUser)    // POST /api/users
    api.GET("/posts", listPosts)      // GET /api/posts
}
```

## Nested Groups

Groups can be nested:

```go
api := app.Group("/api")
{
    v1 := api.Group("/v1")
    {
        v1.GET("/users", listUsersV1)  // GET /api/v1/users
        v1.GET("/posts", listPostsV1)  // GET /api/v1/posts
    }
    
    v2 := api.Group("/v2")
    {
        v2.GET("/users", listUsersV2)  // GET /api/v2/users
        v2.GET("/posts", listPostsV2)  // GET /api/v2/posts
    }
}
```

## Group Middleware

Add middleware to a group:

```go
// Method 1: In Group() call
api := app.Group("/api", authMiddleware)

// Method 2: Using Use()
api := app.Group("/api")
api.Use(authMiddleware)

// Both apply to all routes in the group
api.GET("/users", listUsers)    // Has authMiddleware
api.POST("/users", createUser)  // Has authMiddleware
```

### Multiple Middleware

```go
api := app.Group("/api")
api.Use(
    middleware.RateLimit(rateLimitConfig),
    authMiddleware,
    loggingMiddleware,
)
```

### Middleware Inheritance

Nested groups inherit parent middleware:

```go
api := app.Group("/api")
api.Use(authMiddleware)  // Applied to all /api routes

admin := api.Group("/admin")
admin.Use(adminMiddleware)  // Applied in addition to authMiddleware

admin.GET("/users", adminUsers)  // Has both authMiddleware AND adminMiddleware
```

## Common Patterns

### API Versioning

```go
func setupRoutes(app *marten.App) {
    // V1 API
    v1 := app.Group("/api/v1")
    v1.Use(v1Middleware)
    setupV1Routes(v1)
    
    // V2 API
    v2 := app.Group("/api/v2")
    v2.Use(v2Middleware)
    setupV2Routes(v2)
}

func setupV1Routes(g *marten.Group) {
    g.GET("/users", listUsersV1)
    g.POST("/users", createUserV1)
}

func setupV2Routes(g *marten.Group) {
    g.GET("/users", listUsersV2)
    g.POST("/users", createUserV2)
}
```

### Public vs Protected Routes

```go
// Public routes
app.GET("/", home)
app.GET("/health", health)
app.POST("/auth/login", login)
app.POST("/auth/register", register)

// Protected routes
api := app.Group("/api")
api.Use(authMiddleware)
{
    api.GET("/me", getMe)
    api.GET("/profile", getProfile)
    api.PUT("/profile", updateProfile)
}

// Admin routes
admin := app.Group("/admin")
admin.Use(authMiddleware, adminMiddleware)
{
    admin.GET("/dashboard", dashboard)
    admin.GET("/users", adminListUsers)
    admin.DELETE("/users/:id", adminDeleteUser)
}
```

### Resource Groups

```go
func registerUserRoutes(app *marten.App) {
    users := app.Group("/users")
    {
        users.GET("", listUsers)
        users.POST("", createUser)
        users.GET("/:id", getUser)
        users.PUT("/:id", updateUser)
        users.DELETE("/:id", deleteUser)
        
        // Nested resources
        users.GET("/:id/posts", getUserPosts)
        users.POST("/:id/posts", createUserPost)
    }
}

func registerPostRoutes(app *marten.App) {
    posts := app.Group("/posts")
    {
        posts.GET("", listPosts)
        posts.POST("", createPost)
        posts.GET("/:id", getPost)
        posts.PUT("/:id", updatePost)
        posts.DELETE("/:id", deletePost)
        
        // Nested resources
        posts.GET("/:id/comments", getPostComments)
        posts.POST("/:id/comments", createPostComment)
    }
}
```

### Feature Modules

```go
// users/routes.go
package users

func RegisterRoutes(app *marten.App) {
    g := app.Group("/users")
    g.Use(middleware.Auth)
    
    g.GET("", List)
    g.POST("", Create)
    g.GET("/:id", Get)
    g.PUT("/:id", Update)
    g.DELETE("/:id", Delete)
}

// main.go
func main() {
    app := marten.New()
    
    users.RegisterRoutes(app)
    posts.RegisterRoutes(app)
    comments.RegisterRoutes(app)
    
    app.Run(":3000")
}
```

### Webhooks

```go
webhooks := app.Group("/webhooks")
webhooks.Use(webhookAuthMiddleware)
{
    webhooks.POST("/github", handleGitHub)
    webhooks.POST("/stripe", handleStripe)
    webhooks.POST("/slack", handleSlack)
}
```

## Group Methods

Groups support all HTTP methods:

```go
g := app.Group("/api")

g.GET(path, handler)
g.POST(path, handler)
g.PUT(path, handler)
g.DELETE(path, handler)
g.PATCH(path, handler)
g.Handle(method, path, handler)
```

## Route-Specific Middleware in Groups

Add middleware to specific routes within a group:

```go
api := app.Group("/api")
api.Use(authMiddleware)

// All routes have authMiddleware
api.GET("/users", listUsers)
api.POST("/users", createUser)

// This route also has adminMiddleware
api.DELETE("/users/:id", deleteUser, adminMiddleware)
```

## Best Practices

### 1. Use Meaningful Prefixes

```go
// Good
app.Group("/api/v1")
app.Group("/admin")
app.Group("/webhooks")

// Avoid
app.Group("/a")
app.Group("/stuff")
```

### 2. Keep Groups Focused

```go
// Good - each group has a clear purpose
users := app.Group("/users")
posts := app.Group("/posts")

// Avoid - mixing unrelated routes
misc := app.Group("/misc")
misc.GET("/users", listUsers)
misc.GET("/weather", getWeather)
```

### 3. Apply Middleware at the Right Level

```go
// Global middleware for all routes
app.Use(middleware.Logger, middleware.Recover)

// API middleware for API routes
api := app.Group("/api")
api.Use(middleware.RateLimit(cfg))

// Auth middleware for protected routes
protected := api.Group("/protected")
protected.Use(authMiddleware)
```

### 4. Document Your Groups

```go
// Public API endpoints
// No authentication required
public := app.Group("/api/public")

// Protected API endpoints
// Requires valid JWT token
protected := app.Group("/api")
protected.Use(jwtMiddleware)

// Admin endpoints
// Requires admin role
admin := app.Group("/admin")
admin.Use(jwtMiddleware, adminMiddleware)
```

## Next Steps

[:octicons-arrow-right-24: Learn about Error Handling](errors.md)
