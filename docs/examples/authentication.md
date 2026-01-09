# Authentication Example

JWT-based authentication with protected routes.

## Overview

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    app.Use(middleware.Logger, middleware.Recover)

    // Public routes
    app.POST("/auth/login", login)
    app.POST("/auth/register", register)

    // Protected routes
    protected := app.Group("/api")
    protected.Use(JWTMiddleware)
    protected.GET("/me", getMe)
    protected.GET("/profile", getProfile)

    // Admin routes
    admin := app.Group("/admin")
    admin.Use(JWTMiddleware, AdminMiddleware)
    admin.GET("/users", listAllUsers)

    app.Run(":3000")
}
```

## JWT Middleware

```go
func JWTMiddleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        token := c.Bearer()
        if token == "" {
            return c.Unauthorized("missing token")
        }

        claims, err := validateJWT(token)
        if err != nil {
            return c.Unauthorized("invalid token")
        }

        // Store claims in context for handlers
        c.Set("user_id", claims.UserID)
        c.Set("email", claims.Email)

        return next(c)
    }
}
```

## Login Handler

```go
func login(c *marten.Ctx) error {
    var input struct {
        Email    string `json:"email"`
        Password string `json:"password"`
    }

    if err := c.Bind(&input); err != nil {
        return c.BadRequest("invalid request body")
    }

    // Verify credentials (use database in production)
    if !verifyCredentials(input.Email, input.Password) {
        return c.Unauthorized("invalid credentials")
    }

    token, err := generateJWT(userID, input.Email)
    if err != nil {
        return c.ServerError("failed to generate token")
    }

    return c.OK(marten.M{
        "token":      token,
        "expires_in": 3600,
    })
}
```

## Protected Handler

```go
func getMe(c *marten.Ctx) error {
    // Access user data from context
    userID := c.GetString("user_id")
    email := c.GetString("email")

    return c.OK(marten.M{
        "user_id": userID,
        "email":   email,
    })
}
```

## Role-Based Access

```go
func AdminMiddleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        role := c.GetString("role")
        if role != "admin" {
            return c.Forbidden("admin access required")
        }
        return next(c)
    }
}
```

## Key Features Used

| Feature | Usage |
|---------|-------|
| `c.Bearer()` | Extract JWT from Authorization header |
| `c.Set()` / `c.GetString()` | Store/retrieve user data in request context |
| `c.Unauthorized()` | Return 401 response |
| `c.Forbidden()` | Return 403 response |
| Route groups | Organize public/protected/admin routes |

## Testing

```bash
# Login
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'

# Access protected route
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer <token>"
```
