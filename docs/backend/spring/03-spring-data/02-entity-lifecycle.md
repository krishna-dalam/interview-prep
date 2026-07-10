# JPA Entity Lifecycle and Persistence Context

## Interview Priority

⭐⭐⭐⭐⭐

## Entity States

A JPA entity can be in one of four states:

```text
Transient
   ↓ persist()
Managed
   ↓ detach() / clear() / close()
Detached
   ↓ merge()
Managed
   ↓ remove()
Removed
```

## 1. Transient

A newly created object that is not associated with the persistence context.

```java
User user = new User();
user.setName("Alice");
```

No database row exists yet.

## 2. Managed

An entity tracked by the persistence context.

```java
User user = entityManager.find(User.class, 1L);
```

Changes to a managed entity are automatically detected and persisted during flush or transaction commit.

```java
user.setName("Bob");
```

No explicit `save()` is required when the entity is already managed inside a transaction.

## 3. Detached

An entity that was managed but is no longer tracked.

This can happen after:

* `entityManager.detach(entity)`
* `entityManager.clear()`
* Persistence context closes
* Transaction ends, depending on context scope

Changes to a detached entity are not automatically persisted.

## 4. Removed

A managed entity marked for deletion.

```java
entityManager.remove(user);
```

The delete is executed during flush or commit.

---

# Persistence Context

The persistence context is the set of entities currently managed by the `EntityManager`.

It provides:

* Entity identity
* Dirty checking
* First-level caching
* Write-behind
* Lifecycle management

## Entity Identity

Within one persistence context, repeated lookups for the same entity ID return the same Java object instance.

```java
User first = entityManager.find(User.class, 1L);
User second = entityManager.find(User.class, 1L);

System.out.println(first == second); // true
```

---

# Dirty Checking

Hibernate takes a snapshot of managed entity state.

At flush time, it compares the current state with the original state and generates an `UPDATE` when required.

```java
@Transactional
public void renameUser(Long id) {
    User user = repository.findById(id).orElseThrow();
    user.setName("Updated Name");
}
```

Because `user` is managed, Hibernate persists the change when the transaction commits.

## Interview Point

`save()` is often unnecessary for modifying an entity already loaded inside the same transaction.

---

# Flush vs Commit

## Flush

Synchronizes persistence-context changes with the database.

```java
entityManager.flush();
```

Flush can execute SQL, but it does not necessarily commit the database transaction.

## Commit

Finalizes the transaction.

A commit normally triggers a flush first.

```text
Entity changes
      ↓
Flush
      ↓
SQL executed
      ↓
Commit
```

A flushed transaction can still roll back.

---

# persist() vs merge()

## persist()

Makes a transient entity managed.

```java
entityManager.persist(user);
```

The same object instance becomes managed.

## merge()

Copies the state of a detached entity into a managed entity.

```java
User managed = entityManager.merge(detachedUser);
```

Important:

```java
managed != detachedUser
```

`merge()` returns the managed instance. The original detached object remains detached.

---

# First-Level Cache

The persistence context acts as Hibernate's first-level cache.

It is:

* Enabled by default
* Scoped to the `EntityManager` or session
* Not shared across application instances

```java
User first = repository.findById(1L).orElseThrow();
User second = repository.findById(1L).orElseThrow();
```

Within the same persistence context, the second lookup may avoid another database query.

---

# Write-Behind

Hibernate can delay SQL execution until flush or commit.

```java
user.setName("A");
user.setEmail("a@example.com");
```

Hibernate may combine the in-memory changes into one SQL update instead of executing SQL after every setter.

---

# Spring Data save()

`JpaRepository.save()` generally chooses between:

```text
New entity      → persist()
Existing entity → merge()
```

The exact decision depends on how Spring Data determines whether the entity is new.

For a managed existing entity, calling `save()` is usually redundant.

---

# Common Pitfalls

## Updating a Detached Entity

```java
user.setName("New Name");
```

If `user` is detached, the change is not automatically persisted.

## Assuming flush Means Commit

Flush sends SQL to the database, but rollback is still possible.

## Ignoring Persistence-Context Size

Loading and retaining thousands of managed entities can consume significant memory.

For batch processing, periodically use:

```java
entityManager.flush();
entityManager.clear();
```

## Bulk Updates Bypass Managed State

JPQL bulk update and delete operations act directly on the database and can leave managed entities stale.

After bulk operations, clear or refresh the persistence context when necessary.

---

# Common Interview Questions

### What is the persistence context?

A container managed by JPA that tracks entity instances and provides identity, dirty checking, caching, and lifecycle management.

### How does dirty checking work?

Hibernate tracks the original state of managed entities and compares it with their current state during flush.

### Is `save()` required after changing an entity?

Not when the entity is managed inside an active transaction. Dirty checking persists the change.

### What is the difference between `persist()` and `merge()`?

`persist()` makes the supplied transient instance managed. `merge()` copies state into another managed instance and returns it.

### What is the difference between flush and commit?

Flush synchronizes changes with the database. Commit permanently completes the transaction.

### Why can JPA consume too much memory during batch processing?

The persistence context retains managed entities. Flush and clear it periodically.

---

# Interview Scenario

```java
@Transactional
public void updateUser(Long id) {
    User user = repository.findById(id).orElseThrow();
    user.setStatus(Status.ACTIVE);
}
```

Why is there no `repository.save(user)`?

Because the entity is managed. Hibernate detects the change through dirty checking and writes it during flush or commit.

---

# Quick Revision

* Transient: not tracked.
* Managed: tracked by the persistence context.
* Detached: no longer tracked.
* Removed: scheduled for deletion.
* Managed entities support dirty checking.
* First-level cache belongs to the persistence context.
* Flush executes pending SQL; commit completes the transaction.
* `merge()` returns a managed copy.
* Avoid retaining too many managed entities in batch jobs.
