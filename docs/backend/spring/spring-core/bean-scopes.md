# Spring Bean Scopes

## Interview Priority

⭐⭐⭐⭐⭐

## What is Bean Scope?

Bean scope defines **how many instances** of a bean Spring creates and **how long those instances live**.

Changing the scope changes the bean's lifecycle and sharing behavior.

---

# Available Bean Scopes

| Scope       | Description                        |
| ----------- | ---------------------------------- |
| Singleton   | One instance per Spring container  |
| Prototype   | New instance every time requested  |
| Request     | One instance per HTTP request      |
| Session     | One instance per HTTP session      |
| Application | One instance per ServletContext    |
| WebSocket   | One instance per WebSocket session |

The first two are available in all Spring applications.

The remaining scopes are available only in web applications.

---

# Singleton Scope (Default)

```java
@Service
public class UserService {
}
```

Equivalent to:

```java
@Service
@Scope("singleton")
public class UserService {
}
```

Spring creates only **one instance** of the bean.

```text
Application

        UserService
            ▲
   ┌────────┼────────┐
   │        │        │
Controller  API   Scheduler
```

Every component receives the same instance.

---

## Why Singleton?

Advantages:

* Low memory usage
* Faster startup
* Easy dependency management
* Ideal for stateless services

Most Spring Boot services use singleton scope.

---

# Prototype Scope

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

Every request for the bean creates a new instance.

```text
getBean()
   ↓
ReportGenerator #1

getBean()
   ↓
ReportGenerator #2

getBean()
   ↓
ReportGenerator #3
```

Spring creates the object but does **not** manage its full lifecycle after handing it over.

---

## When to Use Prototype?

Useful for:

* Temporary objects
* Builders
* Report generation
* Stateful processing objects

Rare in enterprise applications.

---

# Request Scope

```java
@Component
@RequestScope
public class RequestContext {
}
```

One bean instance per HTTP request.

```text
Request 1
   ↓
RequestContext #1

Request 2
   ↓
RequestContext #2
```

Common use cases:

* Request metadata
* Correlation IDs
* Current user information

---

# Session Scope

```java
@Component
@SessionScope
public class ShoppingCart {
}
```

One instance per user session.

```text
User A
  ↓
ShoppingCart A

User B
  ↓
ShoppingCart B
```

Useful for stateful web applications.

Less common in stateless REST APIs.

---

# Application Scope

```java
@ApplicationScope
```

One bean shared across the entire web application.

Typically used for shared application-wide configuration or metadata.

---

# WebSocket Scope

```java
@Scope("websocket")
```

One instance per WebSocket connection.

Used in chat applications and real-time systems.

---

# Singleton vs Prototype

| Singleton           | Prototype                 |
| ------------------- | ------------------------- |
| One instance        | New instance each request |
| Shared              | Independent               |
| Better memory usage | Higher memory usage       |
| Default             | Must be configured        |

---

# Thread Safety

A singleton bean is shared by multiple threads.

This means:

```text
Thread 1
     \
      \
Singleton Bean
      /
     /
Thread 2
```

If the bean contains mutable shared state, race conditions can occur.

Example (unsafe):

```java
@Service
public class CounterService {

    private int count = 0;

    public void increment() {
        count++;
    }
}
```

Multiple requests calling `increment()` concurrently can produce incorrect results.

---

## Best Practice

Keep singleton beans **stateless**.

Good example:

```java
@Service
public class EmailService {

    public void sendEmail(String email) {
        // send email
    }

}
```

No mutable instance variables are shared between requests.

---

# Prototype Injection into Singleton

Problem:

```java
@Service
public class OrderService {

    private final ReportGenerator generator;

    public OrderService(ReportGenerator generator) {
        this.generator = generator;
    }

}
```

Even if `ReportGenerator` is a prototype bean, constructor injection gives the singleton only one instance at startup.

To obtain a fresh prototype each time, use `ObjectProvider`, `ObjectFactory`, or method injection.

---

# Common Interview Questions

### What is the default bean scope?

Singleton.

---

### Why are most Spring beans singleton?

They are lightweight, efficient, and well suited for stateless business services.

---

### Are singleton beans thread-safe?

Not automatically.

They are safe only if they do not maintain mutable shared state.

---

### When would you use prototype scope?

When a new independent object is required for each use, such as builders or temporary processing objects.

---

### Why are REST services typically singleton?

REST APIs are designed to be stateless, making singleton beans efficient and scalable.

---

# Real-World Example

In a Spring Boot e-commerce application:

* `OrderService` → Singleton
* `PaymentService` → Singleton
* `InventoryService` → Singleton
* `ShoppingCart` (traditional web app) → Session
* `RequestContext` → Request

Business services remain stateless, while user-specific state is isolated to request or session scopes.

---

# Quick Revision

* Singleton is the default scope.
* Prototype creates a new instance for each lookup.
* Request scope is one instance per HTTP request.
* Session scope is one instance per user session.
* Singleton beans are shared across threads.
* Prefer stateless singleton services.
* Injecting a prototype into a singleton requires special handling to get a new instance each time.
