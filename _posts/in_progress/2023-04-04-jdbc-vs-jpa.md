---
title: "JDBC vs JPA"
categories:
  - Blog
tags:
  - Spring
  - JDBC vs JPA
---

### 
JDBC - Java Database Connectivity is not being affected by Jakarta EE, things like servlets, JMS, SOAP standards, all of thoes are migrating 
over to the jakarta.* (start) namespace, but not JDBC it's part of the JDK. 



JPA - Java Persistence API - 

JDBC - manage a connection ,gathering data close connection and so on with database, but lacks of mapping and so on.

JDBC Template - you just give a query and dont need to manage a connection (Template pattern).

JPA - provides an object mapping facility (ORM) to relational databases.
 
Hibernate - default implementation of JPA, it also transle a querry written in object orienter mannder (findById) to a specific database provider, like oracle, mysql,posgresql, etc.
Decoupling from the data store. But it may limited our possibiltiy to build a query we would like as it may generete not best query possilbe (based on performance) 
, is more complex wth joins

