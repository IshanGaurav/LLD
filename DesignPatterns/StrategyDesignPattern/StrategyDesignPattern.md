# The Strategy Design Pattern

## 1. The Problem: The Trap of Inheritance

Imagine you are building a racing game. You create a parent class `Vehicle` with a `drive()` method.

* `PassengerCar` extends `Vehicle` and uses a normal drive.
* `SportsCar` extends `Vehicle` and overrides `drive()` to go extremely fast.
* `OffRoadJeep` extends `Vehicle` and overrides `drive()` to handle dirt.

**The Breaking Point:** Now, you need to add a `RallyCar`. A Rally Car needs the exact same fast `drive()` logic as the `SportsCar`.
Because Java does not support multiple class inheritance, you cannot inherit the `SportsCar`'s drive method without inheriting *everything* else a sports car is. Your only choice is to copy-paste the fast `drive()` code into the `RallyCar` class.

**The Inheritance Problem:** 
1.  **Code Duplication:** Siblings (SportsCar and RallyCar) share behavior, forcing you to duplicate code.
2.  **Rigidity:** An object's behavior is locked at compile-time. A `SportsCar` cannot dynamically switch to "Off-Road" mode if it drives onto dirt during the game.

## 2. The Definition

The Strategy Design Pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from the clients (objects) that use it.

## 3. The Solution: Composition over Inheritance

Instead of forcing the `Vehicle` class to *inherit* its driving behavior, we extract the driving behavior into its own independent Interface.

The `Vehicle` will then *contain* a reference to this interface (a **HAS-A** relationship / Composition).

---

## 4. The Java Implementation

## UML diagram
 <img width="361" height="417" alt="image" src="https://github.com/user-attachments/assets/2eecde24-1c4a-4620-bc2d-21791180783f" />


```java
// 1. The Strategy Interface (The Contract)
public interface DriveStrategy {
    void drive();
}

// 2. Concrete Strategy A
public class NormalDrive implements DriveStrategy {
    @Override
    public void drive() {
        System.out.println("Driving normally to save gas.");
    }
}

// 3. Concrete Strategy B
public class SportsDrive implements DriveStrategy {
    @Override
    public void drive() {
        System.out.println("Burning fuel, driving at top speed!");
    }
}

// 4. The Context Class (The class that USES the strategy)
public class Vehicle {
    // HAS-A relationship (Composition)
    private DriveStrategy driveStrategy; 

    // Constructor Injection
    public Vehicle(DriveStrategy strategy) {
        this.driveStrategy = strategy;
    }

    // Executes the strategy without knowing the underlying details
    public void executeDrive() {
        driveStrategy.drive();
    }

    // The Magic: Allows changing behavior at RUNTIME
    public void setDriveStrategy(DriveStrategy newStrategy) {
        this.driveStrategy = newStrategy;
    }
}

// 5. Execution
public class StrategyPatternDemo {
    public static void main(String[] args) {
        // Create a car with a normal driving strategy
        Vehicle myCar = new Vehicle(new NormalDrive());
        myCar.executeDrive(); // Output: Driving normally to save gas.

        // The terrain changes! Dynamically swap the strategy at runtime.
        System.out.println("Entering the highway...");
        myCar.setDriveStrategy(new SportsDrive());
        myCar.executeDrive(); // Output: Burning fuel, driving at top speed!
    }
}

```

### Line-by-Line Java vs. C++ Breakdown

* `public interface DriveStrategy`
* **The Concept:** This defines the "family of algorithms." Any class that wants to be a driving strategy *must* implement this.
* **C++ Equivalent:** A class with a pure virtual function (`virtual void drive() = 0;`).


* `private DriveStrategy driveStrategy;`
* **The Concept:** The `Vehicle` holds a reference to the interface, not a specific class. This is called "Programming to an Interface." It is what allows us to plug in *any* valid strategy.
* **C++ Equivalent:** Holding a base class pointer: `DriveStrategy* driveStrategy;`.


* `public Vehicle(DriveStrategy strategy)`
* **The Concept:** This is **Dependency Injection**. The `Vehicle` does not create its own strategy using the `new` keyword. The strategy is created outside and pushed (injected) into the Vehicle.


* `myCar.setDriveStrategy(new SportsDrive());`
* **The Concept:** This is the core power of the Strategy pattern. Because we used an object reference instead of rigid inheritance, we can hot-swap the behavior of the `myCar` object while the program is actively running.



---

## Deeper Dive: Real-Life Uses & "When to Use" Rules

### 1. Heavy-Duty Real-World Examples

#### **Navigation Systems (like Google Maps):**
* **The Context:** The Route Calculator.
* **The Strategies:** `WalkingStrategy`, `DrivingStrategy`, `PublicTransitStrategy`, `BikingStrategy`.
* **How it works:** The user enters a destination. When they click the "Car" icon, the app injects the `DrivingStrategy` to calculate highway traffic and one-way streets. If they click the "Train" icon, the app dynamically swaps to the `PublicTransitStrategy` to calculate train schedules. The core Map UI doesn't change, only the routing algorithm does.


#### **E-Commerce Payment Processing (like Amazon or Stripe):**
* **The Context:** The Checkout Cart.
* **The Strategies:** `CreditCardPayment`, `PayPalPayment`, `CryptoPayment`, `ApplePayPayment`.
* **How it works:** Each payment method requires completely different security checks, API calls, and fee calculations. Instead of a 1,000-line `if-else` block in the checkout system, each payment type is its own isolated class.


#### **File Compression Tools (like WinRAR or 7-Zip):**
* **The Context:** The Archiver.
* **The Strategies:** `ZipCompression`, `RarCompression`, `TarCompression`.
* **How it works:** The user selects files and chooses an output format. The application dynamically assigns the correct compression algorithm before processing the bytes.



### 2. WHEN to Use the Strategy Pattern

Use this pattern when you face the following architectural challenges:

* **You have "Algorithm Families":** When you have several different ways to do the exact same conceptual task (e.g., sorting data, formatting text, calculating taxes).
* **You are drowning in `if-else` or `switch` statements:** If your class has a method that looks like `if (type == A) doA() else if (type == B) doB()`, it is screaming for the Strategy Pattern. Extract each block into its own class.
* **You need to hide complex, proprietary code:** If one of your algorithms requires complex data structures or third-party libraries (like a specific cryptography library), putting it in a Strategy keeps that messy code completely isolated from your clean, main business logic.
* **You need Runtime Flexibility:** When the behavior of the object needs to change while the program is actively running, based on user input or environmental changes (e.g., a game character swapping weapons).

### 3. When NOT to Use the Strategy Pattern (The Dangers)

Do not use this pattern blindly. Interviewers will look for you to point out these specific flaws:

* **Overengineering Simple Problems:** If you only have two algorithms, and they rarely change, creating an Interface, a Context class, and two Concrete Strategy classes is a massive waste of time and makes the code harder to read. A simple `if-else` is perfectly fine for basic logic.
* **The "Client Burden":** To use the Strategy pattern, the Client (the code using the Context) usually has to know *which* strategy to inject. If the user of your API has to understand the deep technical differences between `StrategyA` and `StrategyB` just to use your class, you are leaking implementation details.
* **Increased Memory/Object Overhead:** Every strategy is a full class. If you create a new Strategy object every single time a loop runs, you will strain the Java Garbage Collector. *(Pro-tip: If the strategies hold no internal state variables, they can be implemented as Singletons to save memory!)*
## Real-Life Example

`Google Maps:`
* Car Route
* Bike Route
* Walking Route

## When to Use
* Multiple interchangeable algorithms
* Avoiding large if-else chains
* Following Open-Closed Principle
## Conclusion
* Encapsulate what varies & keep it separate from what remains same.
* Solution to inheritance is not more inheritance.
* Composition should be favoured over inheritance.
* Code to interface & not to concrete implementation (written as "concretion" on the board).
* Do NOT Repeat Yourself (DRY).
## 5. LLD Interview Checkpoint

**Q1: How does the Strategy pattern uphold the Open/Closed Principle (OCP)?**

* **Answer:** The `Vehicle` class is *closed for modification* (you never have to touch its code again to add new driving styles). But it is *open for extension* (you can create a brand new `HoverDrive` class that implements `DriveStrategy` and inject it into the `Vehicle` seamlessly).

**Q2: When should I use the Strategy Pattern?**

* **Answer:** Whenever you see a massive `if-else` or `switch` statement that chooses different behaviors based on a type or flag (e.g., `if (type == "CREDIT_CARD") payCredit(); else if (type == "PAYPAL") payPayPal();`). Extract those blocks into Strategies.

