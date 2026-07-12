# Hibernate Caching (First-Level & Second-Level Cache)

## Interview Priority

⭐⭐⭐⭐⭐

## Why Do We Need Caching?

Every database query has a cost:

* Network latency
* Database CPU
* Disk I/O
* Connection pool usage

Caching reduces unnecessary database access.

---

# Hibernate Cache Levels

```text
                Hibernate

        +----------------------+
        | First Level Cache    |
        +----------------------+
                    |
        +----------------------+
        | Second Level Cache   |
        +----------------------+
                    |
               Database
```

Hibernate provides:

* First-Level Cache
* Second-Level Cache

Query Cache is an optional feature built on top of the second-level cache.

---

# First-Level Cache

## What is it?

The First-Level Cache is the Persistence Context.

Every `EntityManager` (or Hibernate `Session`) has its own cache.

It is:

* Enabled by default
* Mandatory
* Session scoped

---

## Example

```java
User user1 = entityManager.find(User.class, 1L);

User user2 = entityManager.find(User.class, 1L);
```

Generated SQL:

```sql
SELECT * FROM users WHERE id = 1;
```

Only **one query** executes.

The second lookup returns the same managed object.

---

## Identity Guarantee

```java
user1 == user2
```

Returns:

```text
true
```

Within one persistence context, there is only one managed instance for a given entity ID.

---

# When Is the First-Level Cache Cleared?

It is cleared when:

```java
entityManager.clear();
```

or

```java
entityManager.close();
```

or when the transaction/session ends.

---

# Second-Level Cache

The Second-Level Cache is shared across sessions.

```text
Request A

↓

Session A

↓

Second-Level Cache

↓

Database

------------------------

Request B

↓

Session B

↓

Second-Level Cache

↓

Database
```

Unlike the first-level cache, multiple sessions can reuse cached entities.

---

# Supported Cache Providers

Hibernate does not provide a cache implementation itself.

Common providers:

* Ehcache
* Caffeine (through JCache integrations)
* Infinispan

The provider is configurable.

---

# Enabling Second-Level Cache

Example:

```java
@Entity
@Cacheable
```

Hibernate can then cache entity instances according to the configured strategy.

---

# Cache Concurrency Strategies

Common strategies:

### READ_ONLY

For immutable data.

Examples:

* Countries
* Languages
* Product categories

---

### READ_WRITE

Supports updates using a concurrency strategy.

Suitable for most business entities.

---

### NONSTRICT_READ_WRITE

Allows stale data for better performance.

Suitable when occasional stale reads are acceptable.

---

### TRANSACTIONAL

Used with cache providers that support transactional semantics.

Less common.

---

# Query Cache

Caches query results.

Example:

```java
select *
from product
where category='BOOK'
```

Instead of executing the SQL repeatedly, Hibernate can cache the list of matching identifiers.

Important:

The Query Cache works together with the Second-Level Cache.

Caching only the query result is not enough if the corresponding entities are not cached.

---

# Hibernate Cache vs Redis

This is a very common interview question.

| Hibernate Cache                                                                  | Redis                              |
| -------------------------------------------------------------------------------- | ---------------------------------- |
| ORM-level cache                                                                  | Distributed cache                  |
| Mostly transparent to Hibernate                                                  | Explicitly used by the application |
| Typically local to an application instance unless backed by a clustered provider | Shared across services             |
| Entity-oriented                                                                  | General-purpose key/value store    |

Hibernate cache is designed to optimize ORM access.

Redis is a broader application cache.

---

# First-Level vs Second-Level

| First-Level          | Second-Level               |
| -------------------- | -------------------------- |
| Enabled by default   | Must be configured         |
| Session scoped       | Shared across sessions     |
| Mandatory            | Optional                   |
| Cannot be disabled   | Can be enabled selectively |
| No provider required | Requires a cache provider  |

---

# Cache Flow

Without cache:

```text
Application

↓

Database
```

First-Level Cache:

```text
Application

↓

Persistence Context

↓

Database
```

Second-Level Cache:

```text
Application

↓

Persistence Context

↓

Second-Level Cache

↓

Database
```

---

# Cache Invalidation

Whenever data changes, cached entries may need to be updated or evicted.

Incorrect invalidation can lead to stale data.

Choosing the right cache strategy depends on:

* Update frequency
* Consistency requirements
* Performance goals

---

# Common Interview Questions

### What is the First-Level Cache?

The persistence context associated with an `EntityManager` or Hibernate session.

---

### Is the First-Level Cache enabled by default?

Yes.

---

### Can two sessions share the First-Level Cache?

No.

Each session has its own cache.

---

### What is the Second-Level Cache?

A cache shared across sessions to reduce repeated database access.

---

### Does Hibernate provide a Second-Level Cache implementation?

No.

A cache provider must be configured.

---

### Is Redis the same as the Hibernate Second-Level Cache?

No.

Redis is a distributed key/value store.

Hibernate's Second-Level Cache is an ORM-level optimization.

---

### Should every entity be cached?

No.

Frequently changing entities may perform poorly when cached due to invalidation overhead.

---

# Real-World Example

An e-commerce application has a `Country` table.

Characteristics:

* Changes very rarely.
* Read thousands of times each minute.

Using a READ_ONLY Second-Level Cache:

* The first request loads the data from the database.
* Subsequent requests retrieve it from the cache.
* Database load decreases significantly.

By contrast, a rapidly changing `Inventory` entity is usually a poor candidate for aggressive entity caching because updates and invalidations occur frequently.

---

# Quick Revision

* First-Level Cache = Persistence Context.
* Enabled by default.
* One cache per session.
* Second-Level Cache is shared across sessions.
* Requires an external cache provider.
* Query Cache depends on the Second-Level Cache.
* Redis is an application cache, not a Hibernate cache.
* Cache only data with appropriate read/write characteristics.
