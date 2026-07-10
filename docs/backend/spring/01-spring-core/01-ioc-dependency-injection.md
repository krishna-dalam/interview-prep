# Spring IoC & Dependency Injection

## Interview Priority

⭐⭐⭐⭐⭐

## What is Spring?

Spring is a Java framework that simplifies enterprise application development by providing dependency management, configuration, transaction support, security, data access, and many other infrastructure features.

At its core is the **IoC (Inversion of Control) Container**.

---

# What is Inversion of Control (IoC)?

Traditionally, an object creates the dependencies it needs.

```java
OrderService service = new OrderService(new EmailService());
```

The class is responsible for creating its own dependencies.

With IoC:

```text
Application
      |
      v
Spring Container
      |
      +--> Creates Objects
      |
      +--> Manages Objects
      |
      +--> Injects Dependencies
```

The application no longer creates objects directly.

The Spring container manages object creation and wiring.

---

# Why IoC?

Without IoC

* Tight coupling
* Difficult testing
* Difficult dependency management
* Hard to replace implementations

With IoC

* Loose coupling
* Easier unit testing
* Better maintainability
* Easier configuration
* Cleaner architecture

---

# What is Dependency Injection (DI)?

Dependency Injection is the mechanism used by Spring to provide required dependencies to an object.

Instead of:

```java
public class OrderService {

    private EmailService emailService = new EmailService();

}
```

Spring injects it.

```java
public class OrderService {

    private final EmailService emailService;

    public OrderService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

---

# Types of Dependency Injection

## Constructor Injection ⭐ Recommended

```java
@Service
public class OrderService {

    private final EmailService emailService;

    public OrderService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

Advantages

* Immutable dependencies
* Easy testing
* Dependencies are mandatory
* Recommended by Spring

---

## Setter Injection

```java
@Service
public class OrderService {

    private EmailService emailService;

    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

Used for optional dependencies.

---

## Field Injection

```java
@Autowired
private EmailService emailService;
```

Not recommended because:

* Difficult to test
* Hidden dependencies
* Reflection-based injection
* Cannot create immutable objects

---

# Spring IoC Container

The IoC container is responsible for:

* Creating beans
* Managing bean lifecycle
* Injecting dependencies
* Managing scopes
* Destroying beans

Two main implementations:

* BeanFactory
* ApplicationContext

---

# BeanFactory vs ApplicationContext

| BeanFactory         | ApplicationContext                          |
| ------------------- | ------------------------------------------- |
| Basic container     | Advanced container                          |
| Lazy initialization | Eager singleton initialization (by default) |
| Minimal features    | Events, AOP, Environment, i18n, etc.        |

Most Spring Boot applications use `ApplicationContext`.

---

# What is a Bean?

A bean is simply an object managed by the Spring container.

Example

```java
@Service
public class OrderService {
}
```

Spring creates and manages this object.

---

# How are Beans Created?

Common annotations:

```java
@Component
@Service
@Repository
@Controller
@RestController
@Configuration
```

All of these ultimately register Spring-managed beans.

---

# Bean Wiring

Example

```java
@Service
class PaymentService {
}

@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService){
        this.paymentService = paymentService;
    }

}
```

Spring automatically injects `PaymentService` into `OrderService`.

---

# Advantages of Constructor Injection

* Prevents NullPointerException due to missing dependencies
* Easier unit testing
* Supports immutable classes
* Makes dependencies explicit

This is the preferred approach in modern Spring Boot applications.

---

# Common Interview Questions

### What is IoC?

A design principle where object creation and dependency management are delegated to the Spring container instead of application code.

---

### What is Dependency Injection?

A technique where required dependencies are supplied by the container rather than being instantiated manually.

---

### Constructor Injection vs Field Injection?

Constructor injection is preferred because it:

* Makes dependencies explicit
* Enables immutability
* Simplifies testing
* Avoids reflection-based injection

---

### Why is Spring loosely coupled?

Because components depend on abstractions and the IoC container manages their concrete implementations.

---

### What is a Bean?

An object whose lifecycle is managed by the Spring IoC container.

---

# Real-World Example

Consider an e-commerce application:

```
OrderService
        |
        +--> PaymentService
        |
        +--> InventoryService
        |
        +--> NotificationService
```

`OrderService` does not instantiate these services itself.

The Spring container creates each bean, resolves dependencies, and injects them automatically, making the application modular and easier to test.

---

# Quick Revision

* IoC delegates object creation to Spring.
* DI is how Spring supplies dependencies.
* A bean is a Spring-managed object.
* `ApplicationContext` is the primary container in Spring Boot.
* Prefer constructor injection over field or setter injection.
* Use stereotypes like `@Service`, `@Repository`, and `@Controller` to register beans.
