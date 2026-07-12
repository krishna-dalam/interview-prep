# Monolith vs Microservices

## Interview Priority

⭐⭐⭐⭐⭐

## What is a Monolith?

A monolithic application is deployed as a single unit.

Example:

```text
                Monolith

+-----------------------------------+

 Authentication

 Order

 Payment

 Inventory

 Notification

 Reporting

+-----------------------------------+
```

Everything is packaged into one application.

---

# Advantages

* Simple to develop
* Easy debugging
* Single deployment
* Easy local development
* Simple transactions
* Less infrastructure

---

# Disadvantages

* Large codebase
* Slow deployments
* Difficult scaling
* Tight coupling
* Large blast radius
* Technology lock-in

---

# What are Microservices?

Microservices split an application into independently deployable services.

```text
        API Gateway

             |

---------------------------------------

|        |         |         |

Order   Payment  Inventory Notification

```

Each service owns:

* Business logic
* Database
* Deployment
* Scaling

---

# Characteristics

Each microservice should:

* Have a single business capability
* Be independently deployable
* Own its data
* Communicate over APIs or events
* Be independently scalable

---

# Advantages

* Independent deployment
* Independent scaling
* Smaller codebases
* Team autonomy
* Technology flexibility
* Better fault isolation

---

# Disadvantages

* Operational complexity
* Distributed systems problems
* Network latency
* Distributed transactions
* Monitoring complexity
* Higher infrastructure cost

---

# Monolith vs Microservices

| Monolith            | Microservices             |
| ------------------- | ------------------------- |
| Single deployment   | Multiple deployments      |
| Single database     | Database per service      |
| Simple debugging    | Distributed debugging     |
| Easier transactions | Distributed transactions  |
| Simple testing      | Integration complexity    |
| Scale entire app    | Scale individual services |

---

# When Should You Choose a Monolith?

Good for:

* MVPs
* Small teams
* Simple business domains
* Early-stage startups
* Low operational maturity

A modular monolith is often an excellent starting point.

---

# When Should You Choose Microservices?

Good for:

* Large engineering teams
* Independent business domains
* High scale
* Frequent deployments
* Independent scaling requirements

---

# Database Per Service

Each service owns its data.

Example:

```text
Order Service

↓

Order Database

--------------------

Payment Service

↓

Payment Database
```

Services should not access another service's database directly.

Instead, communicate through APIs or events.

---

# Why Not Share a Database?

Problems include:

* Tight coupling
* Schema conflicts
* Difficult deployments
* Ownership confusion
* Reduced autonomy

A shared database often recreates monolithic coupling.

---

# Communication

Microservices communicate using:

Synchronous

* REST
* gRPC

Asynchronous

* Kafka
* RabbitMQ
* SQS
* SNS

Choose based on latency, coupling, and reliability requirements.

---

# Common Challenges

* Service discovery
* API Gateway
* Authentication
* Distributed tracing
* Configuration
* Retry
* Circuit breaker
* Observability
* Distributed transactions

These topics build on the fundamentals introduced here.

---

# Common Interview Questions

### Is a microservice just a small service?

No.

A microservice is organized around a business capability and can be developed, deployed, and scaled independently.

---

### Should every application use microservices?

No.

Microservices introduce operational complexity and are not justified for every application.

---

### Why is a database per service recommended?

It preserves service autonomy and reduces coupling between teams and deployments.

---

### Can two services share the same database?

Technically yes, but it is generally discouraged because it couples services and weakens independent evolution.

---

### What is the biggest challenge with microservices?

Managing distributed system concerns such as communication, failures, consistency, monitoring, and deployment.

---

# Real-World Example

An e-commerce platform may consist of:

```text
User Service

Order Service

Payment Service

Inventory Service

Notification Service
```

When order traffic increases during a sale:

* Scale the Order Service.
* Leave the User and Notification services unchanged.

With a monolith, the entire application would typically need to scale together.

---

# Interview Scenario

**Question:**

"We have a Spring Boot monolith with 50 developers. Would you immediately migrate to microservices?"

A strong answer:

> Not necessarily. I would first identify clear business boundaries, independent scaling needs, deployment bottlenecks, and team ownership issues. If those problems exist, I'd usually evolve toward a **modular monolith** first and then extract services incrementally using patterns like the Strangler Fig pattern. Moving directly to microservices without a compelling need often increases operational complexity without delivering proportional value.

This demonstrates architectural judgment instead of assuming microservices are always the right choice.

---

# Quick Revision

* Monolith = one deployable application.
* Microservices = independently deployable services.
* Each microservice owns its business capability and data.
* Avoid shared databases.
* Use APIs or events for communication.
* Microservices improve scalability and team autonomy.
* They also introduce distributed-system complexity.
* Start with a modular monolith unless there is a clear business need for microservices.
