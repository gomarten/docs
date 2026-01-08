# Philosophy

> The framework you reach for when you want nothing in the way.

## Core Principles

### Zero Dependencies

Marten uses only Go's standard library. No external dependencies means:

- No dependency conflicts
- No security vulnerabilities from third-party code
- No version management headaches
- Predictable, stable behavior

### Simplicity Over Features

Every feature in Marten earns its place. We don't add features just because other frameworks have them. If the standard library does it well, we don't wrap it.

### Explicit Over Magic

No struct tags for routing. No reflection-based dependency injection. No hidden behavior. What you write is what runs.

```go
// Clear and explicit
app.GET("/users/:id", getUser)

// Not magic
// @Route("/users/{id}")
// func getUser(id int) User { ... }
```

### Performance by Default

- Radix tree routing
- Context pooling
- Zero allocations in hot paths
- No reflection in request handling

### Developer Experience

Clean, chainable APIs that read like English:

```go
return c.Status(201).JSON(user)
return c.BadRequest("invalid email")
return c.Header("X-Custom", "value").OK(data)
```

## What Marten Is

- A thin layer over `net/http`
- A routing solution with parameters and groups
- A collection of useful middleware
- A clean context API for handlers

## What Marten Isn't

- An ORM
- A template engine
- A full-stack framework
- A kitchen sink

## Design Decisions

### Why No Generics for Handlers?

Generics add complexity without significant benefit for HTTP handlers. The `any` type with JSON encoding covers 99% of use cases.

### Why Return Errors?

Returning errors from handlers enables:

- Centralized error handling
- Middleware that can inspect errors
- Clean handler code without error checking boilerplate

### Why Context Pooling?

Creating a new context for every request is wasteful. Pooling reduces GC pressure and improves performance under load.

## The Marten Way

1. Start simple
2. Add only what you need
3. Keep handlers focused
4. Let middleware handle cross-cutting concerns
5. Trust the standard library
