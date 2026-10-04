# Go 101 — A Java Engineer's Complete Guide

> Written for engineers who know Java + Spring Boot and want to learn Go from scratch.
> Every concept is mapped to its Java equivalent.
> Covers: Go 1.21+ (generics, slog, structured concurrency improvements).

---

## Table of Contents

1. [Go Philosophy vs Java Philosophy](#1-go-philosophy-vs-java-philosophy)
2. [Tooling & Project Structure](#2-tooling--project-structure)
3. [Types & Data Structures](#3-types--data-structures)
4. [Functions & Methods](#4-functions--methods)
5. [Interfaces & Composition](#5-interfaces--composition)
6. [Error Handling — No Exceptions](#6-error-handling--no-exceptions)
7. [Pointers](#7-pointers)
8. [Packages & Modules](#8-packages--modules)
9. [Concurrency — Goroutines & Channels](#9-concurrency--goroutines--channels)
10. [Context — Cancellation & Deadlines](#10-context--cancellation--deadlines)
11. [The HTTP Layer](#11-the-http-layer)
12. [Database Access](#12-database-access)
13. [Dependency Injection](#13-dependency-injection)
14. [Configuration](#14-configuration)
15. [Testing](#15-testing)
16. [Observability — OTel, Prometheus, slog](#16-observability)
17. [Java → Go Translation Cheatsheet](#17-java--go-translation-cheatsheet)
18. [Go Versions Roadmap (1.21 → 1.24+)](#18-go-versions-roadmap)
19. [Glossary](#19-glossary)

---

## 1. Go Philosophy vs Java Philosophy

Before touching code, understand the **mindset difference**. Go is deliberately opinionated in ways that will feel wrong to a Java engineer at first, and right after six months.

### The Core Trade-offs

| Aspect | Java / Spring Boot | Go |
|--------|-------------------|-----|
| **Abstraction** | Deep, many layers | Minimal — explicit is better than magic |
| **Type system** | Rich: generics, inheritance, variance | Simple: structs, interfaces, generics (1.18+) |
| **Error handling** | Exceptions (throw/catch) | Explicit return values `(T, error)` |
| **Concurrency** | Threads, thread pools, `synchronized` | Goroutines, channels, `sync` package |
| **DI framework** | Spring IoC (required) | Usually no framework — constructor functions |
| **Runtime** | JVM: large, warm-up needed, JIT | Small binary, instant startup, compiled |
| **Memory** | GC, ~50-500MB heap typical | GC, ~5-20MB typical, lower overhead |
| **Startup time** | 3-30 seconds (Spring context loading) | < 10ms |
| **Verbosity** | High (boilerplate, getters/setters) | Low-medium (but explicit error handling adds lines) |
| **Annotation magic** | Everywhere (`@Service`, `@Cacheable`, `@Transactional`) | None — behaviour must be explicit in code |
| **Philosophy** | Convention over configuration | Explicit over implicit |

### What Go Does Not Have (And Why)

| Java Feature | Go Equivalent | Why Go chose this |
|-------------|--------------|-------------------|
| Classes with inheritance | Structs + embedding | Inheritance creates tight coupling; composition is cleaner |
| Exceptions | `error` return value | Exceptions hide control flow; explicit errors are readable |
| Method overloading | Separate function names | Overloading makes code harder to grep and understand |
| Generics (pre-1.18) | `interface{}` / `any` | Simplicity; added in Go 1.18 when clearly needed |
| Annotations | Function calls | Reflection-based magic is hard to reason about |
| `null` pointer exceptions | `nil` (same danger, different name) | Go is honest that nil exists rather than pretending otherwise |
| Checked exceptions | N/A | Every error is unchecked — you choose whether to handle it |
| `finally` block | `defer` | More powerful: stacks multiple deferred calls |
| Package manager (Maven) | `go mod` (built-in) | One tool, no ecosystem fragmentation |

### The "Less Is More" Principle

Go has 25 keywords (Java has 51). The language spec fits in one webpage. This is deliberate.

Rob Pike (Go creator): *"Complexity is multiplicative: fixing a problem by making one part of the system more complex slowly but surely adds complexity to other parts."*

You will miss Spring Boot's magic for the first few weeks. Then you will appreciate being able to read any Go codebase and immediately understand what it does — there is no hidden framework behaviour.

---

## 2. Tooling & Project Structure

### Installation & First Program

```bash
# Install Go (macOS)
brew install go

# Verify
go version  # go version go1.21.0 darwin/arm64

# Create a new project
mkdir sme-payout-service && cd sme-payout-service
go mod init github.com/niyo/sme-payout-service
# Creates: go.mod (like pom.xml but simpler)
```

```go
// main.go — the entry point
package main

import "fmt"

func main() {
    fmt.Println("SME Payout Service starting...")
}
```

```bash
go run main.go          # compile and run (like mvn spring-boot:run)
go build -o server .    # compile to binary (like mvn package)
./server                # run the binary

go test ./...           # run all tests (like mvn test)
go test ./... -v        # verbose output
go test ./... -race     # detect data races (crucial for concurrent code)
go vet ./...            # static analysis (like SonarQube lite)
gofmt -w .              # format all code (no debates — one style)
golangci-lint run       # comprehensive linter
```

### The go.mod File (Your pom.xml)

```
module github.com/niyo/sme-payout-service   ← module path (like Maven groupId:artifactId)

go 1.21                                      ← minimum Go version

require (
    github.com/gin-gonic/gin v1.9.1          ← dependencies (like Maven <dependency>)
    go.mongodb.org/mongo-driver v1.13.0
    github.com/spf13/viper v1.18.0
    go.opentelemetry.io/otel v1.21.0
)
```

```bash
go get github.com/gin-gonic/gin@v1.9.1   # add dependency (like mvn dependency:add)
go mod tidy                               # clean up unused deps (like mvn dependency:purge)
go mod download                           # download all deps (like mvn dependency:resolve)
```

### Project Structure (Microservice)

There is no enforced structure in Go (unlike Spring Boot's `src/main/java`). The Go community converged on this layout for services:

```
sme-payout-service/
├── cmd/
│   └── server/
│       └── main.go              ← entrypoint (like SmeApplication.java)
├── internal/                    ← private packages (cannot be imported by other modules)
│   ├── config/
│   │   └── config.go            ← config loading (like application.yml + @ConfigurationProperties)
│   ├── domain/
│   │   ├── payout.go            ← domain structs (like @Document entities)
│   │   └── errors.go            ← domain-specific error types
│   ├── handler/
│   │   └── payout_handler.go    ← HTTP handlers (like @RestController)
│   ├── service/
│   │   ├── payout_service.go    ← interface (like CorporateService.java)
│   │   └── payout_service_impl.go ← implementation (like CorporateServiceImpl.java)
│   ├── repository/
│   │   └── payout_repository.go ← DB access (like MongoRepository)
│   ├── middleware/
│   │   └── auth.go              ← JWT middleware (like SmeAuthenticationFilter)
│   └── kafka/
│       ├── producer.go          ← Kafka producer (like KafkaTemplate)
│       └── consumer.go          ← Kafka consumer (like @KafkaListener)
├── pkg/                         ← public packages (reusable across modules)
│   └── httputil/
│       └── response.go
├── go.mod
├── go.sum                       ← locked dependency hashes (like pom.xml resolved versions)
└── Dockerfile
```

**`internal/` is enforced by the compiler** — packages inside `internal/` cannot be imported by code outside the parent directory. This is Go's way of enforcing module boundaries without explicit visibility modifiers.

---

## 3. Types & Data Structures

### Structs — Go's "Classes"

Go has no classes. It has **structs** — data containers. Behaviour is added separately via methods.

```java
// Java class
public class PayoutsTransaction {
    private String id;
    private String corporateId;
    private double amount;
    private TransferType transferType;
    private TransactionStatus status;
    private ZonedDateTime createdAt;

    // constructor, getters, setters (or Lombok)
}
```

```go
// Go struct — just data, no methods attached yet
type PayoutsTransaction struct {
    ID           string
    CorporateID  string
    Amount       float64
    TransferType string
    Status       TransactionStatus
    CreatedAt    time.Time
}

// Creating instances — no "new" keyword, no constructor
txn := PayoutsTransaction{
    ID:           "TX-12345",
    CorporateID:  "CORP-789",
    Amount:       50000.00,
    TransferType: "NEFT",
    Status:       StatusPending,
    CreatedAt:    time.Now(),
}

// Partial initialization — unset fields get zero values
txn := PayoutsTransaction{
    CorporateID: "CORP-789",
    Amount:      50000.00,
}
// txn.ID == ""          (zero value for string)
// txn.Status == ""      (zero value for TransactionStatus which is string)
// txn.CreatedAt == time.Time{} (zero value)
```

### Zero Values — No `null` NPEs (Almost)

Every type in Go has a **zero value** — the default when no value is assigned:

| Type | Zero Value | Java Equivalent |
|------|-----------|----------------|
| `int`, `int64`, `float64` | `0` | `0` |
| `string` | `""` (empty string) | `null` |
| `bool` | `false` | `false` |
| `pointer` (`*T`) | `nil` | `null` |
| slice (`[]T`) | `nil` | `null` (but usable — append works on nil slice) |
| map (`map[K]V`) | `nil` | `null` (but must initialize before writing) |
| struct | all fields zero-valued | object with null fields |
| interface | `nil` | `null` |

```go
var s string    // s == ""    (not null — safe to use)
var i int       // i == 0
var b bool      // b == false
var p *string   // p == nil  (pointer — must check before dereferencing)

// Strings: zero value is safe to use directly
if s == "" {
    s = "default"
}

// Pointers: must check nil before use
if p != nil {
    fmt.Println(*p)
}
```

### Constructors — Use Factory Functions

Go has no constructors. The convention is a `NewXxx` function:

```go
// This is Go's constructor pattern
func NewPayoutsTransaction(corporateID string, amount float64, transferType string) *PayoutsTransaction {
    return &PayoutsTransaction{
        ID:           generateID(),    // uuid.New().String()
        CorporateID:  corporateID,
        Amount:       amount,
        TransferType: transferType,
        Status:       StatusPending,
        CreatedAt:    time.Now(),
    }
}

// Usage
txn := NewPayoutsTransaction("CORP-789", 50000.00, "NEFT")
```

### Exported vs Unexported — Visibility

Go has no `public`/`private`/`protected`. Visibility is controlled by **capitalisation**:

```go
// Exported (public) — starts with uppercase, accessible from other packages
type PayoutsTransaction struct {
    ID          string    // exported field
    CorporateID string    // exported field
    internalRef string    // unexported — only accessible within this package
}

func NewPayoutsTransaction(...) *PayoutsTransaction { }  // exported function
func validateAmount(amount float64) bool { }             // unexported function

// Rule: if it starts with uppercase → any package can use it
//       if it starts with lowercase → only this package can use it
```

This replaces all of Java's `public`, `private`, `protected`, `package-private`. Two rules instead of four.

### Slices — Go's List

```java
// Java
List<String> employees = new ArrayList<>();
employees.add("emp-1");
employees.add("emp-2");
String first = employees.get(0);
int size = employees.size();
```

```go
// Go — slice (dynamic array)
employees := []string{}          // empty slice (or: var employees []string)
employees = append(employees, "emp-1")
employees = append(employees, "emp-2")
first := employees[0]            // direct index access
size := len(employees)

// Initialize with values
employees := []string{"emp-1", "emp-2", "emp-3"}

// Slice of structs
transactions := []PayoutsTransaction{
    {ID: "TX-1", Amount: 1000},
    {ID: "TX-2", Amount: 2000},
}

// Append multiple at once
more := []string{"emp-4", "emp-5"}
employees = append(employees, more...)  // ... spreads the slice
```

### Maps — Go's HashMap

```java
// Java
Map<String, CorporateDTO> cache = new HashMap<>();
cache.put("CORP-1", dto);
CorporateDTO dto = cache.get("CORP-1");  // returns null if missing
```

```go
// Go
cache := make(map[string]*CorporateDTO)    // must initialize before writing
cache["CORP-1"] = &CorporateDTO{Code: "CORP-1"}

// Safe read — always use the two-value form
dto, ok := cache["CORP-1"]
if !ok {
    // key doesn't exist — no null, just the zero value and ok=false
    return fmt.Errorf("corporate %s not found", code)
}

// Iterate
for key, value := range cache {
    fmt.Printf("code=%s name=%s\n", key, value.Name)
}

// Delete
delete(cache, "CORP-1")

// Check existence without getting value
if _, exists := cache["CORP-1"]; exists { ... }
```

### Enums — Constants with iota

```java
// Java enum
public enum TransactionStatus {
    PENDING, SUCCESS, FAILED, REVERSED
}
```

```go
// Go — typed string constant (most readable for status fields)
type TransactionStatus string

const (
    StatusPending  TransactionStatus = "PENDING"
    StatusSuccess  TransactionStatus = "SUCCESS"
    StatusFailed   TransactionStatus = "FAILED"
    StatusReversed TransactionStatus = "REVERSED"
)

// Or use iota for integer enums (auto-incrementing)
type Priority int

const (
    PriorityLow Priority = iota   // 0
    PriorityMedium                 // 1
    PriorityHigh                   // 2
)
```

### Generics (Go 1.18+)

```go
// Generic function — like Java <T>
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Usage
amounts := Map(transactions, func(t PayoutsTransaction) float64 {
    return t.Amount
})

// Generic struct — like Java List<T>
type Result[T any] struct {
    Data  T
    Error error
}
```

---

## 4. Functions & Methods

### Functions

```go
// Basic function
func add(a, b int) int {
    return a + b
}

// Multiple return values — this is how Go does error handling
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}

// Named return values (use sparingly — can reduce clarity)
func minMax(arr []int) (min, max int) {
    min, max = arr[0], arr[0]
    for _, v := range arr {
        if v < min { min = v }
        if v > max { max = v }
    }
    return  // naked return — returns the named values
}

// Variadic functions (like Java varargs)
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
sum(1, 2, 3)           // direct
sum(nums...)           // spread a slice
```

### Methods — Attaching Behaviour to Structs

```java
// Java: method belongs to the class
public class PayoutsService {
    public boolean isEligible(PayoutsTransaction txn) {
        return txn.getAmount() > 0 && txn.getStatus() == Status.PENDING;
    }
}
```

```go
// Go: method has a "receiver" — the struct it operates on
// This is how you add methods to structs (like class methods in Java)

type PayoutsService struct {
    repo   PayoutsRepository
    kafka  KafkaProducer
}

// Method with POINTER receiver — can modify the struct, efficient for large structs
func (s *PayoutsService) Process(txn *PayoutsTransaction) error {
    if txn.Amount <= 0 {
        return fmt.Errorf("invalid amount: %f", txn.Amount)
    }
    txn.Status = StatusProcessing   // modifies the transaction
    return s.repo.Save(txn)
}

// Method with VALUE receiver — gets a copy, cannot modify original
func (t PayoutsTransaction) IsEligible() bool {
    return t.Amount > 0 && t.Status == StatusPending
}

// Usage
svc := &PayoutsService{repo: repo, kafka: kafka}
err := svc.Process(txn)

if txn.IsEligible() { ... }
```

**Pointer vs value receiver rule of thumb:**
- Use **pointer receiver** (`*T`) when the method modifies the struct, or the struct is large
- Use **value receiver** (`T`) for small structs where you want an immutable read
- **Be consistent**: if ANY method uses a pointer receiver, use pointer receivers for ALL methods on that type

### defer — The `finally` Replacement

`defer` schedules a function call to run when the enclosing function returns — no matter how it returns (normal, error, panic).

```java
// Java finally
public void processFile(String path) throws IOException {
    InputStream stream = new FileInputStream(path);
    try {
        // process
    } finally {
        stream.close();   // always runs
    }
}
```

```go
// Go defer
func processFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("open file: %w", err)
    }
    defer f.Close()    // scheduled to run when processFile returns

    // process the file
    // f.Close() runs after this, regardless of how we return
    return nil
}
```

Multiple defers execute in **LIFO order** (last registered, first called):

```go
func connectAndProcess() error {
    db, err := connectDB()
    if err != nil { return err }
    defer db.Close()      // registered 1st, runs LAST

    cache, err := connectCache()
    if err != nil { return err }
    defer cache.Close()   // registered 2nd, runs FIRST

    return process(db, cache)
    // On return: cache.Close() runs, then db.Close()
}
```

This is also how you implement the MDC/cleanup pattern from your Java `MDCFilter`:

```go
func (m *MDCMiddleware) Handle(c *gin.Context) {
    correlationID := c.GetHeader("x-correlation-id")
    if correlationID == "" {
        correlationID = "app-sme-" + uuid.New().String()
    }

    // Store in context (Go's version of MDC)
    ctx := context.WithValue(c.Request.Context(), CorrelationIDKey, correlationID)
    c.Request = c.Request.WithContext(ctx)

    // defer ensures cleanup even if handler panics
    defer func() {
        // cleanup if needed
    }()

    c.Next()
}
```

---

## 5. Interfaces & Composition

### Interfaces — Implicit, Not Declared

In Java, a class must explicitly declare `implements SomeInterface`. In Go, a type **automatically satisfies** an interface if it has all the required methods. No `implements` keyword.

```java
// Java — must explicitly declare implements
public interface PayoutsService {
    PayoutsResult process(PayoutsTransaction txn) throws ServiceException;
}

public class PayoutsServiceImpl implements PayoutsService {  // ← explicit
    @Override
    public PayoutsResult process(PayoutsTransaction txn) throws ServiceException { ... }
}
```

```go
// Go — implicit satisfaction
type PayoutsService interface {
    Process(ctx context.Context, txn *PayoutsTransaction) (*PayoutsResult, error)
}

// PayoutsServiceImpl automatically satisfies PayoutsService
// because it has the Process method with the right signature
type PayoutsServiceImpl struct {
    repo   PayoutsRepository
    kafka  KafkaProducer
}

func (s *PayoutsServiceImpl) Process(ctx context.Context, txn *PayoutsTransaction) (*PayoutsResult, error) {
    // implementation
}

// This works — no "implements" declaration needed
var svc PayoutsService = &PayoutsServiceImpl{}
```

**Why this matters for testing:** You don't need to modify a struct to make it satisfy an interface. You can define an interface in your test file for exactly the methods you need, and any compatible type will satisfy it.

```go
// In your test file — define the minimal interface you need
type MockCardHost interface {
    Debit(ctx context.Context, req DebitRequest) (*DebitResponse, error)
}
```

### The `error` Interface

`error` is just a built-in interface:

```go
type error interface {
    Error() string
}
```

Any type with an `Error() string` method IS an `error`. This is how you create custom error types:

```go
// Custom error type (like your ValidationException.java)
type ValidationError struct {
    Field   string
    Message string
    Code    string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on %s: %s (code: %s)", e.Field, e.Message, e.Code)
}

// Usage — can be used anywhere an error is expected
func validateAmount(amount float64) error {
    if amount <= 0 {
        return &ValidationError{
            Field:   "amount",
            Message: "must be positive",
            Code:    "INVALID_AMOUNT",
        }
    }
    return nil
}

// Type assertion to get the specific error type back
err := validateAmount(-100)
var valErr *ValidationError
if errors.As(err, &valErr) {
    fmt.Println(valErr.Code)   // INVALID_AMOUNT
}
```

### Composition — Struct Embedding (Not Inheritance)

Go has no inheritance. Instead, it has **embedding** — you embed one struct inside another and get all its methods promoted.

```go
// Like Java's AbstractAuditingEntity — but via embedding, not inheritance
type AuditFields struct {
    CreatedAt   time.Time
    CreatedBy   string
    ModifiedAt  time.Time
    ModifiedBy  string
}

func (a *AuditFields) SetCreated(by string) {
    a.CreatedAt = time.Now()
    a.CreatedBy = by
}

// Embed AuditFields into PayoutsTransaction
type PayoutsTransaction struct {
    AuditFields          // embedded — all fields and methods are "promoted"
    ID           string
    CorporateID  string
    Amount       float64
}

// Usage — access embedded fields directly
txn := &PayoutsTransaction{ID: "TX-1"}
txn.SetCreated("system")         // promoted method from AuditFields
fmt.Println(txn.CreatedAt)       // promoted field from AuditFields
fmt.Println(txn.AuditFields.CreatedAt)  // also valid — explicit access
```

### The Empty Interface — `any`

`any` (alias for `interface{}`) accepts any type. Like `Object` in Java.

```go
// any accepts any type
func printAnything(v any) {
    fmt.Println(v)
}

printAnything("hello")
printAnything(42)
printAnything(PayoutsTransaction{})

// Type assertion to get the concrete type back
func process(v any) {
    switch t := v.(type) {
    case string:
        fmt.Println("string:", t)
    case int:
        fmt.Println("int:", t)
    case *PayoutsTransaction:
        fmt.Println("transaction:", t.ID)
    default:
        fmt.Printf("unknown type: %T\n", v)
    }
}
```

**Avoid `any` where possible** — use specific types or generics. `any` loses type safety.

---

## 6. Error Handling — No Exceptions

This is the biggest mindset shift from Java. Go has **no exceptions, no try/catch**. Errors are values, returned alongside results.

### The Pattern

```java
// Java — throw and catch
public PayoutsResult processDebit(DebitRequest request) throws ServiceException {
    try {
        return cardHostClient.debit(request);
    } catch (TimeoutException e) {
        throw new ServiceException("Card host timeout", e);
    } catch (InvalidAccountException e) {
        throw new ValidationException("Invalid account: " + e.getMessage());
    }
}

// Caller
try {
    PayoutsResult result = service.processDebit(request);
} catch (ServiceException e) {
    log.error("Failed", e);
}
```

```go
// Go — explicit error returns
func (s *PayoutsService) ProcessDebit(ctx context.Context, req DebitRequest) (*PayoutsResult, error) {
    result, err := s.cardHostClient.Debit(ctx, req)
    if err != nil {
        // Wrap the error with context — like Java's chained exceptions
        return nil, fmt.Errorf("card host debit failed: %w", err)
    }
    return result, nil
}

// Caller — must handle the error
result, err := service.ProcessDebit(ctx, req)
if err != nil {
    log.Error("debit failed", "error", err)
    return err
}
// result is guaranteed non-nil here
```

**The `%w` verb in `fmt.Errorf`** wraps an error, preserving the original for unwrapping:

```go
// Wrapping errors through the call stack
func (r *PayoutsRepository) FindByID(ctx context.Context, id string) (*PayoutsTransaction, error) {
    result, err := r.collection.FindOne(ctx, bson.M{"_id": id}).Decode(&txn)
    if err == mongo.ErrNoDocuments {
        return nil, fmt.Errorf("transaction %s not found: %w", id, ErrNotFound)
    }
    if err != nil {
        return nil, fmt.Errorf("database query failed: %w", err)
    }
    return &txn, nil
}

// Caller checks for specific error types
txn, err := repo.FindByID(ctx, "TX-123")
if errors.Is(err, ErrNotFound) {
    // return 404
}
if err != nil {
    // return 500
}
```

### Sentinel Errors

```go
// Define sentinel errors in your domain package — like Java's custom exceptions
var (
    ErrNotFound     = errors.New("not found")
    ErrInvalidInput = errors.New("invalid input")
    ErrUnauthorized = errors.New("unauthorized")
    ErrConflict     = errors.New("already exists")
)

// Check with errors.Is (works through wrapped errors)
if errors.Is(err, ErrNotFound) { ... }

// Check custom type with errors.As (like Java instanceof)
var valErr *ValidationError
if errors.As(err, &valErr) {
    // valErr has the concrete type now
    fmt.Println(valErr.Code)
}
```

### panic and recover — The Last Resort

`panic` is NOT the equivalent of throwing exceptions. It signals a programmer error (like Java's `Error`, not `Exception`). Use it only for unrecoverable situations (nil dereference, invalid state that means the program is broken).

```go
// panic — for truly unrecoverable programmer errors
func mustParseURL(raw string) *url.URL {
    u, err := url.Parse(raw)
    if err != nil {
        panic(fmt.Sprintf("invalid hardcoded URL %q: %v", raw, err))
        // If this panics, the URL is wrong in the code — fix the code, not the error handling
    }
    return u
}

// recover — catch a panic (like catch(Throwable) in Java)
// Use only in top-level handlers (HTTP middleware, Kafka consumer wrappers)
func safeHandler(fn func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic recovered: %v", r)
        }
    }()
    fn()
    return nil
}
```

**Rule**: Return `error` for expected failure conditions. `panic` only for bugs.

---

## 7. Pointers

Java passes objects by reference automatically. In Go, you explicitly choose.

### Value vs Pointer

```go
type Corporate struct {
    Code    string
    Balance float64
}

// VALUE — a copy is made. Modifications don't affect the original.
func doubleBalance(c Corporate) Corporate {
    c.Balance *= 2          // modifies the copy
    return c                // return the modified copy
}

// POINTER — no copy. Modifications affect the original.
func doubleBalanceInPlace(c *Corporate) {
    c.Balance *= 2          // modifies the original
}

corp := Corporate{Code: "CORP-1", Balance: 1000.0}

// Value call — corp.Balance is unchanged
modified := doubleBalance(corp)
// corp.Balance == 1000.0, modified.Balance == 2000.0

// Pointer call — corp.Balance changes
doubleBalanceInPlace(&corp)   // & takes the address of corp
// corp.Balance == 2000.0
```

### When to Use Pointers

```go
// Use *T (pointer) when:
// 1. The function needs to modify the struct
func (s *PayoutsService) Process(txn *PayoutsTransaction) error { ... }

// 2. The struct is large (avoid copying overhead)
type LargeBatchFile struct {
    Transactions []PayoutsTransaction  // 10,000+ items
    Metadata     map[string]string
}
func processFile(f *LargeBatchFile) error { ... }  // pass by pointer

// 3. The value could be nil (optional parameter)
func findCorporate(code string) (*Corporate, error) {
    // return nil when not found (vs Java's Optional<T>)
    if notFound {
        return nil, ErrNotFound
    }
    return &corporate, nil
}

// Use T (value) when:
// 1. Small structs (just a few scalars)
// 2. The function doesn't need to modify it
// 3. You want the caller to know the function won't modify it
func formatAmount(amount float64) string {
    return fmt.Sprintf("₹%.2f", amount)
}
```

### Go Has No `Optional<T>`

Java uses `Optional<T>` to signal that a value might be absent. Go uses pointer `*T` (may be nil) or the two-return-value pattern:

```go
// Pattern 1: pointer return (nil = not found)
func (r *Repo) FindByCode(ctx context.Context, code string) (*Corporate, error)

// Pattern 2: boolean (the "comma ok" idiom)
value, ok := myMap["key"]
if !ok { ... }   // key not present

// Pattern 3: sentinel error
corporate, err := repo.FindByCode(ctx, code)
if errors.Is(err, ErrNotFound) { ... }
```

---

## 8. Packages & Modules

### Packages — Like Java Packages, But Simpler

```go
// Every .go file starts with its package declaration
package service  // this file belongs to the "service" package

// All files in the same directory must have the same package name
// (except test files which use "service_test" for black-box testing)
```

```go
// Importing packages
import (
    "context"                            // standard library
    "fmt"
    "time"

    "github.com/gin-gonic/gin"           // third-party
    "go.mongodb.org/mongo-driver/mongo"

    "github.com/niyo/sme-payout/internal/domain"   // your own package
    "github.com/niyo/sme-payout/pkg/httputil"
)

// Use imported package
ctx := context.Background()
router := gin.New()
txn := domain.NewPayoutsTransaction(...)
```

**Blank import (`_`)** — import for side effects only (registers a driver, runs `init()`):

```go
import _ "go.uber.org/automaxprocs"   // sets GOMAXPROCS to container CPU limit at startup
```

### init() Functions

```go
// init() runs automatically when the package is loaded — like @PostConstruct but at package level
// A package can have multiple init() functions, in multiple files

package config

var globalConfig *Config

func init() {
    // Load config before any other code in this package runs
    cfg, err := loadFromEnv()
    if err != nil {
        log.Fatal("failed to load config:", err)
    }
    globalConfig = cfg
}
```

**Avoid `init()` for most things** — it makes code harder to test and reason about. Prefer explicit initialization in `main()` or factory functions. Use `init()` only for true package-level setup (registering drivers, etc.).

---

## 9. Concurrency — Goroutines & Channels

This is Go's signature feature. The mental model is completely different from Java threads.

### Goroutines — Lightweight Threads

```java
// Java — create a thread (expensive: ~1MB stack each, OS-managed)
Thread thread = new Thread(() -> processPayment(txn));
thread.start();

// Or thread pool
ExecutorService pool = Executors.newFixedThreadPool(10);
pool.submit(() -> processPayment(txn));
```

```go
// Go — start a goroutine (cheap: ~2KB initial stack, multiplexed by Go runtime)
go processPayment(txn)   // that's it — the "go" keyword

// Anonymous goroutine
go func() {
    processPayment(txn)
}()

// Go creates as many goroutines as you need — millions is fine
for _, txn := range batch {
    go processPayment(txn)   // starts 10,000 goroutines? No problem.
}
```

**Goroutines vs Java virtual threads**: Virtual threads (Java 21) are similar in intent — lightweight, many can exist. Goroutines are Go's original design (since 2009) and have a slightly smaller footprint with the same principles.

### Channels — Communication Between Goroutines

In Java, goroutines communicate via shared memory (with `synchronized`, locks). In Go, the preferred model is **channels** — goroutines pass data to each other instead of sharing it.

*"Don't communicate by sharing memory; share memory by communicating."* — Go Proverb

```go
// Channel creation — a typed pipe between goroutines
results := make(chan PayoutsResult)    // unbuffered: sender blocks until receiver reads
results := make(chan PayoutsResult, 100)  // buffered: sender blocks only when buffer is full

// Send to channel (blocks if channel is full / unbuffered)
results <- PayoutsResult{Status: "success"}

// Receive from channel (blocks until something is available)
result := <-results

// Close a channel — signals no more values will be sent
close(results)

// Range over channel — receives until closed
for result := range results {
    fmt.Println(result.Status)
}
```

### Fan-out Pattern — Process Batch in Parallel

```go
// Process 1000 salary debits concurrently, collect results
func processBatch(ctx context.Context, transactions []PayoutsTransaction) []error {
    results := make(chan error, len(transactions))   // buffered — won't block

    // Fan out: start one goroutine per transaction
    for _, txn := range transactions {
        txn := txn   // capture loop variable (Go 1.22+ fixes this)
        go func() {
            err := processOne(ctx, txn)
            results <- err
        }()
    }

    // Collect all results
    errs := make([]error, 0)
    for range transactions {
        if err := <-results; err != nil {
            errs = append(errs, err)
        }
    }
    return errs
}
```

### sync.WaitGroup — Wait for Multiple Goroutines

```go
// Like Java's CountDownLatch
func processBatch(transactions []PayoutsTransaction) {
    var wg sync.WaitGroup

    for _, txn := range transactions {
        wg.Add(1)    // increment counter
        txn := txn
        go func() {
            defer wg.Done()   // decrement when goroutine finishes
            processOne(txn)
        }()
    }

    wg.Wait()   // blocks until counter reaches 0
}
```

### sync.Mutex — Protecting Shared State

```go
// When goroutines MUST share memory (avoid when possible)
type MetricsCollector struct {
    mu     sync.Mutex   // protects the map below
    counts map[string]int64
}

func (m *MetricsCollector) Increment(key string) {
    m.mu.Lock()
    defer m.mu.Unlock()   // always unlock, even on panic
    m.counts[key]++
}

func (m *MetricsCollector) Get(key string) int64 {
    m.mu.RLock()   // read lock — multiple readers OK, no writers
    defer m.mu.RUnlock()
    return m.counts[key]
}
```

### select — Like switch for Channels

```go
// select waits on multiple channels, handling whichever is ready first
func processWithTimeout(ctx context.Context, txn PayoutsTransaction) error {
    result := make(chan error, 1)

    go func() {
        result <- callCardHost(txn)
    }()

    select {
    case err := <-result:          // card host responded
        return err
    case <-ctx.Done():             // context cancelled or timed out
        return fmt.Errorf("card host call timed out: %w", ctx.Err())
    case <-time.After(5 * time.Second):  // timeout via timer
        return fmt.Errorf("5 second timeout exceeded")
    }
}
```

---

## 10. Context — Cancellation & Deadlines

`context.Context` is in every Go function signature that does I/O. It carries:
- Cancellation signals (user cancelled, upstream timed out)
- Deadlines (this request must complete by T)
- Request-scoped values (correlation ID, user ID — like Java's MDC)

```java
// Java — no built-in context passing. You use MDC (thread-local) or pass everything manually
public PayoutsResult process(DebitRequest request) {
    String corrId = MDC.get("correlationId");  // thread-local — works for threads, not goroutines
}
```

```go
// Go — context is ALWAYS the first parameter by convention
func (s *PayoutsService) Process(ctx context.Context, txn *PayoutsTransaction) (*PayoutsResult, error) {
    // Pass context to every downstream call
    corporate, err := s.repo.FindCorporate(ctx, txn.CorporateID)
    if err != nil { return nil, err }

    result, err := s.cardHost.Debit(ctx, txn)
    if err != nil { return nil, err }

    return result, nil
}
```

### Creating Contexts

```go
// In main / top-level handler — the root context
ctx := context.Background()   // never cancelled, no deadline

// In tests
ctx := context.TODO()   // placeholder — same as Background(), signals "I'll add proper context later"

// With timeout (most common in HTTP handlers and Kafka workers)
ctx, cancel := context.WithTimeout(parentCtx, 5*time.Second)
defer cancel()   // always call cancel to release resources

// With deadline (absolute time)
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(parentCtx, deadline)
defer cancel()

// With cancellation (manual cancel)
ctx, cancel := context.WithCancel(parentCtx)
// ... later:
cancel()   // signal all goroutines using this context to stop
```

### Storing Values in Context (Go's MDC)

```go
// Define typed keys to avoid collisions
type contextKey string

const (
    CorrelationIDKey contextKey = "correlationId"
    UserIDKey        contextKey = "userId"
)

// In middleware — set values
func MDCMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        correlationID := r.Header.Get("x-correlation-id")
        if correlationID == "" {
            correlationID = "app-sme-" + uuid.New().String()
        }

        ctx := context.WithValue(r.Context(), CorrelationIDKey, correlationID)
        ctx = context.WithValue(ctx, UserIDKey, extractUserFromJWT(r))

        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// In any function downstream — retrieve values
func (s *PayoutsService) Process(ctx context.Context, txn *PayoutsTransaction) error {
    correlationID, _ := ctx.Value(CorrelationIDKey).(string)
    // include in logs
    slog.InfoContext(ctx, "processing debit",
        "correlationId", correlationID,
        "transactionId", txn.ID)
    return nil
}
```

**Important**: Unlike Java's MDC (thread-local), context values flow through goroutines via explicit passing. This is why `ctx` is the first parameter of every function that needs request context — there's no thread-local magic.

---

## 11. The HTTP Layer

### Standard Library: net/http

```go
// Go's standard library HTTP server — no framework needed for simple cases
package main

import (
    "encoding/json"
    "net/http"
)

func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("GET /v1/payouts/{id}", getPayoutHandler)
    mux.HandleFunc("POST /v1/payouts", createPayoutHandler)

    server := &http.Server{
        Addr:         ":8080",
        Handler:      mux,
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    log.Fatal(server.ListenAndServe())
}

func getPayoutHandler(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")   // Go 1.22+: extract path variable

    txn, err := service.FindByID(r.Context(), id)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(txn)
}
```

### Gin — The Most Popular Web Framework

[Gin](https://github.com/gin-gonic/gin) is the most used Go web framework. Think of it as Spring MVC without annotations — explicit routing, middleware chaining.

```bash
go get github.com/gin-gonic/gin
```

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.New()

    // Global middleware (like Spring Security filter chain)
    r.Use(gin.Recovery())          // recover from panics → 500 (like Spring's exception handling)
    r.Use(MDCMiddleware())         // set correlationId in context
    r.Use(JWTAuthMiddleware())     // validate JWT (like SmeAuthenticationFilter)
    r.Use(OTelMiddleware())        // OpenTelemetry tracing

    // Route groups (like @RequestMapping on a controller class)
    v1 := r.Group("/v1")
    {
        payouts := v1.Group("/payouts")
        payouts.Use(RequireRole("PAYOUT_INITIATE"))   // like @Secured
        {
            payouts.GET("", listPayoutsHandler)
            payouts.POST("", initiatePayoutHandler)
            payouts.GET("/:id", getPayoutHandler)
        }

        // Public routes (like AUTH_WHITELIST)
        v1.POST("/auth/login", loginHandler)
    }

    r.Run(":8080")   // start server
}
```

```go
// Handler — equivalent to a @RestController method
func initiatePayoutHandler(c *gin.Context) {
    // Bind and validate request body (like @RequestBody + @Valid)
    var req InitiatePayoutRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "error": err.Error(),
        })
        return
    }

    // Get path variable (like @PathVariable)
    corporateID := c.Param("id")

    // Get query param (like @RequestParam)
    page := c.DefaultQuery("page", "0")

    // Get from context (values set by middleware — like MDC)
    correlationID := c.GetString("correlationId")
    userID := c.GetString("userId")

    // Call service
    result, err := service.InitiatePayout(c.Request.Context(), req)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    // Return JSON response (like ResponseEntity.ok(result))
    c.JSON(http.StatusAccepted, result)
}
```

### Middleware — The Filter Chain

```go
// JWT auth middleware — like SmeAuthenticationFilter
func JWTAuthMiddleware(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "missing authorization header",
            })
            return   // c.Abort() stops the chain — like not calling chain.doFilter()
        }

        claims, err := parseJWT(token, jwtSecret)
        if err != nil {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "invalid token",
            })
            return
        }

        // Store in context for downstream handlers (like SecurityContextHolder)
        c.Set("userId", claims.UserID)
        c.Set("roles", claims.Roles)

        c.Next()   // continue to next middleware/handler (like chain.doFilter())
    }
}

// Role-based access — like @Secured
func RequireRole(role string) gin.HandlerFunc {
    return func(c *gin.Context) {
        roles, _ := c.Get("roles")
        userRoles, _ := roles.([]string)

        if !contains(userRoles, role) {
            c.AbortWithStatusJSON(http.StatusForbidden, gin.H{
                "error": "insufficient permissions",
            })
            return
        }
        c.Next()
    }
}
```

### HTTP Client — The Go Way

```go
// Standard library HTTP client (configure once, reuse)
var httpClient = &http.Client{
    Timeout: 30 * time.Second,
    Transport: &http.Transport{
        MaxIdleConns:        100,
        MaxIdleConnsPerHost: 20,
        IdleConnTimeout:     90 * time.Second,
    },
}

// Typed API client — like your Retrofit services
type CardHostClient struct {
    baseURL    string
    httpClient *http.Client
}

func NewCardHostClient(baseURL string) *CardHostClient {
    return &CardHostClient{baseURL: baseURL, httpClient: httpClient}
}

func (c *CardHostClient) Debit(ctx context.Context, req DebitRequest) (*DebitResponse, error) {
    body, err := json.Marshal(req)
    if err != nil {
        return nil, fmt.Errorf("marshal request: %w", err)
    }

    httpReq, err := http.NewRequestWithContext(
        ctx,                            // carries timeout/cancellation
        http.MethodPost,
        c.baseURL+"/v1/debit",
        bytes.NewReader(body),
    )
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }
    httpReq.Header.Set("Content-Type", "application/json")

    resp, err := c.httpClient.Do(httpReq)
    if err != nil {
        return nil, fmt.Errorf("card host request: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("card host returned status %d", resp.StatusCode)
    }

    var result DebitResponse
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, fmt.Errorf("decode response: %w", err)
    }
    return &result, nil
}

// Or use Resty — like RestClient, fluent API
import "github.com/go-resty/resty/v2"

client := resty.New().SetBaseURL("https://card-host.internal").SetTimeout(30 * time.Second)

resp, err := client.R().
    SetContext(ctx).
    SetBody(req).
    SetResult(&DebitResponse{}).
    Post("/v1/debit")
```

---

## 12. Database Access

### MongoDB with the Official Driver

```bash
go get go.mongodb.org/mongo-driver/v2/mongo
```

```go
// Connection — like your DatabaseConfiguration.java
func NewMongoClient(ctx context.Context, uri string) (*mongo.Client, error) {
    opts := options.Client().
        ApplyURI(uri).
        SetReadPreference(readpref.Primary()).
        SetWriteConcern(writeconcern.Majority()).
        SetServerAPIOptions(options.ServerAPI(options.ServerAPIVersion1))

    client, err := mongo.Connect(opts)
    if err != nil {
        return nil, fmt.Errorf("connect mongo: %w", err)
    }

    // Verify connection
    if err := client.Ping(ctx, nil); err != nil {
        return nil, fmt.Errorf("ping mongo: %w", err)
    }
    return client, nil
}
```

```go
// Repository — like your InwardRemittanceRepository.java
// But in Go: no magic query derivation — you write the queries

type PayoutsRepository struct {
    collection *mongo.Collection
}

func NewPayoutsRepository(db *mongo.Database) *PayoutsRepository {
    return &PayoutsRepository{
        collection: db.Collection("payouts_transactions"),
    }
}

// FindByID — like repository.findById(id)
func (r *PayoutsRepository) FindByID(ctx context.Context, id string) (*PayoutsTransaction, error) {
    var txn PayoutsTransaction
    err := r.collection.FindOne(ctx, bson.M{"_id": id}).Decode(&txn)
    if errors.Is(err, mongo.ErrNoDocuments) {
        return nil, ErrNotFound
    }
    if err != nil {
        return nil, fmt.Errorf("find by id: %w", err)
    }
    return &txn, nil
}

// FindAllByStatus — like repository.findAllByStatusIn(statuses)
func (r *PayoutsRepository) FindByStatuses(ctx context.Context, statuses []TransactionStatus) ([]PayoutsTransaction, error) {
    filter := bson.M{"status": bson.M{"$in": statuses}}
    cursor, err := r.collection.Find(ctx, filter, options.Find().SetSort(bson.D{{Key: "created_at", Value: -1}}))
    if err != nil {
        return nil, fmt.Errorf("find by statuses: %w", err)
    }
    defer cursor.Close(ctx)

    var results []PayoutsTransaction
    if err := cursor.All(ctx, &results); err != nil {
        return nil, fmt.Errorf("decode results: %w", err)
    }
    return results, nil
}

// Save — like repository.save(entity)
func (r *PayoutsRepository) Save(ctx context.Context, txn *PayoutsTransaction) error {
    txn.ModifiedAt = time.Now()

    opts := options.Replace().SetUpsert(true)
    _, err := r.collection.ReplaceOne(ctx, bson.M{"_id": txn.ID}, txn, opts)
    if err != nil {
        return fmt.Errorf("save transaction: %w", err)
    }
    return nil
}

// Struct tags for MongoDB field mapping (like @Field in Spring Data)
type PayoutsTransaction struct {
    ID          string            `bson:"_id"`
    CorporateID string            `bson:"corporate_id"`
    Amount      float64           `bson:"amount"`
    Status      TransactionStatus `bson:"status"`
    CreatedAt   time.Time         `bson:"created_at"`
    ModifiedAt  time.Time         `bson:"modified_at"`
}
```

### PostgreSQL with pgx

For SQL databases (if you ever use them), `pgx` is the most performant PostgreSQL driver:

```go
import "github.com/jackc/pgx/v5/pgxpool"

// Connection pool
pool, err := pgxpool.New(ctx, "postgres://user:pass@localhost:5432/sme_db")

// Query
rows, err := pool.Query(ctx,
    "SELECT id, corporate_id, amount FROM payouts WHERE status = $1",
    "PENDING")
defer rows.Close()

for rows.Next() {
    var txn PayoutsTransaction
    err := rows.Scan(&txn.ID, &txn.CorporateID, &txn.Amount)
}
```

### GORM — If You Want ORM

GORM is the closest thing to Spring Data JPA in Go. Useful for SQL databases:

```go
import "gorm.io/gorm"

type Corporate struct {
    gorm.Model                     // embedded: ID, CreatedAt, UpdatedAt, DeletedAt
    Code    string `gorm:"uniqueIndex"`
    Name    string
    Balance float64
}

// Auto-migrate (creates/updates tables)
db.AutoMigrate(&Corporate{})

// CRUD
db.Create(&corp)
db.First(&corp, "code = ?", "CORP-1")
db.Where("balance > ?", 0).Find(&corps)
db.Save(&corp)
db.Delete(&corp)
```

---

## 13. Dependency Injection

Go has no Spring IoC container. You wire dependencies yourself — explicitly, in `main()` or a setup function. This is more verbose but completely transparent.

### Manual Wiring — The Simple Way

```go
// cmd/server/main.go — wire everything together explicitly
func main() {
    // Config
    cfg, err := config.Load()
    if err != nil { log.Fatal(err) }

    // Infrastructure
    mongoClient, err := database.NewMongoClient(ctx, cfg.MongoURI)
    if err != nil { log.Fatal(err) }
    db := mongoClient.Database(cfg.DBName)

    kafkaProducer, err := kafka.NewProducer(cfg.Kafka)
    if err != nil { log.Fatal(err) }

    // Repositories
    payoutsRepo := repository.NewPayoutsRepository(db)
    corporateRepo := repository.NewCorporateRepository(db)

    // External clients
    cardHostClient := clients.NewCardHostClient(cfg.CardHostURL)
    yblClient := clients.NewYBLClient(cfg.YBLBaseURL, cfg.YBLClientID, cfg.YBLSecret)

    // Services — inject dependencies
    corporateService := service.NewCorporateService(corporateRepo)
    payoutsService := service.NewPayoutsService(payoutsRepo, corporateService, cardHostClient, kafkaProducer)

    // Handlers — inject services
    payoutsHandler := handler.NewPayoutsHandler(payoutsService)
    corporateHandler := handler.NewCorporateHandler(corporateService)

    // HTTP server
    router := setupRouter(payoutsHandler, corporateHandler, cfg)
    router.Run(cfg.ServerAddr)
}
```

This is explicit, readable, and has no magic. If wiring becomes complex (50+ dependencies), use a DI framework.

### Wire (Google) — Compile-Time DI

[Wire](https://github.com/google/wire) generates the wiring code above at compile time. Like Spring IoC but with generated code instead of reflection.

```go
// wire.go — declare providers (constructors) and injectors
//go:build wireinject

func InitializeServer(cfg *config.Config) (*Server, error) {
    wire.Build(
        database.NewMongoClient,
        repository.NewPayoutsRepository,
        repository.NewCorporateRepository,
        clients.NewCardHostClient,
        service.NewCorporateService,
        service.NewPayoutsService,
        handler.NewPayoutsHandler,
        NewServer,
    )
    return nil, nil   // Wire replaces this with generated code
}

// Run: wire gen ./...
// Wire generates: wire_gen.go with the explicit wiring above
```

### fx (Uber) — Runtime DI (Closest to Spring)

[fx](https://github.com/uber-go/fx) is runtime DI, similar in feel to Spring IoC:

```go
import "go.uber.org/fx"

func main() {
    app := fx.New(
        fx.Provide(
            config.Load,
            database.NewMongoClient,
            repository.NewPayoutsRepository,
            service.NewPayoutsService,
            handler.NewPayoutsHandler,
            NewRouter,
        ),
        fx.Invoke(startServer),   // runs when the app starts — like @PostConstruct
    )
    app.Run()
}

// fx handles lifecycle hooks too
func NewKafkaConsumer(lc fx.Lifecycle, cfg *config.Config) *kafka.Consumer {
    consumer := kafka.NewConsumer(cfg)
    lc.Append(fx.Hook{
        OnStart: func(ctx context.Context) error {
            go consumer.Start()   // like @PostConstruct
            return nil
        },
        OnStop: func(ctx context.Context) error {
            return consumer.Close()   // like @PreDestroy
        },
    })
    return consumer
}
```

**Recommendation**: Start with manual wiring. Graduate to `Wire` when wiring becomes complex. Use `fx` only if you want the full lifecycle management.

---

## 14. Configuration

### Viper — The Standard Config Library

[Viper](https://github.com/spf13/viper) is the most popular config library. Like Spring Boot's `application.yml` + `@Value` combined.

```bash
go get github.com/spf13/viper
```

```go
// config/config.go
type Config struct {
    Server struct {
        Port    int    `mapstructure:"port"`
        Timeout int    `mapstructure:"timeout_seconds"`
    } `mapstructure:"server"`

    MongoDB struct {
        URI      string `mapstructure:"uri"`
        Database string `mapstructure:"database"`
    } `mapstructure:"mongodb"`

    Kafka struct {
        BootstrapServers string `mapstructure:"bootstrap_servers"`
        TopicPrefix      string `mapstructure:"topic_prefix"`
        GroupIDPrefix    string `mapstructure:"group_id_prefix"`
    } `mapstructure:"kafka"`

    CardHost struct {
        BaseURL string `mapstructure:"base_url"`
        Timeout int    `mapstructure:"timeout_seconds"`
    } `mapstructure:"card_host"`
}

func Load() (*Config, error) {
    v := viper.New()

    // Like application.yml
    v.SetConfigName("config")      // config.yaml / config.yml / config.json
    v.SetConfigType("yaml")
    v.AddConfigPath("./config")
    v.AddConfigPath(".")

    // Like Spring's environment variable override
    // MONGODB_URI overrides mongodb.uri
    v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    v.AutomaticEnv()   // any env var can override a config key

    if err := v.ReadInConfig(); err != nil {
        return nil, fmt.Errorf("read config: %w", err)
    }

    var cfg Config
    if err := v.Unmarshal(&cfg); err != nil {
        return nil, fmt.Errorf("parse config: %w", err)
    }
    return &cfg, nil
}
```

```yaml
# config/config.yaml (like application.yml)
server:
  port: 8080
  timeout_seconds: 30

mongodb:
  uri: mongodb://localhost:27017
  database: sme_db

kafka:
  bootstrap_servers: localhost:9092
  topic_prefix: com.sme.dev.
  group_id_prefix: sme-dev-

card_host:
  base_url: https://card-host.internal
  timeout_seconds: 5
```

```bash
# Override with environment variables (like Spring's env var override)
MONGODB_URI=mongodb://prod:27017 ./server

# Or a .env file with godotenv:
go get github.com/joho/godotenv
```

---

## 15. Testing

Go has a built-in `testing` package — no JUnit, no TestNG needed.

### Unit Tests

```go
// payout_service_test.go — same package (or service_test for black-box)
package service

import (
    "context"
    "testing"
    "github.com/stretchr/testify/assert"   // like AssertJ
    "github.com/stretchr/testify/mock"     // like Mockito
)

// Mock — like Mockito @Mock (but manual or generated by mockery)
type MockPayoutsRepository struct {
    mock.Mock   // embed testify mock
}

func (m *MockPayoutsRepository) FindByID(ctx context.Context, id string) (*PayoutsTransaction, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*PayoutsTransaction), args.Error(1)
}

func TestPayoutsService_Process_Success(t *testing.T) {
    // Arrange
    mockRepo := new(MockPayoutsRepository)
    mockKafka := new(MockKafkaProducer)
    svc := NewPayoutsService(mockRepo, mockKafka)

    txn := &PayoutsTransaction{ID: "TX-1", Amount: 1000, Status: StatusPending}
    mockRepo.On("FindByID", mock.Anything, "TX-1").Return(txn, nil)
    mockRepo.On("Save", mock.Anything, mock.AnythingOfType("*PayoutsTransaction")).Return(nil)
    mockKafka.On("Publish", mock.Anything, mock.Anything, mock.Anything).Return(nil)

    // Act
    result, err := svc.Process(context.Background(), txn)

    // Assert — like AssertJ
    assert.NoError(t, err)
    assert.NotNil(t, result)
    assert.Equal(t, StatusSuccess, result.Status)
    mockRepo.AssertExpectations(t)
}

func TestPayoutsService_Process_InvalidAmount(t *testing.T) {
    svc := NewPayoutsService(nil, nil)
    txn := &PayoutsTransaction{Amount: -100}

    _, err := svc.Process(context.Background(), txn)

    assert.Error(t, err)
    assert.ErrorIs(t, err, ErrInvalidInput)
}
```

### Table-Driven Tests (Go Idiom — No @ParameterizedTest Needed)

```go
func TestValidateAmount(t *testing.T) {
    tests := []struct {
        name    string
        amount  float64
        wantErr bool
    }{
        {"valid small amount", 100.0, false},
        {"valid large amount", 10_000_000.0, false},
        {"zero amount", 0.0, true},
        {"negative amount", -500.0, true},
        {"above max", 10_000_001.0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := validateAmount(tt.amount)
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

### HTTP Handler Tests

```go
func TestInitiatePayoutHandler(t *testing.T) {
    // Setup
    mockService := new(MockPayoutsService)
    handler := NewPayoutsHandler(mockService)

    router := gin.New()
    router.POST("/v1/payouts", handler.Initiate)

    // Mock service
    mockService.On("Initiate", mock.Anything, mock.AnythingOfType("InitiateRequest")).
        Return(&PayoutsResult{Status: "ACCEPTED"}, nil)

    // Create request
    body := `{"corporateId":"CORP-1","amount":50000}`
    req := httptest.NewRequest(http.MethodPost, "/v1/payouts", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    rec := httptest.NewRecorder()

    // Execute
    router.ServeHTTP(rec, req)

    // Assert
    assert.Equal(t, http.StatusAccepted, rec.Code)
    assert.Contains(t, rec.Body.String(), "ACCEPTED")
}
```

### Generating Mocks with mockery

```bash
go install github.com/vektra/mockery/v2@latest

# Generate mock for an interface
mockery --name=PayoutsRepository --dir=internal/repository --output=internal/mocks
# Creates: internal/mocks/PayoutsRepository.go
```

### Running Tests

```bash
go test ./...                          # all tests
go test ./internal/service/...         # specific package
go test -run TestPayoutsService ./...  # tests matching pattern
go test -v ./...                       # verbose (shows test names)
go test -race ./...                    # data race detector (always run this in CI)
go test -cover ./...                   # coverage percentage
go test -coverprofile=coverage.out ./... && go tool cover -html=coverage.out
```

---

## 16. Observability

### Structured Logging with slog (Go 1.21 built-in)

Go 1.21 added `log/slog` — structured logging in the standard library. Like `logstash-logback-encoder` but built-in.

```go
import "log/slog"

// Setup — once at startup
logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))
slog.SetDefault(logger)

// Usage — structured key-value logging
slog.Info("processing debit",
    "transactionId", txn.ID,
    "corporateId", txn.CorporateID,
    "amount", txn.Amount,
    "transferType", txn.TransferType)

// With context (automatically includes traceId/spanId if OTel is configured)
slog.InfoContext(ctx, "debit completed",
    "status", result.Status,
    "duration_ms", time.Since(start).Milliseconds())

slog.ErrorContext(ctx, "debit failed",
    "error", err,
    "responseCode", resp.Code)

// Output (JSON):
// {"time":"2026-05-06T10:23:45Z","level":"INFO","msg":"debit completed",
//  "status":"SUCCESS","duration_ms":185,"traceId":"abc123","spanId":"def456"}
```

### Prometheus Metrics

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

// Define metrics at package level — like your DatadogMetricsConstants
var (
    // Counter — like Micrometer Counter
    debitTotal = promauto.NewCounterVec(prometheus.CounterOpts{
        Name: "bharat_sme_salary_transfer_debit_total",
        Help: "Number of salary debit transactions",
    }, []string{"status", "transfer_type"})

    // Histogram — like Micrometer Timer
    debitDuration = promauto.NewHistogramVec(prometheus.HistogramOpts{
        Name:    "bharat_sme_cardhost_debit_duration_seconds",
        Help:    "Card host debit call duration",
        Buckets: prometheus.DefBuckets,
    }, []string{"status"})

    // Gauge — like Micrometer Gauge
    pendingPayouts = promauto.NewGauge(prometheus.GaugeOpts{
        Name: "bharat_sme_payouts_pending",
        Help: "Number of payouts currently pending",
    })
)

// Usage in service
func (s *PayoutsService) ProcessDebit(ctx context.Context, txn *PayoutsTransaction) error {
    start := time.Now()

    err := s.cardHost.Debit(ctx, txn)

    status := "success"
    if err != nil { status = "failed" }

    debitTotal.WithLabelValues(status, txn.TransferType).Inc()
    debitDuration.WithLabelValues(status).Observe(time.Since(start).Seconds())
    return err
}

// Expose /metrics endpoint
router.GET("/metrics", gin.WrapH(promhttp.Handler()))
```

### OpenTelemetry — Traces

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/trace"
    "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp"
)

// Setup at startup
func setupOTel(ctx context.Context, serviceName string) (func(), error) {
    exporter, err := otlptracehttp.New(ctx,
        otlptracehttp.WithEndpoint("http://otel-collector:4318"),
        otlptracehttp.WithInsecure(),
    )
    if err != nil { return nil, err }

    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceName(serviceName),
        )),
    )
    otel.SetTracerProvider(tp)

    return func() { tp.Shutdown(ctx) }, nil
}

// Usage — create spans
var tracer = otel.Tracer("github.com/niyo/sme-payout")

func (s *PayoutsService) Process(ctx context.Context, txn *PayoutsTransaction) error {
    ctx, span := tracer.Start(ctx, "payouts.process",
        trace.WithAttributes(
            attribute.String("transaction.id", txn.ID),
            attribute.String("corporate.id", txn.CorporateID),
            attribute.Float64("amount", txn.Amount),
        ))
    defer span.End()

    if err := s.cardHost.Debit(ctx, txn); err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err
    }

    span.SetStatus(codes.Ok, "")
    return nil
}
```

---

## 17. Java → Go Translation Cheatsheet

| Java | Go |
|------|-----|
| `class Foo {}` | `type Foo struct {}` |
| `interface Bar {}` | `type Bar interface {}` |
| `implements Bar` | implicit (if methods match) |
| `extends Base` | struct embedding |
| `new Foo(args)` | `NewFoo(args)` (constructor function) |
| `public` / `private` | uppercase / lowercase first letter |
| `void` | no return type, or `error` |
| `throws Exception` | `error` as last return value |
| `try/catch/finally` | `if err != nil` / `defer` |
| `null` | `nil` |
| `Optional<T>` | `*T` or `(T, bool)` or `(T, error)` |
| `instanceof` | `errors.As()` or type switch |
| `List<T>` | `[]T` (slice) |
| `Map<K,V>` | `map[K]V` |
| `enum` | `type X string` + `const` block |
| `synchronized` | `sync.Mutex` |
| `Thread` | goroutine (`go func(){}()`) |
| `CountDownLatch` | `sync.WaitGroup` |
| `BlockingQueue` | channel (`chan T`) |
| `@Component` / `@Service` | just a struct — wire manually |
| `@Autowired` | constructor parameter |
| `@Value("${prop}")` | `viper.GetString("prop")` |
| `application.yml` | `config.yaml` + Viper |
| `@RestController` | handler function + router |
| `ResponseEntity<T>` | `c.JSON(status, body)` |
| `@GetMapping("/path")` | `router.GET("/path", handler)` |
| `@PathVariable` | `c.Param("name")` |
| `@RequestParam` | `c.Query("name")` |
| `@RequestBody` | `c.ShouldBindJSON(&obj)` |
| `@Valid` | `validate.Struct(obj)` via go-playground/validator |
| `@Cacheable` | manual: check map, call, store |
| `@Scheduled(cron)` | `robfig/cron` library |
| `@Async` | `go func(){}()` |
| `@KafkaListener` | `consumer.ReadMessage()` in goroutine |
| `@Transactional` | explicit `session.WithTransaction()` |
| `MongoRepository` | write methods explicitly with `mongo.Collection` |
| `@SpringBootTest` | `httptest.NewServer()` + real dependencies |
| `@MockBean` | `testify/mock` |
| `@Autowired` constructor | constructor function with parameters |
| `MDC.put("key", val)` | `context.WithValue(ctx, key, val)` |
| `log.info("msg {}", val)` | `slog.Info("msg", "key", val)` |
| `@SneakyThrows` | No equivalent — handle errors explicitly |
| `Lombok @Data` | implement manually or use `go generate` |
| `Lombok @Builder` | builder pattern — implement manually |

---

## 18. Go Versions Roadmap

### Go 1.21 (August 2023) — Your Baseline

- **`log/slog`** — built-in structured logging (use this, not logrus/zap)
- **Slices package** — `slices.Sort()`, `slices.Contains()`, `slices.Filter()`
- **Maps package** — `maps.Keys()`, `maps.Values()`
- **Min/max builtins** — `min(a, b)`, `max(a, b)` (finally!)
- **Guaranteed iteration order for `init()`**
- **Go toolchain management** — `go.mod` specifies exact Go version

### Go 1.22 (February 2024)

- **Loop variable capture fix** — `for i, v := range slice` — `v` is now unique per iteration. The `txn := txn` workaround in goroutines is **no longer needed**
- **`net/http` routing improvements** — `GET /v1/payouts/{id}` pattern matching in stdlib (replaces need for Gorilla Mux for simple cases)
- **`math/rand/v2`** — better random number generation

### Go 1.23 (August 2024)

- **Range over functions** — `for v := range sliceIterator()` (iterator protocol)
- **Timer improvements** — `time.Timer` and `time.Ticker` reset/stop behaviour fixed
- **`unique` package** — string interning for memory efficiency

### Go 1.24 (February 2025)

- **`crypto/sha256` improvements**
- **`os.Root`** — sandboxed file system access
- **`sync.Map` improvements** — better performance for read-heavy workloads
- **`tool` directive in `go.mod`** — track CLI tool dependencies explicitly

### Go 1.25+ (Expected August 2025)

- **Weak pointers** (`weak` package) — GC-friendly caches without memory leaks
- **`structs` package** — introspection utilities
- **Swiss table map implementation** — significant performance improvement for maps (in progress)

### What Stays Stable

Go's compatibility promise is exceptional: **any code that compiled on Go 1.0 (2012) compiles on Go 1.24**. Upgrading Go versions is almost always a `go.mod` version bump with no code changes — unlike Java's frequent breakage between versions.

---

## 19. Glossary

| Term | Definition | Java Equivalent |
|------|-----------|----------------|
| **any** | Alias for `interface{}` — accepts any type | `Object` |
| **blank identifier `_`** | Discard a value you don't need: `_, err := fn()` | No equivalent (must assign) |
| **bson** | Binary JSON format used by MongoDB. Struct tags control field names | `@Field` in Spring Data |
| **build tag** | `//go:build linux` — conditional compilation | Maven profiles (different mechanism) |
| **cap()** | Capacity of a slice (allocated memory, >= len) | N/A (ArrayList capacity) |
| **chan** | Channel — typed pipe for goroutine communication | `BlockingQueue` |
| **closure** | Function that captures variables from surrounding scope | Lambda with captured variables |
| **context.Context** | Carries cancellation, deadlines, request-scoped values | MDC + `ThreadLocal` |
| **defer** | Schedule a function to run when the enclosing function returns | `finally` block |
| **embedding** | Include a struct's fields and methods in another struct | `extends` (but via composition) |
| **error** | Built-in interface: `Error() string`. The standard return for failures | `Exception` |
| **errors.As** | Check if error matches a specific type (through wrapping chain) | `instanceof` + unwrapping |
| **errors.Is** | Check if error matches a sentinel error (through wrapping chain) | Checking exception type |
| **fmt.Errorf** | Create a new error, optionally wrapping another with `%w` | `new Exception("msg", cause)` |
| **go** keyword | Start a new goroutine: `go fn()` | `new Thread(fn).start()` |
| **go.mod** | Module definition and dependency declarations | `pom.xml` |
| **go.sum** | Locked cryptographic hashes of dependencies | `pom.xml` resolved versions |
| **goroutine** | Lightweight thread managed by Go runtime (~2KB stack) | `Thread` (Java 21: virtual thread) |
| **init()** | Package-level function that runs automatically before `main()` | `@PostConstruct` at package level |
| **interface** | Set of method signatures. Satisfied implicitly | Java `interface` — but requires `implements` |
| **iota** | Auto-incrementing constant in a `const` block | N/A |
| **len()** | Length of a slice, map, string, channel | `.size()`, `.length()` |
| **make()** | Initialize slices, maps, channels: `make(map[K]V)` | `new HashMap<>()` |
| **method set** | All methods defined on a type (or *type). Determines interface satisfaction | N/A (all methods visible) |
| **module** | A collection of packages with a shared `go.mod` | Maven module |
| **nil** | Zero value for pointers, interfaces, maps, slices, channels, functions | `null` |
| **package** | A directory of `.go` files sharing a package name | Java package (but stricter: one package per dir) |
| **panic** | Unrecoverable error — like Java `Error`, not `Exception`. Use sparingly | `Error` (RuntimeException in practice) |
| **pointer** | Variable holding a memory address: `*T` | All non-primitive Java references (implicit) |
| **race condition** | Two goroutines access shared memory without synchronization | Thread safety bug |
| **receiver** | The struct a method operates on: `func (s *Service) Method()` | `this` in Java |
| **recover()** | Catch a panic inside a `defer` function | `catch(Throwable)` |
| **select** | Block on multiple channel operations, handle whichever is ready | No equivalent |
| **sentinel error** | A package-level `var` error used for comparison: `errors.Is(err, ErrNotFound)` | Custom exception class |
| **slice** | Dynamic array: `[]T`. Unlike arrays, slices are references to underlying arrays | `ArrayList<T>` |
| **slog** | Go 1.21 structured logging standard library | `logstash-logback-encoder` |
| **struct** | Data container with named fields. Methods added separately via receivers | `class` (data + methods combined) |
| **struct tags** | Metadata on struct fields: `` `json:"name" bson:"_id"` `` | `@JsonProperty`, `@Field` |
| **sync.Mutex** | Mutual exclusion lock for shared memory | `synchronized`, `ReentrantLock` |
| **sync.WaitGroup** | Wait for N goroutines to complete | `CountDownLatch` |
| **type assertion** | `v.(ConcreteType)` — extract concrete type from interface | Cast: `(ConcreteType) v` |
| **type switch** | `switch v := x.(type)` — branch on concrete type | `instanceof` chain or `switch` on class |
| **unexported** | Identifier starts with lowercase — only visible within the package | `private` or `package-private` |
| **variadic** | `func fn(args ...string)` — variable number of arguments | Java varargs `String... args` |

---

*For Go observability specifics (OTel setup, Prometheus, slog correlation), see [`observability-101.md`](./observability-101.md) — the concepts are the same, only the library names differ.*
