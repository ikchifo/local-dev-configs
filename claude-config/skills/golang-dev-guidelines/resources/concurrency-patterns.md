# Concurrency Patterns in Go

## Core Principles

1. **Share memory by communicating** - Use channels, not shared variables
2. **Prevent goroutine leaks** - Every goroutine needs a way to exit
3. **Expose synchronous APIs** - Let callers add concurrency as needed
4. **Use channels for state** - Manage goroutine lifecycle with channels

---

## Preventing Goroutine Leaks

### Problem: Blocked Goroutine

**❌ Bad - Goroutine can leak:**
```go
func search(query string) []Result {
    results := make(chan Result)

    go func() {
        // Search operation
        result := performSearch(query)
        results <- result // Blocks forever if no receiver!
    }()

    // If we return early, goroutine leaks
    return <-results
}
```

### Solution 1: Buffered Channel

**✅ Good - Won't block on send:**
```go
func search(query string) Result {
    // Buffer size 1 ensures send never blocks
    results := make(chan Result, 1)

    go func() {
        result := performSearch(query)
        results <- result // Never blocks
    }()

    return <-results
}
```

### Solution 2: Quit Channel

**✅ Good - Explicit cancellation:**
```go
func search(query string, timeout time.Duration) (Result, error) {
    results := make(chan Result)
    quit := make(chan struct{})

    go func() {
        select {
        case results <- performSearch(query):
        case <-quit:
            return // Exit goroutine cleanly
        }
    }()

    select {
    case result := <-results:
        return result, nil
    case <-time.After(timeout):
        close(quit) // Signal goroutine to exit
        return Result{}, errors.New("timeout")
    }
}
```

### Solution 3: Context (Recommended)

**✅ Best - Use context for cancellation:**
```go
func search(ctx context.Context, query string) (Result, error) {
    results := make(chan Result, 1)

    go func() {
        select {
        case results <- performSearch(query):
        case <-ctx.Done():
            return
        }
    }()

    select {
    case result := <-results:
        return result, nil
    case <-ctx.Done():
        return Result{}, ctx.Err()
    }
}
```

---

## Worker Pool Pattern

**Efficient concurrent processing:**

```go
func ProcessItems(items []Item) []Result {
    numWorkers := runtime.NumCPU()
    jobs := make(chan Item, len(items))
    results := make(chan Result, len(items))

    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go worker(jobs, results, &wg)
    }

    // Send jobs
    for _, item := range items {
        jobs <- item
    }
    close(jobs)

    // Wait for all workers to finish
    go func() {
        wg.Wait()
        close(results)
    }()

    // Collect results
    var output []Result
    for result := range results {
        output = append(output, result)
    }

    return output
}

func worker(jobs <-chan Item, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    for item := range jobs {
        results <- processItem(item)
    }
}
```

---

## Channel Direction Specifications

**Use channel directions to enforce usage:**

```go
// Producer only sends
func producer(ch chan<- int) {
    for i := 0; i < 10; i++ {
        ch <- i
    }
    close(ch)
}

// Consumer only receives
func consumer(ch <-chan int) {
    for val := range ch {
        fmt.Println(val)
    }
}

// Orchestrator has bidirectional channel
func run() {
    ch := make(chan int, 5)
    go producer(ch)  // Converted to send-only
    consumer(ch)     // Converted to receive-only
}
```

---

## Pipeline Pattern

**Chain operations with channels:**

```go
// Stage 1: Generate numbers
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

// Stage 2: Square numbers
func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

// Stage 3: Filter even numbers
func filterEven(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if n%2 == 0 {
                out <- n
            }
        }
    }()
    return out
}

// Compose pipeline
func main() {
    // Set up pipeline
    nums := generate(1, 2, 3, 4, 5)
    squared := square(nums)
    evens := filterEven(squared)

    // Consume output
    for n := range evens {
        fmt.Println(n)
    }
}
```

---

## Fan-Out, Fan-In Pattern

**Distribute work, collect results:**

```go
func fanOut(input <-chan Task, numWorkers int) []<-chan Result {
    workers := make([]<-chan Result, numWorkers)

    for i := 0; i < numWorkers; i++ {
        workers[i] = startWorker(input)
    }

    return workers
}

func startWorker(input <-chan Task) <-chan Result {
    output := make(chan Result)
    go func() {
        defer close(output)
        for task := range input {
            output <- processTask(task)
        }
    }()
    return output
}

func fanIn(channels ...<-chan Result) <-chan Result {
    out := make(chan Result)
    var wg sync.WaitGroup

    // Start goroutine for each input channel
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan Result) {
            defer wg.Done()
            for result := range c {
                out <- result
            }
        }(ch)
    }

    // Close output when all inputs are done
    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}

// Usage
func Process(tasks []Task) []Result {
    input := make(chan Task, len(tasks))

    // Send tasks
    go func() {
        for _, task := range tasks {
            input <- task
        }
        close(input)
    }()

    // Fan-out to workers
    workers := fanOut(input, runtime.NumCPU())

    // Fan-in results
    results := fanIn(workers...)

    // Collect
    var output []Result
    for result := range results {
        output = append(output, result)
    }

    return output
}
```

---

## Rate Limiting with Ticker

**Control operation rate:**

```go
func processWithRateLimit(items []Item, ratePerSecond int) {
    ticker := time.NewTicker(time.Second / time.Duration(ratePerSecond))
    defer ticker.Stop()

    for _, item := range items {
        <-ticker.C // Wait for next tick
        go processItem(item)
    }
}
```

---

## Context-Aware Operations

**Proper context usage:**

```go
func FetchData(ctx context.Context, url string) ([]byte, error) {
    // Create child context with timeout
    ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
    defer cancel()

    // Create request with context
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

// Check context in loops
func ProcessLargeDataset(ctx context.Context, data []Item) error {
    for i, item := range data {
        // Check for cancellation
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }

        if err := processItem(item); err != nil {
            return fmt.Errorf("failed at item %d: %w", i, err)
        }
    }
    return nil
}
```

---

## Managing Background Tasks

**Structured concurrency with cleanup:**

```go
type Server struct {
    ctx    context.Context
    cancel context.CancelFunc
    wg     sync.WaitGroup
}

func NewServer() *Server {
    ctx, cancel := context.WithCancel(context.Background())
    return &Server{
        ctx:    ctx,
        cancel: cancel,
    }
}

func (s *Server) Start() {
    // Start background workers
    s.startWorker("worker-1", s.backgroundJob1)
    s.startWorker("worker-2", s.backgroundJob2)
    s.startWorker("worker-3", s.backgroundJob3)
}

func (s *Server) startWorker(name string, job func(context.Context)) {
    s.wg.Add(1)
    go func() {
        defer s.wg.Done()
        log.Printf("%s started", name)
        job(s.ctx)
        log.Printf("%s stopped", name)
    }()
}

func (s *Server) backgroundJob1(ctx context.Context) {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            // Do work
        }
    }
}

func (s *Server) Shutdown() {
    log.Println("Shutting down...")
    s.cancel()           // Signal all workers to stop
    s.wg.Wait()          // Wait for all to finish
    log.Println("Shutdown complete")
}
```

---

## Expose Synchronous APIs

**Let callers control concurrency:**

```go
// ❌ Bad - Forces async on caller
type Service struct{}

func (s *Service) FetchData(url string) <-chan Result {
    results := make(chan Result, 1)
    go func() {
        // Fetch data asynchronously
        results <- fetchFromURL(url)
    }()
    return results
}

// ✅ Good - Synchronous, caller can add concurrency
func (s *Service) FetchData(ctx context.Context, url string) (Result, error) {
    return fetchFromURL(ctx, url)
}

// Caller can easily make it async if needed
func fetchMultiple(urls []string) []Result {
    results := make(chan Result, len(urls))

    for _, url := range urls {
        go func(u string) {
            result, _ := service.FetchData(context.Background(), u)
            results <- result
        }(url)
    }

    // Collect results...
}
```

---

## Best Practices Summary

1. **Prevent leaks** - Use buffered channels, quit channels, or context
2. **Use context** - For cancellation and timeouts
3. **Channel directions** - Specify send-only (`chan<-`) or receive-only (`<-chan`)
4. **WaitGroups** - Track goroutine completion
5. **Defer cleanup** - `defer close(ch)`, `defer wg.Done()`
6. **Structured concurrency** - Start and stop goroutines in coordinated way
7. **Synchronous APIs** - Let callers add concurrency
8. **Check context** - In long-running loops and operations
9. **Size channels appropriately** - Buffer when you know capacity
10. **Don't leak goroutines** - Every goroutine must have exit path

---

## References

- [Go Concurrency Patterns (Talk)](https://go.dev/talks/2012/concurrency.slide)
- [Go Blog: Concurrency Patterns](https://go.dev/blog/pipelines)
- [Effective Go: Concurrency](https://go.dev/doc/effective_go#concurrency)
