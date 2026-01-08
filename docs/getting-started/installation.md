# Installation

## Requirements

- **Go 1.22** or later
- Any operating system (Linux, macOS, Windows)

## Install

Add Marten to your Go module:

```bash
go get github.com/gomarten/marten
```

## Verify Installation

Create a simple test file:

```go
package main

import (
    "fmt"
    "github.com/gomarten/marten/marten"
)

func main() {
    app := marten.New()
    fmt.Println("Marten installed successfully!")
    _ = app
}
```

Run it:

```bash
go run main.go
# Marten installed successfully!
```

## Import Paths

```go
// Core framework
import "github.com/gomarten/marten/marten"

// Built-in middleware
import "github.com/gomarten/marten/marten/middleware"
```

## Version Pinning

For production, pin to a specific version in your `go.mod`:

```
require github.com/gomarten/marten v1.0.0
```

## Updating

To update to the latest version:

```bash
go get -u github.com/gomarten/marten
```

## Zero Dependencies

Marten has **zero external dependencies**. It only uses Go's standard library, which means:

- ✅ No supply chain risks
- ✅ No version conflicts
- ✅ Smaller binary sizes
- ✅ Faster builds
- ✅ Easier auditing

## Next Steps

Now that Marten is installed, let's build something:

[:octicons-arrow-right-24: Quick Start](quickstart.md)
