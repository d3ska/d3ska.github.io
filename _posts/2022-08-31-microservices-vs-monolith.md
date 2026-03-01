---
title: "Microservices vs Monolith vs Modular Monolith architecture"
categories:
  - Software Design
tags:
  - Software Architecture
  - Monolith
  - Microservices
  - Modular Monolith
---

#### Introduction

Nowadays most people perceive monoliths as something bad and microservices as something that must be adopted. The truth is that both have their pros and cons, and the right choice depends on the context of the project.

<br>

#### Monolith

A monolith is a system with exactly one deployment unit and one code base. All business logic, data access, and UI concerns live together inside a single deployable artifact.

![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/monolith-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/monolith-dark.png){: .dark }

##### Advantages

* **Easier deployment** -- there is just one executable file or directory to deploy.
* **Testing** -- end-to-end testing can be performed faster as all dependencies are centralized in a single unit.
* **Performance** -- a centralized code base means a single in-process call can perform the same function that would require multiple network calls in a microservices setup.
* **Debugging** -- with code located in one code base, it is easier to follow a request and find an issue.

##### Disadvantages

* **Scalability** -- you cannot scale a particular part of the system that may be a bottleneck; the entire application must be scaled as a whole.
* **Reliability** -- if there is an error in any module, it could bring the whole application down.
* **Lack of flexibility** -- the project is tightly coupled to the technology stack. A change in the framework or language affects the entire application, and you cannot choose the most suitable technology for individual parts.
* **Deployment** -- a small change requires the redeployment of the entire application.
* **Team coordination at scale** -- when many teams share a single code base, merge conflicts, broken builds, and stepping on each other's changes become a daily reality. Release coordination turns into a bottleneck: every team has to align on a single deployment schedule, and one team's delay blocks everyone else.

<br>

#### Modular Monolith

A modular monolith is still a single deployment unit, but its internals are organized into well-defined modules, each responsible for a distinct slice of business logic. Modules expose a shared API to one another but do not share their inner implementations. Each module should have high cohesion and be loosely coupled with the rest.

![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/modular-monolith-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/modular-monolith-dark.png){: .dark }

In practice, this means organizing your code into top-level packages that represent business domains, where each module communicates with others only through a public API (typically an interface):

```
com.app
├── order
│   ├── api
│   │   └── OrderService.java          // public interface, other modules depend on this
│   └── internal
│       ├── OrderServiceImpl.java       // package-private implementation
│       ├── OrderRepository.java
│       └── Order.java
├── payment
│   ├── api
│   │   └── PaymentService.java
│   └── internal
│       ├── PaymentServiceImpl.java
│       ├── PaymentRepository.java
│       └── Payment.java
└── shipping
    ├── api
    │   └── ShippingService.java
    └── internal
        ├── ShippingServiceImpl.java
        ├── ShippingRepository.java
        └── Shipment.java
```

The `api` packages contain only interfaces and DTOs. The `internal` packages hold implementations and are package-private, so no other module can reach into them directly. If the `order` module needs to charge a customer, it calls `PaymentService` (the interface), never `PaymentServiceImpl` or `PaymentRepository`. This enforces loose coupling at the code level and makes it much easier to extract a module into its own microservice later if needed.

##### Advantages compared to a classic monolith

* **Reusable modules** -- well-isolated modules can be reused across different parts of the system or even extracted into libraries.
* **Independent and interchangeable** -- modules can be replaced or upgraded without rewriting the rest of the application.
* **Better-organized dependencies** -- explicit module boundaries prevent the tangled dependency graph that classic monoliths tend to grow over time.
* **Clearer team responsibilities** -- each team owns one or more modules, which improves collaboration and accountability.

<br>

#### Microservices

Microservices take the concept of modules a step further: each service has its own code base, its own database, and its own deployment pipeline. They are small, independently deployable applications that communicate with each other over the network.

It is worth noting that microservices are not purely a technical pattern -- they are just as much an **organizational pattern**. The architecture mirrors the team structure: each service is owned by a small, autonomous team that can develop, test, and deploy independently without coordinating with every other team in the company. This aligns closely with Conway's Law, which states that systems tend to reflect the communication structures of the organizations that build them. In many cases, the primary driver for adopting microservices is not a technical limitation of the monolith, but the need to scale the engineering organization -- allowing multiple teams to work in parallel without constantly stepping on each other's toes.

![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/microservices-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/microservices-dark.png){: .dark }

##### Advantages

* **Agility** -- promote agile ways of working with small teams that deploy frequently.
* **Better scalability** -- you can scale just the service that is overloaded, rather than the entire system.
* **High reliability** -- you can deploy changes for a specific service without the threat of bringing down the entire application.
* **Technology freedom** -- in theory, each microservice can be written in a different language, use a different database, and so on.
* **Better division of responsibility** -- each team is aware of which part is their responsibility and has deep knowledge about it.
* **Independent deployments** -- only the changed services need to be deployed, not the whole application.
* **Continuous deployment** -- frequent and faster release cycles become possible.
* **Less costly withdrawing from wrong decisions** -- teams can experiment with new features and roll back if something does not work.

##### Disadvantages

* **Higher complexity** -- distributed systems are inherently harder to design, build, and reason about.
* **Dealing with potential failure** -- services communicate over the network, so there will be situations where calls between services fail. This needs to be handled with patterns like retries, circuit breakers, and fallbacks.
* **Increased network traffic** -- because services are independent units, every interaction requires a network call, which can result in slower response times compared to in-process communication.
* **Difficulty in testing and debugging** -- testing a feature that spans multiple services requires mocking or spinning up many dependencies. Debugging is harder because a single request may traverse several services.
* **Dependency on DevOps** -- in order to be successful with microservices, an organization needs a reliable DevOps team and mature tooling for deploying and managing many services.
* **Organizational challenges** -- teams need another level of communication to coordinate updates and interface changes between services.
* **Exponential infrastructure costs** -- each new microservice can have its own costs for deployment, hosting, monitoring tools, and so on.

##### The Distributed Monolith Anti-Pattern

One trap worth mentioning is the **distributed monolith**. This happens when you have multiple independently deployed services, but they are so tightly coupled that they cannot function or be deployed on their own. Typical symptoms include: services sharing the same database, changes in one service requiring synchronized deployments of several others, and teams unable to release without coordinating with other teams first.

A distributed monolith gives you the worst of both worlds. You pay the full operational cost of a distributed system (network latency, partial failures, complex debugging across services) while gaining none of the autonomy benefits that microservices are supposed to provide. If every release still requires lockstep coordination, you effectively have a monolith with network boundaries added on top.

This usually happens when teams split the monolith along technical layers (a "service" for the API, one for business logic, one for data access) rather than along business domain boundaries. It can also happen when services communicate through a shared database instead of well-defined APIs. Before moving to microservices, it is worth asking whether the team has the discipline and tooling to keep services genuinely independent. If not, a modular monolith is often the safer choice.

<br>

#### Summary

The decision of which architecture to choose depends on the context of the project. A modular monolith makes dependencies more manageable within the application and improves developer interoperability on its modular components, but for small projects the overhead may not be worth the investment.

The key downside of monoliths, both classic and modular, is that they have a single deployment unit, while microservices can be deployed independently. If you need to scale a particular part of your system -- for example, the payment pipeline -- that is impossible with a monolith. Similarly, as an organization grows and teams need more autonomy, microservices can help.

Some helping questions:
* What are we trying to achieve?
* What is the primary architectural driver, and what would be the most suitable approach?
* Is it resilience? Is it scalability? Do we actually need it yet?
* What do we want to optimize, and what would be the trade-off of that decision?

<br>

#### Migrating: The Strangler Fig Pattern

If you already have a monolith and decide to move toward microservices, you don't have to do a risky "big bang" rewrite. The **Strangler Fig** pattern (named by Martin Fowler after the strangler fig tree that gradually grows around its host) lets you incrementally extract functionality from the monolith into new microservices. You place a facade or API gateway in front of the monolith, and as each piece of business logic is reimplemented as a microservice, you route traffic to the new service instead of the old code. Over time the monolith shrinks until it can be decommissioned entirely, or it remains as a smaller, more manageable core. This approach reduces risk because the old system stays operational throughout the migration, and each extracted service can be validated independently before moving on to the next.

In practice, the implementation typically follows these steps:

1. **Place a proxy or API gateway** in front of the monolith. All client traffic flows through this gateway. Initially, the gateway simply forwards every request to the monolith unchanged.
2. **Pick one bounded context to extract.** Start with something that is relatively self-contained and has clear boundaries (for example, notifications or a reporting module). Avoid starting with the most critical or most tangled part of the system.
3. **Build the new microservice** alongside the monolith. Implement the same functionality, backed by its own data store if possible.
4. **Redirect traffic gradually.** Update the gateway routing so that requests for the extracted feature go to the new service instead of the monolith. You can use feature flags or percentage-based routing to do this incrementally and roll back quickly if something goes wrong.
5. **Decommission the old code.** Once the new service handles all traffic and has been validated in production, remove the corresponding code from the monolith.
6. **Repeat** for the next bounded context.

The key insight is that at every step the system is fully functional. There is no "halfway migrated" state where things are broken. You are always running either the old code or the new code for a given feature, never both at the same time in a confusing hybrid.

<br>

#### Database Splitting Strategy

One of the hardest parts of migrating from a monolith to microservices is untangling the database. In a monolith, all modules typically share a single database and query each other's tables freely. You cannot simply extract a service and give it its own database overnight, because dozens of joins and shared queries may depend on those tables.

A practical approach is to do this in stages:

1. **Shared database with logical boundaries.** While still in the monolith (or modular monolith), enforce that each module only accesses its own tables through its own repository layer. No cross-module table joins. If module A needs data from module B, it calls module B's API. This is a code-level discipline change, not an infrastructure change, so the risk is low.
2. **Separate schemas.** Move each module's tables into its own database schema within the same database server. This makes ownership explicit and prevents accidental cross-schema queries, while still keeping operations simple (one database server to manage).
3. **Separate databases.** Once a module is extracted into its own microservice, it gets its own database instance entirely. At this point, the service owns its data completely and can choose the database technology that fits its needs best.

This gradual approach avoids the "big bang" database migration that so often derails monolith-to-microservices transitions. Each step can be validated independently, and you can pause at any stage if the current level of separation is sufficient for your needs.

<br>

#### Recommendation

I highly encourage you to visit the blog of [Kamil Grzybek](https://www.kamilgrzybek.com/), who is a specialist in that topic.

> **Related posts**: [Request-Response vs Publish-Subscribe](/posts/request-response-vs-publish-subscribe/), [What is coupling?](/posts/what-is-coupling/), [Defining Software Architecture](/posts/defining-software-architecture/)
