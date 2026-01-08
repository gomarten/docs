# Error Handling

Marten provides flexible error handling through handler return values and custom error handlers.

## Basic Error Handling

Handlers return errors to signal problems:

```go
func getUser(c *marten.Ctx) error {
    id := c.Param("id")
    
    user, err := db.FindUser(id)
    if err != nil {
        return err // Triggers error handler
    }
    
    return c.OK(user)
}
```

## Response Helpers

Use built-in helpers for common HTTP errors:

```go
func handler(c *marten.Ctx) error {
    // 400 Bad Request
    return c.BadRequest("invalid input")
    
    // 401 Unauthorized
    return c.Unauthorized("login required")
    
    // 403 Forbidden
    return c.Forbidden("access denied")
    
    // 404 Not Found
    return c.NotFound("user not found")
    
    // 500 Internal Server Error
    return c.ServerError("something went wrong")
}
```

These return JSON responses:

```json
{"error": "message"}
```

## Custom Error Handler

Set a global error handler:

```go
app := marten.New()

app.OnError(func(c *marten.Ctx, err error) {
    // Log the error
    log.Printf("Error: %v (request_id: %s)", err, c.RequestID())
    
    // Return response
    c.JSON(500, marten.M{
        "error":      "internal error",
        "request_id": c.RequestID(),
    })
})
```

## Custom Error Types

Define custom error types for different scenarios:

```go
// Validation error
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return e.Field + ": " + e.Message
}

// Not found error
type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return e.Resource + " not found: " + e.ID
}

// Permission error
type ForbiddenError struct {
    Message string
}

func (e *ForbiddenError) Error() string {
    return e.Message
}
```

### Using Custom Errors

```go
func getUser(c *marten.Ctx) error {
    id := c.Param("id")
    
    user, err := db.FindUser(id)
    if err == sql.ErrNoRows {
        return &NotFoundError{Resource: "user", ID: id}
    }
    if err != nil {
        return err
    }
    
    return c.OK(user)
}

func createUser(c *marten.Ctx) error {
    var input CreateUserInput
    if err := c.Bind(&input); err != nil {
        return err
    }
    
    if input.Email == "" {
        return &ValidationError{Field: "email", Message: "is required"}
    }
    
    // ...
}
```

### Handling Custom Errors

```go
app.OnError(func(c *marten.Ctx, err error) {
    var validationErr *ValidationError
    var notFoundErr *NotFoundError
    var forbiddenErr *ForbiddenError
    var bindErr *marten.BindError
    
    switch {
    case errors.As(err, &validationErr):
        c.JSON(400, marten.M{
            "error": "validation_error",
            "field": validationErr.Field,
            "message": validationErr.Message,
        })
        
    case errors.As(err, &notFoundErr):
        c.JSON(404, marten.M{
            "error":    "not_found",
            "resource": notFoundErr.Resource,
            "id":       notFoundErr.ID,
        })
        
    case errors.As(err, &forbiddenErr):
        c.JSON(403, marten.M{
            "error":   "forbidden",
            "message": forbiddenErr.Message,
        })
        
    case errors.As(err, &bindErr):
        c.JSON(400, marten.M{
            "error":   "bad_request",
            "message": bindErr.Message,
        })
        
    default:
        // Log unexpected errors
        log.Printf("Unexpected error: %v", err)
        
        c.JSON(500, marten.M{
            "error":      "internal_error",
            "request_id": c.RequestID(),
        })
    }
})
```

## Panic Recovery

The `Recover` middleware catches panics:

```go
app.Use(middleware.Recover)

app.GET("/panic", func(c *marten.Ctx) error {
    panic("something went wrong!")
    return nil
})

// Returns 500 Internal Server Error instead of crashing
```

## Validation

### Using BindValid

```go
func createUser(c *marten.Ctx) error {
    var input struct {
        Name     string `json:"name"`
        Email    string `json:"email"`
        Password string `json:"password"`
    }
    
    err := c.BindValid(&input, func() error {
        if input.Name == "" {
            return &marten.BindError{Message: "name is required"}
        }
        if input.Email == "" {
            return &marten.BindError{Message: "email is required"}
        }
        if len(input.Password) < 8 {
            return &marten.BindError{Message: "password must be at least 8 characters"}
        }
        return nil
    })
    
    if err != nil {
        return c.BadRequest(err.Error())
    }
    
    // Create user...
    return c.Created(user)
}
```

### Multiple Validation Errors

```go
type ValidationErrors []ValidationError

func (e ValidationErrors) Error() string {
    return "validation failed"
}

func validate(input CreateUserInput) error {
    var errs ValidationErrors
    
    if input.Name == "" {
        errs = append(errs, ValidationError{
            Field:   "name",
            Message: "is required",
        })
    }
    
    if input.Email == "" {
        errs = append(errs, ValidationError{
            Field:   "email",
            Message: "is required",
        })
    }
    
    if len(errs) > 0 {
        return errs
    }
    
    return nil
}

// In error handler
case errors.As(err, &validationErrs):
    c.JSON(400, marten.M{
        "error":  "validation_error",
        "errors": validationErrs,
    })
```

## Error Middleware

Create middleware for error handling:

```go
func ErrorMiddleware(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        err := next(c)
        
        if err == nil {
            return nil
        }
        
        // Log error
        log.Printf("[%s] %s %s: %v",
            c.RequestID(),
            c.Method(),
            c.Path(),
            err,
        )
        
        // Handle specific errors
        var notFound *NotFoundError
        if errors.As(err, &notFound) {
            return c.NotFound(notFound.Error())
        }
        
        // Default to 500
        return c.ServerError("internal error")
    }
}
```

## Best Practices

### 1. Be Specific

```go
// Good - specific error
return &NotFoundError{Resource: "user", ID: id}

// Avoid - generic error
return errors.New("not found")
```

### 2. Don't Expose Internal Errors

```go
// Good - hide internal details
if err != nil {
    log.Printf("Database error: %v", err)
    return c.ServerError("internal error")
}

// Avoid - exposing internal errors
if err != nil {
    return c.ServerError(err.Error()) // Might expose SQL errors
}
```

### 3. Use Appropriate Status Codes

| Status | When to Use |
|--------|-------------|
| 400 | Invalid input, validation errors |
| 401 | Missing or invalid authentication |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict (e.g., duplicate) |
| 422 | Unprocessable entity |
| 429 | Rate limit exceeded |
| 500 | Unexpected server errors |

### 4. Include Request ID

```go
c.JSON(500, marten.M{
    "error":      "internal error",
    "request_id": c.RequestID(),
})
```

This helps correlate errors with logs.

## Next Steps

[:octicons-arrow-right-24: Learn about Testing](testing.md)
