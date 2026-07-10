# JPA Fetch Types and Lazy Loading

## Interview Priority

⭐⭐⭐⭐⭐

## Fetch Types

JPA supports two fetch strategies:

* `LAZY` — load related data only when accessed.
* `EAGER` — load related data immediately.

## Default Fetch Types

| Relationship  | Default |
| ------------- | ------- |
| `@OneToMany`  | LAZY    |
| `@ManyToMany` | LAZY    |
| `@ManyToOne`  | EAGER   |
| `@OneToOne`   | EAGER   |

In production code, explicitly choose fetch behavior rather than relying only on defaults.

---

## Lazy Loading

```java
@OneToMany(mappedBy = "customer", fetch = FetchType.LAZY)
private List<Order> orders;
```

When the customer is loaded, orders are not fetched immediately.

```java
Customer customer = repository.findById(id).orElseThrow();
customer.getOrders().size();
```

The collection is fetched when accessed.

Hibernate commonly uses a proxy or persistent collection wrapper to implement this behavior.

---

## Eager Loading

```java
@ManyToOne(fetch = FetchType.EAGER)
private Customer customer;
```

The related customer is loaded with the entity.

This may happen through:

* A join
* An additional query
* Provider-specific loading behavior

`EAGER` does not guarantee one SQL query.

---

## Why Prefer Lazy Loading?

Lazy loading can:

* Avoid unnecessary database work.
* Reduce memory usage.
* Keep entity queries smaller.
* Improve response times when relationships are not needed.

However, lazy loading can also create extra queries and runtime errors if used carelessly.

---

# LazyInitializationException

This occurs when lazy data is accessed after the persistence context has closed.

```java
public Customer getCustomer(Long id) {
    return repository.findById(id).orElseThrow();
}
```

Later:

```java
customer.getOrders().size();
```

If `orders` was not initialized and the Hibernate session is closed, Hibernate cannot load it.

Result:

```text
LazyInitializationException
```

---

## Common Causes

* Returning JPA entities directly from controllers.
* Accessing lazy relationships outside a transaction.
* Mapping entities to DTOs after leaving the service layer.
* Disabling session scope before required relationships are loaded.

---

# Recommended Solutions

## 1. Fetch Join

```java
@Query("""
    select c
    from Customer c
    left join fetch c.orders
    where c.id = :id
""")
Optional<Customer> findByIdWithOrders(Long id);
```

Use when a specific use case needs the relationship.

---

## 2. EntityGraph

```java
@EntityGraph(attributePaths = "orders")
Optional<Customer> findWithOrdersById(Long id);
```

Useful when fetch requirements differ across repository methods.

---

## 3. DTO Projection

```java
public record CustomerSummary(
    Long id,
    String name,
    long orderCount
) {}
```

Query only the data required by the API.

This is often the best approach for read-heavy endpoints.

---

## 4. Map Inside the Transaction

```java
@Transactional(readOnly = true)
public CustomerResponse getCustomer(Long id) {
    Customer customer = repository.findById(id).orElseThrow();

    return new CustomerResponse(
        customer.getId(),
        customer.getName(),
        customer.getOrders()
            .stream()
            .map(OrderResponse::from)
            .toList()
    );
}
```

The lazy relationship is accessed while the persistence context is active.

---

# Open Session in View

Spring Boot traditionally enables Open Session in View for servlet applications unless configured otherwise.

It keeps the persistence context open through the web request, allowing lazy loading during response serialization.

Potential problems:

* Database queries can execute in the controller or serialization layer.
* Query behavior becomes harder to predict.
* N+1 problems can remain hidden.
* Database connections may be held longer.

Many teams disable it:

```properties
spring.jpa.open-in-view=false
```

Then they fetch and map required data explicitly in the service layer.

---

# EAGER Is Not a Fix

Changing every association to `EAGER` may appear to fix lazy-loading errors, but it often causes:

* Excessive data loading
* Large object graphs
* Slower queries
* Unexpected joins
* Additional queries
* Memory pressure

Fetch strategy should be decided per use case.

---

# Fetch Type vs Query Fetch Plan

`FetchType.LAZY` defines the default association behavior.

A specific query can override it using:

* `JOIN FETCH`
* `@EntityGraph`
* DTO projection

This gives better control than globally making relationships eager.

---

# Serialization Problem

Returning entities directly from REST controllers can trigger lazy loading while Jackson serializes them.

It may also cause:

* N+1 queries
* Infinite recursion in bidirectional relationships
* Exposure of internal fields
* Tight coupling between API and persistence models

Prefer DTOs for API responses.

---

# Common Interview Questions

### What is lazy loading?

Related data is loaded only when the association is accessed.

### What causes `LazyInitializationException`?

A lazy association is accessed after the persistence context has closed.

### Should every relationship be EAGER?

No. Eager loading often fetches unnecessary data and can hurt performance.

### How do you safely load lazy relationships?

Use fetch joins, entity graphs, DTO projections, or access and map them within a transaction.

### What is Open Session in View?

A pattern that keeps the persistence context open for the entire web request, allowing lazy loading outside the service layer.

### Why might you disable Open Session in View?

To prevent hidden database queries in controllers and serialization, and to make transaction and fetch boundaries explicit.

### Does EAGER guarantee a single SQL join?

No. The provider may use joins or additional queries.

---

# Interview Scenario

An endpoint returns 100 customers. During JSON serialization, Jackson accesses each customer's orders.

What can happen?

* One query loads customers.
* One additional query runs for each customer's orders.
* Total: 101 queries.

This is an N+1 problem.

The fix may be a DTO projection, fetch join, entity graph, or a dedicated query depending on the response requirement.

---

# Quick Revision

* To-many relationships default to LAZY.
* To-one relationships default to EAGER.
* Prefer explicit fetch plans.
* `LazyInitializationException` means the persistence context is closed.
* Avoid returning entities directly from controllers.
* DTO projections are often best for API reads.
* Open Session in View can hide poor query behavior.
* EAGER loading is not a universal solution.
