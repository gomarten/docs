# Contributing

We welcome contributions to Marten!

## Getting Started

```bash
git clone https://github.com/gomarten/marten.git
cd marten
go test ./...
```

## Guidelines

### Code Style

- Follow standard Go conventions
- Run `go fmt` before committing
- Keep functions small and focused
- Add comments for exported types and functions

### Pull Requests

1. Fork the repository
2. Create a feature branch
3. Write tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

### Commit Messages

Use clear, descriptive commit messages:

```
feat: add rate limiting middleware
fix: handle empty request body in Bind
docs: update routing examples
test: add edge cases for router
```

## Testing

```bash
# Run all tests
cd tests
go test -v ./...

# Run benchmarks
go test -bench=. -benchmem

# Check coverage
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

## What We're Looking For

- Bug fixes
- Performance improvements
- Documentation improvements
- New middleware (if it fits the philosophy)
- Test coverage improvements

## What We're Not Looking For

- Features that add external dependencies
- Breaking changes to the API
- Features that duplicate standard library functionality

## Questions?

Open an issue for discussion before starting major work.
