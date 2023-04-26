---
title: "Strategy Design Patter"
categories:
  - Design patterns 
  - Behavioral design patterns
tags:
  - Strategy pattern
---


#### Understanding and Implementing the Strategy Design Pattern

The Strategy design pattern is a behavioral design pattern that allows you to define a family of algorithms, place each of them in a separate class, and make their objects interchangeable. This pattern simplifies the process of solving the same problem in different ways. A crucial aspect is the intention, which refers to what we want to achieve, rather than how we achieve it.

Each strategy represents a distinct approach to achieving the desired outcome.

Consider this example, we would like to implement a payment processing module.

Now, how many payment methods can you think of?
Possible payment methods may include:
* Visa Card
* Mastercard Card
* Credit Card
* PayPal
* Cash
* Cryptocurrencies

From the perspective of the person ordering a particular functionality or the code segment responsible for payment, the implementation is not important. The desired outcome is that the payment will be processed.

First, we need to create an interface for our strategy pattern example. In our case, the interface will be responsible for processing the payment amount passed as an argument.

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy1.png)

Now, we must create concrete implementations of algorithms for payment using one of the methods mentioned earlier, such as PayPal, Credit Card, or Cryptocurrencies.

**PayPal strategy**
![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy2.png)

<br>

**CreditCard strategy**
![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy4.png)

<br> 

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy6.png)

With the Strategy pattern, we can use any strategy that implements our interface. Moreover, a class behavior or algorithm can be changed at runtime. This design pattern falls under the behavioral pattern category.	 