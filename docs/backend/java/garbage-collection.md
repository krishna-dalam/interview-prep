# Garbage Collection & Memory Management

## Interview Priority

⭐⭐⭐⭐⭐

## What is Garbage Collection?

Garbage Collection (GC) is the JVM's automatic process of reclaiming memory occupied by objects that are no longer reachable.

Without GC, developers would need to manually allocate and free memory, increasing the risk of memory leaks and crashes.

---

## Why is GC Needed?

When objects become unreachable, they continue occupying heap memory unless reclaimed.

```java
User user = new User();

user = null;
```

The `User` object is now eligible for garbage collection because no live references point to it.

---

# Heap Memory Layout

Modern JVMs divide the heap into generations.

```text
                 Heap
                  |
     --------------------------
     |                        |
 Young Generation      Old Generation
     |
 -----------
|           |
Eden    Survivor
```

---

## Young Generation

New objects are allocated here.

Most Java objects have a short lifetime.

Examples:

* HTTP request objects
* DTOs
* JSON objects
* Temporary collections

These objects are collected frequently.

---

## Old Generation

Objects that survive multiple garbage collection cycles are promoted here.

Examples:

* Application caches
* Singleton beans
* Long-lived services

Collections occur less frequently but usually take longer.

---

# Minor GC

Minor GC cleans the Young Generation.

Characteristics:

* Fast
* Frequent
* Low pause time

---

# Major (Old) GC

Major GC cleans the Old Generation.

Characteristics:

* Less frequent
* Longer pause times
* Higher CPU usage

---

# Full GC

Full GC examines the entire heap.

Characteristics:

* Highest pause time
* Expensive
* Usually indicates memory pressure or tuning issues

Frequent Full GCs in production are a warning sign.

---

# Reachability

An object is eligible for GC only if it is unreachable from GC Roots.

Typical GC Roots include:

* Active thread stacks
* Static fields
* JNI references
* Running threads

Example:

```java
User user = new User();
user = null;
```

The object can now be reclaimed.

---

# Memory Leak

A memory leak occurs when objects are no longer useful but remain reachable.

Example:

```java
List<User> cache = new ArrayList<>();

while (true) {
    cache.add(new User());
}
```

The list keeps growing, preventing the objects from being collected.

Common causes:

* Static collections
* Unbounded caches
* Forgotten listeners
* ThreadLocal misuse
* Long-lived references

---

# OutOfMemoryError

Occurs when the JVM cannot allocate more memory.

Common types:

* Java heap space
* Metaspace
* GC overhead limit exceeded
* Unable to create new native thread

Increasing heap size may postpone the issue but does not fix application-level memory leaks.

---

# Garbage Collectors

### Serial GC

* Single-threaded
* Small applications
* Not suitable for large server workloads

---

### Parallel GC

* Multiple GC threads
* Focus on throughput
* Longer pause times

---

### G1 GC (Default for modern server JVMs)

Designed for large heaps.

Benefits:

* Predictable pause times
* Region-based heap
* Incremental collection
* Good balance between throughput and latency

Common choice for Spring Boot services.

---

### ZGC

Designed for:

* Very large heaps
* Extremely low pause times

Useful for latency-sensitive applications.

---

### Shenandoah

Another low-pause collector with goals similar to ZGC.

---

# Stop-The-World (STW)

During certain GC phases, all application threads pause.

```text
Application Running

↓

Stop The World

↓

Garbage Collection

↓

Resume Application
```

Large or frequent STW pauses can increase API response times.

---

# Common JVM Memory Issues

### High GC Frequency

Possible causes:

* Excessive object creation
* Small heap
* Poor coding practices

---

### Long GC Pauses

Possible causes:

* Large heap
* Large live object set
* Inefficient collector choice

---

### Memory Leak

Symptoms:

* Heap usage continuously increases
* Frequent Full GCs
* Eventually OutOfMemoryError

---

# Monitoring

Common JVM metrics:

* Heap Usage
* Young GC Count
* Old GC Count
* GC Pause Time
* Allocation Rate
* Promotion Rate
* Metaspace Usage

Useful tools:

* JVisualVM
* JConsole
* Java Flight Recorder (JFR)
* Java Mission Control (JMC)
* Eclipse MAT
* `jcmd`
* `jmap`
* `jstack`

---

# Common Interview Questions

### Does setting an object to `null` immediately free memory?

No.

It only makes the object eligible for garbage collection.

---

### Can we force garbage collection?

```java
System.gc();
```

This is only a request to the JVM.

There is no guarantee that GC will run immediately.

---

### Why is G1 GC commonly used?

It provides a good balance between throughput and predictable pause times for server applications.

---

### What causes OutOfMemoryError?

Typical causes include:

* Memory leaks
* Heap too small
* Excessive object creation
* Unbounded caches
* Thread leaks

---

### How would you investigate a Spring Boot service with increasing memory usage?

1. Monitor heap usage.
2. Check GC logs and pause times.
3. Capture a heap dump.
4. Analyze retained objects using Eclipse MAT.
5. Identify large collections, caches, or leaking references.
6. Fix the root cause rather than only increasing heap size.

---

# Real-World Example

A Spring Boot application caches every customer record in a static `HashMap` without eviction.

As traffic grows:

* Heap usage increases.
* Full GCs become more frequent.
* Response times increase due to longer pauses.
* Eventually the application throws `OutOfMemoryError`.

Replacing the unbounded cache with a bounded cache (e.g., Caffeine) or an external cache (e.g., Redis), along with monitoring heap usage, resolves the issue.

---

# Quick Revision

* GC automatically reclaims unreachable objects.
* Heap consists of Young and Old generations.
* Minor GC → Young Generation.
* Major GC → Old Generation.
* Full GC → Entire heap.
* Memory leaks occur when unused objects remain reachable.
* `System.gc()` is only a request.
* G1 GC is the common default for server applications.
* Monitor heap, GC frequency, and pause times in production.
