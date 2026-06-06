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


You are absolutely right to ask for this distinction. In Low-Level Design (LLD) interviews, simply saying "Factory Pattern" is a trap. The interviewer is waiting to see if you know the difference between the three distinct variations.

What we wrote previously was the **Simple Factory** (also called the Factory Idiom). While useful, it is not actually an official "Gang of Four" (GoF) design pattern because it violates the Open/Closed Principle.

Here is the complete breakdown of all three types to add to your GitHub repository.

---

## 5.The Three Types of Factory Patterns

### Type 1: The Simple Factory (The Idiom)

This is what we implemented in the previous section. It is a single class with one massive `create()` method (usually `static`) containing a large `if-else` or `switch` statement.

* **The Problem:** It violates the **Open/Closed Principle (OCP)**. If you want to add a `Drone` transport, you are forced to open the `TransportFactory` class and modify the existing `if-else` logic.
* **When to use it:** When the creation logic is very simple and the number of product types is fixed and rarely changes.

---

### Type 2: The Factory Method Pattern (The GoF Standard)

To fix the Simple Factory's OCP violation, we stop using a single factory class. Instead, we define an interface for creating an object, but let subclasses decide which class to instantiate.


## UML diagram
<img width="645" height="376" alt="image" src="https://github.com/user-attachments/assets/9e9e4018-b5c4-4957-8ec7-6bbe9fbf6552" />

**The Concept:** We move the creation logic from a centralized `if-else` block into separate, dedicated factory classes for each product.

```java
// 1. The Product Interface (Same as before)
interface Transport { void deliver(); }

class Truck implements Transport {
    public void deliver() { System.out.println("Delivering by land."); }
}
class Ship implements Transport {
    public void deliver() { System.out.println("Delivering by sea."); }
}

// 2. The CREATOR Interface (The core of Factory Method)
abstract class Logistics {
    // The Factory Method itself!
    public abstract Transport createTransport(); 

    // Core business logic that uses the factory method
    public void planDelivery() {
        Transport t = createTransport();
        t.deliver();
    }
}

// 3. Concrete Creators (Subclasses decide what to instantiate)
class RoadLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Ship();
    }
}

// 4. Execution
public class FactoryMethodDemo {
    public static void main(String[] args) {
        Logistics logistics = new RoadLogistics();
        logistics.planDelivery(); // Output: Delivering by land.
    }
}

```

**Why this is better (C++ to Java Bridge):** In C++, this is known as a **Virtual Constructor**. By making `createTransport()` an abstract method (pure virtual function), we force the child classes (`RoadLogistics`) to implement it.
Now, if you want to add a `Drone`, you don't touch existing code. You simply create a new class `AirLogistics extends Logistics` that returns a `new Drone()`. **OCP is perfectly maintained.**

---

### Type 3: The Abstract Factory (The Factory of Factories)

The Abstract Factory is the heaviest creational pattern.

* **The Concept:** It is used when you need to create **families of related or dependent objects** without specifying their concrete classes.
* **The Classic LLD Scenario:** Cross-platform User Interfaces (UI). If a user is on Windows, the system must create a Windows-style Button and a Windows-style Checkbox. If on macOS, it must create Mac-style elements. You absolutely cannot mix a Mac Button with a Windows Checkbox.

## UML diagram
<img width="752" height="567" alt="image" src="https://github.com/user-attachments/assets/355ffae4-9dbf-4d93-a9be-55c36792dde4" />

```java
// 1. Abstract Products
interface Button { void paint(); }
interface Checkbox { void paint(); }

// 2. Concrete Products (The Families)
class WinButton implements Button {
    public void paint() { System.out.println("Rendering sharp Windows button."); }
}
class MacButton implements Button {
    public void paint() { System.out.println("Rendering rounded Mac button."); }
}
class WinCheckbox implements Checkbox {
    public void paint() { System.out.println("Rendering Windows checkbox."); }
}
class MacCheckbox implements Checkbox {
    public void paint() { System.out.println("Rendering Mac checkbox."); }
}

// 3. The Abstract Factory Interface
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// 4. Concrete Factories (Produce the specific families)
class WinFactory implements GUIFactory {
    public Button createButton() { return new WinButton(); }
    public Checkbox createCheckbox() { return new WinCheckbox(); }
}

class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// 5. The Client (Oblivious to the OS)
class Application {
    private Button button;
    private Checkbox checkbox;

    // Constructor injection guarantees matching families
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void render() {
        button.paint();
        checkbox.paint();
    }
}

```

**The Takeaway:** The `Application` class doesn't know if it is running on Windows or Mac. It just asks the `GUIFactory` for a button and a checkbox. The Abstract Factory guarantees that the created objects are compatible with each other.

---


### LLD Interview: Factory Types Summary

If an interviewer asks you to summarize them, give them this punchy breakdown:

1. **Simple Factory:** 1 Factory class. Uses an `if-else` statement to return a single product type. (Violates OCP).
2. **Factory Method:** 1 Creator Interface, many Subclass Creators. Relies on **Inheritance** to let subclasses decide which single product to instantiate. (Follows OCP).
3. **Abstract Factory:** 1 Factory Interface, many Concrete Factories. Relies on **Composition** to create a whole *family* of related products.

---

## When to Use
* Multiple object types
* Encapsulating creation logic
* Framework development

---


## 6. LLD Interview Checkpoint

**Q1: What is the main difference between the Factory Pattern and the Strategy Pattern?**

* **Answer:** Factory is about *Creation* (building objects). Strategy is about *Behavior* (swapping algorithms inside an already-built object). You use a Factory to create the car, and you use a Strategy to decide how the car drives.

**Q2: Doesn't the "Simple Factory" violate the Open/Closed Principle (OCP)?**

* **Answer:** Yes! The basic Simple Factory shown above violates OCP because if you add a `Drone`, you have to *modify* the `createTransport` method. To fix this, LLD uses the strict **Factory Method Pattern**, where you create separate `TruckFactory` and `ShipFactory` classes that implement a base `Factory` interface, eliminating the `if-else` block entirely.


