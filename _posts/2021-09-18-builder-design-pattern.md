---
title: "Builder Design Pattern"
categories:
  - Design Patterns
tags:
  - Design Patterns
  - Builder Pattern
  - Creational Patterns
  - SOLID
---

The **Builder** is a creational design pattern that lets you construct complex objects step by step, producing different types and representations of an object using the same construction code.

### The Problem: Telescoping Constructors

Imagine you need to create an object that is complex and comes in many configurations. For example, a car. Some cars will be in the basic version, while others may have many additional features.

One approach is to create a class with common fields and then extend it for each variation. This quickly leads to a class explosion: `CarWithAutopilot`, `CarWithSteeringWheel`, `CarInSportVersion`. Not a scalable solution.

Another idea might be to create just one class, `Car`, which will have all the possibilities as fields: `hasAutopilot`, `isSportVersion`, `hasSteeringWheel`, `additionalWarranty`, `soundSystem`, and so on. If we create a base car, most constructor parameters will be null or false, which is both messy and error-prone. This is the classic **telescoping constructor** problem: constructors with ever-growing parameter lists.

### The Pattern

In the classic Gang of Four (GoF) formulation, a **Director** class orchestrates the building process. The Director defines the order in which to call construction steps, while the Builder provides the implementation for those steps. This separation lets you reuse the same building routine across different builders.

In practice, especially in Java, I reach for the **fluent builder** variant far more often. Here the builder returns `this` from each setter method, enabling a readable method chain without a separate Director class.

```java
Car car = new Car.Builder()
    .withEngine("V8")
    .withAutopilot(true)
    .withSoundSystem("Bose")
    .build();
```

The fluent builder is so common that **Lombok** can generate the entire boilerplate for you with a single `@Builder` annotation:

```java
@Builder
public class Car {
    private final String engine;
    private final boolean autopilot;
    private final String soundSystem;
}

// usage
Car car = Car.builder()
    .engine("V8")
    .autopilot(true)
    .soundSystem("Bose")
    .build();
```

Under the hood Lombok generates the static inner `Builder` class, fluent setters, and the `build()` method. I use this in nearly every project where I need builders. It removes a significant amount of repetitive code.

### Full Example

```java
public class Car {

    private final CarType carType;          // required
    private final Engine engine;            // required
    private final Transmission transmission;// required
    private final int seats;                // required
    private final int additionalWarranty;   // optional
    private final String soundSystem;       // optional
    private final boolean hasHeatedSeats;   // optional
    private final boolean hasHeatedWheel;   // optional

    public Car(Builder builder) {
        this.carType = builder.carType;
        this.engine = builder.engine;
        this.transmission = builder.transmission;
        this.seats = builder.seats;
        this.additionalWarranty = builder.additionalWarranty;
        this.soundSystem = builder.soundSystem;
        this.hasHeatedSeats = builder.hasHeatedSeats;
        this.hasHeatedWheel = builder.hasHeatedWheel;
    }

    // All getters, and NO setters to preserve immutability

    public CarType getCarType() {
        return carType;
    }

    public Engine getEngine() {
        return engine;
    }

    public Transmission getTransmission() {
        return transmission;
    }

    public int getSeats() {
        return seats;
    }

    public int getAdditionalWarranty() {
        return additionalWarranty;
    }

    public String getSoundSystem() {
        return soundSystem;
    }

    public boolean isHasHeatedSeats() {
        return hasHeatedSeats;
    }

    public boolean isHasHeatedWheel() {
        return hasHeatedWheel;
    }

    public static class Builder {

        private final CarType carType;
        private final Engine engine;
        private final Transmission transmission;
        private final int seats;
        private int additionalWarranty;
        private String soundSystem;
        private boolean hasHeatedSeats;
        private boolean hasHeatedWheel;

        public Builder(CarType carType,
                       Engine engine,
                       Transmission transmission,
                       int seats) {
            this.carType = carType;
            this.engine = engine;
            this.transmission = transmission;
            this.seats = seats;
        }

        public Builder additionalWarranty(int additionalWarranty) {
            this.additionalWarranty = additionalWarranty;
            return this;
        }

        public Builder soundSystem(String soundSystem) {
            this.soundSystem = soundSystem;
            return this;
        }

        public Builder hasHeatedSeats(boolean hasHeatedSeats) {
            this.hasHeatedSeats = hasHeatedSeats;
            return this;
        }

        public Builder hasHeatedWheel(boolean hasHeatedWheel) {
            this.hasHeatedWheel = hasHeatedWheel;
            return this;
        }

        public Car build() {
            Car car = new Car(this);
            validateCarObject(car);
            return car;
        }

        private void validateCarObject(Car car) {
            if (car.getSeats() < 1 || car.getSeats() > 9) {
                throw new IllegalStateException(
                    "Seat count must be between 1 and 9, got: " + car.getSeats());
            }
            if (car.getCarType() == CarType.SPORTS_CAR && car.getSeats() > 2) {
                throw new IllegalStateException(
                    "Sports cars cannot have more than 2 seats");
            }
            if (car.getAdditionalWarranty() < 0) {
                throw new IllegalStateException(
                    "Additional warranty cannot be negative");
            }
        }
    }
}
```

```java
public static void main(String[] args) {

    Car car1 = new Car.Builder(SPORTS_CAR, new Engine(6.0, 0), AUTOMATIC, 2)
            .hasHeatedSeats(true)
            .hasHeatedWheel(false)
            .additionalWarranty(3)
            .soundSystem("Sony")
            .build();

    Car car2 = new Car.Builder(CITY_CAR, new Engine(1.5, 0), AUTOMATIC, 5)
            .hasHeatedSeats(true)
            .hasHeatedWheel(true)
            // no additional warranty
            // no sound system
            .build();
}
```

Notice how the required parameters go into the Builder's constructor, while optional ones are set through fluent methods. If an optional parameter is not set, it keeps its default value. No nulls, no confusion about which argument is which.

### Validation in the Builder

The `build()` method is the natural place to enforce invariants. If the caller creates an impossible combination, they get a clear error at construction time, not a mysterious bug later at runtime.

```java
// This will throw IllegalStateException:
// "Sports cars cannot have more than 2 seats"
Car invalid = new Car.Builder(SPORTS_CAR, new Engine(6.0, 0), AUTOMATIC, 5)
        .build();
```

This is one of the key advantages over a plain constructor. With a constructor that takes eight arguments, you typically validate each parameter in isolation. The builder can validate cross-field rules, like "sports cars must have at most 2 seats", because all fields are fully populated before `build()` returns.

If you prefer not to throw exceptions (for example, in a context where invalid input is expected and should be handled gracefully), you can return an `Optional` or a result type instead:

```java
public Optional<Car> buildIfValid() {
    Car car = new Car(this);
    try {
        validateCarObject(car);
        return Optional.of(car);
    } catch (IllegalStateException e) {
        return Optional.empty();
    }
}
```

I tend to prefer the exception approach for domain objects where invalid state indicates a programming error. The `Optional` approach makes more sense when parsing user input or external data where failures are routine.

### Builders and Class Hierarchies

One common pain point is using the builder pattern with inheritance. If `Car` has a subclass `ElectricCar`, how do you make the builder work with both?

The naive approach, having `ElectricCar.Builder` extend `Car.Builder`, breaks the fluent API. When you call a method defined on the parent builder, it returns `Car.Builder`, not `ElectricCar.Builder`. You lose access to the child-specific methods:

```java
// This does NOT compile: additionalWarranty() returns Car.Builder,
// which has no batteryCapacity() method
ElectricCar e = new ElectricCar.Builder(CITY_CAR, new Engine(0, 300), AUTOMATIC, 5)
        .additionalWarranty(2)
        .batteryCapacity(75)  // compile error
        .build();
```

The solution is the **recursive generics** (or "self-type") trick. The parent builder declares a generic type parameter that refers to the concrete builder subclass:

```java
public abstract class Car {

    // ... fields omitted for brevity

    protected abstract static class Builder<T extends Builder<T>> {

        private final CarType carType;
        private final Engine engine;
        private final Transmission transmission;
        private final int seats;
        private int additionalWarranty;

        public Builder(CarType carType, Engine engine,
                       Transmission transmission, int seats) {
            this.carType = carType;
            this.engine = engine;
            this.transmission = transmission;
            this.seats = seats;
        }

        @SuppressWarnings("unchecked")
        protected T self() {
            return (T) this;
        }

        public T additionalWarranty(int additionalWarranty) {
            this.additionalWarranty = additionalWarranty;
            return self();
        }

        public abstract Car build();
    }
}
```

```java
public class ElectricCar extends Car {

    private final int batteryCapacityKwh;

    private ElectricCar(Builder builder) {
        super(builder);
        this.batteryCapacityKwh = builder.batteryCapacityKwh;
    }

    public static class Builder extends Car.Builder<Builder> {

        private int batteryCapacityKwh;

        public Builder(CarType carType, Engine engine,
                       Transmission transmission, int seats) {
            super(carType, engine, transmission, seats);
        }

        public Builder batteryCapacity(int kwh) {
            this.batteryCapacityKwh = kwh;
            return this;
        }

        @Override
        public ElectricCar build() {
            return new ElectricCar(this);
        }
    }
}
```

Now the fluent chain works correctly regardless of call order:

```java
ElectricCar tesla = new ElectricCar.Builder(CITY_CAR, new Engine(0, 300), AUTOMATIC, 5)
        .additionalWarranty(2)   // returns ElectricCar.Builder, not Car.Builder
        .batteryCapacity(75)
        .build();
```

Joshua Bloch covers this idiom in *Effective Java* (Item 2). It is worth noting that this adds complexity and is only justified when you genuinely need builder inheritance. For most cases, a flat builder per class is simpler and sufficient.

### When to Use the Builder Pattern

* **Telescoping constructors.** When a constructor has many parameters (especially optional ones), the builder makes construction readable and self-documenting.
* **Different representations.** When the same construction process should produce different configurations of the same type.
* **Composite or complex objects.** When the object being built is a tree structure or involves many nested parts.
* **Immutability.** The builder accumulates state through mutable setters, then the `build()` method produces a fully initialized object with all fields set to `final`. The builder itself does not make your code immutable. It gives you a clean way to construct objects that are.

### Pros and Cons

**Pros:**

* You can construct objects step by step, defer construction steps, or run steps recursively.
* You can reuse the same construction code when building various representations.
* Facilitates creating immutable objects by centralizing construction logic while keeping the resulting object's fields final.
* Single Responsibility Principle. Complex construction code is isolated from the business logic.

**Cons:**

* The overall complexity of the code increases since the pattern requires creating additional classes.
* If the object has only a few fields, a builder adds unnecessary indirection. A simple constructor or static factory method may be a better fit.

> **Related posts**: [Static Factory Method](/posts/static-factory-method/), [SOLID: The First 5 Principles of Object Oriented Design](/posts/solid-the-first-5-principles-of-object-oriented-design/)
