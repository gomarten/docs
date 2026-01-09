# Context (Ctx)

Request context with helpers for clean handler code.

## Properties

```go
c.Request    *http.Request
c.Writer     http.ResponseWriter
```

## Response Methods

### JSON Response

```go
c.JSON(code int, v any) error
c.OK(v any) error           // 200
c.Created(v any) error      // 201
```

### Text Response

```go
c.Text(code int, text string) error
```

### HTML Response

```go
c.HTML(code int, html string) error
```

### Binary Response

```go
c.Blob(code int, contentType string, data []byte) error
```

### Stream Response

```go
c.Stream(code int, contentType string, r io.Reader) error
```

### No Content

```go
c.NoContent() error         // 204
```

### Error Responses

```go
c.BadRequest(message string) error    // 400
c.Unauthorized(message string) error  // 401
c.Forbidden(message string) error     // 403
c.NotFound(message string) error      // 404
c.ServerError(message string) error   // 500
```

### Status & Headers

```go
c.Status(code int) *Ctx
c.StatusCode() int
c.Header(key, value string) *Ctx
c.GetHeader(key string) string
c.Written() bool
c.Redirect(code int, url string) error
```

## Request Data

### Path Parameters

```go
c.Param(name string) string
c.ParamInt(name string) int
c.ParamInt64(name string) int64
```

### Query Parameters

```go
c.Query(name string) string
c.QueryInt(name string) int
c.QueryInt64(name string) int64
c.QueryBool(name string) bool
c.QueryDefault(name, def string) string
c.QueryValues(name string) []string
c.QueryParams() url.Values
```

### Body Binding

```go
c.Bind(v any) error
c.BindValid(v any, validate func() error) error
```

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
c.Method() string
c.Path() string
c.ClientIP() string
c.Bearer() string
c.RequestID() string
c.IsJSON() bool
c.IsAJAX() bool
c.Context() context.Context
```

## Request-Scoped Storage

```go
c.Set(key string, value any)
c.Get(key string) any
c.GetString(key string) string
c.GetInt(key string) int
c.GetBool(key string) bool
```

## Examples

### JSON API Handler

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

### Streaming Response

```go
func downloadFile(c *marten.Ctx) error {
    file, err := os.Open("large-file.zip")
    if err != nil {
        return c.NotFound("file not found")
    }
    defer file.Close()
    
    c.Header("Content-Disposition", "attachment; filename=file.zip")
    return c.Stream(200, "application/octet-stream", file)
}
```

### HTML Response

```go
func homePage(c *marten.Ctx) error {
    html := "<html><body><h1>Welcome</h1></body></html>"
    return c.HTML(200, html)
}
```

### Form Handling

```go
func createPost(c *marten.Ctx) error {
    title := c.FormValue("title")
    file, err := c.File("image")
    if err != nil {
        return c.BadRequest("image required")
    }
    // Process...
    return c.Created(marten.M{"id": newID})
}
```

### Request Validation

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
        return nil
    }); err != nil {
        return c.BadRequest(err.Error())
    }

    return c.Created(input)
}
```
