---
title: "What is coupling?"
categories:
  - Blog
tags:
  - Metrics
  - Design
  - Coupling
---

In object-oriented paradigm, Coupling refers to the degree of direct knowledge that one element has of another. We may generally divide coupling to **tight** and **loose** one.

<br>
We may refer for an example of those to monolith architecture, we may generally say that classic monolith is rather tight coupled, when the modular is loosely coupled (if the module are separated correctly).
But let's think about less abstract example. A phone and a remote. What would you do if your battery in a new iPhone dies? You would probably get the new one because replacing it is difficult and costs a lot. 
In the other hand what would you do if you noticed that your remote is not working, you will get new batteries and replace it in a seconds.

<br>
<br>

![img]({{site.url}}/assets/blog_images/2022-10-05-what-is-coupling/coupling-ilustration.png)

<br>

##### Why coupling is bad, most of the time?

* Its make refactor or any change of 'HOW' we implement something more difficult
* We are **coupled** to concrete implementation of 'WHAT' our system is doing.
* Change in one place can be a cause of a bug in the other place, which should be completely independent
* Make testing and debugging harder
* Introducing new feature will last much longer
* Code got duplicated as it's part are hard to reuse.

##### Different look on coupling

But mentioned earlier thought and loose coupling, what does it mean in context of our code? <br>
I got much better understanding of it, and how to think about it, after watching a conference talk made by Łukasz Szydło
which can be found [here](https://www.youtube.com/watch?v=Jy6eS9QHJOM).


Let's briefly go through that concept. <br>
So we may separate a coupling in our code on four levels, starting from tightly coupled to loosely coupled one. We may
define on which level our code is by answering for following questions (levels) in context of our piece of code that we
decide to work on.

**How** it's done? <br>
**Where** it took place? <br>
**Who** is doing it? <br>
**What** happened? <br>

We would strive to situation where our part of system can not answer for at least 3 of those (How? Where? Who?).


<table style="width:100%">
  <caption>Coupling levels</caption>
  <tr>
    <th></th>
    <th>Local method</th>
    <th>Local instance</th>
    <th>External instance</th>
    <th>Configurable instance (DI)</th>
    <th>Notification</th>
  </tr>
  <tr>
    <td>How?</td>
    <td style="text-align:center; color: green">V</td>    
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
  </tr>
  <tr>
    <td>Where?</td>
    <td style="text-align:center; color: green">V</td>    
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
  </tr>
  <tr>
   <td>Who?</td>
    <td style="text-align:center; color: green">V</td>    
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
  </tr>  
  <tr>
    <td>What?</td>
    <td style="text-align:center; color: green">V</td>    
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
   </tr>
</table> 



