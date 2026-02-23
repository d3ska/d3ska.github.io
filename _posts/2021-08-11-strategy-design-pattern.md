---
title: "Strategy Design Pattern"
categories:
  - Design patterns
tags:
  - Strategy pattern
---


#### Understanding and Implementing the Strategy Design Pattern

The Strategy design pattern is a behavioral design pattern that allows you to define a family of algorithms, place each of them in a separate class, and make their objects interchangeable. This pattern simplifies the process of solving the same problem in different ways. A crucial aspect is the intention, which refers to what we want to achieve, rather than how we achieve it.

Each strategy represents a distinct approach to achieving the desired outcome.

Consider this example: we would like to implement a payment processing module.

Now, how many payment methods can you think of?
Possible payment methods may include:
* Visa Card
* Mastercard Card
* Credit Card
* PayPal
* Cash
* Cryptocurrencies

From the perspective of the person ordering a particular functionality or the code segment responsible for payment, the implementation is not important. The desired outcome is that the payment will be processed.

#### Why Not Just Use if/else or switch?

A naive approach to supporting multiple payment methods might look like this:

```java
public void processPayment(String method, int amount) {
    if ("paypal".equals(method)) {
        // PayPal-specific logic
    } else if ("creditcard".equals(method)) {
        // Credit card-specific logic
    } else if ("crypto".equals(method)) {
        // Crypto-specific logic
    }
    // ... and on it goes
}
```

This works for two or three methods, but it falls apart quickly. Every new payment method forces me to modify this class, violating the Open/Closed Principle. The method grows into a monolith that is hard to test, hard to read, and easy to break. The Strategy pattern eliminates this branching entirely -- each algorithm lives in its own class, and the caller simply receives the right one.

#### Structure of the Pattern

The Strategy pattern consists of three key participants:

1. **Strategy interface** -- declares the contract that all concrete strategies must fulfill.
2. **Concrete strategies** -- individual classes that implement the interface, each encapsulating a specific algorithm.
3. **Context class** -- holds a reference to a Strategy object and delegates the work to it. The context does not know which concrete strategy it is using; it only interacts through the interface.

<ADD>diagram of Strategy pattern structure showing Strategy interface, ConcreteStrategyA/B, and Context class with a strategy reference</ADD>

#### Implementation

First, I need to create an interface for the strategy. In this case, the interface is responsible for processing the payment amount passed as an argument.

```java
/**
 * Common interface for all strategies.
 */
public interface PayStrategy {

    boolean pay(int paymentAmount);
}
```

The `PayStrategy` interface defines a single method `pay(int paymentAmount)` that returns a boolean. Every concrete payment method will implement this contract.

Now, I must create concrete implementations of algorithms for payment using one of the methods mentioned earlier, such as PayPal, Credit Card, or Cryptocurrencies.

**PayPal strategy**
```java
import java.util.HashMap;
import java.util.Map;

/**
 * Concrete strategy. Implements PayPal payment method.
 */
public class PayByPayPal implements PayStrategy {
    private static final Map<String, String> DATA_BASE = new HashMap<>();
    private boolean signedIn;

    static {
        DATA_BASE.put("amanda1985", "amanda@ya.com");
        DATA_BASE.put("qwerty", "john@amazon.eu");
    }

    private boolean verify(String email, String password) {
        setSignedIn(email.equals(DATA_BASE.get(password)));
        return signedIn;
    }

    /**
     * Save customer data for future shopping attempts.
     */
    @Override
    public boolean pay(int paymentAmount) {
        if (signedIn) {
            System.out.println("Paying " + paymentAmount + " using PayPal.");
            return true;
        } else {
            return false;
        }
    }

    private void setSignedIn(boolean signedIn) {
        this.signedIn = signedIn;
    }
}
```

The `PayByPayPal` class implements `PayStrategy`. It maintains a simple in-memory user database, verifies credentials, and processes the payment only if the user is signed in.

<br>

**CreditCard strategy**
```java
/**
 * Concrete strategy. Implements credit card payment method.
 */
public class PayByCreditCard implements PayStrategy {

    private CreditCard card;

    /**
     * After card validation we can charge customer's credit card.
     */
    @Override
    public boolean pay(int paymentAmount) {
        if (cardIsPresent()) {
            System.out.println("Paying " + paymentAmount + " using Credit Card.");
            card.setAmount(card.getAmount() - paymentAmount);
            return true;
        } else {
            return false;
        }
    }

    private boolean cardIsPresent() {
        return card != null;
    }
}
```

The `PayByCreditCard` class also implements `PayStrategy`. It validates that a credit card is present, prints the payment amount, and deducts the cost from the card balance.

<br>

#### The Context: Order Class

The missing piece that ties everything together is the **Context** class. In this example, it is the `Order` class. The context does not implement any payment logic itself -- it delegates that responsibility to whatever `PayStrategy` it receives.

```java
/**
 * Order class. Doesn't know the concrete payment method (strategy) user has
 * picked. It uses common strategy interface to delegate collecting payment data
 * to strategy object. It can be used to save order to database.
 */
public class Order {
    private int totalCost = 0;
    private boolean isClosed = false;

    public void processOrder(PayStrategy strategy) {
        // Here we could collect and store payment data from the strategy.
    }

    public void setTotalCost(int cost) { this.totalCost += cost; }

    public int getTotalCost() { return totalCost; }

    public boolean isClosed() { return isClosed; }

    public void setClosed() { isClosed = true; }
}
```

The `Order` class holds `totalCost` and `isClosed` state. Its `processOrder(PayStrategy strategy)` method accepts any strategy that implements the `PayStrategy` interface. This is the core of the pattern: the `Order` does not know or care whether the payment goes through PayPal, a credit card, or cryptocurrency. It only knows that it can call the strategy to handle the payment, keeping the order logic completely decoupled from payment logic.

#### When to Use the Strategy Pattern

* **Multiple algorithms for the same task.** When I have several ways to perform an operation (sorting, compression, payment, notification) and the choice depends on context or configuration.
* **Eliminating conditional logic.** When a class contains multiple conditional branches (`if/else`, `switch`) that select behavior based on type -- each branch is a candidate for a strategy.
* **Runtime flexibility.** When I need to swap algorithms at runtime without modifying the classes that use them.
* **Isolating algorithm-specific data.** When different algorithms require different auxiliary data structures that should not leak into the main class.

#### When Not to Use It

* **Only two simple variants exist and they are unlikely to change.** The overhead of an interface and multiple classes can be overkill if the logic is trivial.
* **Clients do not need to know about different strategies.** If the algorithm selection is purely internal and static, a simpler approach (like a template method) may suffice.
* **Modern alternatives fit better.** In Java 8+, I can often pass a `Function` or lambda instead of creating a full strategy class, especially when the strategy is a single method.

With the Strategy pattern, I can use any strategy that implements my interface. Moreover, a class's behavior or algorithm can be changed at runtime. This design pattern falls under the behavioral pattern category.
