# Project Structure

Marten doesn't enforce any project structure. Here are some recommended patterns.

## Simple Application

For small projects or microservices:

```
myapp/
├── main.go
├── go.mod
└── go.sum
```

Everything in one file works great for small services.

## Medium Application

For growing projects:

```
myapp/
├── main.go           # Entry point
├── handlers/         # HTTP handlers
│   ├── users.go
│   ├── posts.go
│   └── health.go
├── middleware/       # Custom middleware
│   └── auth.go
├── models/           # Data models
│   ├── user.go
│   └── post.go
├── go.mod
└── go.sum
```

### Example: `main.go`

```go
package main

import (
    "myapp/handlers"
    "myapp/middleware"
    
    "github.com/gomarten/marten"
    mw "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    
    // Global middleware
    app.Use(mw.Logger, mw.Recover)
    
    // Public routes
    app.GET("/health", handlers.Health)
    
    // API routes
    api := app.Group("/api/v1")
    api.Use(middleware.Auth)
    {
        api.GET("/users", handlers.ListUsers)
        api.POST("/users", handlers.CreateUser)
        api.GET("/users/:id", handlers.GetUser)
    }
    
    app.Run(":3000")
}
```

### Example: `handlers/users.go`

```go
package handlers

import "github.com/gomarten/marten"

func ListUsers(c *marten.Ctx) error {
    // ...
    return c.OK(users)
}

func CreateUser(c *marten.Ctx) error {
    // ...
    return c.Created(user)
}

func GetUser(c *marten.Ctx) error {
    id := c.Param("id")
    // ...
    return c.OK(user)
}
```

## Large Application

For larger projects with multiple domains:

```
myapp/
├── cmd/
│   └── server/
│       └── main.go       # Entry point
├── internal/
│   ├── app/
│   │   └── app.go        # App setup
│   ├── handlers/
│   │   ├── users/
│   │   │   ├── handlers.go
│   │   │   └── routes.go
│   │   └── posts/
│   │       ├── handlers.go
│   │       └── routes.go
│   ├── middleware/
│   │   ├── auth.go
│   │   └── logging.go
│   ├── models/
│   │   ├── user.go
│   │   └── post.go
│   └── store/
│       ├── store.go
│       └── postgres.go
├── pkg/                  # Shared packages
│   └── validator/
│       └── validator.go
├── go.mod
└── go.sum
```

### Example: `internal/app/app.go`

```go
package app

import (
    "myapp/internal/handlers/users"
    "myapp/internal/handlers/posts"
    "myapp/internal/middleware"
    
    "github.com/gomarten/marten"
    mw "github.com/gomarten/marten/middleware"
)

func New() *marten.App {
    app := marten.New()
    
    // Global middleware
    app.Use(
        mw.RequestID,
        mw.Logger,
        mw.Recover,
        mw.CORS(mw.DefaultCORSConfig()),
    )
    
    // Register routes
    users.RegisterRoutes(app)
    posts.RegisterRoutes(app)
    
    return app
}
```

### Example: `internal/handlers/users/routes.go`

```go
package users

import (
    "myapp/internal/middleware"
    "github.com/gomarten/marten"
)

func RegisterRoutes(app *marten.App) {
    g := app.Group("/api/v1/users")
    g.Use(middleware.Auth)
    
    g.GET("", List)
    g.POST("", Create)
    g.GET("/:id", Get)
    g.PUT("/:id", Update)
    g.DELETE("/:id", Delete)
}
```

## Best Practices

### 1. Keep Handlers Thin

Handlers should only handle HTTP concerns:

```go
// Good
func CreateUser(c *marten.Ctx) error {
    var input CreateUserInput
    if err := c.Bind(&input); err != nil {
        return c.BadRequest(err.Error())
    }
    
    user, err := userService.Create(input)
    if err != nil {
        return c.ServerError(err.Error())
    }
    
    return c.Created(user)
}

// Bad - too much business logic in handler
func CreateUser(c *marten.Ctx) error {
    var input CreateUserInput
    c.Bind(&input)
    
    // Don't do this in handlers
    hashedPassword := bcrypt.Hash(input.Password)
    user := User{Name: input.Name, Password: hashedPassword}
    db.Create(&user)
    sendWelcomeEmail(user)
    
    return c.Created(user)
}
```

### 2. Use Dependency Injection

```go
type UserHandler struct {
    store UserStore
}

func NewUserHandler(store UserStore) *UserHandler {
    return &UserHandler{store: store}
}

func (h *UserHandler) Get(c *marten.Ctx) error {
    user, err := h.store.Get(c.Param("id"))
    if err != nil {
        return c.NotFound("user not found")
    }
    return c.OK(user)
}
```

### 3. Group Related Routes

```go
// Good
api := app.Group("/api/v1")
{
    users := api.Group("/users")
    users.GET("", listUsers)
    users.POST("", createUser)
    
    posts := api.Group("/posts")
    posts.GET("", listPosts)
    posts.POST("", createPost)
}

// Avoid flat route definitions for large APIs
app.GET("/api/v1/users", listUsers)
app.POST("/api/v1/users", createUser)
app.GET("/api/v1/posts", listPosts)
// ...
```

### 4. Separate Configuration

```go
type Config struct {
    Port         string
    ReadTimeout  time.Duration
    WriteTimeout time.Duration
}

func LoadConfig() Config {
    return Config{
        Port:         os.Getenv("PORT"),
        ReadTimeout:  30 * time.Second,
        WriteTimeout: 30 * time.Second,
    }
}
```

## Next Steps

[:octicons-arrow-right-24: Learn about routing](../guide/routing.md)
