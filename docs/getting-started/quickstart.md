# Quick Start

Let's build a complete REST API in 5 minutes.

## Create a New Project

```bash
mkdir myapi && cd myapi
go mod init myapi
go get github.com/gomarten/marten
```

## Build the API

Create `main.go`:

```go
package main

import (
    "fmt"
    "log"
    "sync"
    "time"

    "github.com/gomarten/marten"
    "github.com/gomarten/marten/middleware"
)

// User model
type User struct {
    ID        string    `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

// In-memory store
var (
    users   = make(map[string]User)
    usersMu sync.RWMutex
    nextID  = 1
)

func main() {
    app := marten.New()

    // Middleware
    app.Use(
        middleware.RequestID,
        middleware.Logger,
        middleware.Recover,
        middleware.CORS(middleware.DefaultCORSConfig()),
    )

    // Routes
    app.GET("/health", healthCheck)

    api := app.Group("/api/v1")
    {
        api.GET("/users", listUsers)
        api.GET("/users/:id", getUser)
        api.POST("/users", createUser)
        api.PUT("/users/:id", updateUser)
        api.DELETE("/users/:id", deleteUser)
    }

    log.Println("Server running on http://localhost:3000")
    app.RunGraceful(":3000", 10*time.Second)
}

func healthCheck(c *marten.Ctx) error {
    return c.OK(marten.M{"status": "healthy"})
}

func listUsers(c *marten.Ctx) error {
    usersMu.RLock()
    defer usersMu.RUnlock()

    list := make([]User, 0, len(users))
    for _, u := range users {
        list = append(list, u)
    }

    return c.OK(marten.M{"users": list, "total": len(list)})
}

func getUser(c *marten.Ctx) error {
    id := c.Param("id")

    usersMu.RLock()
    user, exists := users[id]
    usersMu.RUnlock()

    if !exists {
        return c.NotFound("user not found")
    }

    return c.OK(user)
}

func createUser(c *marten.Ctx) error {
    var input struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }

    if err := c.BindValid(&input, func() error {
        if input.Name == "" {
            return &marten.BindError{Message: "name is required"}
        }
        if input.Email == "" {
            return &marten.BindError{Message: "email is required"}
        }
        return nil
    }); err != nil {
        return c.BadRequest(err.Error())
    }

    usersMu.Lock()
    id := fmt.Sprintf("%d", nextID)
    nextID++
    user := User{
        ID:        id,
        Name:      input.Name,
        Email:     input.Email,
        CreatedAt: time.Now(),
    }
    users[id] = user
    usersMu.Unlock()

    return c.Created(user)
}

func updateUser(c *marten.Ctx) error {
    id := c.Param("id")

    usersMu.RLock()
    user, exists := users[id]
    usersMu.RUnlock()

    if !exists {
        return c.NotFound("user not found")
    }

    var input struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }

    if err := c.Bind(&input); err != nil {
        return c.BadRequest(err.Error())
    }

    if input.Name != "" {
        user.Name = input.Name
    }
    if input.Email != "" {
        user.Email = input.Email
    }

    usersMu.Lock()
    users[id] = user
    usersMu.Unlock()

    return c.OK(user)
}

func deleteUser(c *marten.Ctx) error {
    id := c.Param("id")

    usersMu.Lock()
    _, exists := users[id]
    if exists {
        delete(users, id)
    }
    usersMu.Unlock()

    if !exists {
        return c.NotFound("user not found")
    }

    return c.NoContent()
}
```

## Run the Server

```bash
go run main.go
```

## Test the API

=== "Create User"

    ```bash
    curl -X POST http://localhost:3000/api/v1/users \
      -H "Content-Type: application/json" \
      -d '{"name": "Alice", "email": "alice@example.com"}'
    ```

    Response:
    ```json
    {
      "id": "1",
      "name": "Alice",
      "email": "alice@example.com",
      "created_at": "2024-01-15T10:30:00Z"
    }
    ```

=== "List Users"

    ```bash
    curl http://localhost:3000/api/v1/users
    ```

    Response:
    ```json
    {
      "users": [
        {
          "id": "1",
          "name": "Alice",
          "email": "alice@example.com",
          "created_at": "2026-01-15T10:30:00Z"
        }
      ],
      "total": 1
    }
    ```

=== "Get User"

    ```bash
    curl http://localhost:3000/api/v1/users/1
    ```

=== "Update User"

    ```bash
    curl -X PUT http://localhost:3000/api/v1/users/1 \
      -H "Content-Type: application/json" \
      -d '{"name": "Alice Smith"}'
    ```

=== "Delete User"

    ```bash
    curl -X DELETE http://localhost:3000/api/v1/users/1
    ```

## What You've Learned

- ✅ Creating a Marten application
- ✅ Adding middleware
- ✅ Defining routes with groups
- ✅ Path parameters
- ✅ JSON binding and validation
- ✅ Response helpers
- ✅ Graceful shutdown

## Next Steps

[:octicons-arrow-right-24: Learn about routing in depth](../guide/routing.md)
