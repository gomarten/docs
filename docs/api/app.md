# App

The core application type.

## Creating an App

```go
app := marten.New()
```

## Methods

### Run

Start the server.

```go
func (a *App) Run(addr string) error
```

```go
app.Run(":8080")
```

### RunGraceful

Start with graceful shutdown support.

```go
func (a *App) RunGraceful(addr string, timeout time.Duration) error
```

```go
app.RunGraceful(":8080", 10*time.Second)
```

Handles `SIGINT` and `SIGTERM` signals, allowing in-flight requests to complete.

### Use

Add global middleware.

```go
func (a *App) Use(mw ...Middleware)
```

```go
app.Use(middleware.Logger, middleware.Recover)
```

### OnError

Set custom error handler.

```go
func (a *App) OnError(fn func(*Ctx, error))
```

```go
app.OnError(func(c *marten.Ctx, err error) {
    log.Printf("Error: %v", err)
    c.ServerError("something went wrong")
})
```

### NotFound

Set custom 404 handler.

```go
func (a *App) NotFound(h Handler)
```

```go
app.NotFound(func(c *marten.Ctx) error {
    return c.JSON(404, marten.M{"error": "not found"})
})
```

### Group

Create a route group.

```go
func (a *App) Group(prefix string) *Group
```

```go
api := app.Group("/api/v1")
api.GET("/users", listUsers)
```

## Route Methods

```go
app.GET(path string, h Handler, mw ...Middleware)
app.POST(path string, h Handler, mw ...Middleware)
app.PUT(path string, h Handler, mw ...Middleware)
app.DELETE(path string, h Handler, mw ...Middleware)
app.PATCH(path string, h Handler, mw ...Middleware)
app.Handle(method, path string, h Handler, mw ...Middleware)
```

## ServeHTTP

App implements `http.Handler`:

```go
func (a *App) ServeHTTP(w http.ResponseWriter, r *http.Request)
```

Use with custom server:

```go
server := &http.Server{
    Addr:         ":8080",
    Handler:      app,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
}
server.ListenAndServe()
```
