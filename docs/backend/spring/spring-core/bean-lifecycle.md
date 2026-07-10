# Spring Bean Lifecycle

## Interview Priority

⭐⭐⭐⭐⭐

## What is the Bean Lifecycle?

The Bean Lifecycle describes the sequence of steps a Spring-managed bean goes through from creation until destruction.

Understanding this helps explain how Spring initializes dependencies, executes custom setup logic, applies proxies (AOP), and performs cleanup.

---

# Bean Lifecycle Flow

```text
Bean Definition
      │
      ▼
Instantiate Bean
      │
      ▼
Populate Dependencies
      │
      ▼
Aware Interfaces
      │
      ▼
BeanPostProcessor (Before Initialization)
      │
      ▼
@PostConstruct
      │
      ▼
afterPropertiesSet()
      │
      ▼
Custom init-method
      │
      ▼
BeanPostProcessor (After Initialization)
      │
      ▼
Bean Ready for Use
      │
      ▼
Application Running
      │
      ▼
@PreDestroy
      │
      ▼
destroy()
      │
      ▼
Custom destroy-method
```

---

# 1. Bean Instantiation

Spring creates an instance of the bean.

```java
@Service
public class UserService {

}
```

At this stage, the object exists but its dependencies have not yet been injected.

---

# 2. Dependency Injection

Spring injects all required dependencies.

```java
@Service
public class UserService {

    private final EmailService emailService;

    public UserService(EmailService emailService){
        this.emailService = emailService;
    }
}
```

---

# 3. Aware Interfaces

If implemented, Spring provides framework-specific objects.

Examples:

* BeanNameAware
* BeanFactoryAware
* ApplicationContextAware

These are used less frequently in application code but are common in framework development.

---

# 4. BeanPostProcessor (Before Initialization)

Spring invokes registered `BeanPostProcessor`s before initialization.

Typical uses:

* Validation
* Custom configuration
* Framework extensions

---

# 5. @PostConstruct

Runs once after dependency injection is complete.

```java
@PostConstruct
public void init() {
    System.out.println("Initializing...");
}
```

Typical use cases:

* Load configuration
* Initialize caches
* Open connections
* Validate required settings

---

# 6. InitializingBean

Spring calls:

```java
afterPropertiesSet()
```

Example:

```java
public class UserService
        implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("Initialized");
    }

}
```

Most applications prefer `@PostConstruct` for initialization.

---

# 7. Custom init-method

Can be configured in Java configuration.

```java
@Bean(initMethod = "init")
public EmailService emailService() {
    return new EmailService();
}
```

---

# 8. BeanPostProcessor (After Initialization)

Spring invokes `BeanPostProcessor`s again.

This phase is important because Spring often creates proxies here.

Examples:

* AOP
* Transactions
* Security
* Logging

---

# 9. Bean Ready

The bean is now available for use.

Example:

```java
@Autowired
private UserService service;
```

---

# 10. Bean Destruction

When the application shuts down, Spring invokes destruction callbacks.

---

## @PreDestroy

```java
@PreDestroy
public void cleanup() {
    System.out.println("Cleanup");
}
```

Typical uses:

* Close files
* Close database connections
* Release resources
* Stop background threads

---

## DisposableBean

```java
public class UserService
        implements DisposableBean {

    @Override
    public void destroy() {
    }

}
```

---

## destroy-method

```java
@Bean(destroyMethod = "close")
public Client client() {
    return new Client();
}
```

---

# BeanPostProcessor

One of Spring's most powerful extension points.

It can modify beans before and after initialization.

Spring itself uses it for:

* AOP
* Transactions
* Security
* Dependency injection enhancements

---

# Initialization Methods

| Method               | Recommended |
| -------------------- | ----------- |
| @PostConstruct       | ✅ Yes       |
| afterPropertiesSet() | Sometimes   |
| init-method          | Rare        |

---

# Destruction Methods

| Method         | Recommended |
| -------------- | ----------- |
| @PreDestroy    | ✅ Yes       |
| destroy()      | Sometimes   |
| destroy-method | Rare        |

---

# Common Interview Questions

### What is the Bean Lifecycle?

The sequence from bean creation, dependency injection, initialization, usage, and destruction.

---

### When is @PostConstruct called?

After dependency injection is complete and before the bean is made available for use.

---

### When is @PreDestroy called?

Just before Spring destroys the bean during application shutdown.

---

### What is BeanPostProcessor?

A Spring extension point that allows custom processing before and after bean initialization. It is heavily used internally for features like AOP and transactions.

---

### Why is Bean Lifecycle important?

It allows applications and frameworks to initialize resources, create proxies, validate configuration, and release resources in a predictable way.

---

# Real-World Example

A Spring Boot service creates a Redis client.

During initialization:

* Configuration is injected.
* `@PostConstruct` validates connectivity.

During shutdown:

* `@PreDestroy` closes the Redis connection gracefully.

This prevents resource leaks and ensures clean application shutdown.

---

# Quick Revision

* Spring creates the bean.
* Dependencies are injected.
* `@PostConstruct` runs after injection.
* `BeanPostProcessor` enables framework features like AOP.
* Bean becomes available for use.
* `@PreDestroy` runs before shutdown.
* Prefer `@PostConstruct` and `@PreDestroy` for application initialization and cleanup.
