# @SpringBootApplication & Component Scanning

## Interview Priority

⭐⭐⭐⭐⭐

## What is @SpringBootApplication?

`@SpringBootApplication` is a convenience annotation that combines three commonly used Spring annotations into one.

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

Equivalent to:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

---

# 1. @Configuration

Marks the class as a source of bean definitions.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public EmailService emailService() {
        return new EmailService();
    }

}
```

Spring registers the returned object as a bean.

---

# 2. @EnableAutoConfiguration

Enables Spring Boot's auto-configuration mechanism.

Instead of manually configuring:

* Tomcat
* DispatcherServlet
* Jackson
* DataSource
* Validation

Spring Boot configures them automatically.

Example:

Add dependency:

```xml
spring-boot-starter-web
```

Spring Boot automatically configures:

* Embedded Tomcat
* Spring MVC
* JSON serialization
* DispatcherServlet

No additional configuration is required.

---

# 3. @ComponentScan

Searches for Spring components.

Spring scans:

* @Component
* @Service
* @Repository
* @Controller
* @RestController
* @Configuration

and registers them as beans.

---

# How Component Scanning Works

Suppose:

```text
com.company

    Application

    controller

    service

    repository

    config
```

Application:

```java
@SpringBootApplication
public class Application {

}
```

Spring scans:

```text
com.company
```

and every sub-package.

---

## Why Package Structure Matters

Correct:

```text
com.company

    Application

    controller

    service

    repository
```

Incorrect:

```text
com.company.app

Application

com.company.service

UserService
```

If `Application` is not placed in a common root package, some beans may not be discovered.

Best practice:

Place the main application class in the root package.

---

# Custom Component Scan

Example:

```java
@ComponentScan(basePackages = {
    "com.company.service",
    "com.company.payment"
})
```

Useful for multi-module projects.

---

# What Happens During Component Scan?

Spring:

1. Scans packages.
2. Finds annotated classes.
3. Creates Bean Definitions.
4. Resolves dependencies.
5. Instantiates singleton beans.
6. Applies AOP proxies.
7. Stores beans in the ApplicationContext.

---

# Bean Discovery Example

```java
@Service
public class OrderService {

}
```

Spring automatically creates:

```text
OrderService Bean
```

without explicit configuration.

---

# Stereotype Annotations

## @Component

Generic Spring bean.

---

## @Service

Business logic.

```java
@Service
public class PaymentService {

}
```

---

## @Repository

Persistence layer.

Benefits:

* Exception translation
* Better readability

---

## @Controller

MVC controller.

Returns views.

---

## @RestController

REST controller.

Returns JSON.

Equivalent to:

```java
@Controller
@ResponseBody
```

---

# @Bean vs @Component

## @Component

Spring creates the object automatically.

```java
@Service
public class EmailService {

}
```

---

## @Bean

Developer creates the object.

```java
@Bean
public EmailService emailService() {
    return new EmailService();
}
```

Use `@Bean` when:

* Configuring third-party libraries.
* Customizing object creation.
* Creating multiple instances with different configurations.

---

# Bean Naming

Default:

```java
@Service
public class OrderService {

}
```

Bean name:

```text
orderService
```

Custom:

```java
@Service("paymentProcessor")
```

---

# Common Interview Questions

### What annotations make up @SpringBootApplication?

* @Configuration
* @EnableAutoConfiguration
* @ComponentScan

---

### Why is @SpringBootApplication placed in the root package?

Because component scanning starts from the package containing the application class and scans its sub-packages.

---

### What is the difference between @Bean and @Component?

`@Component` is automatically discovered during component scanning.

`@Bean` explicitly registers an object returned by a method in a `@Configuration` class.

---

### What happens if a bean is outside the scanned package?

Spring does not discover or register it unless component scanning is configured to include that package.

---

### What is the difference between @Component, @Service, and @Repository?

All register Spring beans.

`@Service` and `@Repository` are semantic specializations.

`@Repository` also enables persistence exception translation.

---

# Real-World Example

A Spring Boot application contains:

* `OrderController`
* `OrderService`
* `OrderRepository`

When the application starts:

1. Component scanning discovers all three classes.
2. Spring creates bean definitions.
3. Dependencies are injected.
4. AOP proxies are applied if required.
5. The beans become available through the `ApplicationContext`.

No manual wiring is required.

---

# Quick Revision

* `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
* Component scanning starts from the package of the main application class.
* Place the main class in the root package.
* `@Component` enables automatic bean discovery.
* `@Bean` is used for explicit bean registration.
* `@Service`, `@Repository`, and `@RestController` are specialized stereotypes.
