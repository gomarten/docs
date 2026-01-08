# Changelog

All notable changes to Marten.

## [Unreleased]

### Added

- Wildcard route support (`*filepath`)
- ETag middleware for automatic caching
- NoCache middleware
- Compress middleware (gzip)
- BodyLimit middleware
- Secure middleware (security headers)
- RequestID middleware
- RateLimit middleware
- BasicAuth middleware
- Timeout middleware
- Response helpers: `OK()`, `Created()`, `NoContent()`, `BadRequest()`, `Unauthorized()`, `Forbidden()`, `NotFound()`, `ServerError()`
- Typed parameter helpers: `ParamInt()`, `ParamInt64()`, `QueryInt()`, `QueryInt64()`, `QueryBool()`
- Request helpers: `ClientIP()`, `Bearer()`, `RequestID()`, `IsJSON()`, `IsAJAX()`
- Request-scoped storage: `Set()`, `Get()`, `GetString()`, `GetInt()`, `GetBool()`
- Cookie helpers: `Cookie()`, `SetCookie()`
- Form helpers: `FormValue()`, `File()`
- Convenience types: `marten.M`, `marten.E()`
- `BindValid()` for validation
- `BindError` type
- IPv6 support in `ClientIP()`
- Graceful shutdown with `RunGraceful()`

### Fixed

- CORS wildcard origin security issue
- CORS OPTIONS double WriteHeader
- Panic recovery error propagation
- Router memory efficiency (lazy handler map initialization)
- Status code tracking in context
- Logger now logs status codes

### Changed

- CORS now properly handles credentials
- Improved string joining performance

## [0.1.0] - Initial Release

### Added

- Core routing with radix tree
- Route groups
- Middleware support
- Context with JSON/Text responses
- Logger middleware
- Recover middleware
- CORS middleware
