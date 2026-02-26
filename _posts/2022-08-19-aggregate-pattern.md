---
title: "Aggregate Pattern"
categories:
  - Blog
tags:
  - Aggregate
  - Patterns
  - Domain Driven Design
  - DDD
---

The Aggregate is one of the main patterns in Domain Driven Design (DDD), introduced by Eric Evans.

As you may already deduce, it aggregates -- but what exactly?

An Aggregate is a cluster of domain objects that can be treated as a single unit, grouping together a cohesive slice of business logic. Importantly, an Aggregate does not simply group everything that "belongs to" an entity from a containment perspective -- a mistake people frequently make. If a `Customer` has addresses, phone numbers, and loyalty points, that alone does not mean all of those should live inside one Aggregate.

An Aggregate should encapsulate the data necessary to make a consistent, independent decision about some part of the business domain. Think of it as a black box that answers a specific question without relying on any external data.

### Key Characteristics

Let's take a look at the characteristics of an Aggregate:

- Can make decisions and carry out a consistent change of a certain data set.
- Serves as a root that stores in its interior a graph of other objects.
- Has a clearly defined identity.
- Its operation can be customized with injected policy code, e.g. using a strategy pattern.
- Increases the readability of the code.
- Acts as a unit of consistent change in the system.
- Can be verified through unit tests.

An example of an Aggregate might be a shopping basket with products, an order with line items, or a hotel reservation with room allocations.

### The Aggregate Root

Every Aggregate has exactly one **Aggregate Root** -- the single entity through which the outside world interacts with the Aggregate. All access to objects inside the Aggregate must go through the root. The root is responsible for enforcing all invariants across the entire Aggregate.

For example, in an `Order` Aggregate, the `Order` entity is the root. External code never directly manipulates an `OrderLineItem`; instead, it calls methods on `Order` (such as `addItem` or `removeItem`), and the root ensures that business rules -- like "the total cannot exceed the credit limit" -- remain satisfied.

```
Order (Aggregate Root)
  ├── OrderLineItem
  ├── OrderLineItem
  └── ShippingDetails
```

Key rules for the Aggregate Root:

- **Global identity**: The root has a globally unique identifier. Entities inside the Aggregate need only local identity (unique within the Aggregate).
- **Single entry point**: External objects may only hold references to the root. Internal objects are accessed exclusively through the root's methods.
- **Invariant enforcement**: The root checks that all invariants are satisfied before and after every state change.

### Transaction Boundaries

An Aggregate defines a **transaction boundary**. Each transaction should modify exactly one Aggregate and persist it as a whole. This rule exists because the Aggregate guarantees internal consistency -- if you try to update multiple Aggregates atomically, you are fighting the design rather than working with it.

In practice, this means:

- A single repository `save()` call persists one Aggregate.
- If a business operation needs to affect multiple Aggregates, use **eventual consistency** through domain events rather than stretching a single transaction across them.

### Referencing Other Aggregates by ID

Aggregates should reference other Aggregates **by identity (ID) only**, not by direct object reference. Holding a direct reference blurs the boundary, makes it tempting to reach across Aggregates within a single transaction, and creates tight coupling between them.

```java
// Correct: reference by ID
public class Order {
    private CustomerId customerId;
    private List<OrderLineItem> lineItems;
    // ...
}

// Avoid: direct object reference to another Aggregate
public class Order {
    private Customer customer; // breaks Aggregate boundary
    // ...
}
```

### What Does an Aggregate Look Like?

![img]({{site.url}}/assets/blog_images/2022-08-19-aggregate-pattern/aggregate-pattern-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-08-19-aggregate-pattern/aggregate-pattern-dark.png){: .dark }

An Aggregate consists of two fundamental structural elements: the **Boundary (B)** and the **Aggregate Root (AR)**.

The **Boundary (B)** encapsulates everything that belongs to the Aggregate. It acts as a protective barrier separating the Aggregate's internals from the rest of the application. No external object is allowed to hold a direct reference to anything that lives inside the boundary.

The **Aggregate Root (AR)** is the sole entry point to the Aggregate's contents. It is the only object within the boundary that outside code may reference directly. The root maintains references to everything inside, but it never exposes them for direct manipulation. Instead of retrieving an internal object and operating on it yourself, you ask the Aggregate (A) to perform the operation on your behalf -- you may not even be aware which internal objects are involved. This is what keeps the invariants safe.

The internals of an Aggregate are composed of the core DDD building blocks: **Entities (E)**, **Value Objects (VO)**, and potentially other nested **Aggregates (A)**. A single Entity with no children is itself a valid Aggregate.

The primary purpose of an Aggregate is to preserve consistency within the domain model. It centralizes business rules and is always persisted atomically -- regardless of how many child Entities and Value Objects reside within the root, they are all saved as a single unit of work.

### Design Guidelines

When designing Aggregates, keep these principles in mind:

1. **Keep Aggregates small.** Large Aggregates lead to concurrency conflicts and performance issues. Favor smaller Aggregates that protect true invariants.
2. **Protect true invariants.** The Aggregate boundary should enclose exactly the data that must be immediately consistent. Everything else can be eventually consistent.
3. **Reference other Aggregates by ID.** This keeps boundaries clean and makes the system easier to scale and distribute.
4. **Use domain events for cross-Aggregate coordination.** When one Aggregate's change needs to trigger a reaction in another, publish an event and let a handler update the second Aggregate in its own transaction.
