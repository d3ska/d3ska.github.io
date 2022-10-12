---
title: "JPA/Hibernate Cascade types"
categories:
  - Blog
tags:
  - JPA
  - Hibernate
  - Cascade types
---

### What is Cascading?

Cascade is the feature provided by hibernate to automatically manage the state of mapped entity whenever the state of its relationship owner entity is affected.
For example, Person-Address relationship, without Person, Address it worthless. So when we delete Person entity, Address entity should also get deleted.

**JPA Cascade Types**

All JPA-specific cascade operations are represented by the javax.persistence.CascadeType enum containing entries:

* PERSIST - save() or persist() operations cascade to related entities.

* MERGE - related entities are merged when the owning entity is merged.
* REMOVE - removes all related entities association when the owning entity is deleted.
* REFRESH - refresh does the same thing for the refresh() operation.
* DETACH - detaches all related entities if a “manual detach” occurs.
* ALL - all is shorthand for all the above cascade operations.

**Hibernate Cascade Types**

Hibernate provides three additional cascade operations which are available in org.hibernate.annotations.CascadeType:

* REPLICATE - The replicate operation is used when we have more than one data source and we want the data in sync. With CascadeType.REPLICATE, a sync operation also propagates to child entities whenever performed on the parent entity.
* SAVE_UPDATE - propagates the same operation to the associated child entity. It's useful when we use Hibernate-specific operations like save, update and saveOrUpdate. 
* LOCK - Unintuitivly, CascadeType.LOCK reattaches the entity and its associated child entity with the persistent context again.


**Remember**, There is no default cascade type in JPA. By default no operations are cascaded.


                                                                          

