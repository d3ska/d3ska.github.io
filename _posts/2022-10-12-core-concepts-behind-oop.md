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

**Object-oriented programming** is a paradigm that organizes code around objects, each combining state (fields) and behavior (methods). Objects interact through well-defined interfaces, and the paradigm builds on four core concepts: encapsulation, abstraction, inheritance, and polymorphism.

### Encapsulation

**Encapsulation** bundles an object's state with the methods that operate on it and restricts direct access to the internal data. Fields are marked `private`, and access is controlled through public methods. This protects the object's invariants: no external code can put the object into an invalid state by modifying fields directly.

```java
public class BankAccount {

    private BigDecimal balance;

    public BankAccount(BigDecimal initialBalance) {
        this.balance = initialBalance;
    }

    public BigDecimal getBalance() {
        return balance;
    }

    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        this.balance = this.balance.add(amount);
    }

    public void withdraw(BigDecimal amount) {
        if (amount.compareTo(balance) > 0) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        this.balance = this.balance.subtract(amount);
    }
}
```

The `balance` field is private. The only way to change it is through `deposit` and `withdraw`, which enforce the rules. No caller can set the balance to a negative value by accessing the field directly.

### Abstraction

Abstraction is sometimes confused with encapsulation, but they address different concerns. Encapsulation hides **data** (private fields behind public methods). **Abstraction** hides **complexity** (a complex implementation behind a simple interface).

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

The caller simply invokes `processor.process(request)` without any knowledge of Stripe, retry policies, or logging. If we later switch to a different payment gateway, the calling code remains untouched.

### Inheritance

**Inheritance** lets a class acquire the fields and methods of another class, creating an IS-A relationship. A Dog IS-A Animal. An ElectricCar IS-A Car.

```java
class Car {

    private final Propulsion propulsion;

    public Car(Propulsion propulsion) {
        this.propulsion = propulsion;
    }

    public boolean isReviewValid() {
        return ...;
    }
}

class ElectricCar extends Car {

    private final Battery battery;

    public ElectricCar(Propulsion propulsion, Battery battery) {
        super(propulsion);
        this.battery = battery;
    }

    public boolean isBatteryCharged() {
        return battery.getChargeLevel() > 0;
    }
}
```

`ElectricCar` inherits everything from `Car` (the `propulsion` field, the `isReviewValid` method) and adds its own `battery` field and method. This avoids duplicating shared logic.

Inheritance is powerful but should be used with care. Deep class hierarchies become rigid and hard to change. In many cases, composition (holding a reference to another object) is more flexible.

> **Related post**: [Prefer Composition over Inheritance](/posts/prefer-composition-over-inheritance/)

### Polymorphism

**Polymorphism** means that the same action can behave differently depending on the type of object performing it. There are two forms in Java.

**Runtime polymorphism** is the more important one. It lets you write code against a parent type and have different implementations execute depending on the actual object. This can be achieved through inheritance:

```java
public abstract class Vehicle {

    abstract void drive();
}

public class Car extends Vehicle {

    @Override
    public void drive() {
        // uses a combustion engine
    }
}

public class ElectricCar extends Vehicle {

    @Override
    public void drive() {
        // uses an electric motor
    }
}
```

Or through interfaces, which is the preferred approach in Java because a class can implement multiple interfaces (unlike single inheritance from a class):

```java
public interface Notifier {
    void send(String message);
}

public class EmailNotifier implements Notifier {

    @Override
    public void send(String message) {
        // send via SMTP
    }
}

public class SmsNotifier implements Notifier {

    @Override
    public void send(String message) {
        // send via SMS gateway
    }
}
```

In both cases, the calling code works with the abstract type and does not need to know the concrete implementation:

```java
void notifyUser(Notifier notifier, String message) {
    notifier.send(message);
}
```

The method accepts any `Notifier`. Whether it sends an email or an SMS depends entirely on the object passed in at runtime.

**Compile-time polymorphism** is method overloading: same method name, different parameter lists.

```java
public boolean validate() {
    return ...;
}

public boolean validate(Collection<Rule> rules) {
    return ...;
}
```

The compiler resolves which method to call based on the arguments provided. This is a simpler form of polymorphism compared to runtime dispatch.
