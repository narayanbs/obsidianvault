In Go, `context.Context` is essentially a way to **carry cancellation, deadlines, and request-scoped values across a call chain**.

A useful mental model is:

> **“This operation belongs to this request, and here's how long it may live, whether it has been cancelled, and a small amount of request metadata.”**

## 1. The basic idea

Imagine an HTTP request:

```text
HTTP handler
    ↓
service
    ↓
repository
    ↓
database
```

The request might be cancelled because the client disconnected. You don't want the database query to keep running anyway.

You pass a `context.Context` down the chain:

```go
func handler(ctx context.Context) {
    service.DoSomething(ctx)
}

func (s *Service) DoSomething(ctx context.Context) {
    repo.Fetch(ctx)
}

func (r *Repository) Fetch(ctx context.Context) {
    db.QueryContext(ctx, "SELECT ...")
}
```

If the context is cancelled, the cancellation can propagate through all those layers.

---

# 2. What is `context.Context`?

The interface is roughly:

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key any) any
}
```

The four methods correspond to four concepts:

| Method       | Meaning                                           |
| ------------ | ------------------------------------------------- |
| `Deadline()` | When should this operation stop?                  |
| `Done()`     | Channel that closes when the context is cancelled |
| `Err()`      | Why was it cancelled?                             |
| `Value()`    | Request-scoped metadata                           |

You usually don't implement `Context` yourself. You create contexts using functions from the `context` package.

---

# 3. `context.Background()`

The root of a context tree is usually:

```go
ctx := context.Background()
```

It never gets cancelled and has no deadline or values.

For example:

```go
func main() {
    ctx := context.Background()

    doSomething(ctx)
}
```

Think of it as:

```text
Background
    │
    └── your application operation
```

You'll often see it at application boundaries such as `main`, tests, or initialization.

---

# 4. `context.WithCancel`

This is probably the most important context constructor.

```go
ctx, cancel := context.WithCancel(parent)
defer cancel()
```

Now you have:

```text
parent
  │
  └── ctx
```

Calling:

```go
cancel()
```

causes `ctx.Done()` to close.

Example:

```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    go worker(ctx)

    time.Sleep(time.Second)
    cancel()
}
```

The worker can detect cancellation:

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("worker stopped")
            return

        default:
            // do work
        }
    }
}
```

The important pattern is:

```go
select {
case <-ctx.Done():
    return
default:
    // continue
}
```

---

# 5. `context.WithTimeout`

This is extremely common in real applications.

```go
ctx, cancel := context.WithTimeout(
    parent,
    2*time.Second,
)
defer cancel()
```

It means:

> "This operation has at most two seconds."

After two seconds, the context is automatically cancelled.

For example:

```go
func GetUser(ctx context.Context, id int) (*User, error) {
    ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()

    return database.GetUser(ctx, id)
}
```

This is particularly useful for:

* database queries
* HTTP calls
* RPCs
* external APIs
* expensive operations

---

# 6. `context.WithDeadline`

Instead of saying "two seconds from now", you can specify an exact deadline:

```go
deadline := time.Now().Add(2 * time.Second)

ctx, cancel := context.WithDeadline(ctx, deadline)
defer cancel()
```

In practice, `WithTimeout` is often more convenient.

These are basically equivalent:

```go
context.WithTimeout(ctx, 2*time.Second)
```

and

```go
context.WithDeadline(ctx, time.Now().Add(2*time.Second))
```

---

# 7. Contexts form a tree

This is the most important conceptual part.

Suppose:

```go
root := context.Background()

ctx1, cancel1 := context.WithCancel(root)
ctx2, cancel2 := context.WithTimeout(ctx1, 5*time.Second)
ctx3, cancel3 := context.WithCancel(ctx2)
```

You have:

```text
root
 │
 └── ctx1
      │
      └── ctx2
           │
           └── ctx3
```

Cancellation propagates **downward**.

If `ctx1` is cancelled:

```text
root
 │
 └── ctx1  ❌
      │
      └── ctx2  ❌
           │
           └── ctx3  ❌
```

But cancelling `ctx3` doesn't cancel its parents:

```text
root
 │
 └── ctx1  ✅
      │
      └── ctx2  ✅
           │
           └── ctx3  ❌
```

This parent → child relationship is the core of Go contexts.

---

# 8. Contexts and HTTP requests

This is where contexts become particularly useful.

An HTTP handler gets a context from the request:

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    service.DoSomething(ctx)
}
```

The server manages the lifetime of that context.

If the client disconnects, the request context can be cancelled.

So:

```text
Client
   │
   │ HTTP request
   ▼
Handler
   │
   │ ctx
   ▼
Service
   │
   │ ctx
   ▼
Database
```

If the request disappears:

```text
Client ❌
   │
   ▼
Handler ❌
   │
   ▼
Service ❌
   │
   ▼
Database query ❌
```

**provided the database operation actually observes the context.**

For example:

```go
rows, err := db.QueryContext(
    ctx,
    "SELECT * FROM users",
)
```

rather than:

```go
rows, err := db.Query(
    "SELECT * FROM users",
)
```

The latter has no way to respond to context cancellation.

---

# 9. Always pass context explicitly

A common Go convention is:

```go
func DoSomething(ctx context.Context, arg string) error
```

rather than:

```go
func DoSomething(arg string, ctx context.Context) error
```

Context normally goes **first**.

For example:

```go
func (s *Service) CreateUser(
    ctx context.Context,
    user User,
) error {
    return s.repo.Create(ctx, user)
}
```

Then:

```go
func (r *Repository) Create(
    ctx context.Context,
    user User,
) error {
    _, err := r.db.ExecContext(
        ctx,
        "INSERT INTO users ...",
    )

    return err
}
```

This makes the lifetime of the operation explicit.

---

# 10. `ctx.Done()` and `select`

A very common pattern is:

```go
select {
case <-ctx.Done():
    return ctx.Err()

case result := <-results:
    return result
}
```

This says:

> "Wait for the result, but stop waiting if the context is cancelled."

For example:

```go
func expensiveOperation(ctx context.Context) error {
    result := make(chan error, 1)

    go func() {
        result <- doExpensiveWork()
    }()

    select {
    case err := <-result:
        return err

    case <-ctx.Done():
        return ctx.Err()
    }
}
```

Now the caller can impose a timeout:

```go
ctx, cancel := context.WithTimeout(
    context.Background(),
    time.Second,
)
defer cancel()

err := expensiveOperation(ctx)
```

---

# 11. `ctx.Err()`

After cancellation:

```go
err := ctx.Err()
```

will typically return one of:

```go
context.Canceled
```

or:

```go
context.DeadlineExceeded
```

For example:

```go
select {
case <-ctx.Done():
    return ctx.Err()
}
```

You can distinguish:

```go
if errors.Is(err, context.DeadlineExceeded) {
    // timeout
}

if errors.Is(err, context.Canceled) {
    // explicitly cancelled
}
```

---

# 12. Context values

Contexts can also carry request-scoped values:

```go
ctx = context.WithValue(ctx, key, value)
```

Then:

```go
value := ctx.Value(key)
```

For example:

```go
type contextKey string

const requestIDKey contextKey = "requestID"

ctx := context.WithValue(
    context.Background(),
    requestIDKey,
    "abc-123",
)
```

Later:

```go
requestID := ctx.Value(requestIDKey)
```

However, **don't treat context as a general-purpose bag of variables.**

A good rule:

> Use context values for request-scoped metadata that crosses API boundaries, not for ordinary function arguments.

Good candidates can include things like:

```text
request ID
trace ID
authentication metadata
```

Bad candidates:

```text
user
database connection
configuration
business data
function parameters
```

If a function needs a `User`, generally just pass:

```go
func DoSomething(ctx context.Context, user User)
```

rather than hiding the user inside the context.

---

# 13. `context.WithValue` has a subtle gotcha

Avoid string keys like:

```go
context.WithValue(ctx, "userID", id)
```

because different packages could accidentally use the same key.

Instead, define your own key type:

```go
type contextKey string

const userIDKey contextKey = "userID"
```

Then:

```go
ctx = context.WithValue(ctx, userIDKey, id)
```

Even better, many packages use an unexported custom type:

```go
type userIDKey struct{}
```

Then:

```go
ctx = context.WithValue(ctx, userIDKey{}, id)
```

---

# 14. Don't store contexts in structs

This is generally a bad idea:

```go
type Service struct {
    ctx context.Context
}
```

Instead:

```go
type Service struct {
    // dependencies...
}
```

and:

```go
func (s *Service) DoSomething(ctx context.Context) error {
    // ...
}
```

Why?

Because a context represents the lifetime of **an operation**, not the lifetime of an object.

A service might live for the entire application:

```text
Service lifetime:  application
Context lifetime:  one request
```

Those lifetimes don't match.

---

# 15. Don't pass `nil` for context

Avoid:

```go
DoSomething(nil)
```

If you don't have a meaningful context, use:

```go
context.Background()
```

or, depending on the situation, derive from an existing context.

Also, don't use `context.TODO()` as a permanent replacement for understanding which context should be used.

`TODO()` is basically:

> "I haven't figured out what the correct context is yet."

---

# 16. A realistic example

Imagine an HTTP endpoint:

```go
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    id := r.PathValue("id")

    user, err := h.service.GetUser(ctx, id)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

Service:

```go
func (s *Service) GetUser(
    ctx context.Context,
    id string,
) (*User, error) {

    ctx, cancel := context.WithTimeout(
        ctx,
        2*time.Second,
    )
    defer cancel()

    return s.repo.GetUser(ctx, id)
}
```

Repository:

```go
func (r *Repository) GetUser(
    ctx context.Context,
    id string,
) (*User, error) {

    var user User

    err := r.db.QueryRowContext(
        ctx,
        "SELECT id, name FROM users WHERE id = $1",
        id,
    ).Scan(&user.ID, &user.Name)

    if err != nil {
        return nil, err
    }

    return &user, nil
}
```

The resulting lifetime is:

```text
HTTP request
     │
     │ r.Context()
     ▼
   Handler
     │
     ▼
   Service
     │
     │ + 2 second timeout
     ▼
 Repository
     │
     ▼
 Database
```

If the HTTP request disappears, cancellation propagates.

If the request remains alive but the database operation takes >2 seconds, the service's timeout cancels the operation.

That's the real power of `context`.

---

# 17. A few rules worth memorizing

If you're learning Go, I'd keep these rules in your head:

**1. Context is for lifetime/cancellation.**

```go
func Foo(ctx context.Context, ...)
```

**2. Pass it down, don't hide it.**

```text
handler → service → repository → database
           │
           └── ctx
```

**3. Derive contexts when you need a shorter lifetime.**

```go
ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
defer cancel()
```

**4. Always call `cancel()` when you create a cancellable context.**

```go
ctx, cancel := context.WithCancel(parent)
defer cancel()
```

**5. Actually use context-aware APIs.**

```go
db.QueryContext(ctx, ...)
http.NewRequestWithContext(ctx, ...)
```

**6. Use `ctx.Done()` for cooperative cancellation.**

```go
select {
case <-ctx.Done():
    return ctx.Err()
case result := <-results:
    return result
}
```

**7. Don't use context as a generic parameter bag.**

Use function arguments for normal data.

---

## The one-sentence mental model

If you remember only one thing:

> **A Go context represents the lifetime of an operation and lets that lifetime—cancellation, deadlines, and limited request metadata—flow through the functions involved in that operation.**

Once that clicks, `context.Context` becomes much less mysterious: **it's basically a cancellation/deadline signal that travels down your call tree.**


# Cancelling goroutines with context



This is one of the most important patterns for writing **cancelable goroutines** in Go.

The key idea is:

> Pass a `context.Context` into the goroutine, and use `ctx.Done()` inside `select` to know when the goroutine should stop.

### Basic pattern

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("worker stopping:", ctx.Err())
            return

        case value := <-jobs:
            fmt.Println("processing", value)
        }
    }
}
```

Then the caller controls cancellation:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

go worker(ctx)

// ... do some work

cancel()
```

When `cancel()` is called, `ctx.Done()` becomes ready, so this case:

```go
case <-ctx.Done():
```

gets selected and the goroutine returns.

---

## Why use `select`?

Imagine a worker waiting for jobs:

```go
for {
    job := <-jobs
    process(job)
}
```

The problem is that the goroutine can get stuck forever waiting for `jobs`.

With context:

```go
for {
    select {
    case <-ctx.Done():
        return

    case job := <-jobs:
        process(job)
    }
}
```

Now the goroutine is waiting for **either**:

```text
             ┌── job arrives ──> process it
select ──────┤
             └── context cancelled ──> exit
```

That's the core pattern.

---

## A complete example

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, jobs <-chan int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("worker cancelled")
            return

        case job, ok := <-jobs:
            if !ok {
                fmt.Println("jobs channel closed")
                return
            }

            fmt.Println("processing job:", job)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())

    jobs := make(chan int)

    go worker(ctx, jobs)

    jobs <- 1
    jobs <- 2

    time.Sleep(time.Second)

    cancel()

    time.Sleep(time.Second)
}
```

Here:

```go
ctx, cancel := context.WithCancel(context.Background())
```

creates a context that can be cancelled.

And:

```go
cancel()
```

signals cancellation to **everyone using that context**.

The worker observes it through:

```go
case <-ctx.Done():
```

---

## Context with a timeout

You don't always need to manually call `cancel()`.

You can give a goroutine a deadline:

```go
ctx, cancel := context.WithTimeout(
    context.Background(),
    2*time.Second,
)
defer cancel()

go worker(ctx, jobs)
```

After 2 seconds:

```go
ctx.Done()
```

becomes ready automatically.

You can inspect why it stopped:

```go
ctx.Err()
```

which will return something like:

```go
context deadline exceeded
```

For manual cancellation, it returns:

```go
context canceled
```

---

## Very important: context doesn't kill the goroutine

This is a common misunderstanding.

Calling:

```go
cancel()
```

does **not** forcibly terminate the goroutine.

It merely sends a cancellation signal.

Your goroutine has to cooperate:

```go
select {
case <-ctx.Done():
    return
}
```

If you do this:

```go
func worker(ctx context.Context) {
    for {
        doSomethingForever()
    }
}
```

then cancellation won't magically stop it.

---

## Context + ticker

Another extremely common pattern is a goroutine that periodically does something but should stop when cancelled:

```go
func worker(ctx context.Context) {
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            fmt.Println("stopping worker")
            return

        case <-ticker.C:
            fmt.Println("doing periodic work")
        }
    }
}
```

Conceptually:

```text
             ticker fires
                  │
                  ▼
              do work
                  │
                  ▼
                select
                  ▲
                  │
          context cancelled
                  │
                  ▼
                return
```

This pattern is extremely common in servers, background workers, polling loops, etc.

---

## One subtle but important thing

Suppose you're sending **to** a channel:

```go
jobs <- job
```

That can block forever if nobody is receiving.

So you can make the send cancelable too:

```go
select {
case jobs <- job:
    // job sent

case <-ctx.Done():
    // cancellation
    return
}
```

Similarly, when receiving:

```go
select {
case job := <-jobs:
    // received job

case <-ctx.Done():
    return
}
```

So a very useful mental template is:

```go
select {
case something := <-someChannel:
    // normal operation

case someChannel <- something:
    // normal operation

case <-ctx.Done():
    // cancellation
    return
}
```

**`ctx.Done()` is just another channel that becomes readable when cancellation happens.**

That connection—**context cancellation + `select` + channels**—is the foundation of graceful goroutine shutdown in Go.