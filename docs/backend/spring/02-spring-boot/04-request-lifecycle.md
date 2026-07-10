# Spring Boot HTTP Request Lifecycle

## Interview Priority

⭐⭐⭐⭐⭐

## Overview

Every HTTP request in a Spring Boot application follows a well-defined lifecycle before a response is returned.

Understanding this flow is essential for debugging, performance tuning, security, and answering Spring Boot interview questions.

---

# High-Level Flow

```text
Client
   │
   ▼
Embedded Tomcat
   │
   ▼
Servlet Filter
   │
   ▼
DispatcherServlet
   │
   ▼
HandlerMapping
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
   │
   ▼
Service
   │
   ▼
Controller
   │
   ▼
HttpMessageConverter
   │
   ▼
JSON Response
```

---

# Step 1 – Client Sends Request

Example:

```http
GET /users/10 HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
```

The request reaches the embedded web server.

---

# Step 2 – Embedded Tomcat

Spring Boot starts an embedded Tomcat by default.

Tomcat:

* Accepts TCP connections.
* Parses the HTTP request.
* Creates `HttpServletRequest` and `HttpServletResponse`.
* Passes the request into the Servlet container.

Tomcat does **not** know anything about Spring MVC.

---

# Step 3 – Servlet Filters

Filters execute before the request reaches Spring MVC.

Typical responsibilities:

* Authentication
* CORS
* Request logging
* Compression
* Correlation IDs

Example:

```java
public class LoggingFilter extends OncePerRequestFilter {
    // doFilterInternal(...)
}
```

Filters operate at the **Servlet** level.

---

# Step 4 – DispatcherServlet

The `DispatcherServlet` is the **Front Controller** of Spring MVC.

Every request enters Spring MVC through this servlet.

Responsibilities:

* Route requests
* Invoke controllers
* Handle exceptions
* Convert request and response bodies

Think of it as the traffic controller for Spring MVC.

---

# Step 5 – HandlerMapping

The `DispatcherServlet` asks:

> "Which controller should handle this request?"

Example:

```java
@GetMapping("/users/{id}")
```

Spring matches:

```text
/users/10
```

to the appropriate controller method.

---

# Step 6 – HandlerInterceptor

Interceptors run after the handler is selected but before the controller executes.

Common use cases:

* Authorization
* Performance timing
* Audit logging
* Locale handling

Methods:

```java
preHandle()

postHandle()

afterCompletion()
```

Unlike filters, interceptors are Spring MVC components.

---

# Step 7 – Controller

Example:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {

    return userService.getUser(id);

}
```

Responsibilities:

* Validate input
* Delegate business logic
* Return a response

Controllers should remain thin.

---

# Step 8 – Service Layer

Business logic belongs here.

```java
@Service
public class UserService {

}
```

Typical responsibilities:

* Validation
* Transactions
* Domain logic
* External service calls

---

# Step 9 – Repository

Handles persistence.

Example:

```java
@Repository
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

The repository interacts with the database.

---

# Step 10 – Database

The SQL query executes.

The result is returned to the repository.

---

# Step 11 – Response Processing

The controller returns an object.

```java
return user;
```

Spring now converts the object into JSON.

---

# HttpMessageConverter

Spring uses `HttpMessageConverter`.

For JSON:

```text
Jackson ObjectMapper
```

Example:

```java
User
```

becomes

```json
{
  "id": 10,
  "name": "John"
}
```

---

# Response Returned

The JSON response is written back through Tomcat.

Tomcat sends:

```http
HTTP/1.1 200 OK
```

to the client.

---

# Exception Flow

If an exception occurs:

```text
Controller

↓

Exception

↓

@ControllerAdvice

↓

Error Response
```

Global exception handlers create consistent API responses.

---

# Where Spring Security Fits

When Spring Security is enabled:

```text
Client

↓

Security Filter Chain

↓

Servlet Filters

↓

DispatcherServlet
```

Security executes **before** the request reaches the controller.

---

# Filters vs Interceptors

| Filter                        | Interceptor                              |
| ----------------------------- | ---------------------------------------- |
| Servlet API                   | Spring MVC                               |
| Runs before DispatcherServlet | Runs after HandlerMapping                |
| Can modify request/response   | Works with controller execution          |
| Good for authentication, CORS | Good for logging, authorization, metrics |

---

# Common Interview Questions

### What is DispatcherServlet?

The front controller of Spring MVC that receives every request and coordinates request handling.

---

### What is HandlerMapping?

It maps an incoming request to the appropriate controller method.

---

### Who converts Java objects to JSON?

`HttpMessageConverter` using Jackson (by default).

---

### Where should business logic live?

In the Service layer.

Controllers should primarily orchestrate requests and responses.

---

### Where does Spring Security execute?

Inside the Security Filter Chain, before the request reaches Spring MVC.

---

# Real-World Example

A request arrives:

```text
GET /orders/100
```

Flow:

1. Tomcat receives the request.
2. Security validates the JWT.
3. Logging filter records the request.
4. DispatcherServlet receives control.
5. HandlerMapping selects `OrderController`.
6. Interceptor starts request timing.
7. Controller delegates to `OrderService`.
8. `OrderService` retrieves data through `OrderRepository`.
9. JPA queries the database.
10. The `Order` object is returned.
11. Jackson serializes it to JSON.
12. The response is sent to the client.

---

# Quick Revision

* Tomcat receives the HTTP request.
* Filters execute first.
* DispatcherServlet is the front controller.
* HandlerMapping finds the controller.
* Interceptors execute around controller invocation.
* Controllers delegate to services.
* Services contain business logic.
* Repositories access the database.
* Jackson converts Java objects to JSON.
* The response is returned through Tomcat.
