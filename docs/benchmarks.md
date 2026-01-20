# Performance Benchmarks

Comprehensive performance comparison of Marten v0.1.3 against popular Go web frameworks.

## Test Environment

- **CPU**: Intel(R) Xeon(R) Platinum 8259CL @ 2.50GHz
- **OS**: Linux (amd64)
- **Go Version**: 1.24.0
- **Marten Version**: v0.1.3
- **Date**: January 2026

## Frameworks Compared

| Framework | Version | Dependencies |
|-----------|---------|--------------|
| **Marten** | v0.1.3 | 0 (zero) |
| **Gin** | v1.9.1 | 9 direct |
| **Echo** | v4.11.4 | 11 direct |
| **Chi** | v5.0.11 | 0 (zero) |
| **Fiber** | v2.52.0 | 15 direct |

## Benchmark Results

### Static Route

Simple GET request to `/hello` returning plain text.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Gin | 1,323 | 1,040 | 9 | 100% (fastest) |
| Echo | 1,421 | 1,024 | 10 | 93% |
| **Marten** | **1,445** | **1,040** | **11** | **92%** |
| Chi | 2,208 | 1,392 | 12 | 60% |
| Fiber | 24,300 | 10,685 | 31 | 5% |

**Marten Performance**: 692,000 requests/second/core (2,539,756 ops in benchmark)

### Param Route

Route with single path parameter `/users/:id`.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Gin | 1,419 | 1,040 | 9 | 100% (fastest) |
| Echo | 1,474 | 1,016 | 10 | 96% |
| **Marten** | **1,536** | **1,048** | **11** | **92%** |
| Chi | 2,520 | 1,720 | 14 | 56% |
| Fiber | 24,571 | 10,676 | 30 | 6% |

**Marten Performance**: 651,000 requests/second/core (2,338,711 ops in benchmark)

### JSON Response

Serializing a struct to JSON response.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Gin | 1,583 | 1,040 | 10 | 100% (fastest) |
| **Marten** | **1,651** | **1,024** | **10** | **96%** |
| Echo | 1,754 | 1,056 | 10 | 90% |
| Chi | 1,890 | 1,408 | 12 | 84% |
| Fiber | 25,841 | 10,707 | 32 | 6% |

**Winner**: Marten has the lowest memory usage (1,024 B/op)  
**Marten Performance**: 606,000 requests/second/core (2,218,782 ops in benchmark)

### JSON Binding

POST request with JSON body parsing.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Chi | 6,810 | 7,410 | 25 | 100% (fastest) |
| **Marten** | **8,339** | **7,547** | **33** | **82%** |
| Gin | 8,634 | 7,612 | 34 | 79% |
| Echo | 8,766 | 7,579 | 33 | 78% |
| Fiber | 51,014 | 13,617 | 53 | 13% |

*Note: Chi's benchmark doesn't fully parse JSON, giving it an advantage.*  
**Marten Performance**: 120,000 requests/second/core (419,924 ops in benchmark)

### Multi-Param Route

Route with three path parameters `/users/:userId/posts/:postId/comments/:commentId`.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Gin | 1,511 | 1,043 | 10 | 100% (fastest) |
| Echo | 1,634 | 1,024 | 11 | 92% |
| **Marten** | **1,841** | **1,112** | **11** | **82%** |
| Chi | 2,836 | 1,720 | 14 | 53% |
| Fiber | 26,456 | 10,721 | 30 | 6% |

**Marten Performance**: 543,000 requests/second/core (1,991,316 ops in benchmark)

### Query Parameters

Parsing query string `?q=golang&page=1&limit=10`.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Echo | 3,789 | 2,016 | 23 | 100% (fastest) |
| Gin | 4,002 | 2,112 | 27 | 95% |
| **Marten** | **5,419** | **2,945** | **35** | **70%** |

**Marten Performance**: 185,000 requests/second/core (618,480 ops in benchmark)

### Large JSON Response

Serializing a larger struct with nested data.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| **Marten** | **2,737** | **1,424** | **11** | **100% (fastest)** |
| Gin | 2,818 | 1,696 | 11 | 97% |
| Echo | 2,867 | 1,456 | 11 | 95% |

**Winner**: Marten is fastest for large JSON responses  
**Marten Performance**: 365,000 requests/second/core (1,328,371 ops in benchmark)

### Route Groups

Grouped routes `/api/v1/users/:id`.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Echo | 2,364 | 1,472 | 15 | 100% (fastest) |
| Gin | 2,365 | 1,456 | 16 | 100% |
| **Marten** | **2,524** | **1,504** | **16** | **94%** |

**Marten Performance**: 396,000 requests/second/core (1,421,040 ops in benchmark)

### Wildcard Routes

Catch-all routes `/files/*filepath`.

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Gin | 1,379 | 1,040 | 9 | 100% (fastest) |
| Echo | 1,473 | 1,032 | 10 | 94% |
| **Marten** | **1,690** | **1,104** | **12** | **82%** |

**Marten Performance**: 592,000 requests/second/core (2,104,170 ops in benchmark)

### Parallel Requests

Concurrent request handling (simulated).

| Framework | ns/op | B/op | allocs/op | Relative |
|-----------|-------|------|-----------|----------|
| Gin | 4,607 | 6,145 | 18 | 100% (fastest) |
| Echo | 4,667 | 6,129 | 19 | 99% |
| **Marten** | **4,717** | **6,145** | **20** | **98%** |

**Marten Performance**: 212,000 requests/second/core (713,434 ops in benchmark)

## Performance Summary

### Overall Rankings

**By Speed (Average):**
1. Gin - 100%
2. Echo - 95%
3. **Marten - 88%** ⭐
4. Chi - 63%
5. Fiber - 8%*

*Fiber's low score is due to `app.Test()` overhead in benchmarks.

### Marten's Strengths

- ✅ **Best large JSON performance** - Beats Gin and Echo
- ✅ **Excellent parallel performance** - 98% of Gin's speed
- ✅ **Competitive overall** - Within 8-18% of Gin/Echo
- ✅ **Zero dependencies** - No external packages
- ✅ **Consistent allocations** - Predictable memory usage


## Real-World Context

### What These Numbers Mean

For a typical web application:

```
Request breakdown:
- Network latency: 1-100ms
- Database query: 1-100ms
- Framework overhead: 0.001-0.002ms (Marten)
- JSON encoding: 0.001-0.003ms
```

**The framework is <1% of total request time.**

### Throughput Comparison

Requests per second per core (theoretical):

| Framework | Static Route | With JSON |
|-----------|--------------|-----------|
| Gin | 756,000 | 632,000 |
| Echo | 704,000 | 570,000 |
| **Marten** | **692,000** | **606,000** |
| Chi | 453,000 | 529,000 |

In practice, you'll be limited by:
- Network bandwidth
- Database connections
- Business logic complexity
- External API calls


## Why Choose Marten?

### Performance + Simplicity

Marten delivers **88% of Gin's performance** with **0 dependencies**.

**Benefits:**
- No dependency management
- Smaller binaries (~2MB vs ~8MB)
- Faster builds
- Easier to audit
- No breaking changes from dependencies

**Trade-off:**
- 8-18% slower than Gin/Echo
- Smaller ecosystem
- Fewer third-party integrations


## Running Benchmarks

### Quick Start

```bash
git clone https://github.com/gomarten/marten
cd marten/benchmarks
go test -bench=. -benchmem
```

### Detailed Benchmarks

```bash
# Run for 3 seconds each
go test -bench=. -benchmem -benchtime=3s

# Compare specific frameworks
go test -bench="Marten|Gin" -benchmem

# JSON benchmarks only
go test -bench="JSON" -benchmem

# Parallel benchmarks
go test -bench="Parallel" -benchmem
```

### Memory Profiling

```bash
# Profile memory allocations
go test -bench=Marten_StaticRoute -memprofile=mem.out
go tool pprof mem.out

# Profile CPU usage
go test -bench=Marten_StaticRoute -cpuprofile=cpu.out
go tool pprof cpu.out
```

## Methodology

### Test Setup

- Each benchmark runs for 1-3 seconds
- Uses `httptest.NewRecorder()` for consistency
- All frameworks use default configuration
- Gin runs in release mode
- No middleware enabled (pure routing performance)

### Metrics Explained

- **ns/op**: Nanoseconds per operation (lower is better)
- **B/op**: Bytes allocated per operation (lower is better)
- **allocs/op**: Number of allocations (lower is better)

### Limitations

- Benchmarks use in-memory testing, not real HTTP
- No network overhead included
- No database or external service calls
- Fiber's `app.Test()` adds overhead not present in production
- Chi's JSON binding benchmark is simplified

## Conclusion

Marten delivers **competitive performance** with **zero dependencies**.

For most applications, the 8-18% performance difference compared to Gin is negligible when network and database latency dominate request time.

---

**Source Code**: [github.com/gomarten/marten/benchmarks](https://github.com/gomarten/marten/tree/main/benchmarks)
