---
title: "Defining Software Architecture"
categories:
  - System Design
tags:
  - System Design
  - Software Architecture
  - Design Principles
---

The software community has been engaged in a longstanding debate over how to define "architecture." Some believe it refers to the underlying structure of a system or how the most important components are interconnected at a high level.

Martin Fowler shared the details of a conversation with Ralph Johnson:

> ... who questioned this phrasing, arguing that there was no objective way to define what was fundamental, or high level and that a better view of architecture was the shared understanding that the expert developers have of the system design.
> A second common style of definition for architecture is that it's "the design decisions that need to be made early in a project", but Ralph complained about this too, saying that it was more like the decisions you wish you could get right early in a project.
> His conclusion was that "Architecture is about the important stuff. Whatever that is."
>
> <cite>Martin Fowler, [Architecture](https://martinfowler.com/architecture/)</cite>

"The important stuff, whatever that is" may sound vague, but it captures something real: architecture is the set of decisions that are expensive to change later. What counts as "important" depends on the system. We can make this more concrete by breaking architecture into four components: structure, characteristics, decisions, and design principles.

### Structure

Structure refers to the architectural style (or styles) a system uses: microservices, monolith, layered, microkernel, event-driven, and so on. But naming the style alone does not describe the architecture. Saying "we use microservices" tells you something about how the system is decomposed, but nothing about its quality attributes, constraints, or trade-offs. A full picture requires the remaining three components.

### Architecture Characteristics

Architecture characteristics (often called the "-ilities") are the non-functional requirements that define what success looks like for a system, independent of its features. A payment system and a social media feed might have the same features at a high level ("process data, return results"), but their architecture characteristics differ dramatically: one prioritizes consistency and auditability, the other prioritizes latency and availability.

Common architecture characteristics include:

- Availability
- Reliability
- Testability
- Scalability
- Security
- Fault Tolerance
- Elasticity
- Performance
- Deployability

Several of these characteristics sound similar but differ in important ways:

| Characteristic pair | Distinction |
|---|---|
| **Reliability** vs **Availability** | Reliability means the system produces correct results consistently. Availability means the system is reachable and responsive. A system can be highly available yet unreliable (it responds quickly but sometimes returns wrong data), or reliable but not always available (when it does respond, the answer is correct). |
| **Scalability** vs **Elasticity** | Scalability is the system's ability to handle increased load by adding resources (planned growth). Elasticity is the system's ability to automatically scale resources up and down in response to real-time demand (dynamic, often cloud-based). A system can be scalable without being elastic if scaling requires manual intervention. |
| **Fault Tolerance** vs **Reliability** | Fault tolerance is the system's ability to continue operating when individual components fail. Reliability is broader: the system consistently behaves as expected over time. Fault tolerance is one strategy for achieving reliability, but reliability also depends on correctness, data integrity, and consistent behavior. |

Not every characteristic matters equally for every system. Part of the architect's job is identifying which characteristics are critical and which are acceptable to trade off.

### Architecture Trade-offs

No system can optimize every characteristic simultaneously. Improving one quality attribute almost always comes at the expense of another. This is one of the fundamental realities of software architecture, and recognizing it early saves teams from chasing impossible goals.

A few common trade-offs:

- **Performance vs Maintainability.** Highly optimized code (custom memory management, hand-tuned queries, denormalized data) is faster but harder to understand, modify, and test. A clean, well-abstracted design is easier to maintain but may introduce overhead through additional layers of indirection.
- **Security vs Performance.** Encrypting all data in transit and at rest, validating every input, and running authorization checks on each request all add latency. Loosening security checks improves response times but widens the attack surface.
- **Availability vs Consistency.** The CAP theorem formalizes this for distributed systems: during a network partition, you must choose between returning a potentially stale response (availability) or rejecting the request until nodes agree (consistency).
- **Scalability vs Simplicity.** A system designed for massive scale (event-driven, eventually consistent, heavily partitioned) is more complex to build, debug, and reason about than a straightforward monolith with a single relational database.

The architect's role is not to eliminate trade-offs but to make them visible and intentional. Every decision should be traceable to a conscious choice about which characteristics matter most for the system at hand.

### Architecture Decisions

Architecture decisions establish the rules and constraints for how a system is built. An architect might decide that only the service layer can access the database directly, or that all inter-service communication must use asynchronous messaging, or that a specific data format is required at system boundaries. These decisions guide development teams on what is permissible and what is not.

When a particular decision cannot be followed in some part of the system due to practical constraints, a **variance** is granted: a documented exception with a clear rationale.

To make this concrete, consider a common decision: should two services communicate synchronously (HTTP/REST) or asynchronously (message queue)?

```java
// Synchronous: Order service calls Inventory service directly
public class OrderService {

    private final InventoryClient inventoryClient;

    public OrderResult placeOrder(Order order) {
        // Blocks until Inventory responds. Simple, but if Inventory
        // is down, Order fails too (tight coupling, lower fault tolerance).
        InventoryResponse response = inventoryClient.reserve(order.getItems());
        if (!response.isSuccess()) {
            return OrderResult.rejected("Items not available");
        }
        return OrderResult.confirmed(order);
    }
}

// Asynchronous: Order service publishes an event
public class OrderService {

    private final EventPublisher eventPublisher;

    public OrderResult placeOrder(Order order) {
        // Returns immediately. Inventory service picks up the event
        // independently. More resilient, but the caller does not get
        // an immediate confirmation of inventory availability.
        eventPublisher.publish(new OrderPlacedEvent(order));
        return OrderResult.accepted(order);
    }
}
```

Choosing synchronous communication favors simplicity and immediate consistency. Choosing asynchronous communication favors fault tolerance and scalability, at the cost of eventual consistency and added complexity. Neither is universally better. The right choice depends on which architecture characteristics the system prioritizes.

A proven way to capture and communicate these decisions is through **Architecture Decision Records (ADRs)**. An ADR is a short document that records a single decision: the context, the decision itself, the alternatives considered, and the consequences. Over time, ADRs form a decision log that gives any team member (current or future) a clear trail of *why* the architecture looks the way it does. Tools like [adr-tools](https://github.com/npryce/adr-tools) or simply a folder of Markdown files in the repository make ADRs lightweight to maintain.

### Design Principles

Design principles are softer than architecture decisions. They are guidelines rather than hard rules. A design principle might recommend that development teams prefer asynchronous messaging over synchronous calls between microservices to improve resilience and decoupling. Unlike a strict architecture decision, a design principle acknowledges that exceptions will occur, but establishes a preferred direction.

### Architecture Governance: Fitness Functions

Defining characteristics and decisions is only half the battle. Enforcing them over time is the other half. This is where **architecture fitness functions** come in. A fitness function is any mechanism that provides an objective assessment of whether a particular architecture characteristic is being maintained:

- **Automated tests** that verify layering rules (e.g., ArchUnit tests ensuring the persistence layer never imports from the presentation layer)
- **Metrics thresholds** such as "95th percentile response time must stay below 200ms"
- **Static analysis rules** that flag cyclic dependencies between modules
- **Deployment pipeline gates** that enforce code coverage or security scan results

Here is a practical example using **ArchUnit** to enforce a layering rule. This test ensures that domain classes never depend on infrastructure classes, protecting the core of your system from leaking implementation details:

```java
@AnalyzeClasses(packages = "com.example.myapp")
class ArchitectureRulesTest {

    @ArchTest
    static final ArchRule domain_should_not_depend_on_infrastructure =
        noClasses()
            .that().resideInAPackage("..domain..")
            .should().dependOnClassesThat()
            .resideInAPackage("..infrastructure..");

    @ArchTest
    static final ArchRule controllers_should_not_access_repositories_directly =
        noClasses()
            .that().resideInAPackage("..controller..")
            .should().dependOnClassesThat()
            .resideInAPackage("..repository..");
}
```

These tests run as part of your normal test suite. If someone introduces a dependency that violates the rule, the build fails immediately rather than letting the violation slip through unnoticed.

By embedding fitness functions in your CI/CD pipeline, you turn architecture decisions from documentation into enforceable constraints. Architecture drift, where the implemented system gradually diverges from the intended design, becomes detectable and preventable.

> **Related posts**: [Microservices vs Monolith vs Modular Monolith architecture](/posts/microservices-vs-monolith/), [Cohesion: Measuring Module Design Quality](/posts/cohesion/), [What is coupling?](/posts/what-is-coupling/)

### References

* Mark Richards and Neal Ford, *Fundamentals of Software Architecture* (O'Reilly)
* Mark Richards and Neal Ford, *Building Evolutionary Architectures* (O'Reilly)
* Martin Fowler, [Architecture](https://martinfowler.com/architecture/)
