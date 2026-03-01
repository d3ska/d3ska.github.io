---
title: "Facade Design Pattern"
categories:
  - Design Patterns
tags:
  - Design Patterns
  - Facade Pattern
  - Structural Patterns
  - SOLID
---

Picture an e-commerce checkout flow. A client places an order and expects a single, clean response: "order confirmed" or "order failed." Behind the scenes, that single action triggers inventory validation, payment processing, shipping scheduling, and customer notification. If the calling code has to orchestrate all four services directly, it ends up tightly coupled to every subsystem, aware of their individual APIs, error handling quirks, and execution order.

The **Facade design pattern** solves this by providing a unified, higher-level interface to a set of interfaces in a subsystem. The client talks to one class. That class knows how to coordinate everything underneath.

### How it Works

A facade wraps a group of collaborating classes and exposes only the operations that external callers actually need. The subsystem classes remain package-private, meaning no outside code can reach them directly. This gives you two things at once: simplicity for callers and freedom to refactor internals without breaking anyone.

![img]({{site.url}}/assets/blog_images/2023-02-07-facade-design-pattern/facad=uml-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-02-07-facade-design-pattern/facad-uml-dark.png){: .dark }

### The Example: Order Fulfillment

Imagine an order fulfillment module with four internal services: inventory checks, payment processing, shipping, and notifications. Each service has its own logic, but callers should never interact with them individually. They go through a single facade.

First, the subsystem classes. All of them are **package-private**.

```java
class InventoryService {

    boolean reserve(String productId, int quantity) {
        System.out.println("InventoryService: reserving " + quantity + "x " + productId);
        // check stock levels, place a reservation
        return true;
    }

    void release(String productId, int quantity) {
        System.out.println("InventoryService: releasing " + quantity + "x " + productId);
    }
}
```

```java
class PaymentService {

    boolean charge(String customerId, double amount) {
        System.out.println("PaymentService: charging $" + amount + " to customer " + customerId);
        // call payment gateway, validate card, process charge
        return true;
    }

    void refund(String customerId, double amount) {
        System.out.println("PaymentService: refunding $" + amount + " to customer " + customerId);
    }
}
```

```java
class ShippingService {

    String schedule(String productId, int quantity, String address) {
        System.out.println("ShippingService: scheduling shipment to " + address);
        // create shipping label, schedule pickup
        return "TRACK-" + System.currentTimeMillis();
    }
}
```

```java
class NotificationService {

    void sendOrderConfirmation(String customerId, String trackingNumber) {
        System.out.println("NotificationService: sending confirmation to customer "
                + customerId + ", tracking: " + trackingNumber);
    }

    void sendOrderFailure(String customerId, String reason) {
        System.out.println("NotificationService: notifying customer "
                + customerId + " of failure: " + reason);
    }
}
```

Now, the **public facade**. It is the only entry point external code ever touches.

```java
public class OrderFulfillmentFacade {

    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    private final NotificationService notificationService;

    public OrderFulfillmentFacade() {
        this.inventoryService = new InventoryService();
        this.paymentService = new PaymentService();
        this.shippingService = new ShippingService();
        this.notificationService = new NotificationService();
    }

    public String fulfill(Order order) {
        if (!inventoryService.reserve(order.getProductId(), order.getQuantity())) {
            notificationService.sendOrderFailure(order.getCustomerId(), "Out of stock");
            return "FAILED: out of stock";
        }

        if (!paymentService.charge(order.getCustomerId(), order.getTotalAmount())) {
            inventoryService.release(order.getProductId(), order.getQuantity());
            notificationService.sendOrderFailure(order.getCustomerId(), "Payment declined");
            return "FAILED: payment declined";
        }

        String trackingNumber = shippingService.schedule(
                order.getProductId(), order.getQuantity(), order.getShippingAddress());

        notificationService.sendOrderConfirmation(order.getCustomerId(), trackingNumber);

        return "SUCCESS: " + trackingNumber;
    }
}
```

```java
public class Order {

    private final String customerId;
    private final String productId;
    private final int quantity;
    private final double totalAmount;
    private final String shippingAddress;

    public Order(String customerId, String productId, int quantity,
                 double totalAmount, String shippingAddress) {
        this.customerId = customerId;
        this.productId = productId;
        this.quantity = quantity;
        this.totalAmount = totalAmount;
        this.shippingAddress = shippingAddress;
    }

    public String getCustomerId() { return customerId; }
    public String getProductId() { return productId; }
    public int getQuantity() { return quantity; }
    public double getTotalAmount() { return totalAmount; }
    public String getShippingAddress() { return shippingAddress; }
}
```

The caller's code is minimal:

```java
OrderFulfillmentFacade facade = new OrderFulfillmentFacade();
Order order = new Order("cust-42", "WIDGET-7", 3, 59.97, "123 Main St");
String result = facade.fulfill(order);
```

One method call. No knowledge of inventory checks, payment gateways, shipping APIs, or notification channels. If you later swap the shipping provider or add a fraud check step, the caller never changes.

### The God Object Trap

Facades can rot. When a facade starts accumulating responsibilities beyond its original scope, it becomes a **god object**: a single class that knows too much and does too much. I have seen this happen when teams treat the facade as a convenient dumping ground for "anything related to orders."

Here is what a bloated facade might look like:

```java
public class ECommerceFacade {

    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    private final NotificationService notificationService;
    private final CustomerService customerService;
    private final ProductCatalogService productCatalogService;
    private final ReviewService reviewService;
    private final ReturnService returnService;
    private final AnalyticsService analyticsService;
    private final TaxService taxService;

    // constructor with 10 dependencies...

    public String fulfillOrder(Order order) { /* ... */ }
    public void cancelOrder(String orderId) { /* ... */ }
    public void processReturn(String orderId) { /* ... */ }
    public List<Product> searchProducts(String query) { /* ... */ }
    public Product getProductDetails(String productId) { /* ... */ }
    public void submitReview(String productId, Review review) { /* ... */ }
    public Customer getCustomerProfile(String customerId) { /* ... */ }
    public void updateCustomerProfile(Customer customer) { /* ... */ }
    public Report generateSalesReport(DateRange range) { /* ... */ }
    public TaxSummary calculateTax(Order order) { /* ... */ }
    // ... and it keeps growing
}
```

This class has ten dependencies and methods spanning order fulfillment, product catalog, customer management, reviews, analytics, and tax calculation. These are clearly separate concerns.

**How to recognize the problem:**
- The constructor parameter list keeps growing (more than four or five dependencies is a red flag)
- Methods in the class serve unrelated use cases
- Changes to one area (say, product search) force you to re-test or redeploy code that handles a completely different area (order fulfillment)
- The class file is hundreds of lines long and hard to navigate

**The fix:** split the monolithic facade into smaller, focused facades, each owning a specific subdomain.

```java
public class OrderFulfillmentFacade {
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    private final NotificationService notificationService;

    public String fulfill(Order order) { /* ... */ }
    public void cancel(String orderId) { /* ... */ }
}

public class ProductCatalogFacade {
    private final ProductCatalogService productCatalogService;
    private final ReviewService reviewService;

    public List<Product> search(String query) { /* ... */ }
    public Product getDetails(String productId) { /* ... */ }
    public void submitReview(String productId, Review review) { /* ... */ }
}

public class CustomerFacade {
    private final CustomerService customerService;

    public Customer getProfile(String customerId) { /* ... */ }
    public void updateProfile(Customer customer) { /* ... */ }
}
```

Each facade now has a focused responsibility and a small set of dependencies. This keeps the pattern healthy and avoids the irony of a "simplification" pattern making things more complex.

### Connection to the Dependency Inversion Principle

The basic facade example above instantiates its dependencies directly. That works for simple cases, but it makes the facade impossible to test in isolation and locks it to specific implementations. The **Dependency Inversion Principle** (the "D" in SOLID) tells us that high-level modules should depend on abstractions, not on concrete classes.

Here is how the order fulfillment facade looks when it depends on interfaces instead:

```java
public interface InventoryPort {
    boolean reserve(String productId, int quantity);
    void release(String productId, int quantity);
}

public interface PaymentPort {
    boolean charge(String customerId, double amount);
    void refund(String customerId, double amount);
}

public interface ShippingPort {
    String schedule(String productId, int quantity, String address);
}

public interface NotificationPort {
    void sendOrderConfirmation(String customerId, String trackingNumber);
    void sendOrderFailure(String customerId, String reason);
}
```

The facade now accepts these interfaces through its constructor:

```java
public class OrderFulfillmentFacade {

    private final InventoryPort inventory;
    private final PaymentPort payment;
    private final ShippingPort shipping;
    private final NotificationPort notification;

    public OrderFulfillmentFacade(InventoryPort inventory, PaymentPort payment,
                                  ShippingPort shipping, NotificationPort notification) {
        this.inventory = inventory;
        this.payment = payment;
        this.shipping = shipping;
        this.notification = notification;
    }

    public String fulfill(Order order) {
        if (!inventory.reserve(order.getProductId(), order.getQuantity())) {
            notification.sendOrderFailure(order.getCustomerId(), "Out of stock");
            return "FAILED: out of stock";
        }

        if (!payment.charge(order.getCustomerId(), order.getTotalAmount())) {
            inventory.release(order.getProductId(), order.getQuantity());
            notification.sendOrderFailure(order.getCustomerId(), "Payment declined");
            return "FAILED: payment declined";
        }

        String trackingNumber = shipping.schedule(
                order.getProductId(), order.getQuantity(), order.getShippingAddress());

        notification.sendOrderConfirmation(order.getCustomerId(), trackingNumber);

        return "SUCCESS: " + trackingNumber;
    }
}
```

Now writing a test is straightforward. You pass in stubs or mocks for each port and verify the coordination logic without touching real payment gateways or databases:

```java
@Test
void shouldRollBackInventoryWhenPaymentFails() {
    // Given
    InventoryPort inventory = mock(InventoryPort.class);
    PaymentPort payment = mock(PaymentPort.class);
    ShippingPort shipping = mock(ShippingPort.class);
    NotificationPort notification = mock(NotificationPort.class);

    when(inventory.reserve("WIDGET-7", 3)).thenReturn(true);
    when(payment.charge("cust-42", 59.97)).thenReturn(false);

    OrderFulfillmentFacade facade = new OrderFulfillmentFacade(
            inventory, payment, shipping, notification);

    Order order = new Order("cust-42", "WIDGET-7", 3, 59.97, "123 Main St");

    // When
    String result = facade.fulfill(order);

    // Then
    assertThat(result).isEqualTo("FAILED: payment declined");
    verify(inventory).release("WIDGET-7", 3);
    verify(notification).sendOrderFailure("cust-42", "Payment declined");
    verifyNoInteractions(shipping);
}
```

This is a meaningful improvement. The facade still simplifies the caller's life, but now you can swap implementations, test in isolation, and evolve subsystems independently.

### Facade vs. Adapter vs. Mediator

These three patterns look similar at a glance because they all sit between components, but they solve different problems:

| Pattern | Purpose | Direction |
|---------|---------|-----------|
| **Facade** | Simplifies access to a subsystem by exposing a unified, higher-level interface. Clients talk to the facade; they never interact with the subsystem directly. | One-way: client -> facade -> subsystem |
| **Adapter** | Makes an existing interface compatible with a different expected interface. It wraps **one** object to translate calls, typically so that two already-built components can work together. | One-to-one translation between interfaces |
| **Mediator** | Coordinates communication between multiple peer objects so they don't reference each other directly. Each colleague talks to the mediator, and the mediator routes or orchestrates the interaction. | Many-to-many, centralized through the mediator |

A practical way to think about it: Facade reduces complexity for the caller, Adapter resolves an interface mismatch, and Mediator untangles direct dependencies between collaborating objects.

### When to Use

I reach for the Facade pattern in a few specific situations:

- **A subsystem has grown complex** and clients need a streamlined entry point. Instead of forcing callers to understand a dozen internal classes, a single facade method covers the common workflow.
- **I want to enforce encapsulation at the package level.** By keeping all implementation classes package-private and exposing only the facade as `public`, I guarantee that external code cannot bypass the intended API.
- **I need to decouple layers.** When a higher-level module depends on a lower-level subsystem, a facade serves as the boundary. If the subsystem internals change, only the facade needs updating, and callers remain untouched.
- **During iterative refactoring.** When working with legacy code, wrapping a messy subsystem behind a facade is a practical first step. It stabilizes the external contract while I clean up internals at my own pace.

### When Not to Use

The Facade pattern is not always the right call:

- **The subsystem is already simple.** If there are only one or two classes with clear APIs, adding a facade introduces an unnecessary layer of indirection. You are just wrapping something that does not need wrapping.
- **Clients genuinely need fine-grained control.** Some callers need to interact with subsystem components individually (for example, configuring a specific service or handling partial workflows). Forcing everything through a facade hides options they actually need.
- **You are using it to avoid proper design.** A facade should simplify access, not paper over a badly structured subsystem. If the internals are a mess, the real fix is refactoring the internals, not hiding them behind a nicer interface.
- **It is becoming a god object.** As discussed above, if the facade keeps growing with unrelated methods and a growing list of dependencies, it has outgrown its role. Split it or reconsider the boundaries.

> **Related posts**: [SOLID Principles](/posts/solid-the-first-5-principles-of-object-oriented-design/), [What is Coupling?](/posts/what-is-coupling/), [Inversion of Control and Dependency Injection](/posts/inversion-of-control-and-the-dependency-injection/)
