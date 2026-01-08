# Basic Auth Middleware

HTTP Basic Authentication for protecting routes.

## Usage

### Simple (Single User)

```go
app.Use(middleware.BasicAuthSimple("admin", "secret123"))
```

### Custom Validator

```go
app.Use(middleware.BasicAuth(middleware.BasicAuthConfig{
    Realm: "Admin Area",
    Validate: func(user, pass string) bool {
        return checkCredentials(user, pass)
    },
}))
```

## Configuration

| Option | Type | Description |
|--------|------|-------------|
| `Realm` | `string` | Authentication realm (default: "Restricted") |
| `Validate` | `func(user, pass string) bool` | Credential validator |

## Examples

### Protect Admin Routes

```go
admin := app.Group("/admin")
admin.Use(middleware.BasicAuthSimple("admin", "secret"))

admin.GET("/dashboard", dashboard)
admin.GET("/users", adminUsers)
```

### Multiple Users

```go
users := map[string]string{
    "alice": "password1",
    "bob":   "password2",
    "admin": "adminpass",
}

app.Use(middleware.BasicAuth(middleware.BasicAuthConfig{
    Realm: "API",
    Validate: func(user, pass string) bool {
        expected, ok := users[user]
        return ok && expected == pass
    },
}))
```

### Database Validation

```go
app.Use(middleware.BasicAuth(middleware.BasicAuthConfig{
    Realm: "API",
    Validate: func(user, pass string) bool {
        u, err := db.FindUser(user)
        if err != nil {
            return false
        }
        return bcrypt.CompareHashAndPassword(
            []byte(u.PasswordHash),
            []byte(pass),
        ) == nil
    },
}))
```

## Accessing User

The authenticated username is stored in the context:

```go
func handler(c *marten.Ctx) error {
    user := c.GetString("user")
    return c.OK(marten.M{"logged_in_as": user})
}
```

## Response

When authentication fails:

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="Restricted"
Content-Type: application/json

{"error": "unauthorized"}
```

## Security Considerations

!!! warning "HTTPS Required"
    Basic Auth sends credentials in base64 (not encrypted). Always use HTTPS in production.

1. **Use HTTPS** - Credentials are not encrypted
2. **Use strong passwords** - Basic auth is vulnerable to brute force
3. **Consider alternatives** - JWT or OAuth for APIs
4. **Constant-time comparison** - Prevent timing attacks

```go
// Good - constant time comparison
import "crypto/subtle"

func validate(user, pass string) bool {
    expectedUser := "admin"
    expectedPass := "secret"
    
    userMatch := subtle.ConstantTimeCompare([]byte(user), []byte(expectedUser)) == 1
    passMatch := subtle.ConstantTimeCompare([]byte(pass), []byte(expectedPass)) == 1
    
    return userMatch && passMatch
}
```
