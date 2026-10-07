Since you already know **C++ and Go**, I’d focus less on “build another CRUD app” and more on projects that force you to deal with **concurrency, networking, memory, OS concepts, performance, and clean architecture**.

## C++ projects

### 1. Build a Redis-like in-memory database

**Difficulty:** ⭐⭐⭐⭐

Build a small server that supports commands such as:

```
SET name Alice
GET name
DEL name
EXPIRE name 30
```

Then add:

- TCP networking
    
- Multiple clients
    
- Thread pool / event loop
    
- Hash tables
    
- TTL expiration
    
- Persistence to disk
    
- A simple RESP-like protocol
    

**What you'll learn:** sockets, concurrency, data structures, serialization, memory management, performance.

**Stretch goal:** implement an event-driven server using `epoll`/`kqueue` rather than one thread per connection.

---

### 2. Write your own `std::vector` / `std::string`

Don't just copy the interface. Implement a small STL-like container library:

```
Vector<int>
String
HashMap<K, V>
UniquePtr<T>
SharedPtr<T>
```

Pay particular attention to:

- RAII
    
- Move semantics
    
- Copy/move constructors
    
- Allocators
    
- Exception safety
    
- Iterators
    
- Template metaprogramming/concepts
    

**What you'll learn:** this is probably one of the best ways to turn “I know C++ syntax” into actual C++ understanding.

---

### 3. Multithreaded web server

Build an HTTP/1.1 server from scratch.

Start with:

```
Browser
   ↓
TCP socket
   ↓
HTTP parser
   ↓
Router
   ↓
Handler
```

Then add:

- Thread pool
    
- Keep-alive
    
- Static file serving
    
- HTTP status codes/headers
    
- Logging
    
- Configuration
    
- Graceful shutdown
    
- Benchmarks
    

**Stretch goal:** compare:

```
1 thread / connection
thread pool
event loop
```

and benchmark them under load.

---

### 4. Build a small compiler/interpreter

Create a language such as:

```
let x = 10;
let y = x * 2;

if (y > 10) {
    print(y);
}
```

Implement:

1. Lexer
    
2. Parser
    
3. AST
    
4. Semantic analysis
    
5. Interpreter
    

Then optionally compile it to bytecode.

**What you'll learn:** parsing, trees, memory management, language design and compiler architecture.

---

### 5. File synchronization tool

Build something like a miniature `rsync`:

```
mysync ./folder user@server:/backup
```

Features:

- Directory traversal
    
- File hashing
    
- Incremental synchronization
    
- Compression
    
- Parallel transfers
    
- Resume interrupted transfers
    
- Progress reporting
    

This combines **filesystem + networking + concurrency + hashing** nicely.

---

## Go projects

Go is particularly good for projects involving **networking and concurrent services**.

### 1. Distributed job queue

Build something like a miniature Celery/RabbitMQ.

Architecture:

```
             ┌── Worker
             │
Client → Queue ── Worker
             │
             └── Worker
```

Implement:

- Job submission
    
- Workers
    
- Job IDs
    
- Retries
    
- Timeouts
    
- Worker heartbeats
    
- Persistent queue
    
- Dead-letter queue
    
- Graceful shutdown
    

Use goroutines and channels extensively.

**Stretch goal:** distribute the queue across multiple nodes.

---

### 2. Reverse proxy / load balancer

Build:

```
                ┌── Server A
Client → Proxy ─┼── Server B
                └── Server C
```

Implement:

- HTTP forwarding
    
- Round-robin
    
- Least-connections
    
- Health checks
    
- Connection pooling
    
- Timeouts
    
- Retries
    
- Rate limiting
    
- Metrics
    

This is an excellent Go project because you'll get deep practice with `net/http`, goroutines, contexts and networking.

---

### 3. Concurrent web crawler

Give it:

```
crawler https://example.com --workers 20
```

It should:

- Crawl links
    
- Avoid duplicates
    
- Respect domain restrictions
    
- Handle robots.txt
    
- Limit concurrency
    
- Store results
    
- Handle retries/timeouts
    
- Produce statistics
    

For example:

```
Pages crawled: 14,382
Unique URLs:   18,923
Errors:          127
Average latency: 143ms
```

**What you'll learn:** goroutines, channels, synchronization, HTTP, cancellation and backpressure.

---

### 4. Distributed key-value store

Build a simplified:

```
SET foo bar
GET foo
DELETE foo
```

server.

Then introduce distributed-systems concepts:

```
Node A ←→ Node B ←→ Node C
```

Implement:

- Replication
    
- Leader election
    
- Heartbeats
    
- Log replication
    
- Persistence
    
- Failure detection
    

**Stretch goal:** implement a simplified version of **Raft**.

This is a significantly harder project, but it will teach you a _lot_.

---

### 5. Docker-like container runner

This one is particularly interesting if you're comfortable with Linux.

Build:

```
mycontainer run /bin/bash
```

Under the hood, explore:

- Linux namespaces
    
- cgroups
    
- process isolation
    
- filesystem isolation
    
- virtual networking
    
- resource limits
    

Go is a great language for this because much of the work is interacting with Linux APIs and processes.

---

## Projects I'd especially recommend

If your goal is **deepening your engineering skills rather than just having projects on GitHub**, I'd pick these six:

|Language|Project|Main skills|
|---|---|---|
|C++|Redis-like database|Networking + concurrency + memory|
|C++|STL/container implementation|Modern C++ + memory|
|C++|Compiler/interpreter|CS fundamentals + architecture|
|Go|Distributed job queue|Concurrency + distributed systems|
|Go|Reverse proxy|Networking + production engineering|
|Go|Raft-based KV store|Distributed systems|

### An interesting progression

You could even make the projects build on one another:

**C++**

```
Custom containers
      ↓
Thread pool
      ↓
TCP server
      ↓
Redis-like database
```

**Go**

```
HTTP server
      ↓
Concurrent crawler
      ↓
Reverse proxy
      ↓
Distributed job queue
      ↓
Raft-based distributed KV store
```

That progression would take you from “I know the language” toward **systems/programming proficiency**.

If you tell me roughly **how comfortable you are with Linux, networking, OS concepts, and data structures**, I can also give you a **3–6 month project roadmap**, ordered from easiest to hardest, with specific features to implement at each stage.