---
name: golang-dev-guidelines
description: Use when writing Go code, setting up Go projects, building microservices/APIs, working with Kubernetes controllers/operators, implementing concurrent systems, or reviewing Go code. Covers best practices from Google Go Style Guide, Effective Go, and Kubebuilder patterns.
---

# Golang Development Guidelines

**Purpose:** Comprehensive best practices and patterns for Go development based on official Go documentation, Google's Go Style Guide, and Kubebuilder best practices.

**When to use:**
- Writing Go code (services, CLIs, libraries, controllers)
- Setting up Go projects
- Building microservices and APIs
- Working with Kubernetes controllers/operators
- Implementing concurrent systems
- Code reviews and refactoring

---

## Quick Start

This skill follows the **500-line modular pattern** with progressive disclosure:
- **Main file (this)**: High-level overview and quick reference
- **Resource files**: Deep dives into specific topics

### Core Principles

1. **Errors are values** - Check every error, add context, never ignore
2. **Composition over inheritance** - Use interfaces and embedding
3. **Accept interfaces, return structs** - Flexible input, concrete output
4. **Simplicity first** - Start simple, add complexity only when needed
5. **Idiomatic Go** - Follow stdlib patterns and conventions

---

## Project Structure

**Standard layout for applications:**

```
myapp/
├── cmd/
│   └── myapp/
│       └── main.go          # Application entrypoint
├── internal/                # Private application code
│   ├── handler/             # HTTP handlers
│   ├── service/             # Business logic
│   ├── repository/          # Data access
│   └── model/               # Domain models
├── pkg/                     # Public libraries (optional)
├── api/                     # API definitions (OpenAPI, proto)
├── deployments/             # Kubernetes manifests
├── go.mod
└── README.md
```

**Organize by domain, not layer:**

```
// ✅ Good
internal/
├── user/
│   ├── handler.go
│   ├── service.go
│   └── repository.go
└── product/
    ├── handler.go
    ├── service.go
    └── repository.go

// ❌ Bad
internal/
├── handlers/
├── services/
└── repositories/
```

---

## Quick Examples

### Error Handling

```go
// Handle errors first, add context
func ProcessFile(path string) (*Result, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("failed to read %s: %w", path, err)
    }

    result, err := parseData(data)
    if err != nil {
        return nil, fmt.Errorf("failed to parse data: %w", err)
    }

    return result, nil
}
```

### Concurrency

```go
// Use context for cancellation
func FetchData(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    return io.ReadAll(resp.Body)
}
```

### Testing

```go
// Table-driven tests
func TestParseEmail(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    Email
        wantErr bool
    }{
        {
            name:  "valid email",
            input: "user@example.com",
            want:  Email{Local: "user", Domain: "example.com"},
        },
        {
            name:    "invalid - missing @",
            input:   "userexample.com",
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseEmail(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if !tt.wantErr && !reflect.DeepEqual(got, tt.want) {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

### HTTP Server

```go
// Layered architecture
type UserHandler struct {
    service UserService
}

func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := r.URL.Query().Get("id")

    user, err := h.service.GetUser(r.Context(), id)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            http.Error(w, "not found", http.StatusNotFound)
            return
        }
        http.Error(w, "internal error", http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

---

## Resource Files (Deep Dives)

Use the `Read` tool to access detailed guides on specific topics:

### 1. Error Handling
**File:** `resources/error-handling.md`

**Covers:**
- Handle errors first (reduce nesting)
- Error wrapping with context (`%w`)
- Custom error types
- Sentinel errors
- Error handling in layers
- When to panic vs return error

**Use when:** Working with errors, debugging, designing error handling strategy

---

### 2. Concurrency Patterns
**File:** `resources/concurrency-patterns.md`

**Covers:**
- Preventing goroutine leaks
- Worker pool pattern
- Pipeline pattern
- Fan-out, fan-in
- Context usage
- Rate limiting
- Managing background tasks

**Use when:** Implementing concurrent operations, background workers, parallel processing

---

### 3. Testing Patterns
**File:** `resources/testing-patterns.md`

**Covers:**
- Table-driven tests
- Test helpers that return errors
- Mocking with interfaces
- Testing HTTP handlers
- Benchmark tests
- Parallel tests
- Golden files pattern

**Use when:** Writing tests, setting up test infrastructure, debugging test failures

---

### 4. Kubernetes Controllers
**File:** `resources/kubernetes-controllers.md`

**Covers:**
- Reconciliation loop pattern
- Idempotent reconciliation
- Status conditions
- Finalizers for cleanup
- Owner references
- Validation webhooks
- Testing controllers

**Use when:** Building Kubernetes operators, controllers, custom resources

---

### 5. Naming and Style
**File:** `resources/naming-and-style.md`

**Covers:**
- Package naming conventions
- Function and method naming
- Variable naming (short vs long)
- Interface naming (`-er` pattern)
- Receiver names
- Avoiding stutter
- Exported vs unexported

**Use when:** Naming things, code reviews, refactoring

---

### 6. Package Design
**File:** `resources/package-design.md`

**Covers:**
- Package organization
- API design (accept interfaces, return structs)
- Export minimally
- Options pattern
- Internal packages
- Avoiding circular dependencies
- Dependency injection

**Use when:** Designing packages, structuring projects, creating libraries

---

## Common Commands

```bash
# Initialize module
go mod init github.com/username/project

# Run tests
go test ./...
go test -race ./...           # With race detector
go test -v -run TestName      # Specific test

# Build
go build -o bin/app ./cmd/app

# Format and lint
gofmt -w .
go vet ./...

# Manage dependencies
go mod tidy
go mod download
go get -u ./...               # Update dependencies

# Generate code
go generate ./...

# Run with environment
go run ./cmd/app
```

---

## When to Use What

| Situation | Resource to Read |
|-----------|------------------|
| Handling errors in API | error-handling.md |
| Implementing concurrent workers | concurrency-patterns.md |
| Writing tests for service layer | testing-patterns.md |
| Building Kubernetes operator | kubernetes-controllers.md |
| Naming packages/functions | naming-and-style.md |
| Structuring a new project | package-design.md |
| Preventing goroutine leaks | concurrency-patterns.md |
| Using status conditions | kubernetes-controllers.md |
| Table-driven tests | testing-patterns.md |
| Options pattern | package-design.md |

---

## Quick Decision Trees

### Error Handling
```
Error occurred?
├─ Can continue? → Use t.Error (testing) or log and continue
├─ Must stop? → Return error with context
├─ Programming error? → Panic
└─ Expected condition? → Use sentinel error or custom type
```

### Concurrency
```
Need concurrency?
├─ I/O bound? → Use goroutines with channels or context
├─ CPU bound? → Worker pool with runtime.NumCPU() workers
├─ Background task? → Use context for lifecycle management
└─ One-time async? → Buffered channel (size 1)
```

### Testing
```
Writing test?
├─ Multiple scenarios? → Table-driven test
├─ Need mock? → Define interface, create mock implementation
├─ HTTP handler? → Use httptest package
├─ Performance? → Benchmark test
└─ Integration? → Use t.TempDir(), clean up in defer
```

---

## Anti-Patterns to Avoid

**❌ Don't:**
- Ignore errors
- Use panics for control flow
- Create goroutines without exit strategy
- Use global variables for dependencies
- Name packages `util`, `common`, `helper`
- Export implementation details
- Use `this` or `self` for receivers
- Create circular dependencies
- Use init() for complex initialization
- Test implementation details

**✅ Do:**
- Check every error
- Return errors, add context
- Use context for goroutine lifecycle
- Inject dependencies explicitly
- Name packages by domain purpose
- Export minimal API surface
- Use short, consistent receiver names
- Design for dependency injection
- Prefer explicit initialization
- Test public API behavior

---

## Integration with Development Workflow

### Before Writing Code
1. **Read:** `package-design.md` - Structure your package
2. **Read:** `naming-and-style.md` - Plan your naming

### While Writing Code
1. **Follow:** Error handling patterns from `error-handling.md`
2. **Follow:** Naming conventions from `naming-and-style.md`
3. **Use:** Concurrency patterns from `concurrency-patterns.md` if needed

### After Writing Code
1. **Write tests** using patterns from `testing-patterns.md`
2. **Review:** Against anti-patterns listed above
3. **Run:** `go vet`, `gofmt`, `go test -race`

### For Kubernetes Controllers
1. **Read:** `kubernetes-controllers.md` for reconciliation patterns
2. **Implement:** Idempotent reconciliation
3. **Use:** Status conditions and finalizers

---

## Progressive Learning Path

**Beginner:**
1. Start with error-handling.md
2. Learn naming-and-style.md
3. Practice testing-patterns.md

**Intermediate:**
4. Study concurrency-patterns.md
5. Master package-design.md
6. Build projects using standard layout

**Advanced:**
7. Implement kubernetes-controllers.md patterns
8. Create reusable libraries
9. Contribute to open source

---

## References

**Official Go Resources:**
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Blog](https://go.dev/blog/)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

**Style Guides:**
- [Google Go Style Guide](https://google.github.io/styleguide/go/)
- [Uber Go Style Guide](https://github.com/uber-go/guide)

**Kubernetes:**
- [Kubebuilder Book](https://book.kubebuilder.io/)
- [Kubebuilder Best Practices](https://kubebuilder.io/reference/good-practices)

**Talks:**
- [Go Best Practices (2013)](https://go.dev/talks/2013/bestpractices.slide)
- [Go Concurrency Patterns](https://go.dev/talks/2012/concurrency.slide)

---

## How to Use This Skill

1. **Start here** for quick reference and overview
2. **Use Read tool** to access specific resource files
3. **Follow examples** for your use case
4. **Reference decision trees** for quick guidance
5. **Check anti-patterns** during code review

**Example workflow:**
```
User: "How should I handle errors in my HTTP handler?"
Assistant: Let me read error-handling.md for detailed patterns...
[Shows layered error handling approach with context]
```

---

**Remember:** Go emphasizes simplicity, clarity, and maintainability. When in doubt, choose the simpler approach and refer to the standard library for inspiration.
