# Spring AOP (Aspect-Oriented Programming)

## Interview Priority

⭐⭐⭐⭐⭐

## What is AOP?

Aspect-Oriented Programming (AOP) is a programming paradigm that allows common functionality to be applied across multiple classes **without modifying their business logic**.

Examples of cross-cutting concerns:

* Logging
* Security
* Transactions
* Caching
* Auditing
* Performance monitoring
* Exception handling

---

# Why Do We Need AOP?

Without AOP:

```java
public void createOrder() {

    log.info("Started");

    checkAuthorization();

    startTransaction();

    // Business Logic

    commitTransaction();

    log.info("Completed");
}
```

Business logic becomes mixed with infrastructure code.

With AOP:

```java
public void createOrder() {

    // Business Logic

}
```

Logging, security, and transaction handling are added automatically.

---

# Core Concepts

## Aspect

A class that contains cross-cutting logic.

Example:

```java
@Aspect
@Component
public class LoggingAspect {

}
```

---

## Join Point

A point during program execution where an aspect can be applied.

In Spring AOP, join points are **method executions**.

---

## Advice

The action executed by an aspect.

Types:

* Before
* After
* After Returning
* After Throwing
* Around

---

## Pointcut

Defines **where** an advice should execute.

Example:

```java
execution(* com.example.service.*.*(..))
```

This matches all methods inside the `service` package.

---

# Types of Advice

## @Before

Runs before a method.

```java
@Before("execution(* com.example.service.*.*(..))")
```

Use cases:

* Logging
* Validation
* Authorization

---

## @After

Runs after a method finishes (whether successful or not).

---

## @AfterReturning

Runs only if the method completes successfully.

Common use cases:

* Audit logging
* Success metrics

---

## @AfterThrowing

Runs only when an exception is thrown.

Common use cases:

* Error logging
* Alerting

---

## @Around ⭐ Most Powerful

Wraps the method execution.

```java
@Around(...)
```

Can:

* Execute code before
* Execute code after
* Modify arguments
* Modify return value
* Prevent method execution

Often used for:

* Performance monitoring
* Transactions
* Caching

---

# How Spring AOP Works

Spring AOP uses **proxies**.

```text
Client
   |
   v
Proxy
   |
   +--> Logging
   +--> Security
   +--> Transaction
   |
   v
Actual Bean
```

The client interacts with the proxy, not the original bean.

The proxy applies aspects before delegating to the target bean.

---

# JDK Dynamic Proxy vs CGLIB

## JDK Dynamic Proxy

* Uses Java interfaces
* Default when the bean implements an interface

---

## CGLIB Proxy

* Creates a subclass at runtime
* Used when there is no interface

---

# Example

```java
@Service
public class PaymentService {

    public void processPayment() {

    }

}
```

Logging Aspect:

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void log() {
        System.out.println("Method Called");
    }

}
```

Execution:

```text
Client

↓

Logging Aspect

↓

PaymentService

↓

Return
```

---

# Real Spring Features Using AOP

The following annotations are implemented using proxies and AOP:

* @Transactional
* @Async
* @Cacheable
* @Secured
* @PreAuthorize
* Method-level validation

---

# Self Invocation Problem

A common interview question.

```java
@Service
public class UserService {

    public void methodA() {

        methodB();

    }

    @Transactional
    public void methodB() {

    }

}
```

`methodB()` is called directly, bypassing the Spring proxy.

Result:

`@Transactional` does **not** take effect.

Reason:

The proxy intercepts external calls only.

---

# Limitations of Spring AOP

* Works only with Spring-managed beans.
* Supports method execution join points only.
* Does not intercept direct internal method calls.
* Does not intercept constructors or fields.

---

# Common Interview Questions

### What is AOP?

A programming technique that separates cross-cutting concerns from business logic.

---

### Why use AOP?

To avoid duplicating infrastructure code such as logging, security, and transactions across multiple classes.

---

### How does Spring implement AOP?

Using runtime proxies (JDK Dynamic Proxy or CGLIB).

---

### When is JDK Dynamic Proxy used?

When the bean implements at least one interface.

---

### When is CGLIB used?

When the bean does not implement an interface.

---

### Why does @Transactional sometimes not work?

A common reason is self-invocation, where a method inside the same class calls another annotated method directly, bypassing the proxy.

---

# Real-World Example

An Order Service uses:

* `@Transactional` for database consistency.
* `@PreAuthorize` for authorization.
* `@Cacheable` for product lookups.
* A logging aspect for request tracing.

All of these concerns are applied by Spring through proxies, allowing the service class to focus solely on business logic.

---

# Quick Revision

* AOP separates cross-cutting concerns.
* Aspect = class containing shared logic.
* Advice = action executed by the aspect.
* Pointcut = where the advice runs.
* Join Point = method execution in Spring AOP.
* Spring AOP works using runtime proxies.
* JDK Proxy → interfaces.
* CGLIB → classes.
* `@Transactional`, `@Async`, and `@Cacheable` rely on AOP.
* Self-invocation bypasses the proxy and prevents AOP advice from running.
