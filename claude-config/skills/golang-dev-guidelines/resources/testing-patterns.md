# Testing Patterns in Go

## Core Principles

1. **Table-driven tests** - Test multiple scenarios systematically
2. **Keep logic in Test functions** - Not in helpers
3. **Prefer `t.Error` over `t.Fatal`** - Continue testing when possible
4. **Test helpers return errors** - Don't use assertions
5. **Name test cases clearly** - Describe what's being tested

---

## Table-Driven Tests

**Standard pattern for testing multiple scenarios:**

```go
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
            want: Email{
                Local:  "user",
                Domain: "example.com",
            },
            wantErr: false,
        },
        {
            name:    "missing @",
            input:   "userexample.com",
            wantErr: true,
        },
        {
            name:    "empty string",
            input:   "",
            wantErr: true,
        },
        {
            name:  "subdomain",
            input: "user@mail.example.com",
            want: Email{
                Local:  "user",
                Domain: "mail.example.com",
            },
            wantErr: false,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseEmail(tt.input)

            if (err != nil) != tt.wantErr {
                t.Errorf("ParseEmail() error = %v, wantErr %v", err, tt.wantErr)
                return
            }

            if !tt.wantErr && !reflect.DeepEqual(got, tt.want) {
                t.Errorf("ParseEmail() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

---

## Specify Field Names in Test Cases

**✅ Good - Clear and maintainable:**

```go
tests := []struct {
    name    string
    input   int
    want    int
    wantErr bool
}{
    {
        name:    "positive number",
        input:   5,
        want:    25,
        wantErr: false,
    },
    {
        name:    "negative number",
        input:   -5,
        want:    0,
        wantErr: true,
    },
}
```

**❌ Bad - Positional, easy to make mistakes:**

```go
tests := []struct {
    string
    int
    int
    bool
}{
    {"positive number", 5, 25, false},
    {"negative number", -5, 0, true},
}
```

---

## Test Helpers That Return Errors

**Don't use assertion helpers:**

```go
// ❌ Bad - Helper calls t.Fatal, stops test
func assertNoError(t *testing.T, err error) {
    if err != nil {
        t.Fatal(err)
    }
}

// ❌ Bad - Hides test logic
func assertEqual(t *testing.T, got, want interface{}) {
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}

// ✅ Good - Helper returns error, test decides what to do
func setupTestDB(t *testing.T) (*sql.DB, error) {
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        return nil, err
    }

    if err := db.Ping(); err != nil {
        return nil, err
    }

    return db, nil
}

// Test has logic
func TestUserRepository(t *testing.T) {
    db, err := setupTestDB(t)
    if err != nil {
        t.Fatalf("failed to setup db: %v", err)
    }
    defer db.Close()

    // Test logic here
}
```

---

## Prefer t.Error Over t.Fatal

**Let tests continue when possible:**

```go
func TestMultipleValidations(t *testing.T) {
    user := &User{
        Name:  "John",
        Email: "invalid-email",
        Age:   -5,
    }

    // ❌ Bad - stops at first failure
    if user.Email == "" {
        t.Fatal("email is empty")
    }
    if user.Age < 0 {
        t.Fatal("age is negative") // Never reached if email check fails
    }

    // ✅ Good - reports all failures
    if user.Email == "" {
        t.Error("email is empty")
    }
    if user.Age < 0 {
        t.Error("age is negative")
    }
    // Both errors will be reported
}
```

**Use `t.Fatal` only when continuing is impossible:**

```go
func TestDatabaseOperations(t *testing.T) {
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatal(err) // Can't continue without DB
    }
    defer db.Close()

    // Rest of test...
}
```

---

## Test Fixtures and Setup

**Use subtests for shared setup:**

```go
func TestUserService(t *testing.T) {
    // Common setup
    db := setupTestDB(t)
    defer db.Close()

    repo := NewUserRepository(db)
    service := NewUserService(repo)

    t.Run("CreateUser", func(t *testing.T) {
        user, err := service.CreateUser("John", "john@example.com")
        if err != nil {
            t.Fatalf("CreateUser() error = %v", err)
        }
        if user.Name != "John" {
            t.Errorf("got name %s, want John", user.Name)
        }
    })

    t.Run("GetUser", func(t *testing.T) {
        // Create user first
        created, _ := service.CreateUser("Jane", "jane@example.com")

        // Test retrieval
        user, err := service.GetUser(created.ID)
        if err != nil {
            t.Fatalf("GetUser() error = %v", err)
        }
        if user.ID != created.ID {
            t.Errorf("got ID %d, want %d", user.ID, created.ID)
        }
    })
}
```

---

## Testing with Interfaces (Mocking)

**Use interfaces for dependencies:**

```go
// Production code
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    Get(ctx context.Context, id int64) (*User, error)
}

type UserService struct {
    repo UserRepository
}

func (s *UserService) CreateUser(ctx context.Context, name, email string) (*User, error) {
    user := &User{Name: name, Email: email}
    if err := s.repo.Create(ctx, user); err != nil {
        return nil, err
    }
    return user, nil
}

// Test code
type mockUserRepository struct {
    createFunc func(ctx context.Context, user *User) error
    getFunc    func(ctx context.Context, id int64) (*User, error)
}

func (m *mockUserRepository) Create(ctx context.Context, user *User) error {
    if m.createFunc != nil {
        return m.createFunc(ctx, user)
    }
    return nil
}

func (m *mockUserRepository) Get(ctx context.Context, id int64) (*User, error) {
    if m.getFunc != nil {
        return m.getFunc(ctx, id)
    }
    return nil, nil
}

// Test
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name       string
        userName   string
        userEmail  string
        repoCreate func(ctx context.Context, user *User) error
        wantErr    bool
    }{
        {
            name:      "successful creation",
            userName:  "John",
            userEmail: "john@example.com",
            repoCreate: func(ctx context.Context, user *User) error {
                user.ID = 123 // Simulate DB assigning ID
                return nil
            },
            wantErr: false,
        },
        {
            name:      "repository error",
            userName:  "Jane",
            userEmail: "jane@example.com",
            repoCreate: func(ctx context.Context, user *User) error {
                return errors.New("db error")
            },
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            mock := &mockUserRepository{
                createFunc: tt.repoCreate,
            }

            service := &UserService{repo: mock}

            user, err := service.CreateUser(context.Background(), tt.userName, tt.userEmail)

            if (err != nil) != tt.wantErr {
                t.Errorf("CreateUser() error = %v, wantErr %v", err, tt.wantErr)
                return
            }

            if !tt.wantErr {
                if user.Name != tt.userName {
                    t.Errorf("got name %s, want %s", user.Name, tt.userName)
                }
            }
        })
    }
}
```

---

## Testing HTTP Handlers

**Use httptest package:**

```go
func TestUserHandler_GetUser(t *testing.T) {
    tests := []struct {
        name           string
        userID         string
        mockGetUser    func(ctx context.Context, id int64) (*User, error)
        wantStatusCode int
        wantBody       string
    }{
        {
            name:   "user found",
            userID: "123",
            mockGetUser: func(ctx context.Context, id int64) (*User, error) {
                return &User{ID: 123, Name: "John"}, nil
            },
            wantStatusCode: http.StatusOK,
            wantBody:       `{"id":123,"name":"John"}`,
        },
        {
            name:   "user not found",
            userID: "999",
            mockGetUser: func(ctx context.Context, id int64) (*User, error) {
                return nil, ErrNotFound
            },
            wantStatusCode: http.StatusNotFound,
        },
        {
            name:           "invalid user id",
            userID:         "abc",
            wantStatusCode: http.StatusBadRequest,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Setup mock service
            mockService := &mockUserService{
                getUserFunc: tt.mockGetUser,
            }

            handler := NewUserHandler(mockService)

            // Create request
            req := httptest.NewRequest("GET", "/users/"+tt.userID, nil)
            w := httptest.NewRecorder()

            // Call handler
            handler.GetUser(w, req)

            // Check status code
            if w.Code != tt.wantStatusCode {
                t.Errorf("got status %d, want %d", w.Code, tt.wantStatusCode)
            }

            // Check body if specified
            if tt.wantBody != "" {
                got := strings.TrimSpace(w.Body.String())
                if got != tt.wantBody {
                    t.Errorf("got body %s, want %s", got, tt.wantBody)
                }
            }
        })
    }
}
```

---

## Benchmark Tests

**Measure performance:**

```go
func BenchmarkProcessData(b *testing.B) {
    data := generateTestData(1000)

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        processData(data)
    }
}

// With different input sizes
func BenchmarkProcessData_Sizes(b *testing.B) {
    sizes := []int{10, 100, 1000, 10000}

    for _, size := range sizes {
        b.Run(fmt.Sprintf("size_%d", size), func(b *testing.B) {
            data := generateTestData(size)
            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                processData(data)
            }
        })
    }
}
```

---

## Testing with Temporary Files

**Use t.TempDir():**

```go
func TestFileProcessor(t *testing.T) {
    // Create temporary directory (auto-cleaned)
    tmpDir := t.TempDir()

    // Create test file
    testFile := filepath.Join(tmpDir, "test.txt")
    content := []byte("test data")
    if err := os.WriteFile(testFile, content, 0644); err != nil {
        t.Fatalf("failed to write test file: %v", err)
    }

    // Test your code
    result, err := ProcessFile(testFile)
    if err != nil {
        t.Fatalf("ProcessFile() error = %v", err)
    }

    // Verify result
    if result != "expected" {
        t.Errorf("got %s, want expected", result)
    }

    // No cleanup needed - t.TempDir() handles it
}
```

---

## Parallel Tests

**Speed up test execution:**

```go
func TestParallelOperations(t *testing.T) {
    tests := []struct {
        name  string
        input int
        want  int
    }{
        {"case1", 1, 2},
        {"case2", 2, 4},
        {"case3", 3, 6},
    }

    for _, tt := range tests {
        tt := tt // Capture range variable
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // Run in parallel

            got := processInput(tt.input)
            if got != tt.want {
                t.Errorf("got %d, want %d", got, tt.want)
            }
        })
    }
}
```

---

## Testing with Timeouts

**Prevent hanging tests:**

```go
func TestLongRunningOperation(t *testing.T) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    done := make(chan bool, 1)
    go func() {
        result := longRunningOperation()
        if result != expected {
            t.Error("unexpected result")
        }
        done <- true
    }()

    select {
    case <-done:
        // Test completed
    case <-ctx.Done():
        t.Fatal("test timeout")
    }
}
```

---

## Golden Files Pattern

**For complex output validation:**

```go
func TestGenerateOutput(t *testing.T) {
    got := generateOutput()

    goldenFile := "testdata/output.golden"

    if *update {
        // Update golden file
        os.WriteFile(goldenFile, got, 0644)
    }

    want, err := os.ReadFile(goldenFile)
    if err != nil {
        t.Fatalf("failed to read golden file: %v", err)
    }

    if !bytes.Equal(got, want) {
        t.Errorf("output mismatch\ngot:\n%s\nwant:\n%s", got, want)
    }
}

var update = flag.Bool("update", false, "update golden files")
```

---

## Best Practices Summary

1. **Table-driven tests** - Test multiple scenarios systematically
2. **Name fields explicitly** - In test struct literals
3. **t.Run for subtests** - Organize related tests
4. **Prefer t.Error** - Continue testing when possible
5. **Helpers return errors** - Don't call t.Fatal in helpers
6. **Use interfaces** - For mocking dependencies
7. **Test public APIs** - Not implementation details
8. **t.TempDir()** - For temporary files/directories
9. **t.Parallel()** - Speed up independent tests
10. **Keep logic in Test functions** - Not hidden in helpers
11. **Benchmark critical paths** - Measure performance
12. **Test error cases** - Not just happy paths

---

## References

- [Go Testing Package](https://pkg.go.dev/testing)
- [Table Driven Tests](https://go.dev/wiki/TableDrivenTests)
- [Google Go Style: Testing](https://google.github.io/styleguide/go/best-practices#testing)
