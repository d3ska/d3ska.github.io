---
title: "Custom Assertions with AssertJ"
categories:
  - Testing
tags:
  - Testing
  - AssertJ
  - Custom Assertions
  - Java
---

Tests that read like domain statements catch bugs faster and survive refactoring better than tests full of low-level field checks. Custom assertions with AssertJ let you write assertions that speak the language of your business domain, and they produce failure messages that immediately tell you what went wrong.

> **Note:** AssertJ is typically already available as a transitive dependency through `spring-boot-starter-test`. If you are not using Spring Boot, add it explicitly. For Maven: `assertj-core` in `org.assertj` group. For Gradle: `testImplementation 'org.assertj:assertj-core:3.x.x'`.

### The Problem

Consider a test verifying that an order was placed correctly:

```java
@Test
void shouldCreateValidOrder() {
    // Given
    var customer = new Customer("John Doe", CustomerTier.PREMIUM);
    var items = List.of(new OrderItem("Widget", 2, Money.of("29.99")));

    // When
    var order = orderService.place(customer, items);

    // Then
    assertThat(order).isNotNull();
    assertThat(order.getStatus()).isEqualTo(OrderStatus.PLACED);
    assertThat(order.getCustomer().getFullName()).isEqualTo("John Doe");
    assertThat(order.getItems()).hasSize(1);
    assertThat(order.getItems().get(0).getProductName()).isEqualTo("Widget");
    assertThat(order.getItems().get(0).getQuantity()).isEqualTo(2);
    assertThat(order.getTotalPrice()).isEqualByComparingTo(Money.of("59.98"));
}
```

This test works, but reading it requires mentally translating `getItems().get(0).getProductName()` into "the order contains a Widget." And when it fails, the message looks like this:

```
expected: "PLACED"
 but was: "PENDING_PAYMENT"
```

That tells you which field was wrong, but not which business rule was violated. In a test suite with hundreds of assertions, these generic messages slow down debugging.

### After: Custom Assertions

With a custom assertion class, the same test becomes:

```java
@Test
void shouldCreateValidOrder() {
    // Given
    var customer = new Customer("John Doe", CustomerTier.PREMIUM);
    var items = List.of(new OrderItem("Widget", 2, Money.of("29.99")));

    // When
    var order = orderService.place(customer, items);

    // Then
    OrderAssert.assertThat(order)
            .isPlaced()
            .belongsTo("John Doe")
            .containsProduct("Widget", 2)
            .hasTotalPrice(Money.of("59.98"));
}
```

The test now reads almost like a specification: the order is placed, it belongs to John Doe, it contains two Widgets, and the total is 59.98. When it fails, the message is:

```
Expected order to be in PLACED status, but was PENDING_PAYMENT
```

That is immediately actionable.

### Building the Assertion Class

A custom assertion extends `AbstractAssert`. The two type parameters are the assertion class itself and the class under test:

```java
public class OrderAssert extends AbstractAssert<OrderAssert, Order> {

    public OrderAssert(Order actual) {
        super(actual, OrderAssert.class);
    }

    public static OrderAssert assertThat(Order actual) {
        return new OrderAssert(actual);
    }
}
```

Each assertion method checks one business condition, produces a descriptive failure message with `failWithMessage`, and returns `this` for chaining:

```java
public OrderAssert isPlaced() {
    isNotNull();
    if (actual.getStatus() != OrderStatus.PLACED) {
        failWithMessage(
                "Expected order to be in PLACED status, but was %s",
                actual.getStatus());
    }
    return this;
}

public OrderAssert belongsTo(String customerName) {
    isNotNull();
    if (!actual.getCustomer().getFullName().equals(customerName)) {
        failWithMessage(
                "Expected order to belong to <%s> but belonged to <%s>",
                customerName, actual.getCustomer().getFullName());
    }
    return this;
}

public OrderAssert containsProduct(String productName, int quantity) {
    isNotNull();
    boolean found = actual.getItems().stream()
            .anyMatch(item -> item.getProductName().equals(productName)
                    && item.getQuantity() == quantity);
    if (!found) {
        failWithMessage(
                "Expected order to contain %d x <%s>, but items were %s",
                quantity, productName, actual.getItems());
    }
    return this;
}

public OrderAssert hasTotalPrice(Money expected) {
    isNotNull();
    if (actual.getTotalPrice().compareTo(expected) != 0) {
        failWithMessage(
                "Expected total price <%s> but was <%s>",
                expected, actual.getTotalPrice());
    }
    return this;
}
```

Notice the pattern: call `isNotNull()` first so you get a clear "expected not null" message rather than a `NullPointerException`. Then check the condition, produce a message that describes the business expectation, and return `this`.

### Organizing Multiple Assertion Classes

When you have custom assertions for several domain objects, wrap the static `assertThat` methods in a single entry point:

```java
public class DomainAssertions {

    public static OrderAssert assertThat(Order actual) {
        return new OrderAssert(actual);
    }

    public static CustomerAssert assertThat(Customer actual) {
        return new CustomerAssert(actual);
    }
}
```

Import `DomainAssertions.assertThat` instead of `Assertions.assertThat` in your tests. The compiler selects the correct overload based on the argument type.

> **Tip:** For larger projects, the `assertj-assertions-generator` Maven plugin can automatically generate custom assertion classes from your domain model. This saves boilerplate when you have many domain objects to cover.

### When to Use Custom Assertions

Custom assertions earn their cost when the same set of checks on a domain object appears across many tests. If you verify `order.getStatus() == PLACED` in fifteen different test methods, an `isPlaced()` assertion pays for itself quickly. It becomes a single point of change if the status field is ever renamed or the check becomes more complex (for example, checking both status and a timestamp).

They also make sense when the thing you are checking is not a simple field comparison. Verifying that "an order contains 2 Widgets" requires navigating a collection and matching on two fields. Hiding that logic inside `containsProduct` makes every test that uses it shorter, more readable, and less error-prone.

Another strong signal: when your failure messages are confusing. If a test fails and the message says `expected: true but was: false`, nobody knows what went wrong without reading the test source. A custom assertion turns that into `Expected order to contain 2 x <Widget>`, which is understandable at a glance in a CI log.

### When to Keep It Simple

For one-off assertions or simple field checks, standard AssertJ is perfectly fine. Writing a custom assertion class for a DTO with three fields that you check in two tests is over-engineering. The maintenance burden of keeping the assertion class in sync with the domain class outweighs the readability benefit.

A reasonable heuristic: if you find yourself copying the same multi-line assertion chain into a third test, extract it into a custom assertion.

> **Related posts**: [Best Practices for Writing Effective and Reliable Tests](/posts/best-practices-for-writing-effective-and-reliable-tests/), [Schools of Unit Testing](/posts/schools-of-unit-tests/)
