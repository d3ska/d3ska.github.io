---
title: "Strategy Design Pattern"
categories:
  - Design Patterns
tags:
  - Design Patterns
  - Strategy Pattern
  - Behavioral Patterns
  - SOLID
---

Imagine a checkout service that needs to support credit cards, PayPal, and bank transfers. The first instinct is usually a chain of `if/else` blocks:

```java
public class CheckoutService {

    public void processPayment(String method, BigDecimal amount) {
        if ("creditcard".equals(method)) {
            // validate card number, expiry, CVV
            // call card network API
            // handle 3-D Secure
        } else if ("paypal".equals(method)) {
            // redirect to PayPal
            // wait for callback
            // verify token
        } else if ("bank_transfer".equals(method)) {
            // generate reference number
            // initiate SEPA transfer
        }
    }
}
```

This works when there are two methods. It stops working the moment a third or fourth arrives. Every new payment method forces a change to `CheckoutService`, violating the Open/Closed Principle. The method grows into a monolith that is hard to test, hard to read, and easy to break.

The **Strategy design pattern** eliminates this branching entirely. It is a behavioral pattern that lets you define a family of algorithms, put each one in its own class, and make them interchangeable. The caller does not know or care which concrete algorithm it is using. It only talks through the interface.

### Structure of the Pattern

The Strategy pattern has three participants:

1. **Strategy interface** -- declares the contract that all concrete strategies must fulfill.
2. **Concrete strategies** -- individual classes that implement the interface, each encapsulating a specific algorithm.
3. **Context class** -- holds a reference to a Strategy and delegates the work to it. The context never knows which concrete strategy it is using.

### Implementation

I will stick with the payment example. The goal is a design where `CheckoutService` can process a payment through any method without containing a single `if` branch.

#### The Strategy Interface

```java
public interface PaymentStrategy {

    PaymentResult pay(BigDecimal amount);
}
```

`PaymentResult` is a simple value object that tells the caller whether the payment succeeded and carries a transaction ID:

```java
public class PaymentResult {

    private final boolean success;
    private final String transactionId;
    private final String message;

    public PaymentResult(boolean success, String transactionId, String message) {
        this.success = success;
        this.transactionId = transactionId;
        this.message = message;
    }

    public boolean isSuccess() { return success; }
    public String getTransactionId() { return transactionId; }
    public String getMessage() { return message; }
}
```

#### Concrete Strategies

**Credit card payment**

```java
public class CreditCardPayment implements PaymentStrategy {

    private final String cardNumber;
    private final String expiryDate;
    private final String cvv;

    public CreditCardPayment(String cardNumber, String expiryDate, String cvv) {
        this.cardNumber = cardNumber;
        this.expiryDate = expiryDate;
        this.cvv = cvv;
    }

    @Override
    public PaymentResult pay(BigDecimal amount) {
        if (!isValidCard()) {
            return new PaymentResult(false, null, "Invalid card details");
        }

        String transactionId = "CC-" + UUID.randomUUID();
        String maskedCard = "**** **** **** " + cardNumber.substring(cardNumber.length() - 4);
        System.out.println("Charged " + amount + " to credit card " + maskedCard);
        return new PaymentResult(true, transactionId, "Credit card payment successful");
    }

    private boolean isValidCard() {
        return cardNumber != null
                && cardNumber.length() == 16
                && expiryDate != null
                && cvv != null
                && cvv.length() == 3;
    }
}
```

**PayPal payment**

```java
public class PayPalPayment implements PaymentStrategy {

    private final String email;
    private final String authToken;

    public PayPalPayment(String email, String authToken) {
        this.email = email;
        this.authToken = authToken;
    }

    @Override
    public PaymentResult pay(BigDecimal amount) {
        if (!isAuthenticated()) {
            return new PaymentResult(false, null, "PayPal authentication failed");
        }

        String transactionId = "PP-" + UUID.randomUUID();
        System.out.println("Charged " + amount + " via PayPal account " + email);
        return new PaymentResult(true, transactionId, "PayPal payment successful");
    }

    private boolean isAuthenticated() {
        return email != null && authToken != null && !authToken.isBlank();
    }
}
```

Both strategies receive everything they need through the constructor. No mutable state, no hidden `verify()` calls that clients might forget. Each class is self-contained and independently testable.

#### The Context: CheckoutService

The context does not implement any payment logic itself. It delegates to whatever `PaymentStrategy` it receives:

```java
public class CheckoutService {

    private PaymentStrategy paymentStrategy;

    public CheckoutService(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public PaymentResult checkout(BigDecimal amount) {
        System.out.println("Processing order for " + amount + "...");
        PaymentResult result = paymentStrategy.pay(amount);

        if (result.isSuccess()) {
            System.out.println("Order completed. Transaction: " + result.getTransactionId());
        } else {
            System.out.println("Payment failed: " + result.getMessage());
        }

        return result;
    }
}
```

`CheckoutService` does not know whether the payment goes through a credit card, PayPal, or anything else. It only knows that it can call `pay()` on its strategy. Adding a new payment method (say, cryptocurrency) means creating one new class that implements `PaymentStrategy`. Nothing in `CheckoutService` changes.

### End-to-End Example

Here is a complete scenario that creates a checkout, processes a payment, and swaps the strategy at runtime:

```java
public class Main {

    public static void main(String[] args) {
        // Pay with credit card
        PaymentStrategy creditCard = new CreditCardPayment("4111111111111111", "12/26", "123");
        CheckoutService checkout = new CheckoutService(creditCard);
        checkout.checkout(new BigDecimal("49.99"));

        // Customer changes their mind and wants to pay with PayPal instead
        PaymentStrategy paypal = new PayPalPayment("john@example.com", "pp-auth-token-abc");
        checkout.setPaymentStrategy(paypal);
        checkout.checkout(new BigDecimal("49.99"));
    }
}
```

Output:

```
Processing order for 49.99...
Charged 49.99 to credit card **** **** **** 1111
Order completed. Transaction: CC-a1b2c3d4-...

Processing order for 49.99...
Charged 49.99 via PayPal account john@example.com
Order completed. Transaction: PP-e5f6g7h8-...
```

The key takeaway: `CheckoutService` was never modified. The behavior changed because a different strategy was injected.

### Strategy with Lambdas (Java 8+)

Because `PaymentStrategy` has a single abstract method, it is a **functional interface**. That means I can skip the concrete class entirely and pass a lambda:

```java
CheckoutService checkout = new CheckoutService(amount -> {
    String txId = "CRYPTO-" + UUID.randomUUID();
    System.out.println("Sent " + amount + " in BTC");
    return new PaymentResult(true, txId, "Crypto payment successful");
});

checkout.checkout(new BigDecimal("0.005"));
```

This is convenient for one-off strategies, testing, or cases where the algorithm is short enough that a dedicated class would be overkill. For anything with real validation or state, I still prefer a full class. Lambdas are great for keeping things concise, but a 30-line lambda defeats the purpose.

You can also make the functional interface explicit with the `@FunctionalInterface` annotation:

```java
@FunctionalInterface
public interface PaymentStrategy {

    PaymentResult pay(BigDecimal amount);
}
```

The annotation is not required for lambda usage, but it prevents someone from accidentally adding a second abstract method and breaking all the lambda call sites.

### Strategy with Spring Dependency Injection

In a Spring application, I do not want to manually instantiate strategies with `new`. Spring can collect all `PaymentStrategy` beans into a `Map` keyed by their bean name, which makes strategy selection clean and configuration-driven.

First, mark each strategy as a Spring component with a meaningful name:

```java
@Component("creditcard")
public class CreditCardPayment implements PaymentStrategy {
    // ... same as before
}

@Component("paypal")
public class PayPalPayment implements PaymentStrategy {
    // ... same as before
}
```

Then inject them as a map in the service:

```java
@Service
public class CheckoutService {

    private final Map<String, PaymentStrategy> strategies;

    public CheckoutService(Map<String, PaymentStrategy> strategies) {
        this.strategies = strategies;
    }

    public PaymentResult checkout(String method, BigDecimal amount) {
        PaymentStrategy strategy = strategies.get(method);
        if (strategy == null) {
            throw new IllegalArgumentException("Unknown payment method: " + method);
        }
        return strategy.pay(amount);
    }
}
```

Spring automatically populates the map with all beans that implement `PaymentStrategy`. The key is the bean name (the string I passed to `@Component`), and the value is the bean instance. Adding a new payment method is now purely additive: create a new `@Component`, and it appears in the map without touching `CheckoutService`.

This is one of those spots where the Strategy pattern and dependency injection reinforce each other. The pattern defines the contract. DI handles the wiring.

### When to Use the Strategy Pattern

* **Multiple algorithms for the same task.** When I have several ways to perform an operation (sorting, compression, payment, notification) and the choice depends on context or configuration.
* **Eliminating conditional logic.** When a class contains multiple conditional branches (`if/else`, `switch`) that select behavior based on type, each branch is a candidate for a strategy.
* **Runtime flexibility.** When I need to swap algorithms at runtime without modifying the classes that use them.
* **Isolating algorithm-specific data.** When different algorithms require different auxiliary data structures that should not leak into the main class.

### When Not to Use It

* **Only two simple variants exist and they are unlikely to change.** The overhead of an interface and multiple classes can be overkill if the logic is trivial.
* **Clients do not need to know about different strategies.** If the algorithm selection is purely internal and static, a simpler approach (like a template method) may suffice.
* **A lambda will do.** In Java 8+, if the strategy is a single method with no state, passing a `Function` or lambda is simpler than creating a dedicated class.

The Strategy pattern is one of the most practical patterns I reach for. It keeps classes focused, makes testing straightforward (just pass a mock strategy), and turns conditional spaghetti into clean, extensible code.

> **Related posts**: [SOLID: The First 5 Principles of Object Oriented Design](/posts/solid-the-first-5-principles-of-object-oriented-design/), [Inversion of Control and Dependency Injection](/posts/inversion-of-control-and-the-dependency-injection/)
