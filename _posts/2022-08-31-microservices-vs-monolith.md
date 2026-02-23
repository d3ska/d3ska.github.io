---
title: "Microservices vs Monolith vs Modular Monolith architecture"
categories:
  - Blog
tags:
  - Architecture
  - Monolith
  - Microservices
---

#### Introduction

Nowadays most people perceive monolith as something bad and microservices as something that must be.
But the truth is that both have their pros and cons, so we have to be aware of what we want to achieve.


<br>

#### Modular Monolith

The modular monolith architecture has separate modules that 'answer' for some business questions.
Those modules should have some shared, exposed API for different modules, but should not share their inner implementations and logic.
Each module should be independent and isolated; in other words, each module should have high cohesion and be loosely coupled with other modules.


##### What's the difference between monolith and modular monolith?

Monolith is a system that has exactly one deployment unit, in one code base.
Typically, it's tightly coupled.

###### Advantages of modular monolith in counter to classic monolith:

* reusable 
* independent and interchangeable 
* has better-organized dependencies
* better collaboration between teams and clearer responsibilities

![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/modular-monolith.png)
![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/monolith.png)

<br>

##### Advantages of a monolith architecture in general

* **Easier deployment** - as we have just one executable file or directory.
* **Testing** - end-to-end testing can be performed faster as all needed dependencies are centralized in a single unit.
* **Performance** - a centralized code base and repository means a single API call can perform the same function that would require numerous API calls in a microservices setup.
* **Debugging** - with code located in one code base, it's easier to follow a request and find an issue.

##### Disadvantages of a monolith architecture

* **Scalability** - we cannot scale a particular part of our system that may be a bottleneck.
* **Reliability** - if there's an error in any module, it could bring the whole application down.
* **Lack of flexibility** - we are highly coupled to the technology that we use, some changes in the framework or language affects the entire application, also we can not choose most suitable technology to some parts of our code.
* **Deployment** - small change in a monolith application requires the redeployment of the entire app.

<br>
<br>

#### Microservices

What are microservices? 
As we know already, modules in a modular monolith are responsible for some part of business logic, but are still in a single code base.
We may think of microservices as modules with the difference that each microservice has its own database and code base and is independent;
they are 'small' apps that communicate with each other.

##### Advantages of a microservices

* **Agility** – Promote agile ways of working with small teams that deploy frequently.
* **Better scalability** - you can scale just the microservice that's overloaded.
* **High reliability** - You can deploy changes for a specific service, without the threat of bringing down the entire application.
* **Possibility to choose** the most suitable technology for our microservice - in theory each microservice can be written in a different language, use a different database and so on.
* **Better division of responsibility between teams** - each team is aware which part is their responsibility and have deep knowledge about it.
* **Independent deployments** - we're able to deploy just those microservices which were changed, not the whole app like in the case of a monolith.
* **Continuous deployment** – we now have frequent and faster release cycles.
* **Less costly withdrawing** from wrong decisions - Teams can experiment with new features and roll back if something doesn't work.

<br>

##### Disadvantages of a microservices

* **Higher complexity** 
* **Dealing with potential failure** - as they are separate deployment units, they need to communicate with each other through the network. There will be situations where communication between some microservices will fail due to network issues; it needs to be handled somehow.
* **Increased Network Traffic** - as they are independent unit, designed to be self-contained they rely on network to communicate with each other. This can result in slower response times.
* **Difficulty in testing and debugging** - it can be difficult to test some functionalities in our microservices as it has a lot of dependencies that would need to be mocked. Debugging is also harder because it forces us to go through different microservices and their instances in order to find the issue.
* **Dependency on DevOps** - in order to be successful with microservice architecture, an organization needs to have a reliable, strong DevOps team. It's because there would be a need for deploying and managing microservices.
* **Organizational challenges** - teams need to create another level of communication to coordinate updates and changes between microservices.
* **Exponential infrastructure costs** - each new microservice can have it own costs for deployments, hosting infrastructure, monitoring tools and so on.


<br>


##### Summary

The decision of which architecture to choose, of course, depends... on the context of the project. A modular monolith makes dependencies more manageable within the application, improving developer interoperability on the modular components of that application, but in small projects it may not be worth the time.
The key downside of monoliths, both classic and modular, is the fact that they have one deployment unit, while microservices can be deployed independently of one another.
So in case we need to scale some particular part of our system, for example a payment system, it would be impossible with a monolith. Or if our organization grows in size and we would like to make the work of teams more agile.
The helping questions may be:
* What are we trying to achieve? 
* What is our architectural driver and what would be more suitable?
* Is it resilience? 
* Is it scalability? 
* Do we need it?
* What we want to optimize and what would be the trade-off of our decision? 

![img ]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/microservices.png)

<br>

##### Migrating: The Strangler Fig Pattern

If you already have a monolith and decide to move toward microservices, you don't have to do a risky "big bang" rewrite. The **Strangler Fig** pattern (named by Martin Fowler after the strangler fig tree that gradually grows around its host) lets you incrementally extract functionality from the monolith into new microservices. You place a facade or API gateway in front of the monolith, and as each piece of business logic is reimplemented as a microservice, you route traffic to the new service instead of the old code. Over time the monolith shrinks until it can be decommissioned entirely -- or it remains as a smaller, more manageable core. This approach reduces risk because the old system stays operational throughout the migration, and each extracted service can be validated independently before moving on to the next.

<br>

##### Recommendation

I highly encourage you to visit the blog of [Kamil Grzybek](https://www.kamilgrzybek.com/), who is a specialist in that topic.
