---
title: "Builder Design Patter"
categories:
  - Blog
tags:
  - Design patterns
  - Builder pattern
  - SOLID
---

### Builder is a creational design pattern that lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.
<br/>
**Problem**

Imagine you have an item that is quite complicated to create and has many possible combinations. For example, it could be a car.
Some cars will be in the basic version while others may have many additional things.<br/>You might think about creating a class with common fields that others will extend as they have additional functions.<br/>
This solution comes to us with another problem, which is many classes extending the Car class, eg CarWithAutopilot, CarWithSteeringWheel, CarInSportVersion
So it is not the optimal solution.</br>
Another idea might be to create just one class, just Car, which will have all the possibilities as fields such as has Autopilot, isSportVersion, hasSteeringWheel, additionalWarranty, soundSystem and so on.
So if we create a base car in most constructor fields we will pass null which looks messy.

**Solution**

Builder design patter suggest extraction production code from its class and place it in separate objects called builders.

**Example**

![img]({{site.url}}/assets/blog_images/2021-09-18-builder-design-pattern/builder1.png)

![img]({{site.url}}/assets/blog_images/2021-09-18-builder-design-pattern/builder2.png)

* Use the Builder pattern to get rid of a “telescopic constructor”.

* Use the Builder pattern when you want your code to be able to create different representations of some product (for example, stone and wooden houses)

* Use the Builder to construct Composite trees or other complex objects.

* Use the Builder when you want your code to be immutable.


**Pros and Cons**

 <table style="width:100%">
  <tr>
    <th>Pros</th>
    <th>Cons</th>
  </tr>
  <tr>
    <td>You can construct objects step-by-step, defer construction steps or run steps recursively.</td>
    <td>The overall complexity of the code increases since the pattern requires creating multiple new classes.</td>
  </tr>
  <tr>
    <td>You can reuse the same construction code when building various representations of products.</td>
    <td></td>
  </tr>
  <tr>
    <td>The object that you created is immutable.</td>
    <td></td>
  </tr>  
  <tr>
    <td>Single Responsibility Principle. You can isolate complex construction code from the business logic of the product.</td>
    <td></td>
   </tr>
</table> 