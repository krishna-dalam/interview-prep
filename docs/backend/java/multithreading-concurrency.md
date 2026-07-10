# Multithreading & Concurrency

## Interview Priority

⭐⭐⭐⭐⭐

## What is Multithreading?

Multithreading allows multiple threads to execute within the same process, sharing memory while performing tasks concurrently.

Example:

* Thread 1 → Handle HTTP Request A
* Thread 2 → Handle HTTP Request B
* Thread 3 → Database Query
* Thread 4 → Background Job

---

## Process vs Thread

| Process                    | Thread                            |
| -------------------------- | --------------------------------- |
| Independent execution unit | Lightweight execution unit        |
| Own memory                 | Shares process memory             |
| Expensive to create        | Cheaper to create                 |
| Communicate via IPC        | Communicate through shared memory |

---

## Thread Lifecycle

```text
NEW
 ↓
RUNNABLE
 ↓
RUNNING
 ↓
BLOCKED / WAITING / TIMED_WAITING
 ↓
TERMINATED
```

---

## Creating Threads

### Extending Thread

```java
class Worker extends Thread {
    public void run() {
        System.out.println("Working");
    }
}
```

### Implementing Runnable (Preferred)

```java
Runnable task = () -> System.out.println("Working");
new Thread(task).start();
```

---

## Why Not Create Threads Manually?

Creating a thread for every request:

* Consumes memory
* Has creation overhead
* Doesn't scale
* Can exhaust system resources

Instead, use thread pools.

---

## ExecutorService

A thread pool manages a fixed or dynamic number of reusable threads.

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);

executor.submit(() -> processOrder());
```

Benefits:

* Reuses threads
* Limits concurrency
* Improves performance
* Easier lifecycle management

---

## Common Thread Pools

* FixedThreadPool
* CachedThreadPool
* SingleThreadExecutor
* ScheduledThreadPool
* WorkStealingPool

---

## Callable vs Runnable

| Runnable                        | Callable                     |
| ------------------------------- | ---------------------------- |
| No return value                 | Returns a value              |
| Cannot throw checked exceptions | Can throw checked exceptions |

Example:

```java
Callable<Integer> task = () -> 100;
Future<Integer> future = executor.submit(task);
```

---

## Future

Represents the result of an asynchronous computation.

```java
Future<String> future =
    executor.submit(() -> "Hello");

String result = future.get();
```

Problem:

`get()` blocks the current thread.

---

## CompletableFuture

Modern approach for asynchronous programming.

```java
CompletableFuture
    .supplyAsync(() -> fetchUser())
    .thenApply(user -> user.getName())
    .thenAccept(System.out::println);
```

Advantages:

* Non-blocking
* Chaining
* Parallel execution
* Better readability

---

## synchronized

Ensures only one thread executes a critical section at a time.

```java
public synchronized void increment() {
    count++;
}
```

Provides:

* Mutual exclusion
* Visibility

---

## ReentrantLock

A more flexible alternative to `synchronized`.

Advantages:

* tryLock()
* Fair locking
* Timeout support
* Interruptible locking

Use when advanced locking behavior is required.

---

## volatile

Ensures visibility of updates across threads.

```java
volatile boolean running = true;
```

Use for:

* Status flags
* Shutdown indicators

Not suitable for compound operations like `count++`.

---

## Atomic Classes

Provide lock-free thread-safe operations.

```java
AtomicInteger counter =
    new AtomicInteger();

counter.incrementAndGet();
```

Examples:

* AtomicInteger
* AtomicLong
* AtomicBoolean
* AtomicReference

---

## ConcurrentHashMap

Thread-safe alternative to `HashMap`.

Benefits:

* High concurrency
* Better performance than synchronized maps
* Segment-free implementation in modern Java

---

## Common Concurrency Problems

### Race Condition

Two threads modify shared data simultaneously.

Example:

```java
count++;
```

May produce incorrect results.

---

### Deadlock

Two threads wait for each other indefinitely.

Example:

```text
Thread A
  Lock A
  Wait Lock B

Thread B
  Lock B
  Wait Lock A
```

Avoid by:

* Consistent lock ordering
* Timeouts
* Minimizing nested locks

---

### Starvation

A thread never gets CPU time or required resources because other threads continuously take precedence.

---

### Livelock

Threads remain active but continuously respond to each other without making progress.

---

## Producer–Consumer Pattern

One thread produces tasks.

Another thread processes them.

Typically implemented using:

* BlockingQueue
* Kafka
* RabbitMQ
* AWS SQS

---

## Interview Questions

### Why is ExecutorService preferred over creating threads?

It reuses threads, limits resource consumption, and scales better.

---

### synchronized vs ReentrantLock?

Use `synchronized` for simple mutual exclusion.

Use `ReentrantLock` when features like timeout, fairness, or interruptible locking are needed.

---

### Future vs CompletableFuture?

`Future` supports asynchronous execution but blocks when retrieving results.

`CompletableFuture` supports non-blocking composition, chaining, and parallel workflows.

---

### volatile vs AtomicInteger?

`volatile` guarantees visibility only.

`AtomicInteger` provides atomic operations in addition to visibility.

---

### HashMap vs ConcurrentHashMap?

`HashMap` is not thread-safe.

`ConcurrentHashMap` supports concurrent access efficiently without locking the entire map.

---

## Real-World Example

In a Spring Boot application:

* HTTP requests are handled by a server thread pool.
* Database operations may run concurrently.
* Background jobs use `ExecutorService` or Spring's `@Async`.
* Shared caches often use `ConcurrentHashMap` or Redis.
* High-throughput services rely on thread pools instead of creating new threads for each request.

---

## Quick Revision

* Prefer thread pools over manual thread creation.
* Use `Runnable` or `Callable` for tasks.
* Use `CompletableFuture` for modern asynchronous programming.
* `volatile` → visibility.
* `synchronized` → mutual exclusion.
* `AtomicInteger` → atomic operations.
* `ConcurrentHashMap` → thread-safe map.
* Know race conditions, deadlocks, starvation, and livelocks.
