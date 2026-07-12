# Optimistic vs Pessimistic Locking

## Interview Priority

⭐⭐⭐⭐⭐

## Why Do We Need Locking?

In concurrent systems, multiple users or services may update the same record simultaneously.

Example:

```text
Account Balance = ₹1000

User A Withdraws ₹200

User B Withdraws ₹500
```

Without locking, one update may overwrite the other.

This is called a **Lost Update**.

---

# Lost Update Example

Thread A

```text
Read Balance = 1000
```

Thread B

```text
Read Balance = 1000
```

Thread A

```text
Balance = 800
```

Thread B

```text
Balance = 500
```

Expected:

```text
300
```

Actual:

```text
500
```

One update is lost.

---

# Locking Strategies

JPA provides two approaches:

* Optimistic Locking
* Pessimistic Locking

---

# Optimistic Locking

Assumption:

> Conflicts are rare.

Instead of locking the row, Hibernate checks whether another transaction modified it.

---

## @Version

```java
@Entity
public class Product {

    @Id
    private Long id;

    @Version
    private Long version;

}
```

Database

```text
id

name

version
```

Example

```text
Product

Version = 5
```

Two users load the same record.

Both see:

```text
Version = 5
```

---

## Update

Thread A

```text
UPDATE product

SET version = 6

WHERE id = 1

AND version = 5
```

Success.

---

Thread B

Attempts

```text
UPDATE product

SET version = 6

WHERE version = 5
```

No rows updated.

Hibernate throws:

```text
OptimisticLockException
```

---

## Advantages

* High throughput
* No database lock
* Better scalability
* Ideal for web applications

---

## Disadvantages

* Retries may be required
* Transactions can fail due to conflicts

---

# Pessimistic Locking

Assumption:

> Conflicts are likely.

The database row is locked immediately.

Other transactions must wait.

---

## Example

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
```

Flow

```text
Thread A

Locks row

↓

Updates row

↓

Commit

↓

Unlock
```

Thread B waits until the lock is released.

---

## Advantages

* Prevents concurrent modifications
* Suitable for high-contention data

---

## Disadvantages

* Blocking
* Deadlocks
* Reduced throughput

---

# Lock Modes

## OPTIMISTIC

Version check during commit.

---

## OPTIMISTIC_FORCE_INCREMENT

Version is incremented even if no data changes.

Useful when related updates should invalidate concurrent changes.

---

## PESSIMISTIC_READ

Shared lock.

Other readers allowed.

Writers blocked.

---

## PESSIMISTIC_WRITE

Exclusive lock.

Only one transaction may update.

---

## PESSIMISTIC_FORCE_INCREMENT

Combines write lock with version increment.

---

# When to Use Optimistic Locking

Examples:

* User profiles
* Product catalog
* Customer records
* Inventory with occasional updates
* Order management

Most CRUD applications.

---

# When to Use Pessimistic Locking

Examples:

* Banking
* Seat booking
* Stock trading
* Wallet balance
* Payment processing

Where conflicts are frequent and correctness is critical.

---

# Deadlocks

Example

```text
Thread A

Locks Account

Waits Wallet

Thread B

Locks Wallet

Waits Account
```

Neither can continue.

Database detects the deadlock.

One transaction is rolled back.

---

# Optimistic vs Pessimistic

| Optimistic                   | Pessimistic                      |
| ---------------------------- | -------------------------------- |
| No DB lock                   | Database row lock                |
| Better throughput            | More blocking                    |
| Retry on conflict            | Wait on conflict                 |
| Best when conflicts are rare | Best when conflicts are frequent |

---

# Common Interview Questions

### What problem does optimistic locking solve?

Lost updates caused by concurrent modifications.

---

### What is @Version?

A version column used to detect concurrent updates.

---

### What happens if two users update the same row?

With optimistic locking:

The second update fails with an `OptimisticLockException`.

---

### Why not always use pessimistic locking?

It reduces concurrency, increases waiting time, and can lead to deadlocks.

---

### Which locking strategy is better?

Neither is universally better.

Choose based on contention:

* Rare conflicts → Optimistic
* Frequent conflicts → Pessimistic

---

# Real-World Example

A movie ticket booking system has only one seat left.

Two users try to reserve it at the same time.

Using optimistic locking:

* Both read the seat.
* First user books successfully.
* Second user's update fails due to a version mismatch.
* The application asks the second user to select another seat or retry.

For systems with extremely high contention, pessimistic locking may be appropriate to serialize access.

---

# Quick Revision

* Locking prevents lost updates.
* Optimistic locking uses `@Version`.
* Pessimistic locking uses database row locks.
* Optimistic locking scales better.
* Pessimistic locking blocks competing transactions.
* `OptimisticLockException` indicates a version conflict.
* Use optimistic locking for most business applications.
* Use pessimistic locking only when contention is expected to be high.
