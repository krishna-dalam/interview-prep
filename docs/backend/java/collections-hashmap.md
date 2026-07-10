# Collections Framework & HashMap Internals

## Interview Priority

⭐⭐⭐⭐⭐

## What is the Collections Framework?

The Java Collections Framework provides reusable data structures and algorithms for storing and manipulating groups of objects.

Common interfaces:

```text
Collection
├── List
│   ├── ArrayList
│   ├── LinkedList
│   └── Vector
├── Set
│   ├── HashSet
│   └── TreeSet
└── Queue
    ├── PriorityQueue
    └── Deque

Map
├── HashMap
├── LinkedHashMap
├── TreeMap
├── ConcurrentHashMap
└── Hashtable
```

---

# ArrayList vs LinkedList

| Feature          | ArrayList      | LinkedList         |
| ---------------- | -------------- | ------------------ |
| Backed by        | Dynamic Array  | Doubly Linked List |
| Random Access    | O(1)           | O(n)               |
| Insert at End    | O(1) amortized | O(1)               |
| Insert in Middle | O(n)           | O(n) (search)      |
| Memory Usage     | Lower          | Higher             |

### Use ArrayList when:

* Reads are frequent
* Random access is required

### Use LinkedList when:

* Frequent insertions/removals at the ends
* Sequential traversal

---

# HashMap

A `HashMap` stores key-value pairs and provides average O(1) lookup using hashing.

Example:

```java
Map<Integer, String> users = new HashMap<>();
users.put(1, "Alice");
users.put(2, "Bob");
```

---

# How HashMap Works

When inserting:

```java
map.put(key, value);
```

The JVM:

1. Computes `hashCode()` of the key.
2. Converts the hash into a bucket index.
3. Stores the entry in that bucket.
4. Handles collisions if multiple keys map to the same bucket.

```text
Buckets

0
1 -> (A)
2
3 -> (B)
4 -> (C)
```

---

# What is a Hash Collision?

A collision occurs when two different keys map to the same bucket.

```text
Bucket 5

Key A
↓

Key B
↓

Key C
```

Java handles collisions by storing multiple entries in the same bucket.

---

# Collision Handling

### Before Java 8

Buckets used a linked list.

```text
Bucket

A
↓

B
↓

C
```

Worst-case lookup: O(n)

---

### Java 8+

If many entries exist in the same bucket (typically more than 8, provided the table is sufficiently large), the linked list is converted into a balanced tree.

```text
        B
      /   \
     A     D
          /
         C
```

Worst-case lookup improves to O(log n).

---

# Load Factor

The default load factor is **0.75**.

```text
capacity × load factor
```

Example:

Capacity = 16

Threshold = 12

When the threshold is exceeded, the table resizes.

---

# Resizing

When the threshold is reached:

* Capacity doubles.
* Existing entries are redistributed into the new bucket array.

Example:

```text
16 buckets
↓

32 buckets
```

Resizing is an expensive operation because existing entries must be rehashed.

---

# Why Are equals() and hashCode() Important?

`HashMap` first uses `hashCode()` to locate the bucket and then `equals()` to find the exact key within that bucket.

If two objects are equal:

```java
obj1.equals(obj2) == true
```

Their hash codes must also be equal.

Failing to maintain this contract can lead to incorrect lookups.

---

# Common Time Complexities

| Operation | Average | Worst           |
| --------- | ------- | --------------- |
| get()     | O(1)    | O(n) / O(log n) |
| put()     | O(1)    | O(n) / O(log n) |
| remove()  | O(1)    | O(n) / O(log n) |

---

# HashMap vs Hashtable

| HashMap             | Hashtable                     |
| ------------------- | ----------------------------- |
| Not thread-safe     | Thread-safe                   |
| Allows one null key | Does not allow null keys      |
| Better performance  | Slower due to synchronization |

For concurrent applications, prefer `ConcurrentHashMap` over `Hashtable`.

---

# HashMap vs ConcurrentHashMap

| HashMap                             | ConcurrentHashMap                        |
| ----------------------------------- | ---------------------------------------- |
| Not thread-safe                     | Thread-safe                              |
| Suitable for single-threaded use    | Designed for concurrent access           |
| Faster in single-threaded scenarios | Better scalability under concurrent load |

---

# HashSet Internals

`HashSet` is internally backed by a `HashMap`.

```java
Set<String> set = new HashSet<>();
set.add("Java");
```

Conceptually:

```text
HashMap

Key = "Java"

Value = Dummy Object
```

The values are placeholders; uniqueness is enforced through the keys.

---

# TreeMap

`TreeMap` stores keys in sorted order.

Internally, it uses a Red-Black Tree.

Time complexity:

* get() → O(log n)
* put() → O(log n)
* remove() → O(log n)

Use `TreeMap` when sorted iteration is required.

---

# LinkedHashMap

Maintains insertion order (or access order if configured).

Common use case:

* LRU cache implementations.

---

# Common Interview Questions

### Why is HashMap lookup O(1)?

The hash code allows direct access to the appropriate bucket, avoiding a full scan in the average case.

---

### Why is HashMap not thread-safe?

Concurrent modifications can lead to inconsistent state and race conditions.

---

### Why must hashCode() and equals() be implemented together?

`hashCode()` selects the bucket, while `equals()` identifies the matching key within that bucket.

---

### Why is the default load factor 0.75?

It balances memory usage and lookup performance. A lower load factor uses more memory but reduces collisions; a higher load factor saves memory but increases collision probability.

---

### Why did Java 8 introduce treeification?

Converting long collision chains into balanced trees improves worst-case lookup performance from O(n) to O(log n).

---

# Real-World Example

A Spring Boot service might use:

* `HashMap` for request-scoped in-memory lookups.
* `ConcurrentHashMap` for shared caches.
* `LinkedHashMap` for implementing an LRU cache.
* `TreeMap` for maintaining sorted configuration or ranking data.

Choosing the right collection depends on concurrency requirements, ordering needs, and access patterns.

---

# Quick Revision

* `HashMap` uses hashing for average O(1) operations.
* Collisions occur when multiple keys map to the same bucket.
* Java 8+ treeifies heavily loaded buckets.
* Default load factor is 0.75.
* Resizing doubles capacity and rehashes entries.
* `HashSet` is backed by a `HashMap`.
* `TreeMap` uses a Red-Black Tree.
* `ConcurrentHashMap` is preferred for shared mutable data in concurrent applications.
