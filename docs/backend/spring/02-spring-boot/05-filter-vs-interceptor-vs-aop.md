# Filter vs Interceptor vs AOP

## Interview Priority

⭐⭐⭐⭐⭐

## Overview

Filters, Interceptors, and AOP all allow you to execute logic outside your business code, but they operate at different layers of the application.

Choosing the correct one is a common interview question.

---

# Request Flow

```text
HTTP Request
      │
      ▼
Embedded Tomcat
      │
      ▼
Filter
      │
      ▼
DispatcherServlet
      │
      ▼
HandlerInterceptor
      │
      ▼
Controller
      │
      ▼
Service
      │
      ▼
Repository
      │
      ▼
Database
```

AOP is different.

It wraps Spring-managed bean methods.

```text
Controller
      │
      ▼
AOP Proxy
      │
      ▼
Service Method
```

---

# Filter

Filters belong to the **Servlet API**.

They execute before Spring MVC begins processing the request.

Example:

```java
@Component
public class LoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain chain)
            throws ServletException, IOException {

        chain.doFilter(request, response);

    }
}
```

Typical responsibilities:

* Authentication
* CORS
* Request logging
* Compression
* Correlation IDs
* Rate limiting

Filters can modify both the request and the response.

---

# Interceptor

Interceptors belong to **Spring MVC**.

They execute after the controller has been selected but before the controller method runs.

Methods:

```java
preHandle()

postHandle()

afterCompletion()
```

Typical responsibilities:

* Authorization
* Audit logging
* Performance timing
* Locale handling
* Request metrics

---

# AOP

AOP works on **Spring beans**, not HTTP requests.

It intercepts method execution using proxies.

Example:

```java
@Around(
"execution(* com.example.service.*.*(..))")
```

Typical responsibilities:

* Transactions
* Performance monitoring
* Logging
* Security
* Caching
* Retry logic

---

# Comparison

| Feature                        | Filter  | Interceptor | AOP         |
| ------------------------------ | ------- | ----------- | ----------- |
| Layer                          | Servlet | Spring MVC  | Spring Bean |
| Works before DispatcherServlet | ✅       | ❌           | ❌           |
| Works with Controllers         | ❌       | ✅           | ✅           |
| Works with Services            | ❌       | ❌           | ✅           |
| Can modify HTTP request        | ✅       | Limited     | ❌           |
| Can modify HTTP response       | ✅       | Limited     | ❌           |
| HTTP-specific                  | ✅       | ✅           | ❌           |
| Method-level interception      | ❌       | ❌           | ✅           |

---

# When to Use Each

## Use Filter

When working with raw HTTP requests.

Examples:

* JWT extraction
* CORS
* Compression
* Correlation ID generation
* Request/Response logging

---

## Use Interceptor

When logic depends on Spring MVC.

Examples:

* User authorization
* API metrics
* Audit logging
* Locale selection

---

## Use AOP

When applying logic to business methods.

Examples:

* Transactions
* Logging service execution
* Performance measurement
* Retry mechanisms
* Caching

---

# Example: Logging

## Filter

Logs every incoming HTTP request.

```text
GET /orders
```

---

## Interceptor

Logs the selected controller.

```text
OrderController.getOrders()
```

---

## AOP

Logs business method execution.

```text
OrderService.createOrder()
```

---

# Example: Authentication

Best choice:

✅ Filter

Reason:

Authentication should happen before Spring MVC processes the request.

---

# Example: Authorization

Best choice:

✅ Interceptor

Reason:

The controller and handler information are available, allowing decisions based on endpoints or annotations.

---

# Example: Transactions

Best choice:

✅ AOP

Reason:

Transactions are business concerns applied around service methods.

---

# Common Interview Questions

### Which executes first?

```text
Filter

↓

DispatcherServlet

↓

Interceptor

↓

Controller

↓

AOP

↓

Service
```

---

### Can AOP intercept controllers?

Yes, because controllers are Spring-managed beans.

However, AOP is more commonly applied to service-layer methods.

---

### Can Filters access Spring beans?

Yes, if they are managed by Spring (for example, registered as components or through Spring Boot configuration).

---

### Why is JWT validation usually implemented in a Filter?

Authentication should occur before request mapping and controller execution.

---

### Why isn't @Transactional implemented with a Filter?

Transactions apply to business methods, not HTTP requests.

Spring implements `@Transactional` using AOP proxies around service-layer methods.

---

# Real-World Example

An e-commerce application receives:

```text
POST /orders
```

Execution flow:

1. **Filter** extracts and validates the JWT.
2. **Filter** generates a correlation ID.
3. **DispatcherServlet** receives the request.
4. **Interceptor** verifies the user has permission to create orders.
5. **Controller** validates input and delegates.
6. **AOP** starts a transaction.
7. **Service** creates the order.
8. **Repository** saves the order.
9. **AOP** commits the transaction.
10. Response is returned.

Each layer has a distinct responsibility.

---

# Quick Revision

* **Filter** → Servlet layer, HTTP concerns.
* **Interceptor** → Spring MVC layer, controller concerns.
* **AOP** → Spring bean layer, business concerns.
* JWT authentication → Filter.
* Authorization → Interceptor.
* Transactions → AOP.
* Logging can be implemented at any layer depending on what you need to observe.
