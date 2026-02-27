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

If you already have a monolith and decide to move toward microservices, you don't have to do a risky "big bang" rewrite. The **Strangler Fig** pattern (named by Martin Fowler after the strangler fig tree that gradually grows around its host) lets you incrementally extract functionality from the monolith into new microservices. You place a facade or API gateway in front of the monolith, and as each piece of business logic is reimplemented as a microservice, you route traffic to the new service instead of the old code. Over time the monolith shrinks until it can be decommissioned entirely -- or it remains as a smaller, more manageable core. This approach reduces risk because the old system stays operational throughout the migration, and each extracted service can be validated independently before moving on to the next.

<br>

#### Recommendation

I highly encourage you to visit the blog of [Kamil Grzybek](https://www.kamilgrzybek.com/), who is a specialist in that topic.
