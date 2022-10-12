---
title: "Core concepts behind OOP"
categories:
  - Blog
tags:
  - Metrics
  - Design
  - Coupling
---
  
### What does mean object-oriented programming?

In programming paradigm, in which we define objects with their behaviours and attributes, those objects are communicating with each other making our app working.


### Core concepts of Object-Oriented Programming

##### Polymorphism

Is the ability of a class to provide different implementation of a method, based on the type of object that has been provided...
In other words we can perform some action in multiple forms or ways.<br>
Let's take a look on an example of code that use inheritance to achieve it:
``` java
public abstract class Vehicle {
    void drive();
}

public Car extends Vehicle {

    @Override
    public void drive(){
        // uses a classic engine
    }
}
public ElectricCar extends Vehicle {

    @Override
    public void drive(){
        // uses a eletric drive
    }
}

-----------------------------------
Vehicle car = new Car();
Vehicle electricCar =  new ElectricCar();

driveToTheChoosenPlace(Vehicle vehicle){
    ...
    vehicle.drive();
}
```

Notice that we can perform same action in different way, depending on the implementation, as long as they are both of the same type, either a Car and ElectricCar **IS-A** Vehicle.
Clients of that code does not know anything about its inner implementation, and they don't really care.

The Car class is driving using a classic engine, where the EletricCar would use an electric drive.  

Worth to mention is the fact that in this case polymorphism is achieved with inheritance.
In practice, inheritance and polymorphism are used together in java to achieve fast performance and readability of code.

But polymorphism can be achieved without inheritance, for example by having same method name with different signatures and performing different actions.

This type of polymorphism is **static or compile-time polymorphism, and it's basically a method overloading.**
```java
public boolean valdiate(){
    return ...;
}

public boolean validate(Collection<Rule> rules){
        return ...;
}
```

There is also **runtime or dynamic polymorphism, which is achieved by the child class that overrides the parent's method:**
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

Its mechanism that allows us to acquire fields and methods of another class by inheriting the class.
By extending we are creating an IS-A relationship, Dog IS-A Animal.
Thanks to it code is more reusable, and it reduces the length of the code in OOP.

```java
// base class
class Car {
    privte Propulsion propulsion;
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
    
    public Car(Propulsion propulsion, Battery battery){
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

It's hiding the state and inner implementation of object from the clients of an API, and make its accessible only by publicly provided methods.  
This allows to hide specific information and controlling access to internal implementation.

For, example, member fields in a class are hidden from other classes, and they/ they behaviours can be accessed only by the shared public methods.
```java
public class Car extends Vehicle {
    privte Engine engine;
    privte Gear engine;
    
    public EngineType getEngineType(){
        return engine.getType();
    }
}
```

##### Abstraction
Abstraction is hiding complexities of implementation and exposing simpler interfaces. 

Imagine that you're driving a car, you only use a gas, stop, steering wheel and maybe few other things. 
But in general you don't know what is happening 'under the hood', how gears are change, how engine is working and so on, its inner implementation that's hidden from the 'user'.

In OOP it means that we are hiding the complex implementation details and expose only the stable API, that's required to use the implementation. 
In Java, it can be done by using interfaces and abstract classes.