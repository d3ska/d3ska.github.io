---
title: "Core concepts behind OOP"
categories:
  - Software Design
tags:
  - OOP
  - Abstraction
  - Encapsulation
  - Inheritance
  - Polymorphism
---

### What does object-oriented programming mean?

It is a programming paradigm in which we define objects with their behaviors and attributes. These objects communicate with each other, making our application work.


### Core concepts of Object-Oriented Programming

##### Polymorphism

Polymorphism is the ability of a class to provide different implementations of a method, based on the type of object that has been provided.
In other words, we can perform the same action in multiple forms or ways.<br>
Let's take a look at an example of code that uses inheritance to achieve it:
``` java
public abstract class Vehicle {
    abstract void drive();
}

public class Car extends Vehicle {

    @Override
    public void drive(){
        // uses a classic engine
    }
}
public class ElectricCar extends Vehicle {

    @Override
    public void drive(){
        // uses an electric drive
    }
}

-----------------------------------
Vehicle car = new Car();
Vehicle electricCar =  new ElectricCar();

driveToTheChosenPlace(Vehicle vehicle){
    ...
    vehicle.drive();
}
```

Notice that we can perform the same action in different ways depending on the implementation, as long as both types share a common parent -- both Car and ElectricCar **IS-A** Vehicle.
Clients of that code do not know anything about the inner implementation, and they don't need to.

The Car class drives using a classic engine, while the ElectricCar uses an electric drive.

Worth mentioning is the fact that in this case polymorphism is achieved through inheritance.
In practice, inheritance and polymorphism are used together in Java to achieve fast performance and readability of code.

But polymorphism can be achieved without inheritance, for example by having the same method name with different signatures performing different actions.

This type of polymorphism is **static or compile-time polymorphism, and it is basically method overloading.**
```java
public boolean validate(){
    return ...;
}

public boolean validate(Collection<Rule> rules){
        return ...;
}
```

There is also **runtime or dynamic polymorphism, which is achieved by a child class that overrides the parent's method:**
```java
public class GenericFile {

    public String getFileInfo() {
        return "Generic File Impl";
    }
}

public class ImageFile extends GenericFile {

    @Override
    public String getFileInfo() {
        return "Image File Impl";
    }
}
```
<br>

##### Inheritance

It is a mechanism that allows us to acquire fields and methods of another class by inheriting from it.
By extending, we create an IS-A relationship: Dog IS-A Animal.
Thanks to this, code becomes more reusable and shorter in OOP.

```java
// base class
class Car {
    private Propulsion propulsion;
    ...

    public Car(Propulsion propulsion, ...){
        this.propulsion = propulsion;
        ...
    }

    public boolean isReviewValid(){
        return ...;
    }

}

class ElectricCar extends Car {

    // the ElectricCar subclass adds one more field
    private Battery battery;

    public ElectricCar(Propulsion propulsion, Battery battery){
        super(propulsion, ...);
        this.battery = battery;
    }

    // the ElectricCar subclass adds one more method
    public boolean isBatteryLoaded(){
        return ...
    }
}
```

<br>

##### Encapsulation

Encapsulation hides the state and inner implementation of an object from the clients of an API, making it accessible only through publicly provided methods.
This allows us to protect specific information and control access to internal implementation details.

For example, member fields in a class are hidden from other classes, and their behaviors can be accessed only through the exposed public methods.
```java
public class Car extends Vehicle {
    private Engine engine;
    private Gear gear;

    public EngineType getEngineType(){
        return engine.getType();
    }
}
```

##### Abstraction
Abstraction hides the complexities of implementation and exposes simpler interfaces.

Imagine that you are driving a car. You use the gas pedal, the brake, the steering wheel, and maybe a few other controls. But you don't know what is happening under the hood -- how gears are changed, how the engine works, and so on. That inner implementation is hidden from the driver.

In OOP, we hide complex implementation details and expose only a stable API that consumers need in order to use the functionality.
In Java, abstraction is commonly achieved through interfaces and abstract classes.

Consider a payment processing system. The caller only sees a clean `PaymentProcessor` interface, while the complexities of communicating with external payment gateways, handling retries, and managing transaction state are hidden behind the implementation:

```java
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
}

public class StripePaymentProcessor implements PaymentProcessor {

    private final StripeClient stripeClient;
    private final RetryPolicy retryPolicy;
    private final TransactionLogger logger;

    public StripePaymentProcessor(StripeClient stripeClient,
                                   RetryPolicy retryPolicy,
                                   TransactionLogger logger) {
        this.stripeClient = stripeClient;
        this.retryPolicy = retryPolicy;
        this.logger = logger;
    }

    @Override
    public PaymentResult process(PaymentRequest request) {
        // All the complexity is hidden behind this simple method:
        // - input validation
        // - currency conversion
        // - communication with Stripe API
        // - retry logic on transient failures
        // - transaction logging
        return retryPolicy.execute(() -> stripeClient.charge(request));
    }
}
```

The caller simply invokes `processor.process(request)` without any knowledge of Stripe, retry policies, or logging. If we later switch to a different payment gateway, the calling code remains untouched -- we just provide a different implementation of `PaymentProcessor`.
