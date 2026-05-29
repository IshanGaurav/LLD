## . SRP: Single Responsibility Principle

### 📖 Definition

The Single Responsibility Principle (SRP) is the "S" in the SOLID principles of object-oriented design. Introduced by Robert C. Martin (Uncle Bob), it states that **a class, module, or function should have one, and only one, reason to change.** In other words, every component of your code should be responsible for a single part of the software's functionality.

### 💭 Key Features

* **High Cohesion:** Code that handles related tasks is kept together, while unrelated tasks are moved elsewhere.
* **Separation of Concerns:** Distinct business logic, user interface, database access, and network communication are separated into their own distinct classes.
* **Easier Testing:** When a class only does one thing, writing unit tests for it becomes incredibly straightforward.
* **Fewer Merge Conflicts:** Because responsibilities are split across multiple files, developers are less likely to edit the exact same file at the exact same time.

### 🧩 How to Implement

1. **Identify the "Reasons to Change":** Look at a class and ask, "Who or what would request a change to this file?" If the answer involves multiple different groups (e.g., the database admin *and* the UI designer), the class violates SRP.
2. **Extract Classes:** Move unrelated methods into new, dedicated classes.
3. **Use Facades or Orchestrators:** If a process requires multiple responsibilities, create a "Manager" or "Service" class whose *only* responsibility is coordinating the other single-responsibility classes.
4. **Follow Naming Conventions:** If you cannot name a class without using the word "And" (e.g., `UserAuthenticatorAndDataManager`), it is doing too much.

### 💻 Code Example (Java)

**🚨 Bad Code (Violates SRP):**
In this example, the `Employee` class is doing three entirely different things: managing employee data, handling database logic, and formatting reports.

```java
public class Employee {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    // Responsibility 1: Managing Employee Data
    public String getName() { return name; }
    public double getSalary() { return salary; }

    // Responsibility 2: Database Operations
    public void saveToDatabase() {
        System.out.println("Saving " + name + " to the database...");
        // SQL logic here
    }

    // Responsibility 3: Formatting & Reporting
    public void generateTaxReport() {
        System.out.println("Generating tax report for " + name);
        // Reporting logic here
    }
}

```

**🌿 Good Code (Follows SRP):**
Here, the responsibilities are split into three distinct classes. If the database switches from SQL to NoSQL, only `EmployeeRepository` changes. If the tax laws change, only `TaxReportGenerator` changes.

```java
// Responsibility 1: Managing Employee Data (The Model)
public class Employee {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() { return name; }
    public double getSalary() { return salary; }
}

// Responsibility 2: Database Operations
public class EmployeeRepository {
    public void save(Employee employee) {
        System.out.println("Saving " + employee.getName() + " to the database...");
    }
}

// Responsibility 3: Formatting & Reporting
public class TaxReportGenerator {
    public void generateReport(Employee employee) {
        System.out.println("Generating tax report for " + employee.getName());
    }
}

```

### ⚖️ Pros & Cons

| 📈 Advantages | 📉 Disadvantages |
| --- | --- |
| **Highly Testable:** Smaller, focused classes are much easier to mock and test. | **File Proliferation:** Results in a larger number of files/classes in your project. |
| **Highly Maintainable:** Bugs are isolated to specific components, making them easier to track down. | **Navigational Complexity:** You may have to jump through multiple files to see how a whole feature works. |
| **Better Readability:** A class that does one thing is very easy to read and understand at a glance. | **Over-fragmentation:** Taking SRP to the extreme can lead to "micro-classes" with only one line of code, which harms readability. |
| **Reusability:** Single-purpose classes act like Lego bricks that can be plugged in anywhere. |  |

---

### 🎉 Conclusion

The Single Responsibility Principle is a highly effective way to prevent "God Classes" (classes that know too much or do too much). By ensuring that each component has only one job, you create an architecture that is flexible, resilient to bugs, and deeply welcoming to new developers. Combined with **DRY**, **KISS**, and **YAGNI**, SRP acts as the ultimate safeguard against spaghetti code.
