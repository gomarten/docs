# Logger Middleware

Logs HTTP requests with method, path, status code, and duration.

## Usage

### Basic Usage

```go
app.Use(middleware.Logger)
```

### With Configuration

```go
app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    Output: os.Stdout,
    Skip: func(c *marten.Ctx) bool {
        return c.Path() == "/health"
    },
}))
```

## Configuration

| Option | Type | Description |
|--------|------|-------------|
| `Output` | `io.Writer` | Where to write logs (default: os.Stdout) |
| `Format` | `func(...)` | Custom format function |
| `Skip` | `func(*Ctx) bool` | Skip logging for certain requests |
| `EnableColors` | `bool` | Enable colored output (default: false) |
| `JSONFormat` | `bool` | Output logs in JSON format (default: false) |

## Output

```
GET /users 200 1.234ms 192.168.1.1
POST /users 201 5.678ms 192.168.1.1
GET /users/999 404 0.456ms 192.168.1.1
```

## Examples

### Basic Logger

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

### Skip Health Checks

```go
app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    Skip: func(c *marten.Ctx) bool {
        return c.Path() == "/health" || c.Path() == "/ready"
    },
}))
```

### Custom Output

```go
logFile, _ := os.OpenFile("access.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)

app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    Output: logFile,
}))
```

### Custom Format

```go
app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    Format: func(method, path string, status int, duration time.Duration, clientIP string) string {
        return fmt.Sprintf("[%s] %s %s %d %v\n",
            time.Now().Format(time.RFC3339),
            method,
            path,
            status,
            duration,
        )
    },
}))
```

### Colored Output

```go
app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    EnableColors: true,
}))
```

Output with ANSI colors:
- GET requests in blue
- POST requests in green
- PUT requests in yellow
- DELETE requests in red
- 2xx status in green
- 4xx status in yellow
- 5xx status in red

### JSON Format

```go
app.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
    JSONFormat: true,
}))
```

Output:
```json
{"time":"2026-01-14T10:30:00Z","method":"GET","path":"/users","status":200,"duration":"1.234ms","client_ip":"192.168.1.1"}
```

## Custom Logger Middleware

For more control, create your own middleware:

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
