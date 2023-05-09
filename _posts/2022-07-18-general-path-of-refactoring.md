---
title: "How to think and approach refactoring?"
categories:
  - Blog
tags:
  - Refactoring
  - Clean code
---

### Introduce

When we hear refactoring we often think about legacy code, and the other way
around. What is legacy then? Well, in our context I would say that it is a code where we
feel fear of change. But it is not a sentence, and we may refactor real legacy system
without feeling overwhelmed and without feeling fear, if we approach it properly.


### Refactoring vs Rewrite

Firstly, what does refactoring mean? The definition that I think is the most
appropriate is as follows “It changes the structure of the code without changing the observable behavior.”

What is the rewrite then?

In fact, we can make some rewrites during refactoring, for example while we would
refactor a class, we rewrite a function, or during refactoring of a module, we may
rewrite a class, so in other words, rewrite of some component is actually a refactor for
the higher order component, 'a parent'.


### Why we often feel uncertain when thinking of refactoring

* Code difficult to understand
* High coupling of system components
* Concentration of knowledge between a small group of people
* Various ways of communicating with the people involved in the projects.
* Omitting information about relational business needs.


### How to think about and when to decide for refactoring?

Many people think of refactoring solely in the context of clean code. Don't get me wrong, clean code is undoubtedly important and makes our lives much easier. However, considering refactoring solely for clean code is an oversimplification. A good analogy for refactoring is viewing it as an investment with a return on that investment (ROI). Many programmers are perfectionists and strive for perfection in every case, but does this always provide real value and a great ROI, or does it just satisfy our sense of perfectionism?

Let's discuss a theoretical example: You've refactored a part of a microservice that was previously unreadable and hard to understand at first glance. It took a week to complete, and you feel really great about it. It is an undeniable fact that the refactored part is in better shape than before, but is it always worth the effort? Assume that previously, it took a programmer about 30 minutes to figure out what was happening, which function was responsible for what, and so on. After refactoring, it now takes only 10 minutes, saving an average of 20 minutes - a pretty good result, right?

However, upon checking the repository, we discover that the last change in that part of the system took place several months or even years ago. What does this imply? It means that this part of the system is well-established in production, stable, and rarely changed. Even if we optimistically assume that developers visit this part of the system twice a week and need to understand what it does, is the ROI from the refactoring worth it in the long run?


### Time for quick math

Time to needed to understand the code before refactoring: **~30 min**

Time spent on refactoring: **1 work week = 40h = 2400 min**

Time to needed to understand the code after refactoring: **~10 min**

Saved time on average: **~20 min**

When we analyze it in this manner, we can see that our 'investment' **starts to pay off after 60 weeks...**
In this case, one might argue that it was a rather poor investment. However, it was certainly satisfying from a purist's point of view, which most of us are, to some extent.


### How to approach refactoring to make it less intimidating:

We've made a conscious decision to refactor our system. What's next? I suggest answering the following questions and taking these steps, which I personally find helpful.

1. **What problem are we trying to solve? Name it.**<br>
We can't proceed with refactoring if we don't know our driving force. We need to identify what we want to improve or eliminate. Is it higher testability, readability, better latency, or speeding up the time required to introduce new features by reducing coupling and making teams more autonomous, among other things? So, once again, name the problem you're trying to solve.


2. **Choose the appropriate solution.** <br>
There are likely several solutions or ways to achieve our goal. In such cases, consider the different possibilities for addressing the problem and select the one that best suits your driver and context. What are the pros and cons of each, and what are the trade-offs?


3. **Plan the changes.** <br>
Identify what needs to be changed and where. Will it affect other parts of the system? Determine where to start and finish the refactoring. Be aware of the 'reaction chain' in the context of refactoring. It is essential to begin with the most challenging part of the refactoring plan, also known as the “fail fast” approach. This way, we'll discover sooner if we can complete what we've planned. If our refactoring turns out to be impossible for some reason, we'll likely find out early rather than at the end.<br>
Answer the following questions to help guide your approach:
   * Where should the change be made?
   * What is the impact of the change?
   * What are the observable behaviors of the change?


4. **Ensure the safety of the change.** <br>
Refactoring involves changing the code's structure without altering its observable behavior. To make sure the observable behavior doesn't change, confirm that proper tests covering the behavior exist; if not, write them. The more legacy and coupled a system is, the more likely it is that the only sensible solution is to write tests at higher layers, like controllers.


5. **Perform the refactoring.** <br>
After ensuring the safety of the change by creating tests for observable behaviors, proceed with the refactoring, running tests periodically to ensure nothing breaks and that you're moving in the right direction.


6. **Observe the effect.** <br>
After completing the planned changes, step back and evaluate the process from a broader perspective. Answering the following questions may be helpful:
   * What has changed?
   * Will this change make future features easier or harder to implement?
   * Were there any trade-offs?
   * What is the ROI?

Keep in mind that as you refactor, you might need to take one step back, which may initially make your code seem "worse." However, such temporary changes are sometimes necessary and will enable you to complete the entire refactor later, ultimately resulting in progress. As Martin Fowler said, “For each desired change, make the change easy (warning: this may be hard), then make the easy change.”


### Example

Quick note at the beginning: The code is not 'clean' and has been simplified on purpose for the sake of an example.

In almost every system, there is a concept of money or some form of payments and costs. After all, most businesses aim to earn revenue 😅. Let's assume that our application is a 'taxi app,' and the current implementation uses a Double to represent the concept of money. So far, it has been working well, but a new requirement is on the horizon, stating that our system will be released in different markets and countries. As you can imagine, in countries with different currencies and varying decimal places (e.g., 0 in yen or 3 like in dinar in many countries), there will be a need for money conversion, awareness of currency types, and so on. The current implementation is not equipped to handle these requirements.

**First, we need to identify the problem we are solving.** In this case, it is the variable representation of the business concept, which is money. **What solution would be most suitable?**

Money is a widely used concept in our system, so if we don't implement it behind some abstraction, the coupling between the representation of money and the places where it is used will be high, and the cohesion will be low. However, we want the opposite: high cohesion and low coupling. A good idea would be to create a Money object, which will only share a common API, while hiding everything else and making it inaccessible.

```java
public class Money {
   public static final Money ZERO = new Money(0.0);
   private Double value;

   public Money() {
   }

   public Money(Double value) {
      this.value = value;
   }

   public Money add(Money other) {
      return new Money(value + other.value);
   }

   public Money subtract(Money other) {
      return new Money(value - other.value);
   }

   public Double asDouble() {
      return value;
   }
}
```


**Think about where we want to make the change and what the observable behavior is**. As we mentioned earlier, money is a widely used concept, and for various reasons, we don't want to refactor it all at once. First, we want to introduce the new concept to some parts of our system and observe how it behaves. We've decided to proceed with the change in the context of calculating fees for the driver, journey, and so on.

```java
public Double calculateDriverFee(Long journeyId) {
    Journey journey = journeyRepository.getOne(journeyId);
    Double journeyPrice = journey.getPrice();
    DriverFee driverFee = driverFeeRepository.findById(journey.getDriverId());
    Double finalFee;
    if (driverFee.getFeeType().equals(DriverFee.FeeType.FLAT)) {
        finalFee = journeyPrice - driverFee.getAmount();
    } else {
        finalFee = journeyPrice * driverFee.getAmount() / 100;
    }
        
   return new Money(Math.max(finalFee, driverFee.getMin() == null ? 0 : driverFee.getMin()));
}
```

**Ensure the safety of the change:** Unfortunately, we didn't find any tests for that function in our system. Therefore, we need to create new ones to ensure the safety of the change.

```java
@Test
void shouldCalculateDriversFlatFee() {
   //given
   Driver driver = testDataProvider.aDriver();
   Journey journey = testDataProvider.aJourney(driver, 60);
   testDataProvider.driverHasFee(driver, DriverFee.FeeType.FLAT, 10);
        
   //when
   Double calculatedFee = driverFeeService.calculateDriverFee(journey.getId());
        
   //then
   assertThat(50).isEqualTo(calculatedFee);
}
```
So, we created tests for all observable behaviors of the part we're refactoring, and all of them are green. Now, we have our 'safety net,' and we are secured in some way.

**Perform the refactoring:** Let's proceed with changing our implementation and introducing the new concept of money.

```java
public Money calculateDriverFee(Long journeyId) {
    Journey journey = journeyRepository.getOne(journeyId);
    Money journeyPrice = journey.getPrice();
    DriverFee driverFee = driverFeeRepository.findById(journey.getDriverId());
    Money finalFee;
    if (driverFee.getFeeType().equals(DriverFee.FeeType.FLAT)) {
        finalFee = journeyPrice.subtract(new Money(driverFee.getAmount()));
    } else {
        finalFee = journeyPrice.percentage(driverFee.getAmount());
    }
    
    return new Money(Math.max(finalFee.asDouble(), driverFee.getMin() == null ? 0 : driverFee.getMin()));
}
```

We managed to add a new value object, which is money, but tests are now failing because the API of the method has changed. We need to adjust our tests accordingly. Keep in mind that we should not change the tests and the real implementation simultaneously, but in this case, the change is trivial, so we can do it while remaining cautious.

```java
@Test
void shouldCalculateDriversFlatFee() {
   //given
   Driver driver = testDataProvider.aDriver();
   Journey journey = testDataProvider.aJourney(driver, 60);
   testDataProvider.driverHasFee(driver, DriverFee.FeeType.FLAT, 10);
   
   //when
   Money calculatedFee = driverFeeService.calculateDriverFee(journey.getId());
   
   //then
   assertThat(new Money(50)).isEqualTo(calculatedFee);
}
```

<br>

**Tests are green, and our new concept has been successfully connected!** 

Remember, we only refactored some parts of our system, not everything. So, what will we do with clients and different parts of the system where a Double is still used to represent money? You may have noticed the 'asDouble' method, which simply returns the inner value of Money. This simple method allows us to decide where we want to stop our refactoring while maintaining backward compatibility.

Imagine that clients of the calculateDriverFee function, who further process the value, are not yet adapted to the concept of money. In that case, we can easily maintain backward compatibility by calling the asDouble() method. This approach means we are not forced to refactor the entire system at once, especially if it's highly coupled at the moment. By strategically implementing changes and using methods like asDouble(), we can make the refactoring process more manageable while preserving the functionality of the system.


**Observe the effect** 

We changed the concept of money from Double to the Money object, and now Double is just an implementation detail. As a result, we can easily change the inner implementation to BigDecimal or any other type we want, since it's hidden behind a stable API and doesn't share its inner implementation. This approach would require modifications more or less in just one place - our Money class (besides the parts where we decided to cut off our changes, if there are any).

Now, introducing the concept of currencies, decimal places, and so on becomes much easier, safer, and more pleasant 🙂. By using the Money object, we have established a more robust foundation for handling different monetary scenarios and adapting to new requirements. This refactoring not only improves the overall quality of the code but also increases the maintainability and flexibility of the system in the long run.