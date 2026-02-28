---
title: "Specification Design Pattern"
categories:
- Design Patterns
tags:
- Design Patterns
- Specification Pattern
- Behavioral Patterns
- DDD

---

Business rules rarely stay fixed. The conditions under which a shipment can be rerouted, an order approved, or a discount applied often vary by vendor, market, or regulatory environment. When those rules live as if-else chains inside business methods, every change risks breaking existing logic and every new vendor means more branching.

### The Problem: Hardcoded Conditional Logic

Imagine a shipment system where the client can change the destination of a package, but only if certain rules are satisfied. A first attempt might look like this:

```java
public class DpdVendor implements Vendor {

    @Override
    public boolean changeDestinationTo(ShipmentDetails details, Localization newDest) {
        if (!List.of(ShipmentStatus.IN_PREPARATION, ShipmentStatus.IN_WAREHOUSE)
                .contains(details.getStatus())) {
            return false;
        }
        if (details.getHistory().stream()
                .map(History::getAction)
                .anyMatch(action -> Action.CHANGE_DESTINATION == action)) {
            return false;
        }
        // ... perform the change
        return true;
    }
}
```

This works fine for one vendor. But what happens when InPost has different rules? Or when FedEx allows destination changes even after shipping, but only within the same city? You end up duplicating conditional logic across vendors, and every new rule or vendor forces modifications to existing code. This violates the Open-Closed Principle and makes it impossible to compose or swap rules at runtime.

### The Pattern

The **Specification pattern** solves this by extracting each business rule into its own class behind a common **`isSatisfiedBy`** interface. Each specification answers one question: given a candidate object, is this condition met?

```java
public interface Specification<T> {

    boolean isSatisfiedBy(T candidate);
}
```

In practice, business requirements are rarely a single condition. Sometimes all rules must hold, sometimes just one suffices, and sometimes you need the negation of an existing rule. Three generic composites cover every boolean combination:

```java
class AndSpecification<T> implements Specification<T> {

    private final Set<Specification<T>> specs;

    public AndSpecification(Set<Specification<T>> specs) {
        this.specs = specs;
    }

    @Override
    public boolean isSatisfiedBy(T candidate) {
        return specs.stream()
                .allMatch(spec -> spec.isSatisfiedBy(candidate));
    }
}

class OrSpecification<T> implements Specification<T> {

    private final Set<Specification<T>> specs;

    public OrSpecification(Set<Specification<T>> specs) {
        this.specs = specs;
    }

    @Override
    public boolean isSatisfiedBy(T candidate) {
        return specs.stream()
                .anyMatch(spec -> spec.isSatisfiedBy(candidate));
    }
}

class NotSpecification<T> implements Specification<T> {

    private final Specification<T> spec;

    public NotSpecification(Specification<T> spec) {
        this.spec = spec;
    }

    @Override
    public boolean isSatisfiedBy(T candidate) {
        return !spec.isSatisfiedBy(candidate);
    }
}
```

These composites are fully generic. They work for any type `T` and let you express arbitrary boolean logic over your specifications. The domain-specific rules only need to implement the `Specification` interface for their particular type.

### Applying It: Shipment Rules

Back to the shipment problem. Each business rule becomes a focused class implementing `Specification<ShipmentDetails>`:

```java
class NotShippedYet implements Specification<ShipmentDetails> {

    private final Collection<ShipmentStatus> statusesBeforeShipped;

    public NotShippedYet(Collection<ShipmentStatus> statusesBeforeShipped) {
        this.statusesBeforeShipped = statusesBeforeShipped;
    }

    @Override
    public boolean isSatisfiedBy(ShipmentDetails shipmentDetails) {
        return statusesBeforeShipped.contains(shipmentDetails.getStatus());
    }
}


public class NotChangedDestinationBefore implements Specification<ShipmentDetails> {

    @Override
    public boolean isSatisfiedBy(ShipmentDetails shipmentDetails) {
        return shipmentDetails.getHistory().stream()
                .map(History::getAction)
                .noneMatch(action -> Action.CHANGE_DESTINATION == action);
    }
}
```

Each rule is a small, focused class that can be tested in isolation. Adding a new rule means adding a new class, not modifying existing ones. And because the composites are generic, we can combine these rules immediately:

```java
// "allow change only if the package HAS been shipped"
Specification<ShipmentDetails> hasBeenShipped = new NotSpecification<>(
        new NotShippedYet(List.of(ShipmentStatus.IN_PREPARATION, ShipmentStatus.IN_WAREHOUSE))
);
```

With the rules extracted, the vendor implementation becomes trivial. It no longer knows or cares what the rules are. It just asks whether the given specification is satisfied:

```java
public class DpdVendor implements Vendor {

    @Override
    public boolean changeDestinationTo(ShipmentDetails shipmentDetails, Localization newDestination,
                                       Specification<ShipmentDetails> spec) {
        if (spec.isSatisfiedBy(shipmentDetails)) {
            //...
            return true;
        }
        //...
        return false;
    }
}
```

The caller decides which rules apply. Different vendors, markets, or runtime configurations can pass in different specifications without touching the vendor code.

```java
@Test
void shouldNotBeAbleToChangeDestinationWhenPackageIsShipped() {
    // Given
    var shipmentDetails = ShipmentDetails.withStatus(ShipmentStatus.SHIPPED);
    var newDestination = Localization.of(59.91, 10.75);
    // And
    var notChangedBefore = new NotChangedDestinationBefore();
    var notShippedYet = new NotShippedYet(List.of(ShipmentStatus.IN_PREPARATION, ShipmentStatus.IN_WAREHOUSE));
    // And
    var spec = new AndSpecification<>(Set.of(notChangedBefore, notShippedYet));

    // When
    var wasChanged = dpdVendor.changeDestinationTo(shipmentDetails, newDestination, spec);

    // Then
    assertFalse(wasChanged);
}
```

```java
@Test
void shouldBeAbleToChangeDestinationWhenPackageIsShippedButClientHasNotChangedItBefore() {
    // Given
    var shipmentDetails = ShipmentDetails.withStatus(ShipmentStatus.SHIPPED);
    var newDestination = Localization.of(41.90, 12.49);
    // And
    var notChangedBefore = new NotChangedDestinationBefore();
    var notShippedYet = new NotShippedYet(List.of(ShipmentStatus.IN_PREPARATION, ShipmentStatus.IN_WAREHOUSE));
    var spec = new OrSpecification<>(Set.of(notChangedBefore, notShippedYet));

    // When
    var wasChanged = dpdVendor.changeDestinationTo(shipmentDetails, newDestination, spec);

    // Then
    assertTrue(wasChanged);
}
```

### Data-Driven Specifications

The class-per-rule approach works well when rules are known at compile time. But in some systems, business rules change frequently: a new vendor is onboarded, a regulatory constraint is added for a specific market, or a promotion introduces temporary shipping conditions. Redeploying the application for every rule change is impractical.

Because the Specification pattern separates the rule definition from the evaluation, we can define rules as data and interpret them at runtime. Consider a JSON format where each node is either a composite (`AND`, `OR`, `NOT`) or a leaf rule with a field, operator, and expected values:

```json
{
  "type": "AND",
  "specs": [
    {
      "field": "status",
      "operator": "IN",
      "values": ["IN_PREPARATION", "IN_WAREHOUSE"]
    },
    {
      "field": "historyActions",
      "operator": "NONE_MATCH",
      "values": ["CHANGE_DESTINATION"]
    }
  ]
}
```

This JSON encodes the same logic as `NotShippedYet` and `NotChangedDestinationBefore` composed with `AndSpecification`. A parser walks the tree and builds the corresponding specification objects:

```java
class SpecificationParser {

    public Specification<ShipmentDetails> parse(JsonNode node) {
        String type = node.get("type").asText();

        return switch (type) {
            case "AND" -> new AndSpecification<>(parseChildren(node));
            case "OR" -> new OrSpecification<>(parseChildren(node));
            case "NOT" -> new NotSpecification<>(parse(node.get("spec")));
            default -> new FieldSpecification(
                    node.get("field").asText(),
                    node.get("operator").asText(),
                    parseValues(node.get("values"))
            );
        };
    }

    // parseChildren iterates node.get("specs") and calls parse on each
    // parseValues collects the JSON array into a Set<String>
}
```

The leaf nodes become `FieldSpecification` objects that resolve a field from the candidate and evaluate it against the operator:

```java
class FieldSpecification implements Specification<ShipmentDetails> {

    private final String field;
    private final String operator;
    private final Set<String> values;

    FieldSpecification(String field, String operator, Set<String> values) {
        this.field = field;
        this.operator = operator;
        this.values = values;
    }

    @Override
    public boolean isSatisfiedBy(ShipmentDetails candidate) {
        Object fieldValue = resolveField(candidate, field);
        return evaluate(operator, fieldValue, values);
    }

    // resolveField maps "status" -> candidate.getStatus().name(), etc.
    // evaluate applies the operator (IN, NONE_MATCH, EQUALS) to the resolved value
}
```

The key insight is that `AndSpecification`, `OrSpecification`, and `NotSpecification` are reused as-is. The parser simply constructs them from data instead of code. Rules can now live in a database, a configuration file, or an external service. When a business analyst adds a new rule or changes an existing one, the application picks it up without redeployment.

This approach does trade compile-time safety for runtime flexibility. Field names and operators become strings, so typos and invalid configurations surface at runtime rather than compile time. In practice, validation at load time and thorough integration tests help mitigate that risk.

### When to Use the Specification Pattern

The Specification pattern earns its place when business rules are a first-class domain concept that varies across contexts. If different vendors, regions, or customer tiers each have their own set of conditions, encoding those conditions as interchangeable specification objects keeps the code flexible without scattering conditional logic across the codebase. It is particularly valuable when rules need to be composed dynamically, such as requiring both "not shipped yet" and "no prior destination change" for one vendor while another vendor only needs one of those conditions. The pattern also fits naturally in systems where rules might be configured externally or swapped at runtime.

### When to Keep It Simple

Not every conditional deserves the Specification treatment. If your business logic has a small, stable set of conditions that are unlikely to change or be reused elsewhere, a straightforward if statement is clearer and more maintainable. The pattern introduces indirection (interfaces, composite classes, separate specification objects) and that indirection is only justified when it buys you genuine flexibility. When in doubt, start with the simple approach and refactor toward Specification if and when the conditional logic starts to multiply.
