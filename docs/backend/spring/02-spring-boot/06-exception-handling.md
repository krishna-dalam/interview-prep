# Exception Handling in Spring Boot

## Interview Priority

⭐⭐⭐⭐⭐

## Why Do We Need Exception Handling?

Without centralized exception handling, every controller would need repetitive try-catch blocks.

Bad example:

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {

    try {
        return userService.getUser(id);
    } catch (Exception e) {
        throw new ResponseStatusException(
            HttpStatus.INTERNAL_SERVER_ERROR
        );
    }

}
```

This leads to:

* Duplicate code
* Inconsistent responses
* Difficult maintenance

Spring Boot provides centralized exception handling.

---

# Exception Flow

```text
Client
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
 Exception
   │
   ▼
@ControllerAdvice
   │
   ▼
Standard Error Response
```

---

# @ExceptionHandler

Handles specific exceptions.

Example:

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<String> handle(
        UserNotFoundException ex) {

    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ex.getMessage());
}
```

Useful for handling exceptions in a single controller.

---

# @ControllerAdvice

Provides **global** exception handling across all controllers.

```java
@ControllerAdvice
public class GlobalExceptionHandler {

}
```

This is the recommended approach for REST APIs.

---

# @RestControllerAdvice

Equivalent to:

```text
@ControllerAdvice
+
@ResponseBody
```

Designed specifically for REST applications.

Usually preferred in Spring Boot REST APIs.

---

# Common Exceptions

### Resource Not Found

```java
throw new UserNotFoundException();
```

Response:

```http
404 Not Found
```

---

### Validation Failure

```http
400 Bad Request
```

---

### Unauthorized

```http
401 Unauthorized
```

---

### Forbidden

```http
403 Forbidden
```

---

### Unexpected Error

```http
500 Internal Server Error
```

---

# Designing Error Responses

Instead of returning:

```json
{
  "error": "Something went wrong"
}
```

Prefer a consistent structure.

Example:

```json
{
  "timestamp": "2026-07-10T12:00:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "User not found",
  "path": "/users/10"
}
```

Benefits:

* Easy to debug
* Consistent across APIs
* Better client integration

---

# Validation Errors

Example:

```java
@PostMapping
public void createUser(
        @Valid @RequestBody UserRequest request) {

}
```

Invalid input throws:

```text
MethodArgumentNotValidException
```

Global handler:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
```

Return validation messages instead of stack traces.

---

# Custom Exceptions

```java
public class UserNotFoundException
        extends RuntimeException {

    public UserNotFoundException(Long id) {
        super("User not found: " + id);
    }

}
```

Business logic:

```java
throw new UserNotFoundException(id);
```

The controller remains clean.

---

# ResponseEntity

Allows full control over:

* Status code
* Headers
* Body

Example:

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(user);
```

---

# What Should NOT Be Returned?

Never expose:

* Stack traces
* SQL errors
* Database details
* Internal class names
* Sensitive configuration

Return user-friendly messages while logging detailed errors internally.

---

# Logging Exceptions

Good practice:

```text
Client

↓

Error Response

↓

Application Logs

↓

Monitoring System
```

Log the exception with enough context (request ID, user ID, etc.) but avoid leaking internal details to API consumers.

---

# Common Interview Questions

### What is @ControllerAdvice?

A global mechanism for handling exceptions across multiple controllers.

---

### Difference between @ExceptionHandler and @ControllerAdvice?

* `@ExceptionHandler` handles exceptions for a specific controller.
* `@ControllerAdvice` applies globally.

---

### Why use custom exceptions?

They improve readability, separate business errors from infrastructure errors, and make global handling simpler.

---

### Should every exception return HTTP 500?

No.

Return the status code that best represents the error:

* 400 → Invalid request
* 401 → Authentication required
* 403 → Permission denied
* 404 → Resource not found
* 409 → Conflict
* 500 → Unexpected server error

---

### Should stack traces be returned to clients?

No.

Return a sanitized error response and log the full exception internally.

---

# Real-World Example

Request:

```http
GET /users/100
```

Flow:

1. Controller calls `UserService`.
2. User is not found.
3. `UserNotFoundException` is thrown.
4. `@RestControllerAdvice` catches the exception.
5. A structured 404 response is returned.
6. The exception is logged with the request ID for troubleshooting.

The client receives a clean, consistent error while developers retain full diagnostic information.

---

# Quick Revision

* Use `@RestControllerAdvice` for global REST exception handling.
* Use `@ExceptionHandler` to map exceptions to HTTP responses.
* Create custom exceptions for business errors.
* Return structured error responses.
* Log detailed exceptions internally.
* Never expose stack traces or implementation details to clients.
* Use appropriate HTTP status codes instead of always returning 500.
