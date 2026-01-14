# Error Handling in Go

## Core Principles

1. **Errors are values** - Process them deliberately, don't ignore them
2. **Handle errors first** - Reduce nesting, improve readability
3. **Add context** - Make errors meaningful and debuggable
4. **Return, don't panic** - Panics for truly exceptional cases only

---

## Handle Errors First (Reduce Nesting)

**❌ Bad - Deep nesting:**
```go
func ProcessFile(path string) error {
    data, err := os.ReadFile(path)
    if err == nil {
        result, err := parseData(data)
        if err == nil {
            if err := saveResult(result); err == nil {
                return nil
            } else {
                return err
            }
        } else {
            return err
        }
    } else {
        return err
    }
}
```

**✅ Good - Early returns:**
```go
func ProcessFile(path string) error {
    data, err := os.ReadFile(path)
    if err != nil {
        return fmt.Errorf("failed to read file %s: %w", path, err)
    }

    result, err := parseData(data)
    if err != nil {
        return fmt.Errorf("failed to parse data: %w", err)
    }

    if err := saveResult(result); err != nil {
        return fmt.Errorf("failed to save result: %w", err)
    }

    return nil
}
```

---

## Error Wrapping with Context

**Always wrap errors with context using `%w`:**

```go
func LoadUserData(userID int64) (*User, error) {
    // Add context at each layer
    data, err := fetchFromDB(userID)
    if err != nil {
        return nil, fmt.Errorf("load user %d: failed to fetch from db: %w", userID, err)
    }

    user, err := unmarshalUser(data)
    if err != nil {
        return nil, fmt.Errorf("load user %d: failed to unmarshal: %w", userID, err)
    }

    return user, nil
}
```

**Place `%w` at the end of error strings:**
```go
// ✅ Good - %w at the end
fmt.Errorf("database query failed for user %d: %w", id, err)

// ❌ Bad - %w not at the end
fmt.Errorf("%w: database query failed for user %d", err, id)
```

---

## Custom Error Types

**For errors that need programmatic inspection:**

```go
// Define error type
type ValidationError struct {
    Field   string
    Value   interface{}
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s (value: %v)",
        e.Field, e.Message, e.Value)
}

// Usage
func ValidateUser(u *User) error {
    if u.Email == "" {
        return &ValidationError{
            Field:   "email",
            Value:   u.Email,
            Message: "required field",
        }
    }

    if !strings.Contains(u.Email, "@") {
        return &ValidationError{
            Field:   "email",
            Value:   u.Email,
            Message: "invalid format",
        }
    }

    return nil
}

// Caller can inspect
if err := ValidateUser(user); err != nil {
    var valErr *ValidationError
    if errors.As(err, &valErr) {
        // Handle validation error specifically
        log.Printf("Validation failed for field: %s", valErr.Field)
        return
    }
    // Handle other errors
    return err
}
```

---

## Sentinel Errors

**For well-known error conditions:**

```go
var (
    ErrNotFound      = errors.New("resource not found")
    ErrAlreadyExists = errors.New("resource already exists")
    ErrUnauthorized  = errors.New("unauthorized")
)

func GetUser(id int64) (*User, error) {
    user, exists := userCache[id]
    if !exists {
        return nil, ErrNotFound
    }
    return user, nil
}

// Caller checks with errors.Is
user, err := GetUser(123)
if errors.Is(err, ErrNotFound) {
    // Handle not found case
    return nil
}
if err != nil {
    // Handle other errors
    return err
}
```

---

## Error Handling in Layers

**Clean separation between layers:**

```go
// Repository layer - returns simple errors
type UserRepository struct {
    db *sql.DB
}

func (r *UserRepository) FindByID(ctx context.Context, id int64) (*User, error) {
    var user User
    err := r.db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = $1", id).
        Scan(&user.ID, &user.Name, &user.Email)

    if err == sql.ErrNoRows {
        return nil, ErrNotFound
    }
    if err != nil {
        return nil, err // Let caller wrap with context
    }

    return &user, nil
}

// Service layer - adds business context
type UserService struct {
    repo *UserRepository
}

func (s *UserService) GetUser(ctx context.Context, id int64) (*User, error) {
    user, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("get user %d: %w", id, err)
    }

    // Business logic validations
    if !user.IsActive {
        return nil, fmt.Errorf("user %d is not active", id)
    }

    return user, nil
}

// Handler layer - converts to HTTP errors
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id, _ := strconv.ParseInt(r.URL.Query().Get("id"), 10, 64)

    user, err := h.service.GetUser(r.Context(), id)
    if err != nil {
        // Convert to appropriate HTTP status
        if errors.Is(err, ErrNotFound) {
            http.Error(w, "user not found", http.StatusNotFound)
            return
        }

        var valErr *ValidationError
        if errors.As(err, &valErr) {
            http.Error(w, valErr.Error(), http.StatusBadRequest)
            return
        }

        // Log internal errors, return generic message
        log.Printf("internal error: %v", err)
        http.Error(w, "internal server error", http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

---

## When to Panic vs Return Error

**Panic only for:**
- Programming errors that should never happen in production
- Initialization failures (e.g., config parsing)
- Truly exceptional situations

```go
// ✅ Good use of panic - initialization
func init() {
    config, err := LoadConfig()
    if err != nil {
        panic(fmt.Sprintf("failed to load config: %v", err))
    }
}

// ❌ Bad - don't panic in library code
func ProcessData(data []byte) Result {
    if len(data) == 0 {
        panic("empty data") // Bad!
    }
    // ...
}

// ✅ Good - return error instead
func ProcessData(data []byte) (*Result, error) {
    if len(data) == 0 {
        return nil, errors.New("empty data")
    }
    // ...
}
```

---

## Error Checking in Loops

**Extract error checks to reduce nesting:**

```go
// ❌ Bad - nested error checks
func ProcessItems(items []Item) error {
    for _, item := range items {
        result, err := process(item)
        if err == nil {
            if err := save(result); err == nil {
                continue
            } else {
                return err
            }
        } else {
            return err
        }
    }
    return nil
}

// ✅ Good - early returns
func ProcessItems(items []Item) error {
    for _, item := range items {
        result, err := process(item)
        if err != nil {
            return fmt.Errorf("failed to process item %v: %w", item.ID, err)
        }

        if err := save(result); err != nil {
            return fmt.Errorf("failed to save result for item %v: %w", item.ID, err)
        }
    }
    return nil
}
```

---

## Error Helper Functions

**Create helpers for common error patterns:**

```go
// Helper for database operations
func dbError(op string, err error) error {
    if err == nil {
        return nil
    }
    if err == sql.ErrNoRows {
        return ErrNotFound
    }
    return fmt.Errorf("db %s: %w", op, err)
}

// Usage
func (r *Repository) GetUser(id int64) (*User, error) {
    var user User
    err := r.db.QueryRow("SELECT * FROM users WHERE id = $1", id).
        Scan(&user.ID, &user.Name)

    if err := dbError("get user", err); err != nil {
        return nil, err
    }

    return &user, nil
}
```

---

## Best Practices Summary

1. **Always check errors** - Never ignore `err` values
2. **Handle first** - Use early returns to reduce nesting
3. **Add context** - Wrap errors with meaningful information
4. **Use `%w`** - Place at end of format string for proper unwrapping
5. **Custom types** - For errors that need inspection
6. **Sentinel errors** - For well-known conditions
7. **Return, don't panic** - Prefer errors over panics
8. **Layer appropriately** - Different layers add different context
9. **Check with `errors.Is` and `errors.As`** - For type-safe error inspection
10. **Log internal errors** - But return generic messages to users

---

## References

- [Go Blog: Error Handling and Go](https://go.dev/blog/error-handling-and-go)
- [Go Blog: Working with Errors](https://go.dev/blog/go1.13-errors)
- [Google Go Style Guide: Error Handling](https://google.github.io/styleguide/go/best-practices#error-handling)
