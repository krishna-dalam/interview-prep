# Spring Boot Auto Configuration

## Interview Priority

⭐⭐⭐⭐⭐

## What is Auto Configuration?

Auto Configuration is Spring Boot's mechanism for automatically configuring Spring beans based on:

* Dependencies on the classpath
* Existing beans
* Configuration properties
* Application type

It follows the principle:

> **Convention over Configuration**

Instead of manually configuring common infrastructure, Spring Boot configures it automatically.

---

# Example

Dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Without writing any configuration, Spring Boot automatically creates:

* Embedded Tomcat
* DispatcherServlet
* Jackson ObjectMapper
* RequestMappingHandlerMapping
* MessageConverters
* ErrorController

Your application is immediately ready to serve REST APIs.

---

# How Auto Configuration Works

```text
Application Starts
        │
        ▼
@SpringBootApplication
        │
        ▼
@EnableAutoConfiguration
        │
        ▼
Load Auto Configuration Classes
        │
        ▼
Evaluate Conditions
        │
        ▼
Register Beans
        │
        ▼
Application Ready
```

---

# @EnableAutoConfiguration

This annotation enables Spring Boot's auto-configuration mechanism.

It instructs Spring Boot to load eligible auto-configuration classes.

```java
@SpringBootApplication
```

internally includes:

```java
@EnableAutoConfiguration
```

---

# Auto Configuration Classes

Examples include:

* WebMvcAutoConfiguration
* JacksonAutoConfiguration
* DataSourceAutoConfiguration
* SecurityAutoConfiguration
* RedisAutoConfiguration
* CacheAutoConfiguration

Each class configures a specific feature.

---

# Conditional Configuration

Auto configuration is **conditional**.

Beans are only created when certain conditions are satisfied.

Example:

```java
@ConditionalOnClass
```

Create a bean only if a class exists on the classpath.

---

## Common Conditional Annotations

### @ConditionalOnClass

```java
@ConditionalOnClass(DataSource.class)
```

Only configure if `DataSource` is available.

---

### @ConditionalOnMissingBean

```java
@ConditionalOnMissingBean
```

Create the bean only if the application has not already defined one.

This allows developers to override Boot defaults.

---

### @ConditionalOnProperty

```java
@ConditionalOnProperty(
    name = "cache.enabled",
    havingValue = "true"
)
```

Enable configuration based on application properties.

---

### @ConditionalOnBean

Configure only if another bean already exists.

---

### @ConditionalOnWebApplication

Only configure for web applications.

---

# Example

Suppose your project contains:

```xml
spring-boot-starter-data-jpa
```

Spring Boot automatically configures:

* EntityManagerFactory
* TransactionManager
* Hibernate integration
* DataSource (if configuration is available)

No XML or manual bean definitions are required.

---

# Back-Off Mechanism

Spring Boot never forces its own bean if you provide one.

Example:

```java
@Bean
public ObjectMapper objectMapper() {

    return new ObjectMapper();

}
```

Spring Boot detects your bean and skips creating its default `ObjectMapper`.

This is called the **back-off mechanism**.

---

# Spring Boot 2 vs Spring Boot 3

### Spring Boot 2

Auto-configuration classes were listed in:

```text
META-INF/spring.factories
```

---

### Spring Boot 3

Auto-configuration classes are listed in:

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

This change improves startup performance and simplifies configuration discovery.

---

# Example Startup

Dependency:

```text
spring-boot-starter-web
```

Spring Boot detects:

* Spring MVC
* Servlet API
* Embedded Tomcat

Result:

```text
Tomcat
DispatcherServlet
Jackson
MVC Configuration
```

are automatically configured.

---

# Why Auto Configuration Matters

Without Spring Boot:

* Create DispatcherServlet manually
* Configure Jackson
* Configure Tomcat
* Configure MVC
* Configure message converters
* Configure exception resolvers

With Spring Boot:

```java
@SpringBootApplication
```

Everything is configured automatically.

---

# Common Interview Questions

### What is Auto Configuration?

A feature that automatically configures Spring beans based on the application's dependencies, configuration, and environment.

---

### How does Spring Boot know which beans to create?

It evaluates auto-configuration classes using conditional annotations and registers only the beans whose conditions are satisfied.

---

### What is @ConditionalOnMissingBean?

It tells Spring Boot to create a bean only if the application has not already defined one.

---

### How can you override an auto-configured bean?

Define your own bean of the same type in the application context.

Spring Boot's default bean backs off.

---

### Why doesn't Boot configure JPA in every application?

Because the JPA auto-configuration is conditional.

If the required dependencies or configuration are missing, it is skipped.

---

### Where are auto-configuration classes stored?

* Spring Boot 2 → `spring.factories`
* Spring Boot 3 → `AutoConfiguration.imports`

---

# Real-World Example

An application includes:

* `spring-boot-starter-web`
* `spring-boot-starter-data-jpa`
* `spring-boot-starter-security`

During startup, Spring Boot automatically configures:

* Embedded Tomcat
* Spring MVC
* Jackson
* DataSource
* Hibernate
* TransactionManager
* Spring Security filter chain

The developer focuses on business logic rather than infrastructure setup.

---

# Quick Revision

* Auto Configuration reduces manual setup.
* Enabled by `@EnableAutoConfiguration`.
* Based on classpath, properties, and existing beans.
* Uses conditional annotations.
* Supports a back-off mechanism for customization.
* Spring Boot 3 uses `AutoConfiguration.imports`.
* Starter dependencies trigger relevant auto-configuration.
