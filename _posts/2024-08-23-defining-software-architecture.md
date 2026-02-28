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

Not every characteristic matters equally for every system. Part of the architect's job is identifying which characteristics are critical and which are acceptable to trade off.

### Architecture Decisions

Architecture decisions establish the rules and constraints for how a system is built. An architect might decide that only the service layer can access the database directly, or that all inter-service communication must use asynchronous messaging, or that a specific data format is required at system boundaries. These decisions guide development teams on what is permissible and what is not.

When a particular decision cannot be followed in some part of the system due to practical constraints, a **variance** is granted: a documented exception with a clear rationale.

A proven way to capture and communicate these decisions is through **Architecture Decision Records (ADRs)**. An ADR is a short document that records a single decision: the context, the decision itself, the alternatives considered, and the consequences. Over time, ADRs form a decision log that gives any team member (current or future) a clear trail of *why* the architecture looks the way it does. Tools like [adr-tools](https://github.com/npryce/adr-tools) or simply a folder of Markdown files in the repository make ADRs lightweight to maintain.

### Design Principles

Design principles are softer than architecture decisions. They are guidelines rather than hard rules. A design principle might recommend that development teams prefer asynchronous messaging over synchronous calls between microservices to improve resilience and decoupling. Unlike a strict architecture decision, a design principle acknowledges that exceptions will occur, but establishes a preferred direction.

### Architecture Governance: Fitness Functions

Defining characteristics and decisions is only half the battle. Enforcing them over time is the other half. This is where **architecture fitness functions** come in. A fitness function is any mechanism that provides an objective assessment of whether a particular architecture characteristic is being maintained:

- **Automated tests** that verify layering rules (e.g., ArchUnit tests ensuring the persistence layer never imports from the presentation layer)
- **Metrics thresholds** such as "95th percentile response time must stay below 200ms"
- **Static analysis rules** that flag cyclic dependencies between modules
- **Deployment pipeline gates** that enforce code coverage or security scan results

By embedding fitness functions in your CI/CD pipeline, you turn architecture decisions from documentation into enforceable constraints. Architecture drift, where the implemented system gradually diverges from the intended design, becomes detectable and preventable.

### References

* Mark Richards and Neal Ford, *Fundamentals of Software Architecture* (O'Reilly)
* Mark Richards and Neal Ford, *Building Evolutionary Architectures* (O'Reilly)
* Martin Fowler, [Architecture](https://martinfowler.com/architecture/)
