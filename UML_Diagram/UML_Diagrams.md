### 1. The Anatomy of a Single Class

In UML, a class is represented as a rectangle divided into three horizontal compartments.

* **Top Compartment (Name):** The name of the class. It is usually bold and centered. If the class is `abstract`, the name is written in *italics*.
* **Middle Compartment (Attributes):** The state/variables of the class.
* **Bottom Compartment (Methods):** The behavior/functions of the class.

**Visibility (Access Modifiers)**
UML uses simple symbols placed before an attribute or method to denote its access level (Encapsulation):

* `+` **Public:** Accessible from anywhere.
* `-` **Private:** Accessible only within the class.
* `#` **Protected:** Accessible within the package and by subclasses.
* `~` **Package/Default:** Accessible only within the same package.

**Example format:** `- variableName: DataType`
`+ methodName(parameter: DataType): ReturnType`

---

### 2. Mapping Object Relations (The Connectors)

The most critical part of LLD is showing how objects interact. UML uses different lines and arrows to represent the strength and type of relationship.

| Relationship | Concept | UML Connector | Example |
| --- | --- | --- | --- |
| **Inheritance** | IS-A | Solid line with a **hollow triangle** pointing to the Parent. | `Dog` ──▷ `Animal` |
| **Realization** | Implements an Interface | Dashed line with a **hollow triangle** pointing to the Interface. | `StripePayment` - - ▷ `PaymentGateway` |
| **Association** | Uses / General relationship | Solid line (with or without a standard open arrow). | `Driver` ──> `Car` |
| **Aggregation** | HAS-A (Weak relationship) | Solid line with a **hollow diamond** at the Parent class. | `Department` ◇── `Professor` |
| **Composition** | Belongs-to (Strong relationship) | Solid line with a **solid/filled diamond** at the Parent class. | `House` ♦── `Room` |

*Note: Multiplicity is often written at the ends of these lines to show how many objects are involved (e.g., `1` to `*` means one-to-many, like 1 Department has * (many) Professors).*

---

### 3. Putting It Together: A Mini System

Imagine you are asked to design a basic **Parking Lot**. Here is how you would think about the UML connectors before writing the Java code:

1. **Composition:** The `ParkingLot` class has a solid diamond pointing to a `Level` class. (If the parking lot is demolished, the floors/levels are destroyed with it).
2. **Aggregation:** The `Level` class has a hollow diamond pointing to a `ParkingSpot` class. (Spots make up a level, but you could theoretically re-paint and change the spots without destroying the physical level).
3. **Inheritance:** You have a parent `Vehicle` class. You draw solid lines with hollow arrows pointing from `Car`, `Truck`, and `Motorcycle` up to the `Vehicle` class.

---

Let's design a **Library Management System**. This is a quintessential low-level design interview problem because it perfectly balances all four pillars of OOP and utilizes every major UML relationship.

### 1. Requirements Analysis

Before drawing or coding, we must define what our system needs to do:

* A library contains physical books (`BookItem`).
* Each physical book maps back to a central book profile containing metadata like title, author, and ISBN (`Book`).
* There are different types of users: `Patron` (can borrow books) and `Librarian` (can manage books). Both share common identity details (`Account`).
* System users interact with physical books when checking them out or returning them.

### 2. Translating Requirements to UML Relationships

When designing this system, the classes connect through clear structural links:

* **Inheritance (IS-A):** Both `Patron` and `Librarian` share core details like an ID, name, and password. Instead of duplicating fields, they extend a base `Account` class.
* **Composition (Strong Belongs-to):** A `Library` owns specific physical copies of books (`BookItem`). If a particular library branch is permanently shut down, its physical stock is decommissioned with it.
* **Aggregation (Weak HAS-A):** A physical `BookItem` points to its metadata blueprint (`Book`). Multiple physical copies share the same metadata profile. If a physical copy is destroyed, the literary work's profile still exists in the catalog.
* **Association (Uses):** An `Account` interacts with a `BookItem` during a checkout or return transaction. This is a temporary usage relationship rather than permanent ownership.

### 3. Concrete Java Implementation

Here is how these architectural relationships translate directly into structured Java code:

```java
// Base class illustrating Inheritance
public abstract class Account {
    protected String id;
    protected String name;

    public Account(String id, String name) {
        this.id = id;
        this.name = name;
    }

    // Abstract method to be overridden by child classes
    public abstract void displayDashboard();
}

// Child class extending Account
public class Patron extends Account {
    public Patron(String id, String name) {
        super(id, name);
    }

    @Override
    public void displayDashboard() {
        System.out.println("Patron Dashboard: View borrowed books.");
    }

    // Association relationship: Method accepts BookItem as a temporary argument
    public void borrowBook(BookItem bookItem) {
        bookItem.updateStatus(true);
    }
}

// Metadata class representing central book details
public class Book {
    private String isbn;
    private String title;
    private String author;

    public Book(String isbn, String title, String author) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
    }
}

// Physical item demonstrating Aggregation
public class BookItem {
    private String barcode;
    private boolean isBorrowed;
    private Book metadata; // Aggregation: Reference to independent metadata object

    public BookItem(String barcode, Book metadata) {
        this.barcode = barcode;
        this.metadata = metadata;
        this.isBorrowed = false;
    }

    public void updateStatus(boolean status) {
        this.isBorrowed = status;
    }
}

// Central coordinator demonstrating Composition
public class Library {
    private String branchName;
    private List<BookItem> inventory; // Composition: Library owns these lifecycles

    public Library(String branchName) {
        this.branchName = branchName;
        this.inventory = new ArrayList<>();
    }

    // Creating and adding items directly controls their lifecycle
    public void addNewBookCopy(String barcode, Book metadata) {
        BookItem newCopy = new BookItem(barcode, metadata);
        inventory.add(newCopy);
    }
}

```

To help you visualize exactly how these boxes and arrows fit together, use the interactive UML simulator below. You can select different classes to inspect their internal structure and see how each relationship type manifests in a system architecture diagram.


<img width="717" height="631" alt="image" src="https://github.com/user-attachments/assets/364833a3-0dbe-467f-9e46-348209b4f555" />

