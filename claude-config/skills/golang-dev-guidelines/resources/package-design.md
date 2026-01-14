# Package Design in Go

## Core Principles

1. **Packages group related functionality** - Clear, focused purpose
2. **Design for clients** - Easy to use correctly, hard to use incorrectly
3. **Accept interfaces, return structs** - Flexible inputs, concrete outputs
4. **Export minimally** - Hide implementation details
5. **Avoid circular dependencies** - Clean architecture

---

## Package Organization

**Organize by domain, not by layer:**

```
// ❌ Bad - Organized by layer
project/
├── controllers/
│   ├── user_controller.go
│   └── product_controller.go
├── services/
│   ├── user_service.go
│   └── product_service.go
└── repositories/
    ├── user_repository.go
    └── product_repository.go

// ✅ Good - Organized by domain
project/
├── user/
│   ├── handler.go      // HTTP handlers
│   ├── service.go      // Business logic
│   ├── repository.go   // Data access
│   └── user.go         // Types
└── product/
    ├── handler.go
    ├── service.go
    ├── repository.go
    └── product.go
```

---

## Standard Project Layout

**For applications:**

```
myapp/
├── cmd/
│   └── myapp/
│       └── main.go          # Application entrypoint
├── internal/
│   ├── user/                # Internal packages
│   ├── product/
│   └── auth/
├── pkg/
│   └── api/                 # Public library (if needed)
├── api/
│   ├── openapi.yaml         # API definitions
│   └── proto/               # Protocol buffers
├── deployments/
│   └── kubernetes/          # K8s manifests
├── scripts/                 # Build scripts
├── go.mod
└── README.md
```

**For libraries:**

```
mylib/
├── mylib.go                 # Main package file
├── option.go                # Options pattern
├── internal/                # Private implementation
│   └── helper/
├── examples/                # Usage examples
├── go.mod
└── README.md
```

---

## Package Responsibilities

**Each package should have a clear, single responsibility:**

```go
// ✅ Good - Clear purpose
package jwt
// Provides JWT token generation and validation

package cache
// Provides in-memory caching with expiration

package email
// Sends emails via SMTP

// ❌ Bad - Multiple responsibilities
package utils
// Contains random helper functions

package helpers
// Contains everything that doesn't fit elsewhere
```

---

## API Design: Accept Interfaces

**Functions should ask for the minimum interface they need:**

```go
// ❌ Bad - Requires specific type
func ProcessFile(file *os.File) error {
    data, err := io.ReadAll(file)
    // ...
}

// ✅ Good - Accepts interface
func ProcessFile(r io.Reader) error {
    data, err := io.ReadAll(r)
    // ...
}

// Now works with files, buffers, network connections, etc.
```

**Define interfaces in consumer packages:**

```go
// ❌ Bad - Provider defines interface
package database

type Repository interface {
    Get(id int) (*User, error)
}

type PostgresRepo struct{}
func (p *PostgresRepo) Get(id int) (*User, error) {}

// ✅ Good - Consumer defines interface
package service

// Service defines what it needs
type UserRepository interface {
    Get(id int) (*User, error)
}

type UserService struct {
    repo UserRepository  // Accepts any implementation
}

// Provider just implements the interface
package postgres

type UserRepository struct {
    db *sql.DB
}

func (r *UserRepository) Get(id int) (*User, error) {}
```

---

## API Design: Return Structs

**Return concrete types, not interfaces:**

```go
// ✅ Good - Returns struct
func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

// ❌ Bad - Returns interface (usually)
func NewUserService(repo UserRepository) UserService {
    return &userService{repo: repo}
}

// Exception: When you need to return different implementations
func NewCache(typ string) Cache {
    switch typ {
    case "memory":
        return NewMemoryCache()
    case "redis":
        return NewRedisCache()
    default:
        return NewMemoryCache()
    }
}
```

---

## Export Minimally

**Only export what's necessary:**

```go
package user

// ✅ Good - Export interface, hide implementation
type Service interface {
    Get(id int) (*User, error)
    Create(user *User) error
}

type service struct {  // Unexported
    repo repository    // Unexported
}

type repository struct {  // Unexported
    db *sql.DB
}

// Factory function returns interface
func NewService(db *sql.DB) Service {
    return &service{
        repo: &repository{db: db},
    }
}

// ❌ Bad - Exposing implementation details
type Service struct {  // Exported
    Repo *Repository   // Exported field
}

type Repository struct {  // Exported
    DB *sql.DB           // Exported field
}
```

---

## Options Pattern

**For functions with many optional parameters:**

```go
// Server represents an HTTP server
type Server struct {
    addr           string
    timeout        time.Duration
    maxConnections int
    logger         Logger
}

// Option is a functional option for Server
type Option func(*Server)

// WithTimeout sets the server timeout
func WithTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.timeout = d
    }
}

// WithMaxConnections sets max connections
func WithMaxConnections(n int) Option {
    return func(s *Server) {
        s.maxConnections = n
    }
}

// WithLogger sets the logger
func WithLogger(l Logger) Option {
    return func(s *Server) {
        s.logger = l
    }
}

// NewServer creates a new Server with options
func NewServer(addr string, opts ...Option) *Server {
    // Set defaults
    s := &Server{
        addr:           addr,
        timeout:        30 * time.Second,
        maxConnections: 100,
        logger:         &defaultLogger{},
    }

    // Apply options
    for _, opt := range opts {
        opt(s)
    }

    return s
}

// Usage
server := NewServer(
    ":8080",
    WithTimeout(60*time.Second),
    WithMaxConnections(200),
    WithLogger(myLogger),
)
```

---

## Internal Packages

**Use `internal/` to prevent external imports:**

```
myapp/
├── user/
│   └── user.go              # Public API
└── internal/
    └── database/
        └── conn.go          # Cannot be imported by external packages

// ✅ OK
import "myapp/user"

// ❌ Error - Cannot import internal package
import "myapp/internal/database"
```

---

## Avoid Circular Dependencies

**Bad design that creates cycles:**

```go
// ❌ Bad - Circular dependency
package user
import "myapp/order"

type User struct {
    Orders []order.Order  // user depends on order
}

package order
import "myapp/user"

type Order struct {
    User user.User  // order depends on user
}
```

**Solutions:**

**1. Extract common types:**

```go
// ✅ Good - Extract to shared package
package domain

type User struct {
    ID   int
    Name string
}

type Order struct {
    ID     int
    UserID int
}

// User package imports domain
package user
import "myapp/domain"

// Order package imports domain
package order
import "myapp/domain"
```

**2. Use interfaces:**

```go
// ✅ Good - Use interface
package user

type User struct {
    ID   int
    Name string
}

package order

type Owner interface {  // Don't import user package
    ID() int
}

type Order struct {
    Owner Owner
}
```

**3. Dependency Injection:**

```go
// ✅ Good - Inject dependencies
package user

type Service struct {
    orderClient OrderClient  // Interface defined in user package
}

type OrderClient interface {
    GetUserOrders(userID int) ([]Order, error)
}

package order

type Service struct{}

// Implements user.OrderClient
func (s *Service) GetUserOrders(userID int) ([]Order, error) {}

// Wire them together in main
package main

func main() {
    orderSvc := order.NewService()
    userSvc := user.NewService(orderSvc)  // Inject
}
```

---

## Package Documentation

**Document the package purpose:**

```go
// Package jwt provides JWT token generation and validation.
//
// This package implements RFC 7519 JSON Web Tokens with support
// for HS256, HS384, and HS512 signing algorithms.
//
// Example usage:
//
//	token, err := jwt.Generate(claims, secret)
//	if err != nil {
//	    return err
//	}
//
//	claims, err := jwt.Validate(token, secret)
package jwt
```

---

## Init Functions

**Use sparingly - mainly for registration:**

```go
// ✅ Good use case - registering SQL driver
package postgres

import (
    "database/sql"
    _ "github.com/lib/pq"  // Calls init() to register driver
)

// ✅ Good use case - registering codec
func init() {
    codec.Register("json", &JSONCodec{})
    codec.Register("protobuf", &ProtobufCodec{})
}

// ❌ Bad - Complex initialization
func init() {
    // Don't do complex work in init
    db, err := sql.Open("postgres", connectionString)
    if err != nil {
        panic(err)
    }
    globalDB = db
}

// ✅ Good - Explicit initialization
func Initialize() error {
    db, err := sql.Open("postgres", connectionString)
    if err != nil {
        return err
    }
    globalDB = db
    return nil
}
```

---

## Dependency Management

**Organize dependencies clearly:**

```go
// ✅ Good - Clear dependency structure
package service

type Service struct {
    userRepo    UserRepository
    emailClient EmailClient
    cache       Cache
    logger      Logger
}

func NewService(
    userRepo UserRepository,
    emailClient EmailClient,
    cache Cache,
    logger Logger,
) *Service {
    return &Service{
        userRepo:    userRepo,
        emailClient: emailClient,
        cache:       cache,
        logger:      logger,
    }
}

// ❌ Bad - Hidden global dependencies
package service

var (
    db     *sql.DB       // Global state
    cache  *redis.Client
    logger *log.Logger
)

type Service struct{}

func NewService() *Service {
    // Uses global variables
    return &Service{}
}
```

---

## Versioning

**Use semantic import versioning for major versions:**

```go
// v1
module github.com/myuser/mylib

// v2 or higher
module github.com/myuser/mylib/v2

// Import in client code
import "github.com/myuser/mylib/v2"
```

---

## Best Practices Summary

1. **Organize by domain** - Not by technical layer
2. **Single responsibility** - Each package has clear purpose
3. **Accept interfaces** - Ask for minimum needed
4. **Return structs** - Concrete types for callers
5. **Export minimally** - Hide implementation details
6. **Use internal/** - Prevent external imports
7. **Avoid cycles** - Extract common types or use interfaces
8. **Options pattern** - For many optional parameters
9. **Document packages** - Explain purpose and usage
10. **Explicit initialization** - Avoid complex init() functions
11. **Clear dependencies** - No hidden global state
12. **Follow stdlib patterns** - Learn from standard library

---

## References

- [Go Blog: Package Names](https://go.dev/blog/package-names)
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
- [Google Go Style Guide: Package Design](https://google.github.io/styleguide/go/best-practices#package-design)
- [Effective Go: Package Names](https://go.dev/doc/effective_go#package-names)
