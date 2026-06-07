# Examples

Complete working examples to get you started with Marten.

## Available Examples

### [REST API](rest-api.md)
Build a complete CRUD API with validation, error handling, and proper HTTP semantics.

### [Authentication](authentication.md)
JWT-based authentication with protected routes and token validation.

### [File Server](file-server.md)
Serve static files using wildcard routes and the Static middleware.

### [Microservices](microservices.md)
Patterns for building microservices with Marten — health checks, tracing, graceful shutdown, async jobs.

## Running the Bundled Examples

All examples are in the `examples/` directory of the repository:

```bash
cd examples/basic
go run main.go
```

## Quick Reference

| Example | Highlights |
|---------|------------|
| [basic](https://github.com/gomarten/marten/tree/main/examples/basic) | Hello World, JSON, path and query params |
| [crud-api](https://github.com/gomarten/marten/tree/main/examples/crud-api) | CRUD, validation, `Created`, `NoContent` |
| [auth-jwt](https://github.com/gomarten/marten/tree/main/examples/auth-jwt) | JWT auth, Bearer token, protected groups |
| [middleware](https://github.com/gomarten/marten/tree/main/examples/middleware) | All built-in middleware in one app |
| [groups](https://github.com/gomarten/marten/tree/main/examples/groups) | Route groups, API versioning |
| [error-handling](https://github.com/gomarten/marten/tree/main/examples/error-handling) | Custom error types and `OnError` handler |
| [file-server](https://github.com/gomarten/marten/tree/main/examples/file-server) | Static files, SPA fallback |
| [marten-demo](https://github.com/gomarten/marten/tree/main/examples/marten-demo) | Full web app with templates and auth |
