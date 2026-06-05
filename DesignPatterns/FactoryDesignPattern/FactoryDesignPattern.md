# The Factory Design Pattern

## 1. The Problem: The `new` Keyword is Toxic

Imagine you are building a logistics application. You start with truck deliveries. Throughout your massive codebase, you write `Transport t = new Truck();` in 50 different places.

**The Breaking Point:** Business is booming, and the company expands to sea deliveries using Ships. Now, you have to hunt down all 50 places where you wrote `new Truck()` and change it to complex `if-else` logic:
`if (seaDelivery) { t = new Ship(); } else { t = new Truck(); }`

**The Creation Problem:** 1. **Tight Coupling:** Your main business logic is completely chained to concrete classes (`Truck` and `Ship`).
2. **Maintenance Nightmare:** If you add a `Drone` delivery next month, you have to modify your core business code all over again.

## 2. The Definition

The Factory Design Pattern provides an interface or a dedicated class for creating objects, hiding the complex instantiation logic from the user. It delegates the responsibility of object creation from the client code to a central "Factory."

## 3. The Solution: Centralized Creation

We remove the `new` keyword from the main application entirely. Instead, we create a `TransportFactory`. The main application simply asks the factory, "Give me a sea transport," and the factory handles the dirty work of actually calling `new Ship()`.

---

## 4. The Java Implementation

## UML diagram
<img width="440" height="493" alt="image" src="https://github.com/user-attachments/assets/66ea1208-bcd5-4990-95be-a691bbd9a5da" />


```java
// 1. The Product Interface (The Contract)
public interface Transport {
    void deliver();
}

// 2. Concrete Product A
public class Truck implements Transport {
    @Override
    public void deliver() {
        System.out.println("Delivering cargo by land in a box.");
    }
}

// 3. Concrete Product B
public class Ship implements Transport {
    @Override
    public void deliver() {
        System.out.println("Delivering cargo by sea in a container.");
    }
}

// 4. The Creator (The Simple Factory)
public class TransportFactory {
    
    // Encapsulates the creation logic in one single place
    public static Transport createTransport(String type) {
        if (type == null) {
            return null;
        }
        if (type.equalsIgnoreCase("TRUCK")) {
            return new Truck();
        } else if (type.equalsIgnoreCase("SHIP")) {
            return new Ship();
        }
        
        throw new IllegalArgumentException("Unknown transport type: " + type);
    }
}

// 5. Execution (The Client)
public class FactoryPatternDemo {
    public static void main(String[] args) {
        
        // The client does NOT use 'new Truck()' or 'new Ship()'. 
        // It relies entirely on the Factory.
        Transport landTransport = TransportFactory.createTransport("TRUCK");
        landTransport.deliver(); 
        
        Transport seaTransport = TransportFactory.createTransport("SHIP");
        seaTransport.deliver(); 
    }
}

```

### Line-by-Line Java vs. C++ Breakdown

* `public static Transport createTransport(String type)`
* **The Concept:** This is the heart of the Factory. It is marked `static` so we can call it without creating a `TransportFactory` object first. It returns the generic `Transport` interface, not a specific class.
* **C++ Equivalent:** A static class method returning a base class pointer: `static Transport* createTransport(string type)`.


* `if (type.equalsIgnoreCase("TRUCK"))`
* **The Concept:** In Java, you cannot use `==` to compare the actual text of Strings (it only compares memory addresses). You must use `.equals()` or `.equalsIgnoreCase()`.


* `throw new IllegalArgumentException(...)`
* **The Concept:** If the factory is asked to build something it doesn't know about, it throws a runtime exception to stop the program safely rather than returning a dangerous `null` value.



---

## 5. LLD Interview Checkpoint

**Q1: What is the main difference between the Factory Pattern and the Strategy Pattern?**

* **Answer:** Factory is about *Creation* (building objects). Strategy is about *Behavior* (swapping algorithms inside an already-built object). You use a Factory to create the car, and you use a Strategy to decide how the car drives.

**Q2: Doesn't the "Simple Factory" violate the Open/Closed Principle (OCP)?**

* **Answer:** Yes! The basic Simple Factory shown above violates OCP because if you add a `Drone`, you have to *modify* the `createTransport` method. To fix this, LLD uses the strict **Factory Method Pattern**, where you create separate `TruckFactory` and `ShipFactory` classes that implement a base `Factory` interface, eliminating the `if-else` block entirely.


