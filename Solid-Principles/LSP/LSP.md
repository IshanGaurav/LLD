## LSP: Liskov Substitution Principle

### 📖 Definition

The Liskov Substitution Principle (LSP) is the "L" in the SOLID principles of object-oriented design. Introduced by computer scientist Barbara Liskov in 1987, it states that **objects of a superclass should be replaceable with objects of its subclasses without breaking the application.** In other words, a subclass must behave in a way that the base class expects, honoring the "contract" established by the parent.

### 💭 Key Features

* **Behavioral Subtyping:** It isn't enough for a subclass to just inherit properties; it must also maintain the logical behavior expected by the parent class.
* **No Unexpected Exceptions:** A subclass should not throw exceptions for behaviors that the base class normally handles smoothly.
* **Refused Bequest Prevention:** Prevents situations where a subclass inherits a method from a parent but leaves it blank or intentionally crashes it because the subclass doesn't actually support that action.
* **Predictability:** Ensures that client code using the base class can safely operate on any subclass without needing to know the specific subclass type.

### 🧩 How to Implement

1. **Design by Contract:** Ensure subclasses enforce the same rules (pre-conditions and post-conditions) as the parent class.
2. **Avoid "Dummy" Implementations:** If you find yourself writing a subclass method that just returns `null` or throws an `UnsupportedOperationException`, your inheritance hierarchy is flawed.
3. **Favor Interfaces or Smaller Base Classes:** If subclasses only share *some* behaviors, don't force them into a massive single base class. Break behaviors down into interfaces (like `IFlyable`, `ISwimmable`).
4. **Is-A vs. Behaves-Like-A:** Ensure the subclass truly *behaves like* the parent, not just conceptually *is* the parent (e.g., in geometry, a Square *is a* Rectangle, but in programming, a Square does not *behave like* a Rectangle because changing a Square's width also changes its height).

### 💻 Code Example (Java)

**🚨 Bad Code (Violates LSP):**
In this classic violation, a `Penguin` inherits the `fly()` method from `Bird`, but because penguins can't fly, it throws an exception. Any code expecting a generic `Bird` to fly will crash when handed a `Penguin`.

```java
// Base class
class Bird {
    public void fly() {
        System.out.println("I am flying high!");
    }
}

// Subclass violating LSP
class Penguin extends Bird {
    @Override
    public void fly() {
        // Violates the contract of the Bird class
        throw new UnsupportedOperationException("Help! Penguins cannot fly!");
    }
}

public class Main {
    public static void makeBirdFly(Bird bird) {
        bird.fly(); // This will crash if a Penguin is passed in
    }
}

```

**🌿 Good Code (Follows LSP):**
Here, we fix the hierarchy. We remove `fly()` from the base `Bird` class and create a specific interface for flying capabilities. `Penguin` remains a `Bird` but is no longer forced to implement flying.

```java
// Base class with universal behaviors
class Bird {
    public void eat() {
        System.out.println("I am eating.");
    }
}

// Specific behavior interface
interface Flyable {
    void fly();
}

// Subclass that behaves perfectly as expected
class Sparrow extends Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("I am flying high!");
    }
}

// Subclass that does not break any base class contracts
class Penguin extends Bird {
    public void swim() {
        System.out.println("I am swimming fast!");
    }
}

public class Main {
    public static void makeBirdFly(Flyable flyingBird) {
        flyingBird.fly(); // 100% safe, only accepts birds that can actually fly
    }
}

```

### ⚖️ Pros & Cons

| 📈 Advantages | 📉 Disadvantages |
| --- | --- |
| **High Reliability:** Client code won't unexpectedly crash when swapping out object types. | **Complex Hierarchies:** Fixing LSP violations often requires creating more interfaces and base classes. |
| **True Polymorphism:** You can confidently write code targeting base classes or interfaces. | **Refactoring Cost:** Correcting a deeply ingrained LSP violation in legacy code can be incredibly time-consuming. |
| **Easier Maintenance:** Clearer contracts mean developers know exactly what a subclass is supposed to do. | **Requires Strict Discipline:** It is very easy to accidentally violate LSP for the sake of quick code reuse. |

---

### 🎉 Conclusion

The Liskov Substitution Principle is the ultimate test of your inheritance structures. While inheritance is a powerful tool for code reuse, LSP reminds us that it must be used logically. If a subclass cannot seamlessly stand in for its parent, the inheritance relationship is fundamentally broken. By adhering to LSP alongside **SRP** and **OCP**, you ensure that your object-oriented designs are not just extensible, but mathematically sound and crash-resistant.
