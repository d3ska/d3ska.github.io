---
title: "Aggregate Pattern"
categories:
  - Design Patterns
tags:
  - Aggregate
  - Design Patterns
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

### A Complete Example: Order Aggregate

Theory is useful, but the pattern clicks once you see it in code. Here is a full `Order` Aggregate that enforces two business invariants: line items cannot be empty when the order is placed, and the order total must not exceed a given credit limit.

```java
public class OrderId {
    private final UUID value;

    public OrderId(UUID value) {
        this.value = value;
    }

    public static OrderId generate() {
        return new OrderId(UUID.randomUUID());
    }

    public UUID getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        return value.equals(((OrderId) o).value);
    }

    @Override
    public int hashCode() {
        return value.hashCode();
    }
}
```

The `OrderLineItem` is a **Value Object**. It has no identity of its own and is fully defined by its attributes. Two line items with the same product, quantity, and price are considered equal.

```java
public class OrderLineItem {
    private final ProductId productId;
    private final String productName;
    private final int quantity;
    private final BigDecimal unitPrice;

    public OrderLineItem(ProductId productId, String productName, int quantity, BigDecimal unitPrice) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        if (unitPrice.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Unit price must be positive");
        }
        this.productId = productId;
        this.productName = productName;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }

    public BigDecimal lineTotal() {
        return unitPrice.multiply(BigDecimal.valueOf(quantity));
    }

    public ProductId getProductId() {
        return productId;
    }

    public int getQuantity() {
        return quantity;
    }

    public BigDecimal getUnitPrice() {
        return unitPrice;
    }
}
```

Now the **Aggregate Root** itself. Notice that the `Order` class is the only way to add, remove, or place items. No external code touches `OrderLineItem` directly.

```java
public class Order {
    private final OrderId id;
    private final CustomerId customerId;
    private final List<OrderLineItem> lineItems;
    private final BigDecimal creditLimit;
    private OrderStatus status;
    private final List<DomainEvent> domainEvents;

    public Order(OrderId id, CustomerId customerId, BigDecimal creditLimit) {
        this.id = id;
        this.customerId = customerId;
        this.creditLimit = creditLimit;
        this.lineItems = new ArrayList<>();
        this.status = OrderStatus.DRAFT;
        this.domainEvents = new ArrayList<>();
    }

    public void addItem(ProductId productId, String productName, int quantity, BigDecimal unitPrice) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify a placed order");
        }
        OrderLineItem newItem = new OrderLineItem(productId, productName, quantity, unitPrice);
        BigDecimal projectedTotal = calculateTotal().add(newItem.lineTotal());

        if (projectedTotal.compareTo(creditLimit) > 0) {
            throw new CreditLimitExceededException(
                    "Adding this item would exceed the credit limit of " + creditLimit
            );
        }
        lineItems.add(newItem);
    }

    public void removeItem(ProductId productId) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify a placed order");
        }
        boolean removed = lineItems.removeIf(item -> item.getProductId().equals(productId));
        if (!removed) {
            throw new IllegalArgumentException("Product not found in order: " + productId);
        }
    }

    public void place() {
        if (lineItems.isEmpty()) {
            throw new IllegalStateException("Cannot place an order with no items");
        }
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Order has already been placed");
        }
        this.status = OrderStatus.PLACED;
        domainEvents.add(new OrderPlaced(id, customerId, calculateTotal()));
    }

    public BigDecimal calculateTotal() {
        return lineItems.stream()
                .map(OrderLineItem::lineTotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public OrderId getId() {
        return id;
    }

    public OrderStatus getStatus() {
        return status;
    }

    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearDomainEvents() {
        domainEvents.clear();
    }
}
```

A few things to notice:

- **Invariants are checked inside the root.** The credit limit check happens in `addItem`, and the "must have items" check happens in `place`. The caller cannot accidentally bypass these rules.
- **State transitions are explicit.** An order starts as `DRAFT`, and once placed, it cannot be modified. The root controls the lifecycle.
- **Domain events are collected.** When the order is placed, an `OrderPlaced` event is recorded. We will cover what happens with that event in the next section.
- **No setters.** The Aggregate exposes behavior (verbs like `addItem`, `removeItem`, `place`), not data mutation.

The Aggregate is easy to test because all you need is the root object, no database, no Spring context, no mocking frameworks.

```java
@Test
void shouldNotExceedCreditLimit() {
    // Given
    var order = new Order(
            OrderId.generate(),
            new CustomerId(UUID.randomUUID()),
            new BigDecimal("100.00")
    );
    order.addItem(new ProductId(UUID.randomUUID()), "Keyboard", 1, new BigDecimal("80.00"));

    // When / Then
    assertThrows(CreditLimitExceededException.class, () ->
            order.addItem(new ProductId(UUID.randomUUID()), "Monitor", 1, new BigDecimal("250.00"))
    );
}

@Test
void shouldNotPlaceEmptyOrder() {
    // Given
    var order = new Order(
            OrderId.generate(),
            new CustomerId(UUID.randomUUID()),
            new BigDecimal("500.00")
    );

    // When / Then
    assertThrows(IllegalStateException.class, order::place);
}

@Test
void shouldPublishOrderPlacedEventWhenPlaced() {
    // Given
    var order = new Order(
            OrderId.generate(),
            new CustomerId(UUID.randomUUID()),
            new BigDecimal("500.00")
    );
    order.addItem(new ProductId(UUID.randomUUID()), "Keyboard", 1, new BigDecimal("80.00"));

    // When
    order.place();

    // Then
    assertThat(order.getDomainEvents()).hasSize(1);
    assertThat(order.getDomainEvents().get(0)).isInstanceOf(OrderPlaced.class);
}
```

### Eventual Consistency Through Domain Events

The "one transaction per Aggregate" rule inevitably raises a question: what happens when placing an order needs to update inventory, notify the warehouse, and charge the customer? Those are separate Aggregates with their own consistency boundaries.

The answer is **domain events**. Instead of stretching a transaction across multiple Aggregates, the originating Aggregate records what happened, and other parts of the system react to that event in their own transactions.

Here is a simple domain event interface and the `OrderPlaced` event from the example above:

```java
public interface DomainEvent {
    Instant occurredAt();
}

public class OrderPlaced implements DomainEvent {
    private final OrderId orderId;
    private final CustomerId customerId;
    private final BigDecimal totalAmount;
    private final Instant occurredAt;

    public OrderPlaced(OrderId orderId, CustomerId customerId, BigDecimal totalAmount) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
        this.occurredAt = Instant.now();
    }

    @Override
    public Instant occurredAt() {
        return occurredAt;
    }

    public OrderId getOrderId() {
        return orderId;
    }

    public CustomerId getCustomerId() {
        return customerId;
    }

    public BigDecimal getTotalAmount() {
        return totalAmount;
    }
}
```

The event is published after the Order Aggregate's transaction commits successfully. A handler then processes it in a separate transaction:

```java
@Component
public class OrderPlacedHandler {

    private final InventoryService inventoryService;
    private final NotificationService notificationService;

    public OrderPlacedHandler(InventoryService inventoryService,
                              NotificationService notificationService) {
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }

    @EventListener
    public void handle(OrderPlaced event) {
        inventoryService.reserveStock(event.getOrderId());
        notificationService.notifyWarehouse(event.getOrderId());
    }
}
```

The critical insight here is that the `Order` Aggregate and the `Inventory` Aggregate are **eventually consistent**. There is a brief window where the order has been placed but inventory has not yet been reserved. This is perfectly fine for most business domains, and in fact it is how the real world works. When you place an order in a physical store, the clerk does not freeze the entire warehouse while ringing you up.

If the event handler fails (maybe the inventory service is temporarily down), the event can be retried. This is why domain events are often persisted to a durable store or published through a message broker like Kafka or RabbitMQ. The Outbox pattern is a common approach: events are saved to an "outbox" table within the same transaction as the Aggregate, and a separate process publishes them to the broker. This guarantees that events are never lost, even if the broker is unavailable at commit time.

Eventual consistency through domain events trades immediate consistency for better scalability, lower contention, and clearer Aggregate boundaries. I have found that teams resist it at first because "what if something goes wrong between the two steps?" But once you design for it (with idempotent handlers, retries, and dead-letter queues), the system becomes more resilient than one that relies on distributed transactions.

### Common Mistakes

#### Making Aggregates Too Large

This is the single most common mistake I see. Developers model an Aggregate by asking "what data belongs together?" instead of "what data must be immediately consistent?" Those are very different questions.

Consider an e-commerce system with customers, orders, and reviews. A tempting first design might look like this:

```java
// Too large: the Customer Aggregate tries to own everything
public class Customer {
    private final CustomerId id;
    private String name;
    private String email;
    private List<Address> addresses;
    private List<Order> orders;         // dozens to thousands
    private List<Review> reviews;       // dozens to thousands
    private LoyaltyPoints loyaltyPoints;
    // ...
}
```

This Aggregate becomes a bottleneck. Every time a customer places an order or writes a review, you load the entire object graph into memory, lock it, modify it, and persist it back. Two users cannot even place orders for the same customer concurrently without conflicting.

The fix is to ask: "Does placing an order require immediate consistency with the customer's reviews?" Almost certainly not. `Order` and `Review` should be separate Aggregates that reference the `Customer` by ID:

```java
// Right-sized: each Aggregate protects its own invariants
public class Customer {
    private final CustomerId id;
    private String name;
    private String email;
    private List<Address> addresses;
}

public class Order {
    private final OrderId id;
    private final CustomerId customerId; // reference by ID
    private List<OrderLineItem> lineItems;
}

public class Review {
    private final ReviewId id;
    private final CustomerId customerId; // reference by ID
    private final ProductId productId;
    private String content;
    private int rating;
}
```

A good rule of thumb: if loading your Aggregate requires joining more than two or three tables, it is probably too large. Vaughn Vernon, in "Implementing Domain-Driven Design", advocates designing small Aggregates first and merging them only when you discover a true consistency requirement.

#### Exposing Internal Collections

Returning a mutable reference to an internal collection lets callers bypass the root entirely:

```java
// Dangerous: caller can add items without invariant checks
public List<OrderLineItem> getLineItems() {
    return lineItems;
}
```

Instead, return an unmodifiable view or a defensive copy:

```java
public List<OrderLineItem> getLineItems() {
    return Collections.unmodifiableList(lineItems);
}
```

#### Modifying Multiple Aggregates in One Transaction

If you find yourself doing something like this in a service method, it is a design smell:

```java
@Transactional
public void placeOrder(OrderId orderId) {
    Order order = orderRepository.findById(orderId);
    Inventory inventory = inventoryRepository.findByProductIds(order.getProductIds());

    order.place();
    inventory.reserve(order.getProductIds());

    orderRepository.save(order);
    inventoryRepository.save(inventory);
}
```

This modifies two Aggregates (`Order` and `Inventory`) in the same transaction. It works on a single database, but it does not scale. It creates lock contention and makes it impossible to distribute these Aggregates across separate services later. The domain event approach described above is the correct alternative.

### When to Use the Aggregate Pattern

Aggregates earn their complexity when you have genuine business invariants that span multiple objects. If adding a line item to an order must check the credit limit, and removing a line item must recalculate the total, and placing the order must verify that the items are not empty, then grouping those objects behind a single root makes sense. The root becomes the single place where those rules live, and the rest of the system does not need to know about them.

Aggregates are also valuable when you need clear transaction boundaries in a system that will eventually be distributed. By designing Aggregates early, you make it straightforward to extract bounded contexts into separate services later, because each Aggregate already communicates with others through events and IDs rather than direct references.

### When Not to Use

Not every entity needs to be wrapped in an Aggregate. If you have a simple CRUD resource with no cross-object invariants, an Aggregate adds ceremony without benefit. A `Tag` entity that has an ID and a name, with no rules beyond "name cannot be blank", does not need an Aggregate boundary around it. A plain entity or even a Value Object is sufficient.

Similarly, if your system is a straightforward data pipeline where objects flow through transformations without enforcing domain rules, Aggregates are the wrong tool. The pattern exists to protect business invariants. If there are no invariants to protect, there is nothing for the pattern to do.

I have also seen teams apply Aggregates in read-heavy reporting systems where the primary concern is querying, not enforcing rules. In those cases, a simpler read model (possibly backed by CQRS) is more appropriate. Aggregates shine on the write side of a system, where consistency matters.

> **Related posts**: [Specification Design Pattern](/posts/specification-design-pattern/), [Microservices vs Monolith](/posts/microservices-vs-monolith/)
