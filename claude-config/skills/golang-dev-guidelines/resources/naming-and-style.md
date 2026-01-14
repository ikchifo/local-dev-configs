# Naming and Style Conventions in Go

## Core Principles

1. **Clarity over brevity** - Code is read more than written
2. **Use short names in small scopes** - Long names in large scopes
3. **Avoid repetition** - Context provides meaning
4. **Follow Go conventions** - Consistency with stdlib
5. **MixedCaps, not underscores** - `myVariable`, not `my_variable`

---

## Package Naming

**Keep package names short, clear, and singular:**

```go
// ✅ Good
package user
package http
package auth
package cache

// ❌ Bad
package users          // Plural
package userPackage    // Redundant "package"
package user_service   // Underscores
package userUtilities  // Generic "utilities"
```

**Avoid generic names:**

```go
// ❌ Bad
package util
package common
package helper
package lib

// ✅ Good - Be specific
package stringutil
package httputil
package sqldb
```

**Package names provide context:**

```go
// ❌ Bad - Repetitive
package user

type UserService struct {}  // "user.UserService" is redundant
func NewUserService() {}    // "user.NewUserService()" is redundant

// ✅ Good - Let package name provide context
package user

type Service struct {}      // "user.Service" - clear
func NewService() {}        // "user.NewService()" - clear
func (s *Service) Get() {}  // "user.Service.Get()" - clear
```

---

## Function and Method Naming

**Noun-like names for values, verb-like for actions:**

```go
// ✅ Good - Returns value (noun)
func Name() string
func Config() Config
func Database() *sql.DB

// ✅ Good - Performs action (verb)
func Run()
func Start()
func Fetch()
func Parse()
func Create()
```

**Avoid redundancy with package/receiver:**

```go
// ❌ Bad - Redundant
func (u *User) GetUserName() string

// ✅ Good - Package/receiver provides context
func (u *User) Name() string

// Usage: user.Name() - clear and concise
```

**Boolean getters:**

```go
// ✅ Good - Question-like
func IsValid() bool
func HasPrefix() bool
func CanAccess() bool

// ❌ Bad - Not question-like
func GetValid() bool
func CheckPrefix() bool
```

---

## Variable Naming

**Short names in small scopes:**

```go
// ✅ Good - Small scope, short name
func Sum(nums []int) int {
    var s int
    for _, n := range nums {
        s += n
    }
    return s
}

// ✅ Good - Larger scope, descriptive name
type Server struct {
    connectionPool *sql.DB
    requestTimeout time.Duration
    maxRetries     int
}
```

**Common short names:**

```go
// Receivers (first letter of type)
func (u *User) Name() string
func (s *Server) Start() error
func (c *Client) Connect() error

// Loop variables
for i := 0; i < len(items); i++ {}
for k, v := range map {}

// Common abbreviations
var (
    ctx  context.Context
    err  error
    req  *http.Request
    resp *http.Response
    db   *sql.DB
    tx   *sql.Tx
    cfg  Config
    msg  string
)
```

**Avoid single-letter variables in large scopes:**

```go
// ❌ Bad - Hard to search and understand
var (
    c *Client  // What kind of client?
    d *sql.DB  // OK if small scope
    s string   // Too generic
)

// ✅ Good - Clear purpose
var (
    httpClient *http.Client
    database   *sql.DB
    username   string
)
```

---

## Interface Naming

**Single-method interfaces end in `-er`:**

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

type Stringer interface {
    String() string
}

type Handler interface {
    Handle(ctx context.Context) error
}
```

**Multi-method interfaces are nouns:**

```go
type File interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
    Close() error
    Stat() (FileInfo, error)
}

type Repository interface {
    Create(ctx context.Context, item *Item) error
    Get(ctx context.Context, id string) (*Item, error)
    Update(ctx context.Context, item *Item) error
    Delete(ctx context.Context, id string) error
}
```

---

## Receiver Names

**Use short, consistent names (not `this` or `self`):**

```go
// ✅ Good - First letter(s) of type
func (u *User) Name() string {}
func (s *Server) Start() error {}
func (db *Database) Connect() error {}

// ✅ Good - Consistent across all methods
type UserService struct{}

func (us *UserService) Create() {}
func (us *UserService) Get() {}
func (us *UserService) Update() {}

// ❌ Bad - Inconsistent
func (service *UserService) Create() {}
func (us *UserService) Get() {}
func (this *UserService) Update() {} // Don't use "this"
```

**Pointer vs value receivers:**

```go
// Use pointer receiver when:
// 1. Method modifies the receiver
// 2. Receiver is large struct
// 3. Consistency (some methods need pointer, so all use pointer)

type Counter struct {
    count int
}

// ✅ Pointer - modifies receiver
func (c *Counter) Increment() {
    c.count++
}

// ✅ Pointer - consistency
func (c *Counter) Value() int {
    return c.count
}

// Value receiver for small, immutable types
type Point struct {
    X, Y int
}

// ✅ Value - small, immutable
func (p Point) String() string {
    return fmt.Sprintf("(%d, %d)", p.X, p.Y)
}
```

---

## Error Variable Naming

**Prefix with `Err` for sentinel errors:**

```go
var (
    ErrNotFound      = errors.New("not found")
    ErrUnauthorized  = errors.New("unauthorized")
    ErrInvalidInput  = errors.New("invalid input")
    ErrTimeout       = errors.New("operation timed out")
)
```

**Error types end with `Error`:**

```go
type ValidationError struct {
    Field   string
    Message string
}

type NetworkError struct {
    Op  string
    Err error
}

type AuthenticationError struct {
    Reason string
}
```

---

## Constant Naming

**Use MixedCaps, not SCREAMING_CASE:**

```go
// ✅ Good
const (
    MaxRetries = 3
    DefaultTimeout = 30 * time.Second
    APIVersion = "v1"
)

// ❌ Bad - Go is not C
const (
    MAX_RETRIES = 3
    DEFAULT_TIMEOUT = 30
    API_VERSION = "v1"
)
```

**Enum-like constants:**

```go
type Status int

const (
    StatusPending Status = iota
    StatusRunning
    StatusComplete
    StatusFailed
)

// Or with explicit values
const (
    StatusPending  Status = 0
    StatusRunning  Status = 1
    StatusComplete Status = 2
    StatusFailed   Status = 3
)
```

---

## File Naming

**Lowercase with underscores for separation:**

```go
// ✅ Good
user.go
user_service.go
user_service_test.go
http_handler.go
database_test.go

// ❌ Bad
User.go
userService.go
user-service.go  // Use underscores, not hyphens
```

---

## Avoid Stutter

**Package name provides context:**

```go
// ❌ Bad - Stutter
package auth

type AuthService struct{}    // auth.AuthService
type AuthConfig struct{}     // auth.AuthConfig
func NewAuthService() {}     // auth.NewAuthService()

// ✅ Good - No stutter
package auth

type Service struct{}        // auth.Service
type Config struct{}         // auth.Config
func NewService() {}         // auth.NewService()
```

**Import renaming when necessary:**

```go
import (
    "crypto/rand"
    mathrand "math/rand"  // Rename to avoid conflict
)

func example() {
    n, _ := rand.Read(buf)          // crypto/rand
    x := mathrand.Intn(100)         // math/rand
}
```

---

## Exported vs Unexported

**Capital first letter = exported (public):**

```go
// Exported - available to other packages
type User struct {
    ID   int     // Exported field
    Name string  // Exported field
}

func NewUser() *User {}  // Exported function

// Unexported - internal to package
type userCache struct {
    data map[int]*User
}

func validateUser(u *User) error {}  // Unexported function
```

**Hide implementation details:**

```go
package user

// ✅ Good - Export interface, hide implementation
type Repository interface {
    Get(id int) (*User, error)
}

type sqlRepository struct {  // Unexported
    db *sql.DB
}

func NewRepository(db *sql.DB) Repository {  // Exported factory
    return &sqlRepository{db: db}
}

// ❌ Bad - Exposing implementation
type SQLRepository struct {  // Exported
    DB *sql.DB               // Exported field
}
```

---

## Acronyms and Initialisms

**Keep acronyms uppercase or lowercase together:**

```go
// ✅ Good
type HTTPServer struct{}
type URLParser struct{}
type IDGenerator struct{}
var userID int
var apiURL string

// ❌ Bad
type HttpServer struct{}
type UrlParser struct{}
type IdGenerator struct{}
var userId int      // Should be userID
var apiUrl string   // Should be apiURL
```

---

## Comment Style

**Complete sentences with proper punctuation:**

```go
// ✅ Good - Complete sentences
// User represents a user in the system.
// It contains authentication and profile information.
type User struct {
    // ID is the unique identifier for the user.
    ID int

    // Name is the user's full name.
    Name string
}

// NewUser creates a new User with the given name.
// It returns an error if the name is empty.
func NewUser(name string) (*User, error) {
    if name == "" {
        return nil, errors.New("name is required")
    }
    return &User{Name: name}, nil
}

// ❌ Bad - Incomplete sentences
// user struct
type User struct {
    ID   int    // id
    Name string // name
}

// new user
func NewUser(name string) (*User, error) {}
```

---

## Variable Declaration Style

**Use `:=` for new variables:**

```go
// ✅ Good
user := NewUser()
count := 0
names := []string{}

// Use var when:
// 1. Zero value is desired
var count int  // 0

// 2. Type differs from right side
var timeout time.Duration = 5 * time.Second

// 3. Declaring multiple variables
var (
    name  string
    age   int
    email string
)
```

---

## Best Practices Summary

1. **MixedCaps** - Not snake_case or kebab-case
2. **Short names in small scopes** - Long names in large scopes
3. **Avoid stutter** - `user.Service`, not `user.UserService`
4. **Receivers**: First letter(s) of type, consistent across methods
5. **Interfaces**: `-er` for single method, noun for multiple
6. **Errors**: `Err` prefix for sentinels, `Error` suffix for types
7. **Exported = Capitalized** - Unexported = lowercase
8. **Acronyms**: All caps or all lowercase (`HTTPServer`, `userID`)
9. **Package names**: Short, singular, lowercase, no underscores
10. **Avoid generic names**: `util`, `common`, `helper`
11. **Comments**: Complete sentences for exported identifiers
12. **Files**: lowercase_with_underscores.go

---

## References

- [Effective Go: Names](https://go.dev/doc/effective_go#names)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Google Go Style Guide](https://google.github.io/styleguide/go/)
