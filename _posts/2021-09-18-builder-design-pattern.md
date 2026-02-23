---
title: "Builder Design Pattern"
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
Some cars will be in the basic version, while others may have many additional things.<br/>You might think about creating a class with common fields that others will extend as they have additional functions.<br/>
This solution comes with another problem, which is many classes extending the Car class, e.g. CarWithAutopilot, CarWithSteeringWheel, CarInSportVersion.
So it is not the optimal solution.<br/>
Another idea might be to create just one class, just Car, which will have all the possibilities as fields such as hasAutopilot, isSportVersion, hasSteeringWheel, additionalWarranty, soundSystem, and so on.
So if we create a base car, in most constructor fields we will pass null, which looks messy.

**Solution**

The Builder design pattern suggests extracting the construction code from its class and placing it in separate objects called builders.

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

Under the hood Lombok generates the static inner `Builder` class, fluent setters, and the `build()` method. I use this in nearly every project where I need builders -- it removes a significant amount of repetitive code.

**Example**

```java
public class Car {
    //final attributes
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

    //All getter, and NO setter to preserve immutability

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

        //Return the finally constructed Car object
        public Car build() {
            Car car = new Car(this);
            validateCarObject(car);
            return car;
        }

        private void validateCarObject(Car car) {
            //Do some basic validations to check
            //if Car object does not break any assumption of system
        }
    }
}
```

```java
public static void main(String[] args) {

    Car car1 = new Car.Builder(SPORTS_CAR, new Engine( volume: 6.0,  mileage: 0), AUTOMATIC,  seats: 2)
            .hasHeatedSeats(true)
            .hasHeatedWheel(false)
            .additionalWarranty(3)
            .soundSystem("Sony")
            .build();

    Car car2 = new Car.Builder(CITY_CAR, new Engine( volume: 1.5,  mileage: 0), AUTOMATIC,  seats: 5)
            .hasHeatedSeats(true)
            .hasHeatedWheel(true)
            //no additional warranty
            //no sound system
            .build();
}
```

* Use the Builder pattern to get rid of a "telescopic constructor".

* Use the Builder pattern when you want your code to be able to create different representations of some product (for example, stone and wooden houses).

* Use the Builder to construct Composite trees or other complex objects.

* Use the Builder when you want to facilitate creating immutable objects. The builder accumulates state through mutable setters, then the `build()` method produces a fully initialized object with all fields set to `final`. The builder itself does not make your code immutable -- it gives you a clean way to construct objects that are.


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
    <td>If the object has only a few fields, a builder adds unnecessary indirection -- a simple constructor or static factory method may be a better fit.</td>
  </tr>
  <tr>
    <td>Facilitates creating immutable objects by centralizing construction logic while keeping the resulting object's fields final.</td>
    <td></td>
  </tr>
  <tr>
    <td>Single Responsibility Principle. You can isolate complex construction code from the business logic of the product.</td>
    <td></td>
   </tr>
</table>
