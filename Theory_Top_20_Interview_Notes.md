# Theory Top 20 Interview Notes

Quick revision notes for networking, backend, microservices, APIs, concurrency, Go, caching, and databases.

## Networking

### 1. HTTP vs HTTPS

**HTTP** stands for HyperText Transfer Protocol. It is used by browsers and servers to exchange web data.

**HTTPS** is HTTP over TLS/SSL. It encrypts the communication between client and server.

| Point | HTTP | HTTPS |
| --- | --- | --- |
| Security | Not encrypted | Encrypted using TLS |
| Port | 80 | 443 |
| Data safety | Data can be intercepted | Data is protected |
| Certificate | Not required | Requires SSL/TLS certificate |
| Use case | Rare for production | Standard for production |

**Interview answer:** HTTPS provides confidentiality, integrity, and authentication. It prevents attackers from reading or modifying data in transit.

**Little deeper:** HTTP sends data as plain text, so anyone sitting between client and server can potentially read or modify it. HTTPS uses TLS to encrypt the connection and also verifies the server identity using certificates. This is why login pages, payment pages, APIs, and production websites should always use HTTPS.

### 2. What Happens When You Type A URL In Browser?

Example: `https://example.com/products`

1. Browser checks cache for DNS, page, assets, and certificates.
2. Browser parses the URL into protocol, domain, path, and query parameters.
3. DNS resolves the domain name to an IP address.
4. Browser opens a TCP connection to the server.
5. For HTTPS, TLS handshake happens to create an encrypted connection.
6. Browser sends an HTTP request.
7. Server processes the request and returns an HTTP response.
8. Browser parses HTML, CSS, and JavaScript.
9. Browser requests additional assets like images, CSS, JS, and fonts.
10. Browser renders the page.

**Important protocols involved:** DNS, TCP, TLS, HTTP/HTTPS.

**Interview tip:** Do not answer only "request goes to server and response comes back." Mention DNS resolution, TCP connection, TLS handshake for HTTPS, HTTP request/response, and browser rendering. That gives a complete systems-level answer.

### 3. TCP vs UDP

| Point | TCP | UDP |
| --- | --- | --- |
| Full form | Transmission Control Protocol | User Datagram Protocol |
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable | Best effort |
| Ordering | Maintains order | No ordering guarantee |
| Speed | Slower | Faster |
| Error handling | Retransmission, acknowledgements | Minimal |
| Use cases | HTTP, HTTPS, SSH, databases | Video calls, gaming, DNS, streaming |

**Interview answer:** TCP is used when reliability matters. UDP is used when low latency matters more than perfect delivery.

**Little deeper:** TCP creates a connection using a handshake, tracks packets, retransmits lost packets, and delivers data in order. UDP simply sends packets without setting up a connection, so the application must handle any reliability requirement itself. This is why video calls can tolerate a few dropped packets, but banking APIs cannot.

### 4. DNS Basics

DNS stands for Domain Name System. It converts human-readable domain names like `google.com` into IP addresses like `142.250.x.x`.

**DNS flow:**

1. Browser checks local DNS cache.
2. OS checks its DNS cache.
3. Request goes to recursive DNS resolver.
4. Resolver asks root server.
5. Root server points to TLD server, like `.com`.
6. TLD server points to authoritative name server.
7. Authoritative server returns the IP address.

**Common DNS records:**

| Record | Meaning |
| --- | --- |
| A | Maps domain to IPv4 |
| AAAA | Maps domain to IPv6 |
| CNAME | Alias for another domain |
| MX | Mail server record |
| TXT | Text metadata, often used for verification |
| NS | Name server record |

**Little deeper:** DNS results are cached at many levels: browser, OS, resolver, and sometimes ISP. This improves speed but can delay DNS changes. The TTL value on DNS records controls how long a resolver can cache the answer.

## Backend / Microservices

### 5. Monolith vs Microservices

**Monolith:** Entire application is built and deployed as one unit.

**Microservices:** Application is split into small independent services, each handling a specific business capability.

| Point | Monolith | Microservices |
| --- | --- | --- |
| Deployment | One deployment | Independent deployments |
| Scaling | Scale whole app | Scale individual services |
| Complexity | Simpler initially | More operational complexity |
| Communication | In-process calls | Network calls |
| Failure isolation | Lower | Better |
| Team ownership | Harder at scale | Easier for large teams |

**Little deeper:** A monolith is not bad. For small teams and early products, a monolith is often faster to build and easier to debug. Microservices become useful when the application and team size grow enough that independent ownership, deployment, and scaling are worth the extra complexity.

### 6. Why Microservices?

Microservices are useful when a system grows large and different parts need independent development, scaling, and deployment.

**Benefits:**

- Independent deployment.
- Independent scaling.
- Better fault isolation.
- Smaller codebases per service.
- Technology flexibility.
- Team ownership becomes easier.

**Trade-offs:**

- Network latency.
- Distributed transactions are hard.
- More monitoring and DevOps complexity.
- Service discovery, retries, tracing, and logging become important.

**Interview answer:** Microservices help large teams move faster and scale parts of the system independently, but they add distributed system complexity.

**Example:** In an e-commerce app, user service, payment service, inventory service, order service, and notification service can scale separately. During a sale, inventory and order services may need more replicas, while the user profile service may not.

### 7. API Gateway And Why Kong?

An **API Gateway** is a single entry point for client requests. It routes requests to backend services and handles cross-cutting concerns.

**Responsibilities:**

- Routing.
- Authentication and authorization.
- Rate limiting.
- Load balancing.
- Request/response transformation.
- Logging and monitoring.
- SSL termination.

**Why Kong?**

Kong is a popular open-source API gateway built on Nginx/OpenResty. It is fast, plugin-based, and commonly used for microservices.

**Kong advantages:**

- High performance.
- Plugin ecosystem.
- Supports rate limiting, auth, logging, transformations.
- Works well with Kubernetes.
- Can be used for REST, gRPC, and service mesh scenarios.

**Little deeper:** Without an API gateway, every service may need to implement the same concerns again and again, like authentication, rate limiting, logging, and SSL handling. A gateway keeps these concerns centralized and gives clients a clean single entry point.

### 8. Synchronous vs Asynchronous Communication

**Synchronous communication:** Caller waits for the response.

Examples: REST API call, gRPC call.

**Asynchronous communication:** Caller does not wait for immediate processing. Message is sent to a queue or broker.

Examples: Kafka, RabbitMQ, SQS, Pub/Sub.

| Point | Synchronous | Asynchronous |
| --- | --- | --- |
| Waiting | Caller waits | Caller does not wait |
| Coupling | Tightly coupled | Loosely coupled |
| Latency | Immediate response | Eventual processing |
| Failure handling | Direct error | Retry/DLQ patterns |
| Use case | User-facing request | Background jobs, events |

**Interview answer:** Use synchronous calls when an immediate response is required. Use asynchronous communication for decoupling, reliability, and background processing.

**Example:** Payment authorization should usually be synchronous because the user needs an immediate result. Sending a confirmation email can be asynchronous because it can happen after the main order is placed.

## gRPC / APIs

### 9. Why gRPC Over REST?

gRPC is a high-performance RPC framework that usually uses HTTP/2 and Protocol Buffers.

**Advantages over REST:**

- Faster serialization using protobuf.
- Strongly typed contracts.
- HTTP/2 multiplexing.
- Supports streaming.
- Auto-generated client/server code.
- Good for internal microservice communication.

**REST is still good for:**

- Public APIs.
- Browser-friendly APIs.
- Simple CRUD services.
- Human-readable JSON.

**Interview answer:** gRPC is preferred for internal service-to-service communication where performance, strict contracts, and streaming matter. REST is simpler and more widely compatible for public APIs.

**Little deeper:** REST usually exposes resources like `/users/1` and sends JSON. gRPC exposes functions like `GetUser()` and sends protobuf binary data. Because the contract is defined in a `.proto` file, both client and server can generate strongly typed code, which reduces mismatch errors between services.

### 10. Serialization vs Deserialization

**Serialization:** Converting an object/data structure into a format that can be stored or sent over the network.

Example: Go struct to JSON/protobuf bytes.

**Deserialization:** Converting serialized data back into an object/data structure.

Example: JSON/protobuf bytes to Go struct.

**Common formats:** JSON, XML, Protocol Buffers, Avro, MessagePack.

**Example:** When an API sends a Go struct as JSON, that is serialization. When another service reads that JSON body into its own struct, that is deserialization.

### 11. What Is Protobuf?

Protocol Buffers, or protobuf, is a binary serialization format created by Google.

You define message schemas in `.proto` files, then generate code for languages like Go, Java, Python, and JavaScript.

Example:

```proto
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

**Benefits:**

- Compact binary format.
- Faster than JSON in many cases.
- Strongly typed schema.
- Backward and forward compatibility when field numbers are managed carefully.
- Works naturally with gRPC.

**Important point:** In protobuf, field numbers are part of the binary contract. You should not reuse old field numbers for a different meaning because older clients may still understand that number in the old way.

## Concurrency

### 12. Concurrency vs Parallelism

**Concurrency:** Handling multiple tasks by switching between them. It is about structure.

**Parallelism:** Running multiple tasks at the exact same time. It requires multiple CPU cores.

Example:

- One chef handling multiple dishes by switching tasks: concurrency.
- Multiple chefs cooking multiple dishes at once: parallelism.

**Interview answer:** Concurrency is about dealing with many tasks at once. Parallelism is about executing many tasks at once.

**Little deeper:** A concurrent program can run on a single CPU core by switching between tasks. A parallel program needs multiple cores to literally execute multiple tasks at the same instant. Go makes it easy to write concurrent programs using goroutines and channels, and the runtime can run them in parallel when multiple CPU cores are available.

### 13. Goroutine vs Thread

| Point | Goroutine | Thread |
| --- | --- | --- |
| Managed by | Go runtime | Operating system |
| Initial stack | Small, grows dynamically | Larger fixed stack |
| Creation cost | Cheap | More expensive |
| Scheduling | Go scheduler | OS scheduler |
| Count | Can create thousands/millions | Usually fewer |

**Interview answer:** Goroutines are lightweight functions managed by the Go runtime. Many goroutines are multiplexed onto fewer OS threads.

**Little deeper:** An OS thread is relatively expensive because it is scheduled by the operating system and usually has a larger stack. A goroutine starts with a small stack that grows and shrinks as needed. This allows Go servers to handle many concurrent tasks like API requests, background jobs, and network calls efficiently.

### 14. Mutex vs Channel

**Mutex:** Used to protect shared memory from concurrent access.

**Channel:** Used to communicate data between goroutines.

| Point | Mutex | Channel |
| --- | --- | --- |
| Purpose | Lock shared state | Pass messages |
| Style | Shared memory with locking | Communicate by sharing |
| Best for | Counters, maps, critical sections | Pipelines, worker pools, signaling |
| Risk | Deadlock if lock not released | Deadlock if send/receive blocks forever |

**Rule of thumb:** Use a mutex when protecting shared state is simple. Use channels when goroutines need to coordinate or pass ownership of data.

**Example:** If many goroutines increment a shared counter, a mutex is usually simple and clear. If one goroutine produces jobs and many workers consume them, a channel is usually a better fit.

### 15. Race Condition And Deadlock

**Race condition:** Happens when multiple goroutines access shared data concurrently and at least one writes, causing unpredictable results.

Example:

```go
counter++
```

This is not atomic. It includes read, increment, and write.

**Deadlock:** Happens when goroutines wait forever for each other and no progress is possible.

Common deadlock examples:

- Sending to an unbuffered channel with no receiver.
- Receiving from a channel with no sender.
- Locking a mutex and never unlocking it.
- Two goroutines waiting for locks held by each other.

**Detection:** Go can detect some deadlocks at runtime with `fatal error: all goroutines are asleep - deadlock!`.

**How to avoid races and deadlocks:**

- Keep shared state small.
- Prefer clear ownership of data.
- Unlock mutexes with `defer` where appropriate.
- Close channels only from the sender side.
- Use `context` for cancellation.
- Use `go test -race` to detect data races.

## Caching

### 16. Cache Hit vs Cache Miss

**Cache hit:** Requested data is found in cache.

**Cache miss:** Requested data is not found in cache, so the system fetches it from the original source.

| Point | Cache Hit | Cache Miss |
| --- | --- | --- |
| Speed | Fast | Slower |
| Backend load | Low | Higher |
| Example | Redis returns value | DB query required |

**Little deeper:** A high cache hit ratio means most requests are served from cache, which usually means lower latency and less database load. A high miss ratio may mean the cache is too small, TTL is too short, keys are not designed well, or traffic is accessing many unique values.

### 17. Cache Invalidation

Cache invalidation means removing or updating stale cache data.

**Common strategies:**

- **TTL-based invalidation:** Cache expires after a fixed time.
- **Write-through:** Update cache whenever DB is updated.
- **Write-around:** Write to DB directly, cache on next read.
- **Write-back:** Write to cache first, DB later.
- **Manual invalidation:** Application explicitly deletes or updates cache key.

**Hard part:** Keeping cache fresh while still getting performance benefits.

**Common interview phrase:** Cache invalidation is hard because data changes in the original source, but old values may still exist in cache. If stale data is dangerous, use shorter TTLs, explicit invalidation, or write-through patterns.

### TTL

TTL stands for Time To Live. It defines how long cached data remains valid.

Example: If a Redis key has TTL of 5 minutes, it expires automatically after 5 minutes.

**Why TTL is useful:**

- Prevents stale data from living forever.
- Controls memory usage.
- Reduces need for manual invalidation.

### CDN Caching

A CDN, or Content Delivery Network, caches content near users geographically.

**Used for:**

- Images.
- CSS and JavaScript files.
- Videos.
- Static pages.
- API responses in some cases.

**Benefits:**

- Lower latency.
- Reduced origin server load.
- Better global performance.
- Improved availability.

### Local Cache vs Distributed Cache

| Point | Local Cache | Distributed Cache |
| --- | --- | --- |
| Location | Inside app process | External system like Redis |
| Speed | Very fast | Fast, but network call needed |
| Scope | Single instance | Shared across instances |
| Consistency | Hard with many instances | Easier to centralize |
| Example | In-memory map | Redis, Memcached |

**Use local cache** for small, frequently accessed data where slight inconsistency is acceptable.

**Use distributed cache** when multiple services/instances need shared cached data.

### Why Caching Improves Performance

Caching improves performance by storing frequently accessed data closer to the application or user.

**Benefits:**

- Reduces database load.
- Reduces network calls.
- Lowers latency.
- Improves throughput.
- Helps absorb traffic spikes.

## Database

### 18. ACID Properties

ACID describes reliable database transaction behavior.

| Property | Meaning |
| --- | --- |
| Atomicity | Transaction is all-or-nothing |
| Consistency | Transaction moves DB from one valid state to another |
| Isolation | Concurrent transactions do not interfere incorrectly |
| Durability | Once committed, data persists even after failure |

**Example:** During money transfer, debit and credit must both happen or both fail.

**Little deeper:** ACID is important when correctness matters more than raw speed. For example, orders, payments, account balances, and inventory counts usually need transaction guarantees. Some distributed systems relax strict ACID behavior for scalability and use eventual consistency instead.

### 19. Indexing Basics

An index is a data structure that improves read/query performance.

Without index, database may scan the whole table.

With index, database can find rows faster, often using B-tree-like structures.

Example:

```sql
CREATE INDEX idx_users_email ON users(email);
```

**Benefits:**

- Faster `WHERE` queries.
- Faster joins.
- Faster sorting in some cases.

**Trade-offs:**

- Slower writes.
- Extra storage.
- Bad indexes may not help.

**Interview answer:** Indexes speed up reads by avoiding full table scans, but they add write overhead because the index must also be updated.

**Little deeper:** Indexes are most useful on columns used often in `WHERE`, `JOIN`, `ORDER BY`, and `GROUP BY`. But indexing every column is bad because inserts, updates, and deletes become slower and storage usage increases.

## Why Go Is Fast

Go is fast because it compiles to native machine code and has a lightweight runtime.

**Reasons:**

- Compiled language.
- Simple type system.
- Efficient garbage collector.
- Goroutines are lightweight.
- Built-in concurrency model.
- Static binaries are easy to deploy.
- Low overhead compared to many interpreted languages.

**Detailed explanation:**

Go code is compiled ahead of time into machine code, so it does not need an interpreter at runtime. Its language design is also intentionally simple: no heavy inheritance model, no complex runtime reflection by default, and no hidden object model like some dynamic languages. This keeps execution predictable.

Go is also fast for backend systems because network I/O and concurrency are built into the standard library and runtime. A Go web server can create many goroutines for concurrent requests without creating one heavy OS thread per request.

**Important nuance:** Go is not always faster than C, C++, or Rust for raw CPU-heavy work, but it is usually very fast for web servers, microservices, network tools, CLIs, and concurrent systems because it balances performance, simple code, and efficient concurrency.

**Interview answer:** Go is fast because it is compiled to native code, has lightweight goroutines, an efficient scheduler, good standard-library networking support, escape analysis, and a garbage collector optimized for low pause times.

## Memory Management In Go

Go manages memory automatically using stack allocation, heap allocation, escape analysis, and garbage collection.

**Important ideas:**

- Local variables usually live on stack.
- Variables that outlive a function may escape to heap.
- Escape analysis decides whether a variable can stay on stack.
- Garbage collector cleans unused heap memory.

Example:

```go
func userName() *string {
  name := "Wasif"
  return &name
}
```

Here `name` escapes to heap because its address is returned.

**Stack vs heap:**

- **Stack:** Fast memory used for function calls and local variables. It is automatically cleaned when the function returns.
- **Heap:** Memory used for values that must live beyond the current function call. Heap memory is cleaned by the garbage collector.

**Escape analysis:**

Escape analysis is the compiler's process of deciding whether a variable can stay on the stack or must move to the heap. If a value is returned by pointer, stored globally, captured by a goroutine, or used in a way that outlives the function, it may escape to heap.

Example:

```go
func makeUser() User {
  user := User{Name: "Wasif"}
  return user
}
```

Here `user` can often stay on the stack because it is returned by value.

```go
func makeUserPtr() *User {
  user := User{Name: "Wasif"}
  return &user
}
```

Here `user` usually escapes to heap because the returned pointer must remain valid after the function returns.

**Why this matters:** More heap allocation means more work for the garbage collector. Writing simple code, avoiding unnecessary pointers, reusing buffers carefully, and understanding allocation hotspots can improve performance.

## Garbage Collection In Go

Go uses automatic garbage collection to free unused heap memory.

**How it works at a high level:**

1. Finds reachable objects starting from roots like globals and stacks.
2. Marks reachable objects as live.
3. Sweeps unreachable objects and frees memory.

Go GC is designed to keep pause times low, which is important for backend services.

**Trade-off:** GC improves developer productivity and memory safety, but it adds runtime overhead.

**More detail:**

Go's garbage collector mainly focuses on keeping application pause times low. It runs concurrently with the program for much of its work, so backend services do not stop for a long time whenever memory is collected.

**Reachable vs unreachable:**

- If an object can still be reached from running code, it is live.
- If no code can reach the object anymore, it is garbage.

Example:

```go
func process() {
  data := make([]byte, 1024)
  _ = data
}
```

After `process()` returns, if nothing else references `data`, that memory can eventually be collected.

**GC tuning idea:** Go exposes `GOGC`, which controls how aggressively the garbage collector runs. A lower value runs GC more often and may reduce memory usage. A higher value runs GC less often and may improve throughput but use more memory.

**Interview answer:** Go GC automatically finds heap objects that are no longer reachable and frees them. It is designed to run mostly concurrently with low pause times, which makes it suitable for production backend services.

## Basics

### Process vs Thread

**Process:** An independent running program with its own memory space.

**Thread:** A smaller execution unit inside a process. Threads in the same process share memory.

| Point | Process | Thread |
| --- | --- | --- |
| Memory | Separate memory | Shared process memory |
| Communication | IPC needed | Easier shared-memory communication |
| Creation cost | Higher | Lower |
| Crash impact | Usually isolated | Can crash entire process |

### Race Condition

A race condition happens when program behavior depends on unpredictable timing between concurrent operations.

In Go, data races can be detected using:

```bash
go test -race ./...
```

### Deadlock

A deadlock happens when tasks wait forever for each other.

Example causes:

- Circular lock dependency.
- Channel send without receiver.
- Channel receive without sender.
- Waiting on a `WaitGroup` whose counter never reaches zero.

### Starvation

Starvation happens when a goroutine or thread waits for a very long time because other tasks keep getting resources first.

Example: A low-priority task never gets CPU time because high-priority tasks keep running.

## Go-Specific

### What Is Goroutine?

A goroutine is a lightweight concurrent function managed by the Go runtime.

Example:

```go
go sendEmail(user)
```

Goroutines are cheaper than OS threads and are scheduled by the Go scheduler.

**More detail:**

When you write `go someFunction()`, Go starts that function in a new goroutine and immediately continues executing the current function. The goroutine may run now or later depending on scheduler decisions.

Goroutines are useful for:

- Handling many requests concurrently.
- Running background jobs.
- Calling external APIs without blocking the whole program.
- Building worker pools and pipelines.

Example:

```go
func main() {
  go fmt.Println("runs in another goroutine")
  fmt.Println("main continues")
  time.Sleep(time.Millisecond)
}
```

**Important:** If the `main` function exits, the program exits even if other goroutines are still running. Use `WaitGroup`, channels, or context cancellation to manage goroutine lifetimes.

### What Is Channel?

A channel is a typed communication mechanism between goroutines.

Example:

```go
ch := make(chan int)

go func() {
  ch <- 10
}()

value := <-ch
```

**More detail:**

Channels allow goroutines to send and receive values safely. A channel has a type, so `chan int` can only carry integers and `chan string` can only carry strings.

Channel operations:

```go
ch <- value   // send
value := <-ch // receive
close(ch)     // close
```

**Important rules:**

- Send to a closed channel causes panic.
- Receive from a closed channel returns the zero value immediately.
- Only the sender should usually close the channel.
- A nil channel blocks forever on send and receive.

Channels are excellent for signaling completion, passing jobs to workers, and building pipelines.

### Buffered vs Unbuffered Channel

**Unbuffered channel:** Send blocks until another goroutine receives.

```go
ch := make(chan int)
```

**Buffered channel:** Send blocks only when buffer is full. Receive blocks only when buffer is empty.

```go
ch := make(chan int, 5)
```

| Point | Unbuffered | Buffered |
| --- | --- | --- |
| Capacity | 0 | Fixed capacity |
| Send | Blocks until receiver ready | Blocks when buffer full |
| Receive | Blocks until sender ready | Blocks when buffer empty |
| Use case | Synchronization | Queue-like behavior |

**More detail:**

An unbuffered channel is a direct handoff. Sender and receiver must meet at the same time. This is useful when you want synchronization.

A buffered channel works like a small queue. Senders can place values into the buffer until it is full. Receivers can take values until it is empty.

Example use case:

```go
jobs := make(chan Job, 100)
```

Here workers can consume jobs from the buffer while producers add jobs. The buffer can smooth short bursts, but it should not be used as unlimited storage.

### What Is Mutex?

A mutex is a lock used to protect shared data.

Example:

```go
var mu sync.Mutex
var count int

mu.Lock()
count++
mu.Unlock()
```

Use `defer mu.Unlock()` when the critical section can return early.

**More detail:**

Maps, counters, slices, and structs can be unsafe if multiple goroutines read and write them at the same time. A mutex creates a critical section where only one goroutine can access the protected data.

Safer version:

```go
mu.Lock()
defer mu.Unlock()
count++
```

**Common mistakes:**

- Forgetting to unlock.
- Locking the same mutex twice in the same goroutine.
- Holding a lock while doing slow network or database calls.
- Copying a struct that contains a mutex.

### RWMutex

`sync.RWMutex` allows multiple readers or one writer.

Use it when reads are much more frequent than writes.

Example:

```go
var mu sync.RWMutex
var users = map[int]string{}

mu.RLock()
name := users[1]
mu.RUnlock()

mu.Lock()
users[2] = "Asha"
mu.Unlock()
```

**More detail:**

`RLock()` allows many readers to read at the same time. `Lock()` gives exclusive access to one writer. While a writer holds the lock, no reader can read. While readers hold `RLock()`, a writer must wait.

**When to use:** Use `RWMutex` when reads are very frequent and writes are rare. If writes are frequent, a normal `Mutex` may be simpler and sometimes faster.

### WaitGroup

`sync.WaitGroup` waits for multiple goroutines to finish.

Example:

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
  wg.Add(1)
  go func(id int) {
    defer wg.Done()
    fmt.Println(id)
  }(i)
}

wg.Wait()
```

**Common mistake:** Calling `wg.Add(1)` inside the goroutine can create a race with `wg.Wait()`.

**More detail:**

`WaitGroup` has an internal counter:

- `Add(n)` increases the counter.
- `Done()` decreases the counter by 1.
- `Wait()` blocks until the counter becomes 0.

**Correct pattern:** Call `Add(1)` before starting the goroutine. Call `Done()` with `defer` inside the goroutine so it runs even if the function returns early.

**Interview answer:** WaitGroup is used when the main goroutine needs to wait for a group of worker goroutines to complete.

### Select Statement

`select` lets a goroutine wait on multiple channel operations.

Example:

```go
select {
case msg := <-ch:
  fmt.Println(msg)
case <-time.After(time.Second):
  fmt.Println("timeout")
}
```

**Use cases:**

- Timeouts.
- Cancellation.
- Reading from multiple channels.
- Non-blocking channel operations using `default`.

**More detail:**

`select` is like `switch`, but for channels. If multiple cases are ready, Go picks one pseudo-randomly. If no case is ready, it blocks unless there is a `default` case.

Non-blocking receive:

```go
select {
case msg := <-ch:
  fmt.Println(msg)
default:
  fmt.Println("no message ready")
}
```

Using `select` with context:

```go
select {
case result := <-resultCh:
  return result, nil
case <-ctx.Done():
  return nil, ctx.Err()
}
```

This is common in production Go code because services need timeouts and cancellation.

### Context Package

The `context` package carries deadlines, cancellation signals, and request-scoped values across API boundaries.

Common functions:

| Function | Use |
| --- | --- |
| `context.Background()` | Root context |
| `context.WithCancel()` | Manual cancellation |
| `context.WithTimeout()` | Cancel after duration |
| `context.WithDeadline()` | Cancel at specific time |
| `context.WithValue()` | Request-scoped values |

Example:

```go
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

result, err := callService(ctx)
```

**Interview answer:** Context is used to propagate cancellation and timeouts across goroutines and service calls.

**More detail:**

Context helps prevent goroutine leaks. If a request is cancelled by the client or times out, downstream work should stop instead of continuing uselessly.

Common places context is used:

- HTTP handlers.
- Database queries.
- gRPC calls.
- External API calls.
- Worker goroutines.

Example with cancellation:

```go
ctx, cancel := context.WithCancel(context.Background())

go func() {
  defer cancel()
  doWork()
}()

<-ctx.Done()
```

**Best practices:**

- Pass `context.Context` as the first parameter.
- Do not store context inside structs unless there is a strong reason.
- Always call `cancel()` to release resources.
- Use `context.WithValue()` only for request-scoped metadata, not normal function parameters.

### How Scheduler Works In Go

The Go scheduler maps many goroutines onto a smaller number of OS threads.

The scheduler decides which goroutine runs, pauses goroutines that are blocked, and resumes them when they are ready.

**Important points:**

- Goroutines are scheduled cooperatively and preemptively.
- Blocking syscalls do not necessarily block the whole program.
- The scheduler helps Go handle many concurrent tasks efficiently.

**More detail:**

The scheduler maintains queues of runnable goroutines. When a goroutine blocks on I/O, channel receive, channel send, mutex, sleep, or syscall, the scheduler can run another goroutine instead of wasting the thread.

This is one reason Go is strong for I/O-heavy backend services. Thousands of goroutines can wait on network calls while a smaller number of OS threads keep doing useful work.

**Preemption:** Modern Go can preempt long-running goroutines so one busy goroutine does not unfairly block others forever.

**GOMAXPROCS:** `GOMAXPROCS` controls how many OS threads can execute Go code at the same time. By default, it is usually set to the number of available CPU cores.

**Interview answer:** The Go scheduler multiplexes many goroutines onto OS threads. When one goroutine blocks, the scheduler can run another, which makes concurrency efficient.

### GMP Model Basics

The Go scheduler uses the GMP model:

| Term | Meaning |
| --- | --- |
| G | Goroutine |
| M | Machine, an OS thread |
| P | Processor, scheduler context required to run Go code |

**How it works:**

- G represents the work to run.
- M is the OS thread that executes code.
- P holds runnable goroutines and scheduling resources.
- An M must have a P to execute Go code.

**Simple answer:** Go runs many goroutines by scheduling Gs onto Ms using Ps. This is why goroutines are lightweight and scalable.

**More detail:**

Think of it like this:

- **G:** The task, meaning the goroutine's stack and execution state.
- **M:** The actual OS thread that runs code.
- **P:** The permission and resources needed to run Go code.

An M needs a P to execute Go code. Each P has a local run queue of goroutines. There is also a global run queue. If one P runs out of work, it can steal work from another P. This is called work stealing.

**Blocking example:**

If a goroutine blocks on a network operation, Go can detach the waiting goroutine and allow the thread to run other goroutines. This keeps CPU resources active instead of letting threads sit idle.

**Interview answer:** In the GMP model, goroutines are scheduled as Gs, OS threads are Ms, and Ps provide the execution context. The scheduler uses local queues, global queues, and work stealing to run many goroutines efficiently.

## Last-Minute Interview Lines

- HTTP transfers data; HTTPS transfers encrypted data.
- DNS converts domain names to IP addresses.
- TCP is reliable; UDP is faster but best effort.
- Microservices improve independent scaling and deployment, but increase distributed system complexity.
- API Gateway centralizes routing, auth, rate limiting, and monitoring.
- gRPC is fast, strongly typed, and great for internal microservices.
- Serialization converts objects to bytes; deserialization converts bytes back to objects.
- Protobuf is a compact binary serialization format.
- Concurrency is handling many tasks; parallelism is executing many tasks at the same time.
- Goroutines are lightweight runtime-managed functions.
- Mutex protects shared memory; channels communicate between goroutines.
- Race condition is unpredictable shared-state access.
- Deadlock is forever waiting.
- Cache hit is fast; cache miss goes to original source.
- Cache invalidation is one of the hardest parts of caching.
- ACID makes transactions reliable.
- Indexes speed up reads but slow down writes.
- Go is fast because it compiles to native code and has efficient concurrency.
- Go GC automatically frees unused heap memory with low pause goals.
- GMP means Goroutine, Machine, Processor.
