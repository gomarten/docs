# Examples

Complete working examples to get you started with Marten.

## Available Examples

### [REST API](rest-api.md)
Build a complete CRUD API with validation, error handling, and proper HTTP semantics.

### [Authentication](authentication.md)
JWT-based authentication with protected routes and token validation.

### [File Server](file-server.md)
Serve static files using wildcard routes.

### [Microservices](microservices.md)
Patterns for building microservices with Marten.

## Running Examples

All examples are in the `examples/` directory:

```bash
cd examples/basic
go run main.go
```

## Quick Reference

| Example | Features |
|---------|----------|
| basic | Hello World, JSON, params |
| crud-api | CRUD, validation, error handling |
| middleware | All built-in middleware |
| file-server | Static files, wildcards |
| auth-jwt | JWT auth, protected routes |
| groups | Route groups, versioning |
| error-handling | Custom error handlers |
