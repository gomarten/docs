# Marten

<p align="center">
  <img src="assets/logo.png" alt="Marten" width="300">
</p>

<div align="center" style="margin-bottom: 2rem;">
  <h2 style="margin-bottom: 0.5rem;">The framework you reach for when you want nothing in the way.</h2>
  <p style="color: #666; font-size: 1.1rem;">Fast. Simple. Elegant. Zero dependencies.</p>
</div>

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } __Quick to Start__

    ---

    Get up and running in seconds with a minimal, intuitive API that feels like writing pure Go.

    [:octicons-arrow-right-24: Getting started](getting-started/index.md)

-   :material-lightning-bolt:{ .lg .middle } __Blazing Fast__

    ---

    Radix tree router, context pooling, and zero allocations where it matters. Built for performance.

    [:octicons-arrow-right-24: Benchmarks](#performance)

-   :material-puzzle:{ .lg .middle } __Middleware Ready__

    ---

    14 built-in middleware for logging, auth, CORS, rate limiting, compression, and more.

    [:octicons-arrow-right-24: Middleware](middleware/index.md)

-   :material-scale-balance:{ .lg .middle } __Zero Dependencies__

    ---

    Only Go's standard library. No external dependencies means no supply chain risks.

    [:octicons-arrow-right-24: Philosophy](philosophy.md)

</div>

## Quick Example

```go
package main

import "github.com/gomarten/marten"

func main() {
    app := marten.New()
    
    app.GET("/", func(c *marten.Ctx) error {
        return c.OK(marten.M{"message": "Hello, Marten!"})
    })
    
    app.Run(":3000")
}
```

That's it. No boilerplate. No configuration files. Just code.

## Features

<div class="grid" markdown>

=== "Routing"

    ```go
    // Static routes
    app.GET("/users", listUsers)
    
    // Path parameters
    app.GET("/users/:id", getUser)
    
    // Wildcard routes
    app.GET("/files/*filepath", serveFiles)
    
    // Route groups
    api := app.Group("/api/v1")
    api.GET("/posts", listPosts)
    ```

=== "Context"

    ```go
    func handler(c *marten.Ctx) error {
        // Path params
        id := c.ParamInt("id")
        
        // Query params
        page := c.QueryInt("page")
        
        // JSON binding
        var user User
        c.Bind(&user)
        
        // Response helpers
        return c.OK(user)
    }
    ```

=== "Middleware"

    ```go
    app.Use(
        middleware.Logger,
        middleware.Recover,
        middleware.CORS(middleware.DefaultCORSConfig()),
        middleware.RateLimit(middleware.RateLimitConfig{
            Requests: 100,
            Window:   time.Minute,
        }),
    )
    ```

=== "Groups"

    ```go
    // API versioning
    v1 := app.Group("/api/v1")
    v1.GET("/users", listUsersV1)
    
    // Protected routes
    admin := app.Group("/admin")
    admin.Use(authMiddleware)
    admin.GET("/dashboard", dashboard)
    ```

</div>

## Performance

Marten performs on par with Gin and Echo while maintaining zero dependencies.

| Benchmark | Marten | Gin | Echo | Chi |
|-----------|--------|-----|------|-----|
| Static Route | 1464 ns/op | 1336 ns/op | 1436 ns/op | 2202 ns/op |
| Param Route | 1564 ns/op | 1418 ns/op | 1472 ns/op | 2559 ns/op |
| JSON Response | 1755 ns/op | 2050 ns/op | 1835 ns/op | 1868 ns/op |

*Benchmarks run on Intel Xeon Platinum 8259CL @ 2.50GHz, Go 1.24*

[:octicons-arrow-right-24: Full benchmark results](benchmarks.md)

!!! tip "Context Pooling"
    Marten uses `sync.Pool` to reuse context objects, minimizing garbage collection pressure under high load.

## Installation

```bash
go get github.com/gomarten/marten
```

Requires Go 1.22 or later.

## Why Marten?

Most Go web frameworks grow until they become the problem they set out to solve. Marten stays small on purpose.

<div class="grid cards" markdown>

-   :material-feather:{ .lg .middle } __Minimal__

    ---

    ~1000 lines of code. Easy to read, understand, and contribute to.

-   :material-eye:{ .lg .middle } __Explicit__

    ---

    No magic. No reflection. No struct tags for routing. What you see is what you get.

-   :material-puzzle-outline:{ .lg .middle } __Composable__

    ---

    Middleware, groups, and handlers compose naturally. Build complex apps from simple pieces.

-   :material-book-open:{ .lg .middle } __Idiomatic__

    ---

    Feels like writing Go, not fighting it. Uses standard library patterns throughout.

</div>

## What Marten Does Not Do

These are intentional omissions:

- ❌ No ORM
- ❌ No template engine  
- ❌ No CLI tools
- ❌ No code generation
- ❌ No reflection magic
- ❌ No struct tags for routing
- ❌ No global state
- ❌ No opinionated project structure

If you need these features, there are excellent libraries that do them well. Marten focuses on being the best routing and middleware layer.

## Next Steps

<div class="grid cards" markdown>

-   [:octicons-rocket-16: __Getting Started__](getting-started/index.md)

    Install Marten and build your first app

-   [:octicons-book-16: __Guide__](guide/index.md)

    Learn the core concepts in depth

-   [:octicons-code-16: __Examples__](examples/index.md)

    See complete, working examples

-   [:octicons-package-16: __API Reference__](api/index.md)

    Detailed API documentation

</div>

## License

MIT License. Use it however you want.
