---
title: "Database SQL vs NoSQL"
categories:
  - Blog
tags:
  - SQL
  - NoSQL
  - Databases
---

#### Introduce

Sometime we stand in front of decision, whether to use SQL or NoSQL Database. 
Here it is, shortly explicitly shown the difference between them and when it may be better to chose which.

<br>

#### SQL vs NoSQL


 <table style="width:100%">
  <tr>
    <th>SQL</th>
    <th>NoSQL</th>
  </tr>
  <tr>
    <td>When you access pattern* is not defined.</td>
    <td>When you access pattern* is defined.</td>
  </tr>
  <tr>
    <td>When you want to perform flexible/ relational queries.</td>
    <td>When your data model fits (for example for friends domain, graphs seems natural choice).</td>
  </tr>
  <tr>
    <td>When you want to enforce field constraints.</td>
    <td>When you need high performance and low latency.</td>
  </tr>  
  <tr>
    <td>When you want to use well documented access language (SQL).</td>
    <td></td>
   </tr>
</table> 


#### How to Pick - Example Scenarios 

*  Small project  +  Low Scale  +  Unkown Access Patterns  =  SQL
*  Large project  +  High Scale  +  Relational Queries   =   SQL with read replicas
*  Medium/Large project  +  High Scale  +  High Performance  =   NoSQL


So we may say that NoSQL fits better for larger projects and SQL are better for smaller ones, but it's general assumption and it of course depends :)



####  Access patterns matrix

It will help yo define how the users and the system access the data to satisfy business needs, and which database you should use.

 <table style="width:100%">
  <tr>
    <th>Field</th>
    <th>NoSQL</th>
  </tr>
  <tr>
    <td>Access pattern</td>
    <td>Provide a name for the access pattern.</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Provide a more detailed description of the access pattern.</td>
  </tr>
  <tr>
    <td>Priority</td>
    <td>Define a priority for the access pattern (high/medium/low). This defines the most relevant access patterns for the application.</td>
  </tr>  
  <tr>
    <td>Read or write</td>
    <td>Is it a read access or write access pattern?</td>
   </tr>
   <tr>
    <td>Type</td>
    <td>Does the pattern access a single item, multiple items, or all items?</td>
   </tr>
   <tr>
    <td>Filter</td>
    <td>Does the access pattern require any filter?</td>
   </tr>
      <tr>
       <td>Sort</td>
       <td>Does the result require any sorting?</td>
      </tr>
</table> 





