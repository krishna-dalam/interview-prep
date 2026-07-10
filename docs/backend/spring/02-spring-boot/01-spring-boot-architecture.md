# Spring Boot Architecture

## Interview Priority

⭐⭐⭐⭐⭐

## What is Spring Boot?

Spring Boot is an opinionated framework built on top of Spring that simplifies application development through:

* Auto Configuration
* Embedded Web Servers
* Starter Dependencies
* Production-ready features
* Externalized Configuration

Its goal is to help developers build production-ready applications with minimal configuration.

---

# Spring vs Spring Boot

| Spring                          | Spring Boot          |
| ------------------------------- | -------------------- |
| Manual configuration            | Auto Configuration   |
| External Tomcat                 | Embedded Tomcat      |
| XML-heavy (historically)        | Annotation-based     |
| Dependency management is manual | Starter dependencies |
| More setup                      | Faster development   |

---

# Spring Boot Startup Flow

```text
main()

↓

SpringApplication.run()

↓

Create ApplicationContext

↓

Load Configuration

↓

Component Scan

↓

Create Beans

↓

Apply Auto Configuration

↓

Start Embedded Server

↓

Application Ready
```

---

# Entry Point

Every Spring Boot application starts with:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

The important method is:

```java
SpringApplication.run(...)
```

This bootstraps the entire application.

---

# What Happens Inside SpringApplication.run()?

High-level steps:

1. Create a `SpringApplication`.
2. Detect application type (Servlet, Reactive, or Non-Web).
3. Create the appropriate `ApplicationContext`.
4. Load configuration files.
5. Perform component scanning.
6. Register beans.
7. Apply auto-configuration.
8. Execute startup callbacks.
9. Start the embedded web server.
10. Mark the application as ready.

---

# ApplicationContext

The `ApplicationContext` is the central Spring container.

Responsibilities:

* Create beans
* Inject dependencies
* Manage bean lifecycle
* Publish events
* Load configuration

In a typical web application, Spring Boot creates an `AnnotationConfigServletWebServerApplicationContext`.

---

# Embedded Web Server

Spring Boot packages a web server inside the application.

Supported servers include:

* Tomcat (default)
* Jetty
* Undertow

This means applications can be started with:

```bash
java -jar app.jar
```

No external application server is required.

---

# Component Scanning

Spring scans the package containing the main application class and its sub-packages for components such as:

* `@Component`
* `@Service`
* `@Repository`
* `@Controller`
* `@RestController`
* `@Configuration`

Matching classes are registered as beans.

---

# Bean Creation

After scanning, Spring:

* Instantiates beans
* Resolves dependencies
* Runs initialization callbacks
* Applies AOP proxies
* Stores beans in the `ApplicationContext`

---

# Auto Configuration

One of Spring Boot's biggest features.

Instead of configuring common infrastructure manually, Spring Boot configures it automatically based on:

* Dependencies on the classpath
* Configuration properties
* Existing beans

Examples:

* DataSource
* Jackson
* DispatcherServlet
* Tomcat
* Spring MVC

Auto-configuration backs off if you define your own bean of the same type.

---

# Starter Dependencies

Example:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This starter pulls in a curated set of dependencies such as:

* Spring MVC
* Jackson
* Validation
* Embedded Tomcat
* Logging

without requiring you to manage each one individually.

---

# External Configuration

Spring Boot supports configuration through:

* `application.properties`
* `application.yml`
* Environment variables
* Command-line arguments

This allows different settings for development, testing, and production.

---

# Production Features

Spring Boot includes:

* Actuator
* Health checks
* Metrics
* Logging
* Graceful shutdown support
* Externalized configuration

These features make applications easier to operate in production.

---

# Common Interview Questions

### What is Spring Boot?

A framework built on Spring that simplifies configuration and provides production-ready features.

---

### What happens when `SpringApplication.run()` is called?

It creates the application context, scans components, creates beans, applies auto-configuration, starts the embedded server, and makes the application ready to serve requests.

---

### Why doesn't Spring Boot require Tomcat installation?

Because it packages an embedded server inside the application.

---

### What is the role of `ApplicationContext`?

It manages beans, dependency injection, lifecycle, configuration, and application infrastructure.

---

### What is the biggest advantage of Spring Boot?

Convention over configuration. It reduces boilerplate while remaining extensible.

---

# Real-World Example

A Spring Boot REST API includes `spring-boot-starter-web` and `spring-boot-starter-data-jpa`.

When the application starts:

* Tomcat is started automatically.
* `DispatcherServlet` is configured.
* Jackson is configured for JSON.
* A `DataSource` is created from configuration.
* Controllers, services, and repositories are discovered and registered as beans.

Developers focus on business logic rather than infrastructure setup.

---

# Quick Revision

* Spring Boot builds on Spring.
* `SpringApplication.run()` bootstraps the application.
* `ApplicationContext` manages beans.
* Component scanning discovers annotated classes.
* Auto-configuration configures common infrastructure automatically.
* Embedded Tomcat removes the need for an external server.
* Starter dependencies provide curated libraries.
* External configuration supports multiple environments.
