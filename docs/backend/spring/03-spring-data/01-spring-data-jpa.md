# Spring Data JPA

## Interview Priority

⭐⭐⭐⭐⭐

## What is Spring Data JPA?

Spring Data JPA is a Spring module that simplifies database access by building on top of JPA and Hibernate.

It eliminates most boilerplate DAO code while providing repository abstractions, query generation, pagination, and transaction integration.

---

# JPA vs Hibernate vs Spring Data JPA

Many interviews start with this question.

| Technology      | Purpose                                |
| --------------- | -------------------------------------- |
| JPA             | Java specification for ORM             |
| Hibernate       | Popular implementation of JPA          |
| Spring Data JPA | Spring abstraction built on top of JPA |

Think of the relationship as:

```text
Application
      │
      ▼
Spring Data JPA
      │
      ▼
JPA Specification
      │
      ▼
Hibernate
      │
      ▼
Database
```

---

# What is ORM?

ORM (Object Relational Mapping) maps Java objects to database tables.

Example:

Database

```text
users
---------
id
name
email
```

Java

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    private String email;

}
```

Spring Data JPA maps rows into Java objects automatically.

---

# Repository Pattern

Instead of writing DAOs manually:

```java
public class UserDao {

    public User findById(Long id) {

    }

}
```

Spring Data JPA provides repositories.

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

No implementation is required.

---

# JpaRepository

Provides built-in operations.

Common methods:

```java
save()

findById()

findAll()

delete()

deleteById()

count()

existsById()
```

---

# Derived Query Methods

Spring can generate SQL from method names.

Example:

```java
findByEmail()

findByStatus()

findByNameContaining()

findByAgeGreaterThan()

findByOrderByCreatedAtDesc()
```

No SQL is written manually.

---

# Custom Queries

When derived methods become too complex:

```java
@Query("""
SELECT u
FROM User u
WHERE u.status='ACTIVE'
""")
```

JPQL operates on entities rather than database tables.

---

# Native Queries

For database-specific SQL:

```java
@Query(
value = "...",
nativeQuery = true
)
```

Use sparingly.

Prefer JPQL unless native SQL is required.

---

# Pagination

Spring Data provides built-in pagination.

```java
Page<User> users =
repository.findAll(pageable);
```

Benefits:

* Smaller responses
* Lower memory usage
* Better database performance

---

# Sorting

```java
Sort.by("name")
```

or

```java
Sort.by(
Direction.DESC,
"createdAt"
)
```

---

# Specifications

Useful for dynamic filtering.

Instead of creating many repository methods:

```text
findByName()

findByNameAndStatus()

findByNameAndStatusAndCity()
```

Specifications allow filters to be composed dynamically.

---

# Transaction Support

Repository methods participate in Spring transactions.

Example:

```java
@Transactional
public void createUser() {

}
```

Transaction management will be covered in a separate topic.

---

# Common Interview Questions

### What is JPA?

A Java specification for Object Relational Mapping (ORM).

---

### What is Hibernate?

A widely used implementation of the JPA specification.

---

### What is Spring Data JPA?

A Spring module that simplifies JPA usage through repository abstractions and additional features.

---

### Why use JpaRepository?

It reduces boilerplate code by providing common CRUD operations and query support.

---

### What is the difference between JPQL and SQL?

* SQL queries database tables.
* JPQL queries Java entities and their relationships.

---

### Should native queries always be preferred?

No.

Use JPQL by default.

Use native SQL only when you need database-specific features or performance optimizations.

---

# Real-World Example

A Spring Boot Order Service defines:

```java
public interface OrderRepository
        extends JpaRepository<Order, Long> {

    List<Order> findByCustomerId(Long id);

}
```

Spring Data automatically generates the implementation, allowing developers to focus on business logic instead of CRUD code.

---

# Quick Revision

* JPA is a specification.
* Hibernate implements JPA.
* Spring Data JPA simplifies JPA usage.
* `JpaRepository` provides CRUD operations.
* Derived query methods generate queries automatically.
* Use JPQL for most custom queries.
* Use native SQL only when necessary.
* Built-in support exists for pagination and sorting.
