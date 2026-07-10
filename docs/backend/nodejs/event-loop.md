# Node.js Event Loop

## Definition

The Node.js event loop is the mechanism that allows Node.js to perform non-blocking input/output operations while primarily executing JavaScript on a single main thread.

Node.js delegates many I/O operations to the operating system or the libuv worker pool. Once an operation completes, its callback is scheduled for execution by the event loop.

## Why Is It Needed?

Backend applications spend significant time waiting for operations such as:

* Database queries
* Network calls
* File operations
* Timers
* Message queue operations

Creating a dedicated thread for every request can consume considerable memory and introduce thread-management overhead.

Node.js uses an event-driven model that allows one process to manage many concurrent I/O operations efficiently.

## How It Works

A simplified request flow is:

1. A request reaches the Node.js server.
2. JavaScript begins processing the request on the main thread.
3. An asynchronous operation, such as a database query, is initiated.
4. Node.js delegates the operation to the operating system or libuv.
5. The main thread continues processing other work.
6. When the operation finishes, its callback is placed in a queue.
7. The event loop executes the callback when the call stack is available.

## Event Loop Phases

The major phases include:

1. Timers
2. Pending callbacks
3. Idle and prepare
4. Poll
5. Check
6. Close callbacks

Common examples:

* `setTimeout` callbacks run in the timers phase.
* I/O callbacks are mainly processed in the poll phase.
* `setImmediate` callbacks run in the check phase.

Promise callbacks and `process.nextTick` are processed through separate queues with higher priority than regular event-loop callbacks.

## Microtasks and Macrotasks

Microtasks include:

* Promise callbacks
* `queueMicrotask`
* `process.nextTick`

Macrotasks include:

* Timers
* I/O callbacks
* `setImmediate`

After the current operation completes, Node.js processes microtasks before moving to the next event-loop phase.

`process.nextTick` has especially high priority. Excessive recursive use can starve the event loop and prevent I/O from being processed.

## Example

```typescript
console.log("start");

setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve().then(() => {
  console.log("promise");
});

process.nextTick(() => {
  console.log("next tick");
});

console.log("end");
```

Expected output:

```text
start
end
next tick
promise
timer
```

The synchronous statements execute first. The `process.nextTick` callback executes before the Promise callback, and the timer runs afterward.

## CPU-Bound Work

Node.js performs well for I/O-heavy workloads but CPU-intensive work can block the event loop.

Examples include:

* Large JSON transformations
* Image processing
* Video processing
* Encryption
* Complex calculations
* Large synchronous loops

When the main thread is blocked:

* Other requests cannot be processed.
* Latency increases.
* Health checks may fail.
* The service may appear unavailable.

## Handling CPU-Intensive Work

Possible solutions include:

* Worker threads
* Child processes
* Background workers
* Message queues
* Separate compute services
* Native libraries
* Breaking large operations into smaller chunks

## Worker Pool

Libuv maintains a worker pool for selected operations, including:

* Some file-system operations
* DNS lookup
* Compression
* Cryptographic functions

The worker pool is different from the main JavaScript thread. Increasing its size can help specific workloads, but it does not solve all event-loop blocking problems.

## Production Risks

### Event-loop blocking

Long-running synchronous operations delay all other requests.

### Microtask starvation

Recursive Promise or `process.nextTick` operations can prevent other phases from executing.

### Large payload processing

Parsing or serializing very large JSON payloads can block the main thread.

### Excessive concurrency

Starting thousands of downstream requests simultaneously can overload databases or external services even though Node.js itself remains responsive.

## Observability

Useful measurements include:

* Event-loop lag
* Event-loop utilization
* Request latency
* CPU usage
* Memory usage
* Garbage collection duration
* Active handles
* Worker-pool utilization

High event-loop lag usually indicates CPU-heavy work, synchronous APIs, excessive garbage collection, or overloaded application logic.

## Interview Answer

Node.js uses a single main JavaScript thread with an event-driven, non-blocking I/O model. When the application initiates an asynchronous operation, Node.js delegates it to the operating system or libuv and continues processing other work. Once the operation completes, its callback is queued and later executed by the event loop.

This model is effective for I/O-heavy applications because one process can manage many concurrent requests. However, CPU-intensive or synchronous work blocks the event loop and delays every request handled by that process. Such workloads should be moved to worker threads, background jobs, or separate compute services.

## Common Interview Questions

### Is Node.js single-threaded?

JavaScript generally runs on one main thread, but the Node.js runtime is not entirely single-threaded. It uses operating-system capabilities, a libuv worker pool, and optional worker threads.

### Can Node.js handle concurrent requests?

Yes. It handles many concurrent I/O operations without requiring one thread per request.

### What blocks the event loop?

CPU-intensive JavaScript, synchronous APIs, large serialization operations, poorly designed loops, and excessive microtasks can block it.

### What is the difference between concurrency and parallelism?

Concurrency means multiple tasks make progress during overlapping periods. Parallelism means multiple tasks execute simultaneously on different CPU cores.

### When should worker threads be used?

Worker threads are suitable for CPU-intensive JavaScript tasks that can run independently from the main event loop.

## Revision Notes

* JavaScript runs mainly on one thread.
* Node.js delegates asynchronous work.
* The event loop processes completed callbacks.
* Microtasks run before normal event-loop tasks.
* `process.nextTick` can starve the loop.
* CPU-bound work blocks every request.
* Use worker threads or background workers for expensive computation.
