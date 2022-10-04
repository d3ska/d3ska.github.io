---
title: "Specification Design Pattern"
categories:
- Blog 
tags:
- Design Patterns
- Specification Design Pattern

---

### Introduction

Imagine that we have some domain objects, which can be presented, passed through or some action can be performed on them
only if they satisfied some rules, those rules may often change and vary from environment, the market, country or the provider of some service. 
 
<br>

##### What problem we're solving?

So the issue that we try to solve is **variable conditional logic**.


Following pattern will allow us to configure and combine with each other different business condition based on the
market or on the implementation of our flow. Additionally, it will be in agree with Open-Closed Principle and Liskov
Substitution Principle, we will be able to easily add other rules. But in order to achieve it, firstly we need to
extract a common interface, its important step, because if we create inappropriate interface it will force us to
refactor in the future. So make sure that you deeply understand the use case and the domain that it is embedded in.

> “The Only Constant in Life Is Change.”- Heraclitus

It also applied to business requirements :D

<br>

### Example

Assume that we are working on shipment project. The client has possibility to change the destination of the package that
he/she has sent, but only if some requirements/ rules will be satisfied. 

Firstly let's create our interface and some implementations of them.

```java
interface Rule {

    boolean isSatisfiedBy(ShipmentDetails shipmentDetails);

}


class NotShippedYet implements Rule {

    private final Collection<ShipmentStatus> statusesBeforeShipped;

    public NotShippedYet(Collection<ShipmentStatus> statusesBeforeShipped) {
        this.statusesBeforeShipped = statusesBeforeShipped;
    }

    @Override
    public boolean isSatisfiedBy(ShipmentDetails shipmentDetails) {
        return statusesBeforeShipped.contains(shipmentDetails.getStatus());
    }
}


public class NotChangedDestinationBefore implements Rule {

    @Override
    public boolean isSatisfiedBy(ShipmentDetails shipmentDetails) {
        return shipmentDetails.getHistory().stream()
                .map(History::getAction)
                .noneMatch(action -> Action.CHANGE_DESTINATION == action);
    }
}
```

We may easily create more of them if there would be such requirement and encapsulate that logic under
stable API of the interface. I mentioned that the logic may differ based on the vendor, so we can also easily implement
some factory method/ facade or set this up on the start-up for our up using Spring profiles or even configuration can be changed in runtime by loading it from DB with usage a bit of reflection, which will imitate dynamic programming. 
So as you see there is few ways in which we can handle it.
<br>

Besides, that probably there would be rules that all has to be satisfied or one of them, it can be also nicely hidden and freely combine with, thanks to AndRule/ OrRule implementation below. 

```java
class AndRule implements Rule {

    private final Set<Rule> rules;

    public AndRule(Set<Rule> rules) {
        this.rules = rules;
    }

    @Override
    public boolean isSatisfiedBy(ShipmentDetails shipmentDetails) {
        return rules.stream()
                .allMatch(rule -> rule.isSatisfiedBy(shipmentDetails));
    }
}

class OrRule implements Rule {

    private final Set<Rule> rules;

    public OrRule(Set<Rule> rules) {
        this.rules = rules;
    }

    @Override
    public boolean isSatisfiedBy(ShipmentDetails shipmentDetails) {
        return rules.stream()
                .anyMatch(rule -> rule.isSatisfiedBy(shipmentDetails));
    }
}
```


To do not leave you without any example of usage, lets take a look on a particular vendor implementation and some tests that cover 'changeDestinationTo' method.
```java
public class DpdVendor implements Vendor {

    @Override
    public boolean changeDestinationTo(ShipmentDetails shipmentDetails, Localization newDestination, Rule rule) {
        if (rule.isSatisfiedBy(shipmentDetails)) {
            //...
            return true;
        }
        //...
        return false;
    }
}
```

```java
@Test
void shouldNotBeAbleToChangeDestinationWhenPackageIsShipped(){
    //given
    var shipmentDetails=ShipmentDetails.withStatus(ShipmentStatus.SHIPPED);
    var newDestination=Localization.of(59.91,10.75);
    //and
    var noChangedDestinationBefore=new NotChangedDestinationBefore();
    var noShippedYet=new NotShippedYet(List.of(ShipmentStatus.IN_PREPARATION,ShipmentStatus.IN_WAREHOUSE));
    //and
    var andRule=new AndRule(Set.of(noChangedDestinationBefore,noShippedYet));

    //when
    var wasChanged=dpdVendor.changeDestinationTo(shipmentDetails,newDestination,andRule);

    //then
    assertFalse(wasChanged);
}
```

```java
@Test
void shouldBeAbleToChangeDestinationWhenPackageIsShippedButClientDoesNotChangedItBefore(){
    //given
    var shipmentDetails=ShipmentDetails.withStatus(ShipmentStatus.SHIPPED);
    var newDestination=Localization.of(41.90,12.49);
    //and
    var noChangedDestinationBefore=new NotChangedDestinationBefore();
    var noShippedYet=new NotShippedYet(List.of(ShipmentStatus.IN_PREPARATION,ShipmentStatus.IN_WAREHOUSE));
    var orRule=new OrRule(Set.of(noChangedDestinationBefore,noShippedYet));

    //when
    var wasChanged=dpdVendor.changeDestinationTo(shipmentDetails,newDestination,orRule);

    //then
    assertTrue(wasChanged);
}
```

<br>

#### Summarization

Specification Design Pattern is very flexible and desirable in some cases. 
**In some cases** exactly so it's worth to think twice and don't do overengineering in place where
just simple 'if's' would be more suitable.