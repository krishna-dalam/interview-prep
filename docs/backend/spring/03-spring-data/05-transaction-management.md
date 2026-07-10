# Spring Transaction Management

## Interview Priority

⭐⭐⭐⭐⭐

## What Is a Transaction?

A transaction groups multiple database operations into one logical unit of work.

It should either:

* Complete fully, or
* Roll back fully

Example:

```text
Create order
Reduce inventory
Create payment record
```

If one step fails, the transaction should usually roll back.

---

## ACID Properties

* **Atomicity** — all operations succeed or fail together
* **Consistency** — data remains valid
* **Isolation** — concurrent transactions do not interfere incorrectly
* **Durability** — committed data survives failures

---

# `@Transactional`

```java
@Transactional
public void createOrder() {
    orderRepository.save(order);
    inventoryRepository.reduceStock();
}
```

Spring opens a transaction before the method starts and commits it when the method completes successfully.

If a qualifying exception occurs, Spring rolls it back.

---

# How `@Transactional` Works Internally

Spring uses AOP proxies.

```text
Caller
  |
  v
Transaction Proxy
  |
  +--> Begin transaction
  |
  v
Target method
  |
  +--> Commit or rollback
```

High-level flow:

1. Spring creates a proxy around the bean.
2. The caller invokes the proxy.
3. The proxy starts or joins a transaction.
4. The target method executes.
5. Spring commits on success.
6. Spring rolls back on failure.

---

# Rollback Rules

By default, Spring rolls back for:

* `RuntimeException`
* `Error`

It does not automatically roll back for checked exceptions.

```java
@Transactional(rollbackFor = Exception.class)
public void process() throws Exception {
}
```

Use explicit rollback rules only when the business requirement needs them.

---

# Self-Invocation Problem

```java
@Service
public class OrderService {

    public void create() {
        saveOrder();
    }

    @Transactional
    public void saveOrder() {
    }
}
```

`saveOrder()` is called directly within the same object.

The proxy is bypassed, so the transaction may not be applied.

Common fixes:

* Move the transactional method to another bean
* Call through a proxied dependency
* Redesign the service boundary

---

# Propagation

Propagation defines how a method behaves when a transaction already exists.

## `REQUIRED` — Default

* Joins the current transaction
* Creates one if none exists

```java
@Transactional
```

Use for most service methods.

## `REQUIRES_NEW`

* Suspends the current transaction
* Creates a new independent transaction

Useful for operations such as audit logging that must commit separately.

## `SUPPORTS`

* Joins a transaction if one exists
* Runs without one otherwise

## `NOT_SUPPORTED`

* Suspends any existing transaction
* Runs non-transactionally

## `MANDATORY`

* Requires an existing transaction
* Throws an exception if none exists

## `NEVER`

* Must run without a transaction
* Throws an exception if one exists

## `NESTED`

* Uses a savepoint within the current transaction
* Support depends on the transaction manager and database

---

# Important `REQUIRES_NEW` Pitfall

```java
@Transactional
public void placeOrder() {
    auditService.writeAudit();
}
```

If `writeAudit()` uses `REQUIRES_NEW`, it needs another database connection while the outer transaction still holds one.

Under load, this can exhaust the connection pool.

---

# Isolation Levels

Isolation controls how concurrent transactions see each other's changes.

## `READ_UNCOMMITTED`

May allow dirty reads.

## `READ_COMMITTED`

Prevents dirty reads.

Common database default.

## `REPEATABLE_READ`

Repeated reads of the same row return the same result within the transaction.

## `SERIALIZABLE`

Strongest isolation.

Transactions behave almost as if executed sequentially, but concurrency is lower.

Example:

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

---

# Common Concurrency Anomalies

## Dirty Read

A transaction reads uncommitted data from another transaction.

## Non-Repeatable Read

The same row returns different values within one transaction.

## Phantom Read

A repeated query returns additional or missing rows because another transaction inserted or deleted matching data.

## Lost Update

Two transactions update the same row, and one overwrites the other's change.

Use optimistic or pessimistic locking where needed.

---

# Read-Only Transactions

```java
@Transactional(readOnly = true)
public Order getOrder(Long id) {
}
```

Benefits may include:

* Communicating intent
* Hibernate optimizations
* Avoiding unnecessary dirty checking in some scenarios
* Routing to read replicas in custom infrastructure

It is not a security guarantee and does not always prevent writes at the database level.

---

# Transaction Boundaries

Place transactions around business operations, usually in the service layer.

Good:

```java
@Transactional
public void placeOrder() {
    validateOrder();
    saveOrder();
    reserveInventory();
}
```

Avoid very large transactions that include:

* Long external API calls
* User interaction
* Large batch processing
* Slow file operations

Long transactions hold locks and connections for longer.

---

# Database and External Systems

A database transaction does not automatically include:

* Kafka
* REST calls
* Email
* Another microservice
* Another independent database

This does not provide distributed atomicity:

```java
@Transactional
public void createOrder() {
    orderRepository.save(order);
    kafkaTemplate.send("orders", event);
}
```

The database commit may succeed while the Kafka send fails, or vice versa.

Use patterns such as:

* Transactional outbox
* Saga
* Idempotent consumers
* Retry and compensation

---

# Common Interview Questions

### How does `@Transactional` work?

Spring creates an AOP proxy that opens, commits, or rolls back a transaction around the method call.

### Why does `@Transactional` sometimes not work?

Common reasons:

* Self-invocation
* Bean not managed by Spring
* Method not invoked through the proxy
* Wrong rollback assumptions
* Exception caught and not rethrown
* Transaction annotation placed on an unsuitable method

### Does Spring roll back checked exceptions?

Not by default.

### Where should `@Transactional` be placed?

Usually on service-layer methods representing a business use case.

### What is the difference between `REQUIRED` and `REQUIRES_NEW`?

`REQUIRED` joins the current transaction. `REQUIRES_NEW` suspends it and starts an independent one.

### Can one transaction cover two microservices?

Not through a normal local database transaction. Distributed workflows usually use Saga, outbox, and compensation patterns.

---

# Interview Scenario

```java
@Transactional
public void registerUser() {
    userRepository.save(user);
    emailClient.sendWelcomeEmail(user);
}
```

What is wrong?

The database transaction remains open while waiting on an external system.

Also, email sending cannot be rolled back if the database operation later fails.

A better design:

1. Save the user and an outbox event in one transaction.
2. Commit.
3. Publish the event asynchronously.
4. Send the email with retry and idempotency.

---

# Quick Revision

* `@Transactional` is implemented through AOP proxies.
* `REQUIRED` is the default propagation.
* Runtime exceptions trigger rollback by default.
* Self-invocation bypasses the proxy.
* Isolation controls concurrent visibility.
* Keep transaction boundaries short.
* Avoid external calls inside long database transactions.
* Local transactions do not solve distributed consistency.
* Use outbox or Saga for cross-service workflows.
