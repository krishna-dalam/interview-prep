# N+1 Query Problem in JPA

## Interview Priority

⭐⭐⭐⭐⭐

## What Is the N+1 Problem?

The N+1 problem occurs when:

* One query loads a list of parent entities.
* An additional query runs for each parent to load related data.

Example:

```text
1 query to load customers
+
100 queries to load orders for each customer
=
101 total queries
```

This can severely impact latency and database load.

---

## Example

```java
List<Customer> customers = customerRepository.findAll();

for (Customer customer : customers) {
    System.out.println(customer.getOrders().size());
}
```

Possible SQL:

```sql
select * from customers;
```

Then:

```sql
select * from orders where customer_id = 1;
select * from orders where customer_id = 2;
select * from orders where customer_id = 3;
```

One query loads customers, followed by one query per customer.

---

# Why Does It Happen?

Typical causes include:

* Accessing lazy relationships in a loop.
* Serializing entities directly to JSON.
* Eager associations being fetched through secondary queries.
* Mapping entities to DTOs without controlling the fetch plan.
* Iterating over related entities outside an optimized query.

The issue is not limited to `LAZY` loading. `EAGER` associations can also produce N+1 queries.

---

# How to Detect It

## SQL Logging

Enable SQL output in non-production environments:

```properties
spring.jpa.show-sql=true
logging.level.org.hibernate.SQL=DEBUG
```

Bind values can be logged separately when needed.

Look for the same query executing repeatedly with different IDs.

## Hibernate Statistics

Hibernate statistics can expose:

* Query count
* Entity fetch count
* Collection fetch count

## Observability

In production, watch:

* Database query rate
* Endpoint latency
* Database CPU
* Connection-pool usage
* Repeated query fingerprints

---

# Solutions

## 1. Fetch Join

```java
@Query("""
    select distinct c
    from Customer c
    left join fetch c.orders
""")
List<Customer> findAllWithOrders();
```

This loads customers and orders in one query.

### Why `distinct`?

A join can return one row per parent-child combination.

`distinct` prevents duplicate parent entities in the JPA result.

---

## 2. Entity Graph

```java
@EntityGraph(attributePaths = "orders")
@Query("select c from Customer c")
List<Customer> findAllWithOrders();
```

Useful when the same entity requires different fetch plans for different use cases.

---

## 3. DTO Projection

```java
public record CustomerSummary(
    Long id,
    String name,
    long orderCount
) {}
```

```java
@Query("""
    select new com.example.CustomerSummary(
        c.id,
        c.name,
        count(o)
    )
    from Customer c
    left join c.orders o
    group by c.id, c.name
""")
List<CustomerSummary> findCustomerSummaries();
```

DTO projections fetch only the data required by the API.

They are often the best option for read endpoints.

---

## 4. Batch Fetching

Hibernate can load lazy associations in batches.

```properties
spring.jpa.properties.hibernate.default_batch_fetch_size=50
```

Instead of one query per parent, Hibernate may issue:

```sql
select *
from orders
where customer_id in (?, ?, ?, ...);
```

This reduces query count but may not eliminate it completely.

Entity-level configuration is also possible:

```java
@BatchSize(size = 50)
```

---

# Fetch Join Trade-Offs

Fetch joins are useful, but they have limitations.

## Large Result Sets

Joining a parent with a large child collection can produce many database rows.

Example:

```text
100 customers
×
100 orders each
=
10,000 joined rows
```

This increases:

* Network transfer
* Memory usage
* Hibernate deduplication work

## Multiple Collections

Fetching multiple to-many collections in one query can create a Cartesian product.

```text
Customer
  ├── Orders
  └── Addresses
```

Joining both may multiply rows dramatically.

## Pagination

Pagination with collection fetch joins is problematic because the database paginates rows rather than unique parent entities.

Hibernate may paginate in memory or produce incorrect-looking page sizes depending on the query and version.

---

# Safe Pagination Pattern

For paginated parent-child data:

1. Query only the parent IDs for the requested page.
2. Fetch the parent entities and required relationships using those IDs.
3. Preserve the original order.

Example:

```text
Query 1:
Fetch 20 customer IDs

Query 2:
Fetch customers and orders for those 20 IDs
```

This is often called two-step pagination.

---

# Choosing the Right Fix

| Requirement                               | Recommended approach |
| ----------------------------------------- | -------------------- |
| Need full entity graph for a small result | Fetch join           |
| Need configurable relationship loading    | Entity graph         |
| Need a read-only API response             | DTO projection       |
| Need to reduce many lazy queries broadly  | Batch fetching       |
| Need pagination with collections          | Two-step query       |
| Need only counts or aggregates            | Aggregate projection |

---

# Common Interview Questions

### What is the N+1 problem?

One query loads parent entities, and N additional queries load related data for each parent.

### Does LAZY loading always cause N+1?

No. N+1 occurs when lazy associations are accessed repeatedly without an optimized fetch plan.

### Can EAGER loading cause N+1?

Yes. A provider may load eager associations through separate queries.

### How do you solve N+1?

Use fetch joins, entity graphs, DTO projections, batch fetching, or purpose-built queries.

### Why not use fetch joins everywhere?

They can create large result sets, Cartesian products, memory pressure, and pagination problems.

### Why are DTO projections often preferred for REST APIs?

They fetch only required fields, avoid exposing entities, and provide predictable query behavior.

---

# Interview Scenario

An endpoint returns 50 orders with customer details and line items.

A poor implementation may execute:

```text
1 query for orders
50 queries for customers
50 queries for line items
```

Total:

```text
101 queries
```

A better solution depends on the response:

* Use a DTO projection for a summary endpoint.
* Use a controlled fetch plan for a detailed endpoint.
* Use batch fetching if multiple lazy relationships must be initialized.
* Avoid joining multiple large collections in one query.

---

# Production Checklist

Before approving a JPA endpoint:

* Check generated SQL.
* Count database round trips.
* Avoid returning entities directly.
* Test with realistic data volume.
* Review pagination behavior.
* Check connection-pool pressure.
* Measure query latency.
* Add indexes for filtering and joining columns.
* Avoid fetching unused fields and relationships.

---

# Quick Revision

* N+1 means one parent query plus one related query per parent.
* Both LAZY and EAGER associations can cause it.
* Fetch join works well for small, controlled graphs.
* Entity graphs provide reusable fetch plans.
* DTO projections are ideal for many read APIs.
* Batch fetching reduces repeated lazy-load queries.
* Collection fetch joins and pagination do not combine cleanly.
* Always inspect generated SQL.
