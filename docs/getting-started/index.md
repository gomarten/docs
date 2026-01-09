# Getting Started

Welcome to Marten! This guide will help you get up and running quickly.

## What is Marten?

Marten is a minimal, elegant web framework for Go. It provides:

- **Fast routing** with a radix tree router
- **Clean context** with helpers for common tasks
- **Flexible middleware** system
- **Route groups** for organizing your API
- **Zero dependencies** beyond Go's standard library

## Prerequisites

- Go 1.22 or later
- Basic knowledge of Go

## Quick Install

```bash
go get github.com/gomarten/marten
```

## Your First App

Create a new file called `main.go`:

```go
package main

import (
    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

func main() {
    // Create a new app
    app := marten.New()
    
    // Add middleware
    app.Use(middleware.Logger, middleware.Recover)
    
    // Define routes
    app.GET("/", func(c *marten.Ctx) error {
        return c.OK(marten.M{
            "message": "Welcome to Marten!",
        })
    })
    
    app.GET("/hello/:name", func(c *marten.Ctx) error {
        name := c.Param("name")
        return c.OK(marten.M{
            "message": "Hello, " + name + "!",
        })
    })
    
    // Start the server
    app.Run(":3000")
}
```

Run it:

```bash
go run main.go
```

Test it:

```bash
curl http://localhost:3000/
# {"message":"Welcome to Marten!"}

curl http://localhost:3000/hello/World
# {"message":"Hello, World!"}
```

## Next Steps

<div class="grid cards" markdown>

-   :material-download:{ .lg .middle } __Installation__

    ---

    Detailed installation instructions and requirements

    [:octicons-arrow-right-24: Installation](installation.md)

-   :material-rocket-launch:{ .lg .middle } __Quick Start__

    ---

    Build a complete REST API in 5 minutes

    [:octicons-arrow-right-24: Quick Start](quickstart.md)

-   :material-folder-outline:{ .lg .middle } __Project Structure__

    ---

    Recommended ways to organize your Marten project

    [:octicons-arrow-right-24: Project Structure](structure.md)

</div>
