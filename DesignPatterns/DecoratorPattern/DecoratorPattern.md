# The Decorator Design Pattern

## 1. The Problem: The "Class Explosion"

Imagine you are designing the checkout system for a coffee shop. You start with a base class `Coffee`.

* A customer wants milk, so you create `CoffeeWithMilk extends Coffee`.
* A customer wants sugar, so you create `CoffeeWithSugar extends Coffee`.
* A customer wants both, so you create `CoffeeWithMilkAndSugar extends Coffee`.

**The Breaking Point:** Add Caramel, Whip, Vanilla, and Mocha to the menu. If you rely on inheritance, you will need to create hundreds of individual classes to account for every possible combination of ingredients (`CoffeeWithMilkAndVanillaAndWhip`, etc.). This is an unmaintainable nightmare.

## 2. The Definition

The Decorator Pattern attaches additional responsibilities to an object dynamically at runtime. Decorators provide a flexible alternative to subclassing for extending functionality.

It is often called the **"Wrapper"** pattern because you place an object inside a special wrapper object that contains the new behaviors.

## 3. Real-World Usage

This pattern is heavily used in core libraries and frameworks:

* **Java I/O Streams:** This is the textbook example. When you write `new BufferedReader(new FileReader(new File("test.txt")))`, you are using the Decorator pattern. The `FileReader` reads raw bytes, and the `BufferedReader` wraps it to add the new capability of reading whole lines at a time.
* **UI Components:** Wrapping a basic `TextBox` inside a `ScrollbarDecorator` and then inside a `BorderDecorator`.
* **Web Server Middleware:** Wrapping an incoming HTTP Request object with an `AuthenticationDecorator` to check logins, and then an `AnalyticsDecorator` to log the IP address, before passing it to the main business logic.

---

## 4. The Java Implementation

We will solve the Coffee Shop problem by using Wrappers.

## UML diagram
<img width="569" height="449" alt="image" src="https://github.com/user-attachments/assets/163502ab-e6b5-4d4f-a8b5-a8498a88f50c" />

```java
// 1. The Base Component Interface
public interface Beverage {
    String getDescription();
    double getCost();
}

// 2. The Concrete Component (The core object we will wrap)
public class SimpleCoffee implements Beverage {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }

    @Override
    public double getCost() {
        return 2.00; // Base price
    }
}

// 3. The Base Decorator (Crucial Step: Implements AND HAS-A Beverage)
public abstract class BeverageDecorator implements Beverage {
    // The wrapped object
    protected Beverage wrappedBeverage; 

    public BeverageDecorator(Beverage beverage) {
        this.wrappedBeverage = beverage;
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription(); // Delegates to the wrapper
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost(); // Delegates to the wrapper
    }
}

// 4. Concrete Decorator A
public class Milk extends BeverageDecorator {
    public Milk(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost() + 0.50; // Adds its own cost
    }
}

// 5. Concrete Decorator B
public class Mocha extends BeverageDecorator {
    public Mocha(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription() + ", Mocha";
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost() + 0.90;
    }
}

// 6. Execution
public class DecoratorDemo {
    public static void main(String[] args) {
        // Order a simple coffee
        Beverage myOrder = new SimpleCoffee();
        
        // Wrap it in Milk
        myOrder = new Milk(myOrder);
        
        // Wrap it in Mocha
        myOrder = new Mocha(myOrder);
        
        // Wrap it in a second shot of Mocha!
        myOrder = new Mocha(myOrder);

        // Output: Simple Coffee, Milk, Mocha, Mocha
        System.out.println("Order: " + myOrder.getDescription()); 
        // Output: $4.30 (2.00 + 0.50 + 0.90 + 0.90)
        System.out.println("Total Cost: $" + myOrder.getCost()); 
    }
}

```

### Line-by-Line Java vs. C++ Breakdown

* `public abstract class BeverageDecorator implements Beverage` AND `protected Beverage wrappedBeverage;`
* **The Mind-Bending Concept:** The Decorator *is a* Beverage (Inheritance/Interface), but it also *has a* Beverage inside it (Composition). This is the secret sauce. Because it implements the same interface, you can pass a decorated object anywhere the original object was expected.
* **C++ Equivalent:** A class that inherits from `Beverage` but also holds a `std::unique_ptr<Beverage>`.


* `myOrder = new Milk(myOrder);`
* **The Concept:** You are taking the existing object reference, passing it into the constructor of the `Milk` class, and then reassigning the `myOrder` reference to point to the newly created `Milk` wrapper object on the Heap.


* `return wrappedBeverage.getCost() + 0.50;`
* **The Concept:** This acts like a recursive function. When you call `.getCost()` on the outermost wrapper, it drills all the way down to the `SimpleCoffee` to get the base $2.00, then bubbles back up, adding $0.50, then adding $0.90, etc.


---

## Deeper Dive: Real-Life Uses & "When to Use" Rules

### 1. Heavy-Duty Real-World Examples

* **Database Drivers (e.g., Java JDBC):**
* **How it works:** When you connect a Java app to a database, you use `DriverManager.getConnection()`. This is a classic Factory Method. You pass it a URL (like `jdbc:mysql://...` or `jdbc:postgresql://...`), and the factory dynamically returns the exact connection object for that specific database vendor. Your application code never has to call `new MySQLConnection()`; it just talks to the generic `Connection` interface.


* **Logging Frameworks (e.g., SLF4J or Logback):**
* **How it works:** You almost always start a class with `Logger log = LoggerFactory.getLogger(MyClass.class);`. The factory decides behind the scenes whether to give you a logger that writes to a text file, a console, or an external server like Datadog, depending on your configuration files.


* **Cross-Platform UI Toolkits (e.g., JavaFX or React Native):**
* **How it works:** This is the domain of the **Abstract Factory**. When you write code for a "Button," the framework's factory detects the operating system at runtime. If the app is running on an iPhone, the factory creates an iOS-styled button. If on Android, it creates a Material Design button. The developer only interacts with the generic `Button` interface.



### 2. WHEN to Use the Factory Pattern

Use this pattern when you face the following architectural challenges:

* **Creation is Complex:** If building an object requires 15 lines of setup (e.g., reading a config file, opening a network socket, and authenticating a token), you do not want to copy-paste that setup every time you need the object. Put those 15 lines inside a Factory.
* **You Need Loose Coupling (Dependency Inversion):** If your core business logic relies on concrete classes (like `Truck` or `Ship`), it becomes rigid. A factory allows your business logic to rely only on interfaces (like `Transport`), making the system infinitely expandable.
* **You Want to Save Memory (Object Pooling):** Factories don't *always* have to create new objects. A factory can maintain a cache of existing objects and hand you an old one if it is available, rather than running the expensive `new` keyword every time.

### 3. When NOT to Use the Factory Pattern (The Dangers)

Factories are the most overused design pattern by developers who are just learning LLD. Be ready to explain these downsides:

* **The "Class Explosion" Problem:** To implement a proper Factory Method or Abstract Factory, you have to create a massive amount of boilerplate code. For every new product, you need a new Interface, a Concrete Class, a Factory Interface, and a Concrete Factory. This can turn a simple 5-file project into a 20-file maze.
* **Overcomplicating Simple Instantiation:** If an object is just a simple data container (like the `Product` class we made in the SRP example) and has no complex setup, using a Factory is severe overengineering. Just use `new Product()`.
* **The "God Factory" Anti-Pattern:** Sometimes developers put *all* object creation for the entire application into one massive `AppFactory` class. This violates the Single Responsibility Principle, as the factory now has 100 different reasons to change.

---

## 5. LLD Interview Checkpoint

**Q1: What is the difference between Decorator and Strategy?**

* **Answer:** **Strategy** changes the *guts* of an object (how it behaves internally, like swapping a V8 engine for an Electric motor). **Decorator** changes the *skin* of an object (wrapping it to add new features, like adding a spoiler or a paint job, without touching the original engine).

**Q2: When should I NOT use the Decorator Pattern?**

* **Answer:** When the exact order of execution strictly matters, or when you have to write code that relies on the specific concrete type of an object. Once you wrap a `SimpleCoffee` in a `Milk` decorator, it is no longer recognized as a `SimpleCoffee` type by the system; it is just a generic `Beverage`.



