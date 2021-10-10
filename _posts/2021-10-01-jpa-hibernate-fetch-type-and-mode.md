---
title: "Hibernate Fetch Mode vs Fetch Type"
categories:
  - Blog
tags:
  - JPA
  - Hibernate
  - Fetch modes
  - Fetch types
---

### Fetch Mode vs Fetch Type

In general, FetchMode defines how Hibernate will fetch the data (by select, join or subselect). FetchType, on the other hand, defines whether Hibernate will load data eagerly or lazily.
The exact rules between these two are as follows:

* if the code doesn't set FetchMode, the default one is JOIN and FetchType works as defined
* with FetchMode.SELECT or FetchMode.SUBSELECT set, FetchType also works as defined
* with FetchMode.JOIN set, FetchType is ignored and a query is always eager

#### What does Fetch Mode do?

It defines how hibernate will fetch data from a database.

**FetchMode.SELECT** - generates separate query for each entity that needs to be loaded.
So for example, if we would like to load Customer which has five Addresses hibernate would generate six queries.
This is known as the n + 1 select problem. Executing one query will trigger n additional queries.

FetchMode.SELECT has an optional configuration annotation using the **@BatchSize** annotation.
Hibernate will try to load the orders collection in batches defined by the size parameter.
In our example, we have just five addresses so one query is enough.

We'll still use the same query. **But it will only be run once.** Now we have just two queries: One to load the Customer and one to load the Addresses collection.

**FetchMode.JOIN** - While FetchMode.SELECT loads relations lazily, FetchMode.JOIN loads them eagerly, using join.
This results in just one query for both the Customer and their Addresses.
But it may retrieve data duplicated.

**FetchMode.SUBSELECT** - It as well avoids the N+1 issues as well as JOIN and doesn't duplicate data but it loads all the entities of the associated type into memory.

#### What does Fetch Type do?

**FetchType.LAZY** will only fire for primary table. If in your code you call any other method that has a parent table dependency then it will fire query to get that table information. (FIRES MULTIPLE SELECT)

**FetchType.EAGER** will create join of all table including relevant parent tables directly. (USES JOIN)

**When to Use:** Suppose you compulsorily need to use dependant parent table informartion then choose FetchType.EAGER. If you only need information for certain records then use FetchType.LAZY.

**Remember**, FetchType.LAZY needs an active database session factory at the place in your code where if you choose to retrieve parent table information.
                                                                          
                                                                          
                                                                          

