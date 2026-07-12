# Microservice Communication

## Interview Priority

⭐⭐⭐⭐⭐

## Overview

Microservices communicate using either:

* Synchronous communication
* Asynchronous communication

The choice affects latency, coupling, reliability, scalability, and consistency.

---

# Synchronous Communication

The caller waits for the receiving service to respond.

```text
Order Service
      |
      | Request
      v
Payment Service
      |
      | Response
      v
Order Service
```

Common technologies:

* REST
* gRPC
* GraphQL

---

## REST

REST commonly uses HTTP and JSON.

```http
GET /customers/100
```

Advantages:

* Simple
* Widely supported
* Easy to debug
* Human-readable payloads
* Good external API support

Disadvantages:

* Larger JSON payloads
* Runtime contract validation
* Higher serialization overhead than binary protocols
* Caller and receiver are temporally coupled

Use REST for:

* Public APIs
* Browser-facing services
* Standard CRUD operations
* Integrations where simplicity matters

---

## gRPC

gRPC uses Protocol Buffers and commonly communicates over HTTP/2.

```text
Client
  |
  | Binary request
  v
gRPC Service
```

Advantages:

* Smaller binary payloads
* Strongly typed contracts
* Code generation
* Supports streaming
* Efficient internal communication

Disadvantages:

* Harder to inspect manually
* Browser support requires additional handling
* Contract changes require careful versioning
* Less convenient for public APIs

Use gRPC for:

* Internal service-to-service calls
* Low-latency communication
* High-throughput systems
* Streaming use cases

---

# Asynchronous Communication

The producer sends a message and continues without waiting for immediate processing.

```text
Order Service
      |
      | OrderCreated event
      v
Message Broker
      |
      +------> Inventory Service
      |
      +------> Notification Service
      |
      +------> Analytics Service
```

Common technologies:

* Kafka
* RabbitMQ
* AWS SQS
* AWS SNS
* EventBridge

---

## Advantages

* Loose temporal coupling
* Better fault isolation
* Supports buffering
* Easier independent scaling
* Suitable for long-running workflows
* Enables event-driven architecture

## Disadvantages

* Eventual consistency
* Harder debugging
* Duplicate messages
* Ordering challenges
* More complex error handling
* Requires observability and retry design

---

# Synchronous vs Asynchronous

| Synchronous                     | Asynchronous                      |
| ------------------------------- | --------------------------------- |
| Caller waits                    | Caller continues                  |
| Immediate response              | Eventual processing               |
| Simpler flow                    | More operational complexity       |
| Temporal coupling               | Looser coupling                   |
| Failure affects caller directly | Broker can buffer messages        |
| Suitable for queries            | Suitable for events and workflows |

---

# Commands vs Events

## Command

A command asks a specific service to perform an action.

```text
ReserveInventory
ProcessPayment
SendEmail
```

A command usually has one intended receiver.

## Event

An event states that something already happened.

```text
OrderCreated
PaymentCompleted
InventoryReserved
```

An event may have multiple consumers.

### Interview distinction

```text
Command: "Do this"
Event: "This happened"
```

---

# Request-Response vs Event-Driven Flow

## Request-Response

```text
Order Service
      |
      v
Payment Service
      |
      v
Inventory Service
```

Risks:

* Increased latency
* Cascading failures
* Longer call chains
* More complex timeout handling

## Event-Driven

```text
Order Service
      |
      v
OrderCreated
      |
      +--> Payment
      +--> Inventory
      +--> Notification
```

Benefits:

* Consumers operate independently
* New consumers can be added without changing the producer
* Failures can be retried asynchronously

---

# Timeouts

Every remote synchronous call should have a timeout.

Bad:

```text
Wait indefinitely
```

Good:

```text
Connect timeout: 1 second
Read timeout: 2 seconds
```

Without timeouts:

* Threads remain blocked
* Connection pools become exhausted
* Cascading failures spread across services

---

# Retries

Retries help with transient failures such as:

* Temporary network errors
* Brief service unavailability
* Rate limits
* Connection resets

Use:

* Limited attempts
* Exponential backoff
* Jitter
* Idempotent operations

Do not retry:

* Validation failures
* Authentication failures
* Permanent business errors
* Every timeout without limits

---

# Idempotency

An operation is idempotent when repeating it produces the same final effect.

Example:

```http
PUT /orders/100/status
```

Repeated requests should not create duplicate orders or payments.

For unsafe operations, use an idempotency key:

```http
Idempotency-Key: 45f8...
```

The receiving service stores the key and returns the original result for duplicate requests.

---

# Cascading Failure

Consider:

```text
API
 |
 v
Order Service
 |
 v
Payment Service
 |
 v
Fraud Service
```

If the Fraud Service becomes slow:

* Payment threads wait
* Order threads wait
* API requests time out
* The entire system degrades

Protection mechanisms:

* Timeouts
* Circuit breakers
* Bulkheads
* Retries with backoff
* Fallbacks
* Asynchronous processing

---

# Service Communication with Spring

## REST Client

Modern Spring applications may use:

* `RestClient`
* `WebClient`
* OpenFeign

Example:

```java
public PaymentResponse processPayment(
        PaymentRequest request) {

    return paymentClient.process(request);
}
```

Remote calls should include:

* Timeout configuration
* Error mapping
* Metrics
* Tracing
* Retry policy where appropriate

---

## Kafka Event

```java
kafkaTemplate.send(
    "order-created",
    orderId,
    event
);
```

Consumers must handle:

* Duplicate messages
* Retry
* Dead-letter topics
* Ordering
* Schema evolution
* Idempotency

---

# REST vs gRPC vs Messaging

| Requirement                 | Recommended     |
| --------------------------- | --------------- |
| Public external API         | REST            |
| Immediate response required | REST or gRPC    |
| Low latency internal calls  | gRPC            |
| Browser compatibility       | REST            |
| Long-running workflow       | Messaging       |
| Multiple consumers          | Events          |
| Temporary receiver downtime | Queue or broker |
| Streaming                   | gRPC or Kafka   |
| Loose coupling              | Messaging       |

---

# Avoid Chatty Communication

Bad design:

```text
Order Service calls Customer Service 10 times
Order Service calls Product Service 20 times
Order Service calls Pricing Service 20 times
```

This increases:

* Latency
* Failure probability
* Network traffic
* Coupling

Better options:

* Batch APIs
* API composition
* Local read models
* Cached reference data
* Events to replicate required data
* Redesign service boundaries

---

# Data Ownership

A service should not read another service's database directly.

Bad:

```text
Order Service
      |
      v
Payment Database
```

Good:

```text
Order Service
      |
      v
Payment API or Payment Events
```

Direct database access creates tight coupling and bypasses service rules.

---

# Common Interview Questions

### REST or Kafka—which should you choose?

REST is suitable when an immediate response is required.

Kafka is suitable for asynchronous workflows, multiple consumers, buffering, and loose coupling.

### When would you use gRPC?

For high-throughput, low-latency internal communication with strongly typed contracts.

### Why are synchronous call chains risky?

Each additional dependency increases latency and creates another possible failure point.

### Should every failure be retried?

No. Retry only transient failures and use bounded retries with backoff and jitter.

### Why must consumers be idempotent?

Message brokers can deliver duplicate messages, especially during retries or consumer restarts.

### Can asynchronous communication guarantee immediate consistency?

No. It usually introduces eventual consistency.

### Can services share a database?

They can technically, but it reduces autonomy and creates schema and deployment coupling.

---

# Interview Scenario

An Order Service calls:

1. Payment Service
2. Inventory Service
3. Notification Service

Should all calls be synchronous?

A balanced design:

* Payment may be synchronous when the user needs an immediate payment result.
* Inventory could be synchronous for immediate reservation or asynchronous depending on the business flow.
* Notification should usually be asynchronous because order completion should not fail when email delivery is unavailable.

The correct choice depends on business consistency and response-time requirements.

---

# Quick Revision

* Synchronous communication provides an immediate response but increases temporal coupling.
* Asynchronous communication improves resilience and scalability but introduces eventual consistency.
* REST is simple and suitable for external APIs.
* gRPC is efficient for strongly typed internal communication.
* Messaging suits workflows, buffering, and multiple consumers.
* Configure timeouts for every remote call.
* Retry only transient failures.
* Use backoff and jitter.
* Design APIs and consumers to be idempotent.
* Avoid long synchronous call chains and chatty services.
* Services should communicate through contracts, not shared databases.
