---
title: "Microservices vs Monolith vs Modular Monolith architecture"
categories:
  - Blog
tags:
  - Architecture
  - Monolith
  - Microservices
---

#### Introduce

Now days most of the people are perceiving monolith as something that's necessarily bad and microservices as something that's must be.
But the truth is that both have some pros and cons, so we have to be aware of what we want to achieve.


<br>

#### Modular Monolith

The modular monolith architecture has separate module that 'answers' for some business logic. 
Those modules should have some shared API for different modules, but it should answer the business matters for which that particular module 
is responsible, without sharing its inner implementations and logic. Each module should be independent and isolated in other words.


##### What's the difference between monolith and modular monolith?

Monolith is a system that has exactly one deployment unit, in one code base.
Typically, it's tightly couples. But modular monolith offer some advantages in counter to classic monolith, it's:
* reusable 
* independent and interchangeable 
* has better-organized dependencies

![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/modular-monolith.png)
![img]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/monolith.png)


##### Advantages of a monolith architecture

* **Easier deployment** - as we have just one executable file or directory.
* **Testing** - end-to-end testing can be performed faster as all needed dependencies are centralized in a single unit.
* **Performance** - centralized code base and repository can make that one API can achieve to perform the same function that numerous APIs perform with microservices
* **Debugging** - with code located in one code base, it's easier to follow a request and find an issue.

##### Disadvantages of a monolith architecture

* **Scalability** - we can not scale particular part of our system that may be 'bottleneck'. 
* **Reliability** - if there's an error in any module, it could make whole application down
* **Lack of flexibility** - we are highly coupled to the technology that we use, some changes in the framework or language affects the entire application, also we can not choose most suitable technology to some parts of our code.
* **Deployment** - small change in a monolith application requires the redeployment of the entire app.

<br>
<br>

#### Microservices

What are microservices? 
As we know already modules in modular monolith answers for some part of business logic, but are still in single code base.
We may think of it as modules with that difference that each microservice has its own database, code base and is independent,
there are 'small' apps that communicate with each others. 

##### Advantages of a microservices

* **Agility** – Promote agile ways of working with small teams that deploy frequently.
* **Better scalability** - you can scale just the microservice that's overloaded.
* **High reliability** - You can deploy changes for a specific service, without the threat of bringing down the entire application.
* **Possibility to chose** most suitable technology to our microservice - in theory each microservice can be written in different language, use different database and so on.
* **Better division of responsibility between teams** - each team is aware which part is their responsibility and have deep knowledge about it.
* **Independent deployment's** - we're  able to deploy just those microservices which were changed, not whole app like in case of monolith. 
* **Continuous deployment** – we now have frequent and faster release cycles.
* **Less costly withdrawing** from wrong decisions - Teams can experiment with new features and roll back if something doesn't work.

<br>

##### Disadvantages of a microservices

* **Higher complexity** 
* **Increased Network Traffic** - as they are independent unit, designed to be self-contained they rely on network to communicate with each other. This can result in slower response times.
* **Difficulty in testing and debugging** - it can be difficult to test some functionalities in our microservices as it has a lot of dependencies that would need to be mocked. Debugging is also harder because it forces us to go through different microservices and their instances in order to find the issue.
* **Dependency of DevOps** - in order to be successful with microservice architecture, organization need to have a reliable, strong DevOps team. It's because there would be need for deploying and managing microservices.
* **Organizational challenges** - teams need to create another level of communication to coordinate updates and changes between microservices.
* **Exponential infrastructure costs** - each new microservice can have it own costs for deployments, hosting infrastructure, monitoring tools and so on.





##### Summarization

Decision of which architecture choose, of course depends... on the context of the project. Modular monolith is making dependencies more manageable within the application, improving developer interoperability on the modular components of that application, but in small projects it may be not worth the time.
The key downside of monoliths, both, classic one and modular, is the fact that the have one deployment unit while microservices can be deployed independently of one another.
So in case if we need scale some particular part of our system, for example a payment system, it would be impossible. Or if our organization growth in size, and we would like to make work of teams more agile. 
The helping questions may be:
* What are we trying to achieve? 
* Is it resilience? 
* Is it scalability? 
* Do we need it?
* What we want to optimize and what would be the trade-off of our decision? 

![img ]({{site.url}}/assets/blog_images/2022-08-31-microservices-vs-monolith/microservices.jpg)




##### Recommendation

I highly encourage you to visit blog of [Kamil Grzybek](https://www.kamilgrzybek.com/), which is specialist in that topic.
