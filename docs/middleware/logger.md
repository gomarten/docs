# Logger Middleware

Logs HTTP requests with method, path, status code, and duration.

## Usage

```go
app.Use(middleware.Logger)
```

## Output

```
2024/01/15 10:30:00 GET /users 200 1.234ms
2024/01/15 10:30:01 POST /users 201 5.678ms
2024/01/15 10:30:02 GET /users/999 404 0.456ms
```

## Log Format

```
<timestamp> <method> <path> <status> <duration>
```

## Example

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    app := marten.New()
    
    app.Use(middleware.Logger)
    
    app.GET("/", func(c *marten.Ctx) error {
        return c.OK(marten.M{"message": "hello"})
    })
    
    app.Run(":3000")
}
```

## Custom Logger

For custom logging, create your own middleware:

```go
func CustomLogger(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        start := time.Now()
        
        err := next(c)
        
        log.Printf(
            "[%s] %s %s %d %v",
            c.RequestID(),
            c.Method(),
            c.Path(),
            c.StatusCode(),
            time.Since(start),
        )
        
        return err
    }
}
```

## With Structured Logging

```go
func JSONLogger(next marten.Handler) marten.Handler {
    return func(c *marten.Ctx) error {
        start := time.Now()
        
        err := next(c)
        
        entry := map[string]any{
            "request_id": c.RequestID(),
            "method":     c.Method(),
            "path":       c.Path(),
            "status":     c.StatusCode(),
            "duration":   time.Since(start).Milliseconds(),
            "client_ip":  c.ClientIP(),
        }
        
        data, _ := json.Marshal(entry)
        log.Println(string(data))
        
        return err
    }
}
```
