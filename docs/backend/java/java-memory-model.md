# Java Memory Model (JMM)

## Interview Priority

⭐⭐⭐⭐⭐

## What is the Java Memory Model?

The Java Memory Model (JMM) defines how threads interact through memory. It specifies when changes made by one thread become visible to other threads and provides guarantees around synchronization and ordering.

The JMM is about **visibility**, **ordering**, and **atomicity**, not the physical layout of memory.

---

## Why do we need it?

Without the JMM:

* One thread might not see updates made by another.
* The compiler and CPU could reorder instructions unexpectedly.
* Multithreaded programs could behave inconsistently.

---

## Core Concepts

### Visibility

When one thread updates a shared variable, other threads should eventually see the updated value.

```java
volatile boolean running = true;
```

Using `volatile` ensures that updates to `running` are visible to all threads.

---

### Atomicity

An operation is atomic if it cannot be interrupted.

Atomic:

```java
AtomicInteger counter = new AtomicInteger();
counter.incrementAndGet();
```

Not atomic:

```java
count++;
```

`count++` consists of:

1. Read
2. Increment
3. Write

Multiple threads can interleave these steps.

---

### Ordering

The compiler and CPU may reorder instructions for optimization.

The JMM guarantees correct ordering when synchronization constructs (`volatile`, `synchronized`, locks) are used.

---

## Happens-Before Relationship

The JMM defines *happens-before* rules to guarantee visibility.

Examples:

* Unlock happens-before a subsequent lock on the same monitor.
* A write to a `volatile` variable happens-before every subsequent read of that variable.
* Starting a thread happens-before actions inside that thread.
* Completing a thread happens-before another thread successfully joins it.

---

## volatile vs synchronized

| volatile     | synchronized                  |
| ------------ | ----------------------------- |
| Visibility   | Visibility + Mutual Exclusion |
| No locking   | Uses locking                  |
| No atomicity | Atomic for synchronized block |
| Lightweight  | More expensive                |

Use `volatile` for simple state flags.

Use `synchronized` when multiple operations must execute as one unit.

---

## Common Interview Questions

### Is `volatile` thread-safe?

No. It guarantees visibility, not atomicity.

### Is `count++` atomic?

No.

### When should you use `volatile`?

For shared flags and immutable references where compound operations are not required.

### What problem does JMM solve?

Visibility, instruction reordering, and thread communication.

---

## Quick Revision

* JMM defines thread interaction through memory.
* Three pillars: Visibility, Atomicity, Ordering.
* `volatile` → visibility.
* `synchronized` → visibility + atomicity.
* `count++` is not atomic.
* Happens-before guarantees correct synchronization.
