---
title: "Unraveling the Schools of Unit Testing: The Classic and London Approaches"
categories:
  - Testing
tags:
  - Testing
  - Unit Testing
  - London School
  - Classical School
  - Mocks
---

You write a test for a service that coordinates three collaborators. To isolate the class under test, you mock all three. The test passes. A week later, someone renames a method on one of those collaborators, the compiler catches it in production code, but your mock-heavy test still passes with the old stub. The bug reaches staging. I have been in that exact situation, and the frustrating part was that I had a test covering the scenario. It just was not testing reality.

That experience forced me to think carefully about **when** mocking is the right tool and when it actively harms test quality. It turns out, the testing community has been debating this for decades under two banners: the **Classic School** (also called the "Detroit School") and the **London School**. They primarily differ in how they define the concept of **isolation** in a unit test.

### Classic School

In the **Classic School**, isolation refers to keeping individual tests independent from each other, not isolating the code under test from its collaborators. This school emphasizes running tests in parallel and in-memory, ensuring no shared state (such as databases) where one test could affect the outcome of another. The use of mocks, stubs, and test doubles should be minimal. They are only employed for dependencies that might introduce shared state and disrupt this kind of isolation.

The aim here is to use production code wherever possible. For these tests, a **unit** is defined as a class or even a set of classes. Therefore, we want to test an aggregate collectively without overutilizing mocks, stubs, and test doubles, restricting their usage to dependencies that would change the state.

### London School

In contrast, the **London School** defines a unit as a single class. The goal is to test this class in isolation, unlike the Classic School's approach that aims to test a whole component made up of classes performing a common functionality.

In the London School, we create a test implementation of a class, for instance, `CustomerVerifier`, that performs customer verification based on different implementations. The test implementation might be "always passed verification" or "always failed". Each verification type, such as by name or PESEL (Polish national identification number), would then be tested separately.

### Which School to Choose?

Both schools come with their distinct advantages and disadvantages. Here are the strengths of each approach:

#### Classic School (Detroit or Classical School)

* **Reflects Real Operation:** The Classic School's tests closely mimic the actual functioning of the system, providing a high degree of confidence in the system's real-world performance.

* **Better Coverage of Real Logic:** With a more comprehensive approach to testing, the Classic School achieves superior coverage of the real system logic, since production code is actually exercised.

* **Resilient to Refactoring:** Because Classic tests assert on outcomes rather than internal calls, you can restructure the internals of your code without breaking tests. This is a huge advantage in long-lived codebases.

#### London School

* **Isolation:** The London School excels at isolating the code for testing purposes, making it easier to pinpoint and rectify errors.

* **Simplifies Test Writing:** Utilizing test doubles and mocks helps simplify the graphs of collaborators, making the test writing process faster and easier.

* **Effective for Complex Collaborators' Graph:** This approach is especially beneficial when dealing with a complex collaborators' graph and difficult setup for tests. The extensive use of mocks enables efficient testing of the code in isolation.

#### Pilimon's Heuristic: Readability vs. Responsibility

Jakub Pilimon proposed a practical heuristic in [his presentation "Testing -- Love, Hate, Love"](https://www.youtube.com/watch?v=GjKYLmimGeE) that I have found very useful. The idea is: examine the class being tested and ask whether its dependencies were extracted for **readability** or because they carry genuinely **different responsibilities** crossing architectural layers.

If a dependency was extracted purely for readability (to keep a method short, to name a concept), use the real object. If the dependency represents a separate responsibility or layer (a repository, an external gateway, a message publisher), mocking is appropriate.

Here is a concrete example. Imagine an `OrderFulfillmentService` that depends on a `PriceCalculator` and a `ShipmentGateway`:

```java
class OrderFulfillmentService {

    private final PriceCalculator priceCalculator;
    private final ShipmentGateway shipmentGateway;

    OrderFulfillmentService(PriceCalculator priceCalculator, ShipmentGateway shipmentGateway) {
        this.priceCalculator = priceCalculator;
        this.shipmentGateway = shipmentGateway;
    }

    OrderConfirmation fulfill(Order order) {
        Money totalPrice = priceCalculator.calculate(order);
        ShipmentRequest shipment = shipmentGateway.schedule(order, totalPrice);
        return new OrderConfirmation(order.id(), totalPrice, shipment.trackingNumber());
    }
}
```

`PriceCalculator` is a pure domain class that was extracted for readability. It takes an order and computes a price. No I/O, no side effects. `ShipmentGateway`, on the other hand, is a separate responsibility: it talks to an external shipping provider over HTTP.

Applying Pilimon's heuristic:

```java
@Test
void shouldFulfillOrderWithCorrectTotalAndTracking() {
    // Given - real object for readability extraction, mock for responsibility extraction
    PriceCalculator realCalculator = new PriceCalculator(new DiscountPolicy());
    ShipmentGateway shipmentGateway = mock(ShipmentGateway.class);
    when(shipmentGateway.schedule(any(), any()))
            .thenReturn(new ShipmentRequest("TRACK-001"));

    OrderFulfillmentService service = new OrderFulfillmentService(realCalculator, shipmentGateway);

    // When
    OrderConfirmation confirmation = service.fulfill(sampleOrder());

    // Then
    assertThat(confirmation.totalPrice()).isEqualTo(Money.of(150));
    assertThat(confirmation.trackingNumber()).isEqualTo("TRACK-001");
}
```

Notice that we use the real `PriceCalculator` (readability extraction, same domain layer), but we mock the `ShipmentGateway` (separate responsibility, different layer). If `PriceCalculator` were also mocked, a bug in price computation would slip through the test undetected. If `ShipmentGateway` were real, our test would need an HTTP server running and become slow and flaky.

#### A Quick Decision Guide

When you are writing a test and unsure whether to mock a dependency, walk through these questions:

1. **Does it cross an architectural boundary?** (database, HTTP, messaging, filesystem) If yes, mock it or use an in-memory test double.
2. **Does it introduce shared mutable state?** (a cache, a singleton counter) If yes, mock it to keep tests independent.
3. **Is it a pure domain object extracted for readability?** (a calculator, a validator, a formatter with no I/O) If yes, use the real thing.
4. **Is the setup unreasonably complex?** (requires 10+ objects to construct) This might signal a design problem. Consider simplifying the design before reaching for mocks.

In my experience, most applications fall into a pattern where 70-80% of dependencies are readability extractions that should be tested with real objects, and the remaining 20-30% are infrastructure boundaries that deserve mocks.

### A Quick Comparison in Pseudocode

To illustrate the difference, consider testing an `OrderService` that depends on an `InventoryService` and a `PaymentGateway`.

**Classic (Detroit) style** -- use the real collaborators, only fake the external boundary:

```java
@Test
void shouldCompleteOrder() {
    // Given - real objects, only the database is replaced with an in-memory variant
    InventoryService inventory = new InventoryService(inMemoryInventoryRepo);
    PaymentGateway payment = new PaymentGateway(inMemoryPaymentRepo);
    OrderService orderService = new OrderService(inventory, payment);

    // When
    OrderResult result = orderService.placeOrder(sampleOrder());

    // Then
    assertThat(result.isSuccessful()).isTrue();
    assertThat(inMemoryInventoryRepo.stockOf("ITEM-1")).isEqualTo(9);
}
```

**London style** -- mock all collaborators, verify interactions:

```java
@Test
void shouldCompleteOrder() {
    // Given - all collaborators are mocked
    InventoryService inventory = mock(InventoryService.class);
    PaymentGateway payment = mock(PaymentGateway.class);
    when(inventory.reserve("ITEM-1", 1)).thenReturn(true);
    when(payment.charge(any())).thenReturn(PaymentResult.success());
    OrderService orderService = new OrderService(inventory, payment);

    // When
    OrderResult result = orderService.placeOrder(sampleOrder());

    // Then
    assertThat(result.isSuccessful()).isTrue();
    verify(inventory).reserve("ITEM-1", 1);
    verify(payment).charge(any());
}
```

Notice how the Classic test asserts on the resulting **state** (stock count decreased), while the London test verifies the **interactions** (correct methods called with correct arguments). The Classic test would catch a bug where `reserve` is called but does not actually decrement stock. The London test would not, because it never exercises the real `InventoryService`.

### Sociable vs. Solitary Tests

Martin Fowler uses a complementary vocabulary to describe the same spectrum. He calls Classic-style tests **"sociable"** because the unit under test collaborates with its real dependencies. London-style tests are **"solitary"** because every collaborator is replaced with a test double. This terminology is useful because it focuses on the characteristic of the test itself rather than on a school of thought, and it reminds us that the choice is a sliding scale rather than a binary one.

I find this vocabulary more practical in code reviews. Instead of debating "should we follow the London School here?", the question becomes: "should this test be sociable or solitary?" That framing naturally leads to examining the specific dependency in question rather than applying a blanket rule.

### Final Thoughts

After years of working with both approaches, I have a clear preference: **default to Classic (sociable) tests and reach for mocks selectively at architectural boundaries**. This is not a controversial take anymore. The authors of the Mockito mocking library themselves caution that if everything is mocked, are we truly testing production code at all? They explicitly advocate for a mixed approach.

The Classic style gives you tests that catch real bugs, survive refactoring, and serve as living documentation of how the system actually behaves. The London style earns its place when you need to isolate from slow infrastructure, verify specific interactions with external systems, or deal with genuinely complex dependency graphs that would make setup impractical.

The trap to avoid is dogmatic purism in either direction. Mocking everything creates a parallel universe of tests that pass while production breaks. Mocking nothing makes tests slow, brittle to infrastructure changes, and painful to set up. The practical middle ground, guided by heuristics like Pilimon's readability-vs-responsibility distinction, is where I have seen teams write the most effective test suites.

### References

* Martin Fowler, [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html) (2007)
* Vladimir Khorikov, *Unit Testing: Principles, Practices, and Patterns* (Manning, 2020)
* Gerard Meszaros, *xUnit Test Patterns: Refactoring Test Code* (Addison-Wesley, 2007)

> **Related posts**: [Difference between @Mock and @MockBean](/posts/difference-between-mock-and-mock-bean/)
