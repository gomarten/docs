# Benchmarks

Marten performs on par with established Go web frameworks while maintaining zero external dependencies.

## Frameworks Compared

| Framework | Version | Dependencies |
|-----------|---------|--------------|
| Marten | v0.1.1 | 0 (stdlib only) |
| Gin | v1.9.1 | 9 direct |
| Echo | v4.11.4 | 6 direct |
| Chi | v5.0.12 | 0 (stdlib only) |
| Fiber | v2.52.0 | 8 direct |

## Results

Benchmarks run on Intel Xeon Platinum 8259CL @ 2.50GHz, Go 1.24, Linux.

### Static Route

Simple GET request to `/hello` returning text.

| Framework | ns/op | B/op | allocs/op |
|-----------|-------|------|-----------|
| Marten | 1464 | 1040 | 11 |
| Gin | 1336 | 1040 | 9 |
| Echo | 1436 | 1024 | 10 |
| Chi | 2202 | 1360 | 12 |
| Fiber | 24506 | 10311 | 30 |

### Param Route

GET request to `/users/:id` with path parameter extraction.

| Framework | ns/op | B/op | allocs/op |
|-----------|-------|------|-----------|
| Marten | 1564 | 1048 | 11 |
| Gin | 1418 | 1040 | 9 |
| Echo | 1472 | 1016 | 10 |
| Chi | 2559 | 1688 | 14 |
| Fiber | 25582 | 10300 | 29 |

### JSON Response

GET request returning a JSON-serialized struct.

| Framework | ns/op | B/op | allocs/op |
|-----------|-------|------|-----------|
| Marten | 1755 | 1048 | 11 |
| Gin | 2050 | 1064 | 11 |
| Echo | 1835 | 1080 | 11 |
| Chi | 1868 | 1376 | 12 |
| Fiber | 26325 | 10356 | 32 |

### JSON Binding

POST request with JSON body parsing.

| Framework | ns/op | B/op | allocs/op |
|-----------|-------|------|-----------|
| Marten | 8438 | 7194 | 32 |
| Gin | 8950 | 7260 | 33 |
| Echo | 8960 | 7227 | 32 |
| Chi | 6460 | 7025 | 24 |
| Fiber | 47575 | 13251 | 53 |

### Multi-Param Route

GET request to `/users/:userId/posts/:postId/comments/:commentId`.

| Framework | ns/op | B/op | allocs/op |
|-----------|-------|------|-----------|
| Marten | 1837 | 1112 | 11 |
| Gin | 1524 | 1043 | 10 |
| Echo | 1655 | 1024 | 11 |
| Chi | 2869 | 1688 | 14 |
| Fiber | 28778 | 10343 | 29 |

## Key Takeaways

1. **Marten performs on par with Gin and Echo** across all benchmark categories
2. **Zero dependencies** means smaller binary size and faster compilation
3. **Standard net/http compatibility** ensures broad ecosystem support
4. **Chi** is slightly slower due to context-based parameter storage
5. **Fiber** benchmarks show overhead from `app.Test()` method (not representative of real Fasthttp performance)

## Run Your Own

```bash
cd benchmarks
go mod tidy
go test -bench=. -benchmem -benchtime=3s
```

## Interpretation

- **ns/op** - Nanoseconds per operation (lower is better)
- **B/op** - Bytes allocated per operation (lower is better)
- **allocs/op** - Number of heap allocations per operation (lower is better)

Results will vary based on CPU architecture, Go version, and system load. Always benchmark on your target hardware.
