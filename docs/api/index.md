# API Reference

Complete API documentation for Marten.

## Core Types

| Type | Description |
|------|-------------|
| [App](app.md) | Main application instance |
| [Ctx](context.md) | Request context with helpers |
| [Router](router.md) | HTTP routing |
| [Middleware](middleware.md) | Built-in middleware |

## Quick Reference

### Creating an App

```go
app := marten.New()
app.Run(":8080")
```

### Registering Routes

```go
app.GET("/path", handler)
app.POST("/path", handler)
app.PUT("/path/:id", handler)
app.DELETE("/path/:id", handler)
```

### Handler Signature

```go
func handler(c *marten.Ctx) error {
    return c.OK(data)
}
```

### Middleware Signature

```go
func middleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        // before
        err := next(c)
        // after
        return err
    }
}
```

## Convenience Types

```go
// Map shorthand
marten.M{"key": "value"}

// Error response
marten.E("error message")
// Returns: {"error": "error message"}
```
