---
title: "Transactions with Spring and JPA"
categories:
  - Blog
tags:
  - JPA
  - Hibernate
  - Transactions
---

### What is Transactions?

It is a sequence of actions that are treated as a single unit of work. These actions should either complete entirely or take no effect at all.
The main goal of a transaction is to provide ACID characteristics to ensure the consistency and validity of your data.

**ACID transactions**

ACID is an acronym that stands for an atomicity, consistency, isolation, and durability:

* Atomicity - each statement in a transaction (to read, write, update or delete data) is treated as a single unit. Either the entire statement is executed, or none of it is executed. This property prevents data loss and corruption from occurring if, for example, if your streaming data source fails mid-stream.
* Consistency - ensures that transactions only make changes to tables in predefined, predictable ways. Transactional consistency ensures that corruption or errors in your data do not create unintended consequences for the integrity of your table.
* Isolation - when multiple users are reading and writing from the same table all at once, isolation of their transactions ensures that the concurrent transactions don’t interfere with or affect one another. Each request can occur as though they were occurring one by one, even though they're actually occurring simultaneously.
* Durability - ensures that changes to your data made by successfully executed transactions will be saved, even in the event of system failure.

**Configure Transactions**

Spring 3.1 introduces the @EnableTransactionManagement annotation that we can use in a @Configuration class to enable transactional support.

![img]({{site.url}}/assets/blog_images/2021-10-24-transactions-with-spring-and-jpa/image1.png)


However, if we're using a Spring Boot project and have a spring-data-* or spring-tx dependencies on the classpath, then transaction management will be enabled by default.


**@Transactional annotation**

We can annotate a bean with @Transactional either at the class or method level.

The annotation supports further configuration as well:

* The Propagation Type of the transaction - Defines how transactions relate to each other
* The Isolation Level of the transaction - Defines the data contract between transactions
* A Timeout for the operation wrapped by the transaction
* A readOnly flag –  hint for the persistence provider that the transaction should be read only
* The Rollback rules for the transaction

Note that by default, rollback happens for runtime, unchecked exceptions only. The checked exception does not trigger a rollback of the transaction. We can, of course, configure this behavior with the rollbackFor and noRollbackFor annotation parameters.


**What happens in background?**

Spring's declarative transaction support uses AOP at its foundation.

Spring creates proxies for classes that declare @Transactional on the class itself or on members. The proxy is mostly invisible at runtime. It provides a way for Spring to inject behaviors before, after, or around method calls into the object being proxied.  
                                                   
So when you annotate a method with @Transactional, Spring dynamically creates a proxy that implements the same interface(s) as the class you're annotating. And when clients make calls into your object, the calls are intercepted and the behaviors injected via the proxy mechanism.   
                                                                       

**Remember**

* On whichever method you declare @Transactional the boundary of transaction starts and boundary ends when method completes.
If you are using JPA call then all commits are with in this transaction boundary.

* Do not pass entities between transaction boundaries, it is safer to pass an ID or TO.

* Entities are tracked only within the transaction in which they were retrieved.

* Entities returned from REQUIRES_NEW are detached.

