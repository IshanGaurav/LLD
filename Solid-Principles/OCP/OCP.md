## OCP: Open/Closed Principle

### 📖 Definition

The Open/Closed Principle (OCP) is the "O" in the SOLID principles of object-oriented design. Coined by Bertrand Meyer in 1988, it states that **software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.** In simpler terms, you should be able to add new functionality to a system without altering existing, thoroughly tested code.

### 💭 Key Features

* **Extensibility:** The system's behavior can be expanded as new requirements arrive (Open for extension).
* **Stability:** Existing source code is left untouched when adding new features, protecting the core logic (Closed for modification).
* **Polymorphism:** Relies heavily on interfaces or abstract classes to allow different implementations to be swapped in dynamically.
* **Reduced Regression:** Because you are not changing existing code, the risk of breaking current functionality is dramatically minimized.

### 🧩 How to Implement

1. **Use Interfaces and Abstract Classes:** Define the expected behavior in an interface, and let specific classes implement the details.
2. **Avoid Large `if-else` or `switch` Chains:** If you find yourself writing a `switch` statement that checks the "type" of an object to determine what to do, it's usually an OCP violation. Replace it with polymorphism.
3. **Employ Design Patterns:** Patterns like the **Strategy Pattern** or **Decorator Pattern** are classic ways to achieve OCP.
4. **Dependency Injection:** Pass the specific implementation (the extension) into the class that needs it, rather than hardcoding it.

### 💻 Code Example (Java)

**🚨 Bad Code (Violates OCP):**
In this example, every time the store introduces a new customer type (like "VIP" or "EMPLOYEE"), the `DiscountCalculator` class itself must be modified and re-tested.

```java
// Violates OCP
public class DiscountCalculator {
    
    // Adding new types requires changing this method
    public double calculateDiscount(String customerType, double amount) {
        if (customerType.equals("REGULAR")) {
            return amount * 0.05;
        } else if (customerType.equals("PREMIUM")) {
            return amount * 0.10;
        }
        return 0;
    }
}

```

**🌿 Good Code (Follows OCP):**
Here, we abstract the discount logic into an interface. If a new customer type is introduced, we simply create a new class. `DiscountCalculator` never changes.

```java
// Interface for extension
interface DiscountStrategy {
    double calculate(double amount);
}

// Concrete extensions
class RegularDiscount implements DiscountStrategy {
    public double calculate(double amount) { 
        return amount * 0.05; 
    }
}

class PremiumDiscount implements DiscountStrategy {
    public double calculate(double amount) { 
        return amount * 0.10; 
    }
}

// Closed for modification, open for extension
public class DiscountCalculator {
    public double calculateDiscount(DiscountStrategy strategy, double amount) {
        return strategy.calculate(amount);
    }
}

```

### ⚖️ Pros & Cons

| 📈 Advantages | 📉 Disadvantages |
| --- | --- |
| **Fewer Bugs:** Existing code remains untouched, preventing regression bugs. | **Increased Complexity:** Adds layers of abstraction (interfaces/classes) that can be difficult to track. |
| **Highly Scalable:** Adding new features is as simple as creating a new file/class. | **More Upfront Design:** Requires anticipating *where* the system might need to expand in the future. |
| **Easier Testing:** You only need to write tests for the newly added class, not rewrite tests for the old one. | **Risk of Premature Abstraction:** Trying to make everything "pluggable" from day one violates YAGNI. |
| **Parallel Development:** Different developers can work on different extensions simultaneously without merge conflicts. |  |

---

### 🎉 Conclusion

The Open/Closed Principle acts as a protective shield for your codebase. By forcing developers to add new features via *new* code rather than *modifying* old code, OCP ensures that systems can grow infinitely without collapsing under the weight of fragile, interconnected logic. When paired with the **Single Responsibility Principle (SRP)**, it creates an architecture that is highly modular, deeply stable, and ready for future business requirements.
