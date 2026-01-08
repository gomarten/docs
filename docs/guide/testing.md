# Testing

Marten applications are easy to test using Go's standard `testing` package and `net/http/httptest`.

## Basic Testing

### Test Setup

```go
package main

import (
    "net/http/httptest"
    "testing"
    
    "github.com/gomarten/marten/marten"
)

func TestHelloWorld(t *testing.T) {
    // Create app
    app := marten.New()
    app.GET("/", func(c *marten.Ctx) error {
        return c.Text(200, "Hello, World!")
    })
    
    // Create request
    req := httptest.NewRequest("GET", "/", nil)
    rec := httptest.NewRecorder()
    
    // Execute
    app.ServeHTTP(rec, req)
    
    // Assert
    if rec.Code != 200 {
        t.Errorf("expected 200, got %d", rec.Code)
    }
    if rec.Body.String() != "Hello, World!" {
        t.Errorf("unexpected body: %s", rec.Body.String())
    }
}
```

### Testing JSON Responses

```go
func TestJSONResponse(t *testing.T) {
    app := marten.New()
    app.GET("/user", func(c *marten.Ctx) error {
        return c.OK(marten.M{"name": "Alice", "age": 30})
    })
    
    req := httptest.NewRequest("GET", "/user", nil)
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    // Check status
    if rec.Code != 200 {
        t.Fatalf("expected 200, got %d", rec.Code)
    }
    
    // Check content type
    ct := rec.Header().Get("Content-Type")
    if ct != "application/json; charset=utf-8" {
        t.Errorf("unexpected content type: %s", ct)
    }
    
    // Parse JSON
    var resp map[string]any
    if err := json.Unmarshal(rec.Body.Bytes(), &resp); err != nil {
        t.Fatalf("failed to parse JSON: %v", err)
    }
    
    if resp["name"] != "Alice" {
        t.Errorf("expected name=Alice, got %v", resp["name"])
    }
}
```

## Testing with Parameters

### Path Parameters

```go
func TestPathParams(t *testing.T) {
    app := marten.New()
    app.GET("/users/:id", func(c *marten.Ctx) error {
        return c.OK(marten.M{"id": c.Param("id")})
    })
    
    req := httptest.NewRequest("GET", "/users/123", nil)
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    var resp map[string]string
    json.Unmarshal(rec.Body.Bytes(), &resp)
    
    if resp["id"] != "123" {
        t.Errorf("expected id=123, got %s", resp["id"])
    }
}
```

### Query Parameters

```go
func TestQueryParams(t *testing.T) {
    app := marten.New()
    app.GET("/search", func(c *marten.Ctx) error {
        return c.OK(marten.M{
            "q":    c.Query("q"),
            "page": c.QueryInt("page"),
        })
    })
    
    req := httptest.NewRequest("GET", "/search?q=golang&page=2", nil)
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    var resp map[string]any
    json.Unmarshal(rec.Body.Bytes(), &resp)
    
    if resp["q"] != "golang" {
        t.Errorf("expected q=golang, got %v", resp["q"])
    }
}
```

## Testing POST Requests

### JSON Body

```go
func TestCreateUser(t *testing.T) {
    app := marten.New()
    app.POST("/users", func(c *marten.Ctx) error {
        var input struct {
            Name  string `json:"name"`
            Email string `json:"email"`
        }
        if err := c.Bind(&input); err != nil {
            return c.BadRequest(err.Error())
        }
        return c.Created(input)
    })
    
    body := bytes.NewBufferString(`{"name":"Alice","email":"alice@example.com"}`)
    req := httptest.NewRequest("POST", "/users", body)
    req.Header.Set("Content-Type", "application/json")
    rec := httptest.NewRecorder()
    
    app.ServeHTTP(rec, req)
    
    if rec.Code != 201 {
        t.Errorf("expected 201, got %d", rec.Code)
    }
}
```

### Form Data

```go
func TestFormSubmission(t *testing.T) {
    app := marten.New()
    app.POST("/login", func(c *marten.Ctx) error {
        username := c.FormValue("username")
        return c.OK(marten.M{"username": username})
    })
    
    body := strings.NewReader("username=alice&password=secret")
    req := httptest.NewRequest("POST", "/login", body)
    req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
    rec := httptest.NewRecorder()
    
    app.ServeHTTP(rec, req)
    
    if rec.Code != 200 {
        t.Errorf("expected 200, got %d", rec.Code)
    }
}
```

## Testing Middleware

### Testing Custom Middleware

```go
func TestAuthMiddleware(t *testing.T) {
    authMw := func(next marten.Handler) marten.Handler {
        return func(c *marten.Ctx) error {
            if c.Bearer() == "" {
                return c.Unauthorized("missing token")
            }
            return next(c)
        }
    }
    
    app := marten.New()
    app.GET("/protected", func(c *marten.Ctx) error {
        return c.OK(marten.M{"message": "secret"})
    }, authMw)
    
    // Without token
    req := httptest.NewRequest("GET", "/protected", nil)
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    if rec.Code != 401 {
        t.Errorf("expected 401 without token, got %d", rec.Code)
    }
    
    // With token
    req = httptest.NewRequest("GET", "/protected", nil)
    req.Header.Set("Authorization", "Bearer valid-token")
    rec = httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    if rec.Code != 200 {
        t.Errorf("expected 200 with token, got %d", rec.Code)
    }
}
```

### Testing Middleware Order

```go
func TestMiddlewareOrder(t *testing.T) {
    var order []string
    
    mw1 := func(next marten.Handler) marten.Handler {
        return func(c *marten.Ctx) error {
            order = append(order, "mw1-before")
            err := next(c)
            order = append(order, "mw1-after")
            return err
        }
    }
    
    mw2 := func(next marten.Handler) marten.Handler {
        return func(c *marten.Ctx) error {
            order = append(order, "mw2-before")
            err := next(c)
            order = append(order, "mw2-after")
            return err
        }
    }
    
    app := marten.New()
    app.Use(mw1, mw2)
    app.GET("/", func(c *marten.Ctx) error {
        order = append(order, "handler")
        return c.Text(200, "ok")
    })
    
    req := httptest.NewRequest("GET", "/", nil)
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    expected := []string{"mw1-before", "mw2-before", "handler", "mw2-after", "mw1-after"}
    for i, v := range expected {
        if order[i] != v {
            t.Errorf("position %d: expected %s, got %s", i, v, order[i])
        }
    }
}
```

## Table-Driven Tests

```go
func TestRoutes(t *testing.T) {
    app := setupApp() // Your app setup function
    
    tests := []struct {
        name     string
        method   string
        path     string
        body     string
        status   int
        contains string
    }{
        {"list users", "GET", "/users", "", 200, "users"},
        {"get user", "GET", "/users/1", "", 200, "id"},
        {"create user", "POST", "/users", `{"name":"Alice"}`, 201, "Alice"},
        {"not found", "GET", "/nonexistent", "", 404, "not found"},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            var body io.Reader
            if tt.body != "" {
                body = strings.NewReader(tt.body)
            }
            
            req := httptest.NewRequest(tt.method, tt.path, body)
            if tt.body != "" {
                req.Header.Set("Content-Type", "application/json")
            }
            rec := httptest.NewRecorder()
            
            app.ServeHTTP(rec, req)
            
            if rec.Code != tt.status {
                t.Errorf("expected %d, got %d", tt.status, rec.Code)
            }
            
            if !strings.Contains(rec.Body.String(), tt.contains) {
                t.Errorf("body should contain %q, got %q", tt.contains, rec.Body.String())
            }
        })
    }
}
```

## Testing with Dependencies

### Dependency Injection

```go
type UserStore interface {
    Get(id string) (*User, error)
    Create(user *User) error
}

type UserHandler struct {
    store UserStore
}

func (h *UserHandler) Get(c *marten.Ctx) error {
    user, err := h.store.Get(c.Param("id"))
    if err != nil {
        return c.NotFound("user not found")
    }
    return c.OK(user)
}

// In tests
type mockUserStore struct {
    users map[string]*User
}

func (m *mockUserStore) Get(id string) (*User, error) {
    if user, ok := m.users[id]; ok {
        return user, nil
    }
    return nil, errors.New("not found")
}

func TestGetUser(t *testing.T) {
    store := &mockUserStore{
        users: map[string]*User{
            "1": {ID: "1", Name: "Alice"},
        },
    }
    handler := &UserHandler{store: store}
    
    app := marten.New()
    app.GET("/users/:id", handler.Get)
    
    req := httptest.NewRequest("GET", "/users/1", nil)
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    
    if rec.Code != 200 {
        t.Errorf("expected 200, got %d", rec.Code)
    }
}
```

## Benchmarking

```go
func BenchmarkHandler(b *testing.B) {
    app := marten.New()
    app.GET("/users/:id", func(c *marten.Ctx) error {
        return c.OK(marten.M{"id": c.Param("id")})
    })
    
    req := httptest.NewRequest("GET", "/users/123", nil)
    
    b.ResetTimer()
    b.ReportAllocs()
    
    for i := 0; i < b.N; i++ {
        rec := httptest.NewRecorder()
        app.ServeHTTP(rec, req)
    }
}
```

## Test Helpers

Create reusable test helpers:

```go
// testutil/helpers.go
package testutil

func NewTestApp() *marten.App {
    app := marten.New()
    app.Use(middleware.Recover)
    return app
}

func DoRequest(app *marten.App, method, path string, body io.Reader) *httptest.ResponseRecorder {
    req := httptest.NewRequest(method, path, body)
    if body != nil {
        req.Header.Set("Content-Type", "application/json")
    }
    rec := httptest.NewRecorder()
    app.ServeHTTP(rec, req)
    return rec
}

func AssertStatus(t *testing.T, rec *httptest.ResponseRecorder, expected int) {
    t.Helper()
    if rec.Code != expected {
        t.Errorf("expected status %d, got %d: %s", expected, rec.Code, rec.Body.String())
    }
}

func AssertJSON(t *testing.T, rec *httptest.ResponseRecorder, key string, expected any) {
    t.Helper()
    var resp map[string]any
    if err := json.Unmarshal(rec.Body.Bytes(), &resp); err != nil {
        t.Fatalf("failed to parse JSON: %v", err)
    }
    if resp[key] != expected {
        t.Errorf("expected %s=%v, got %v", key, expected, resp[key])
    }
}
```

## Best Practices

1. **Test behavior, not implementation** - Focus on inputs and outputs
2. **Use table-driven tests** - Easy to add new test cases
3. **Test error cases** - Don't just test the happy path
4. **Use meaningful test names** - `TestCreateUser_InvalidEmail` not `TestCreate2`
5. **Keep tests independent** - Each test should set up its own state
6. **Use test helpers** - Reduce boilerplate

## Next Steps

[:octicons-arrow-right-24: See complete examples](../examples/index.md)
