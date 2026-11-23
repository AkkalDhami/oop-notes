# Object-Oriented Programming (OOP) in Java

> A detailed study note covering **Java OOP only**: concepts, syntax, examples, practical reasoning, and common mistakes.

---

## 1. What is Object-Oriented Programming?

**Object-Oriented Programming (OOP)** is a programming paradigm that structures software around **objects**—entities that hold state (data) and provide behavior (methods).

Java is primarily object-oriented, though it also supports primitive types and static members.

### Why OOP?

- Organizes code into reusable, maintainable units.
- Models real-world or domain concepts.
- Encapsulates state and behavior.
- Supports extensibility through inheritance and interfaces.
- Makes large programs easier to reason about when responsibilities are well-designed.

### Four pillars of OOP
| Pillar | Meaning |
|---|---|
| Encapsulation | Protect internal state and expose controlled operations. |
| Abstraction | Expose essential behavior while hiding implementation details. |
| Inheritance | Derive a type from another type to reuse or specialize behavior. |
| Polymorphism | Use a common type/interface while different implementations behave differently. |

---

## 2. Class and Object

### Class

A **class** is a blueprint that defines the data (fields) and behavior (methods) its objects can have.

### Object

An **object** is an instance of a class created at runtime. It has identity, state, and behavior.

```java
class Car {
    String brand;
    int speed;

    void drive() {
        System.out.println(brand + " is driving at " + speed + " km/h");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.brand = "Toyota";
        car1.speed = 80;
        car1.drive();
    }
}
```

`Car` is a class; `car1` references an object. `new Car()` creates the object, and the variable stores a reference to it.

### Object state and behavior
- **State:** values held in fields (`brand`, `speed`).
- **Behavior:** methods (`drive()`).

### Multiple objects
Each instance has its own instance fields.

```java
Car car1 = new Car();
Car car2 = new Car();

car1.speed = 60;
car2.speed = 100;
```

---

## 3. Fields, Methods, and `this`

### Instance fields
Fields declared inside a class, outside methods, and without `static` belong to each object.

### Local variables
Variables declared inside a method or block. They must be initialized before use.

### Methods
Methods define operations or behavior.

```java
class Student {
    String name;
    int age;

    void display() {
        System.out.println(name + " - " + age);
    }
}
```

### `this` keyword
`this` refers to the current object.

```java
class Student {
    String name;

    Student(String name) {
        this.name = name; // field = constructor parameter
    }
}
```

Uses of `this`:
- Distinguish fields from local parameters.
- Call another constructor using `this(...)` (must be first statement).
- Pass or return the current object.
- Call current object's methods/fields.

```java
class Box {
    int length;

    Box(int length) {
        this(length, length, length);
    }

    Box(int length, int width, int height) {
        this.length = length;
    }
}
```

---

## 4. Access Modifiers

Java access modifiers control where a class member can be accessed.

| Modifier | Same class | Same package | Subclass in other package | Other package (non-subclass) |
|---|---:|---:|---:|---:|
| `private` | Yes | No | No | No |
| *(no modifier)* package-private | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes, under subclass access rules | No |
| `public` | Yes | Yes | Yes | Yes |

> Top-level classes can be `public` or package-private; they cannot be `private` or `protected`.

Prefer the narrowest access level that supports the design.

---

## 5. Encapsulation

**Encapsulation** is the practice of keeping an object's internal state protected and allowing controlled access through its public API.

Common pattern: make fields `private`, and provide methods to validate or change state.

```java
class BankAccount {
    private double balance;

    BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        balance = initialBalance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

Why not make every field public?
- Callers could put the object into invalid states.
- Validation becomes scattered.
- Changing implementation details becomes harder.

**Note:** Getters/setters are not automatically good encapsulation. A setter that allows any value may still undermine the object's invariants.

---

## 6. Abstraction

**Abstraction** exposes what a type can do without requiring users to know how it does it.

In Java, abstraction is commonly expressed using:
- Interfaces
- Abstract classes
- Well-designed public methods

Example: callers use `payment.pay(...)` without knowing the payment-processing internals.

```java
abstract class Payment {
    abstract void pay(double amount);

    void receipt() {
        System.out.println("Payment recorded");
    }
}

class CardPayment extends Payment {
    @Override
    void pay(double amount) {
        System.out.println("Paid " + amount + " by card");
    }
}
```

Abstraction focuses on the contract or essential behavior, while encapsulation protects the implementation and state. They are related but not identical.

---

## 7. Constructors

A **constructor** initializes a newly created object. It has the class name and no return type.

### Default constructor
If you declare no constructor at all, Java provides a no-argument default constructor. If you declare any constructor, Java no longer supplies that default one.

```java
class User {
    String name;
    // Compiler-provided no-arg constructor only if no constructor is declared.
}
```

### No-argument constructor
A constructor you explicitly write with no parameters.

```java
class User {
    String name;

    User() {
        name = "Guest";
    }
}
```

### Parameterized constructor
Accepts values to initialize fields.

```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}
```

### Constructor overloading
Multiple constructors with different parameter lists.

```java
class Rectangle {
    int width, height;

    Rectangle() {
        this(1, 1);
    }

    Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
}
```

### Constructor chaining
- `this(...)` invokes another constructor in the same class.
- `super(...)` invokes a superclass constructor.
- Constructor invocation must be first (Java versions with flexible constructor bodies have nuanced rules; for conventional code, keep it first).

---

## 8. `static` Members

`static` members belong to the class rather than an individual instance.

### Static field
One shared field for the class.

### Static method
Called through the class name; it cannot directly access instance fields/methods or use `this`.

```java
class Counter {
    static int total = 0;
    int id;

    Counter() {
        total++;
        id = total;
    }

    static void showTotal() {
        System.out.println(total);
    }
}
```

Usage:

```java
Counter c1 = new Counter();
Counter c2 = new Counter();
Counter.showTotal(); // 2
```

---

## 9. `final` Keyword

`final` means a variable can be assigned only once, a method cannot be overridden, or a class cannot be subclassed, depending on its use.

### Final variable
```java
final int MAX = 100;
// MAX = 200; // compile-time error
```

For a final reference, the reference cannot be reassigned, but the referenced object may still be mutable.

```java
final StringBuilder text = new StringBuilder("Hi");
text.append("!"); // allowed
// text = new StringBuilder("Bye"); // not allowed
```

### Final method
A subclass cannot override it.

### Final class
Cannot be extended. `String` is a familiar final class.

---

## 10. Inheritance

**Inheritance** lets one class (subclass) extend another class (superclass) and inherit accessible members.

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}
```

```java
Dog dog = new Dog();
dog.eat(); // inherited
dog.bark();
```

### Terms
- Superclass / parent / base
- Subclass / child / derived

### Java inheritance types
Java classes support:
- Single inheritance
- Multilevel inheritance
- Hierarchical inheritance

Java does **not** allow a class to extend multiple classes. Multiple type inheritance can be achieved with interfaces.

### `super`
- Access superclass fields (when accessible).
- Invoke superclass methods.
- Call superclass constructors with `super(...)`.

```java
class Animal {
    Animal(String name) {
        System.out.println("Animal: " + name);
    }
}

class Dog extends Animal {
    Dog() {
        super("Dog");
    }
}
```

---

## 11. Method Overloading (Compile-Time Polymorphism)

Overloading allows methods with the same name but different parameter lists in the same class (or inherited context).

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

Overloading is resolved by the compiler based on method signatures and argument types.

**You cannot overload a method only by changing its return type.**

---

## 12. Method Overriding (Runtime Polymorphism)

A subclass provides a specialized implementation of an inherited instance method with a compatible signature.

```java
class Animal {
    void sound() {
        System.out.println("Generic sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Woof");
    }
}
```

```java
Animal animal = new Dog();
animal.sound(); // Woof
```

The method implementation is selected at runtime based on the actual object's class. This is dynamic dispatch.

Use `@Override` to let the compiler catch accidental signature mismatches.

### Overriding rules (high-level)
- Method signature must match (or be compatible according to Java rules).
- Return type may be covariant.
- Cannot reduce visibility.
- `final` methods cannot be overridden.
- `static` methods are hidden, not overridden.
- Private methods are not overridden.

---

## 13. Polymorphism

**Polymorphism** means one interface or superclass reference can represent different concrete types, with behavior varying by implementation.

```java
class Animal {
    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Meow");
    }
}
```

```java
Animal[] animals = { new Dog(), new Cat() };

for (Animal animal : animals) {
    animal.sound();
}
```

The declared reference type controls which members are accessible at compile time. The actual object type controls which overridden instance method runs at runtime.

---

## 14. Abstract Classes

An **abstract class** cannot be instantiated and may contain abstract methods (methods without an implementation) and concrete methods.

```java
abstract class Shape {
    abstract double area();

    void describe() {
        System.out.println("I am a shape");
    }
}

class Circle extends Shape {
    double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}
```

Use an abstract class when related subclasses share state or implementation, or when a common base should define partial behavior.

---

## 15. Interfaces

An **interface** defines a contract a class agrees to implement. A class can implement multiple interfaces.

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    public void fly() {
        System.out.println("Flying");
    }

    public void swim() {
        System.out.println("Swimming");
    }
}
```

### Interface features
- Abstract methods are implicitly `public`.
- Interface fields are implicitly `public static final`.
- Interfaces can include `default`, `static`, and (since Java 9) `private` helper methods.
- Interfaces cannot be instantiated.
- A class can implement multiple interfaces.

### Abstract class vs interface
| Abstract class | Interface |
|---|---|
| Can have instance state and constructors | No ordinary instance fields or constructors |
| A class can extend only one class | A class can implement multiple interfaces |
| Can mix abstract and concrete methods | Can declare abstract, default, static, and private methods |
| Good for shared base state/implementation | Good for capability/contract across unrelated types |

---

## 16. Object Class Methods

Every class directly or indirectly extends `Object` unless it extends another class.

Common methods:
- `toString()` — string representation
- `equals(Object)` — logical equality
- `hashCode()` — hash value used in hash-based collections
- `getClass()` — runtime class information

```java
class Student {
    String name;

    Student(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{name='" + name + "'}";
    }
}
```

### `equals()` and `hashCode()`
If you override `equals`, you should generally also override `hashCode`. Equal objects must have equal hash codes. Unequal objects may share a hash code.

---

## 17. Composition

Composition models a **has-a** relationship by storing one object inside another.

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine = new Engine();

    void startCar() {
        engine.start();
    }
}
```

`Car` has an `Engine`.

Composition is often preferable to inheritance when the relationship is not truly “is-a”. It allows behavior to be assembled and replaced without creating deep inheritance hierarchies.

---

## 18. Association, Aggregation, and Composition

These describe object relationships.

| Relationship | Meaning | Typical ownership |
|---|---|---|
| Association | Objects know/use each other | No ownership implied |
| Aggregation | Whole-part relationship with independently existing parts | Weak/shared ownership |
| Composition | Whole owns its parts as a strong lifecycle/design relationship | Strong ownership |

These terms are conceptual design relationships; Java syntax does not enforce them.

---

## 19. Upcasting and Downcasting

### Upcasting
Treating a subclass object as a superclass type. Usually implicit and safe.

```java
Dog dog = new Dog();
Animal animal = dog;
```

### Downcasting
Converting a superclass reference back to a subclass type. Requires checking that the object really is that subtype.

```java
Animal animal = new Dog();

if (animal instanceof Dog dog) {
    dog.bark();
}
```

Unsafe downcasts can throw `ClassCastException`.

---

## 20. Packages and OOP Organization

Packages group related types and help manage access and naming.

```java
package animals;

public class Dog {
    public void bark() {
        System.out.println("Woof");
    }
}
```

Importing:

```java
import animals.Dog;
```

Package structure is not an OOP pillar, but it is important for organizing classes and access boundaries in Java applications.

---

## 21. Nested and Inner Classes

Java allows classes inside other classes.

Types:
- Static nested class
- Inner (non-static nested) class
- Local class
- Anonymous class

### Static nested class
```java
class Outer {
    static class Nested {
        void show() {
            System.out.println("Nested class");
        }
    }
}
```

### Inner class
A non-static inner class is associated with an instance of its outer class and can access its instance members.

---

## 22. Records (Concise Data-Carrying Classes)

A `record` is a special kind of class designed to hold data with less boilerplate.

```java
record Point(int x, int y) {}

Point p = new Point(3, 4);
System.out.println(p.x());
```

Records automatically provide component accessors, `equals`, `hashCode`, and `toString`. Records are implicitly final and are shallowly immutable: their component references cannot be reassigned, but referenced mutable objects can still mutate.

---

## 23. Enum Types

An `enum` represents a fixed set of constants.

```java
enum Status {
    PENDING,
    APPROVED,
    REJECTED
}
```

Enums can also contain fields, constructors, and methods.

```java
enum Level {
    LOW(1), MEDIUM(2), HIGH(3);

    private final int value;

    Level(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

---

## 24. Object Lifecycle and Garbage Collection

- Objects are created with `new` (or by certain language/runtime mechanisms).
- Constructors initialize them.
- References allow code to access objects.
- When an object is no longer reachable, it may become eligible for garbage collection.
- Java garbage collection manages memory reclamation; timing is not guaranteed.
- Java does not have C++-style deterministic destructors. Avoid relying on finalization (deprecated and unreliable). Use `try-with-resources` for files, sockets, and other `AutoCloseable` resources.

---

## 25. Java vs C++ OOP (Quick Comparison)

| Feature | Java | C++ |
|---|---|---|
| Class inheritance | Single class inheritance | Multiple class inheritance |
| Interfaces / contracts | Interfaces | Abstract classes and other mechanisms; no Java-style interface keyword |
| Memory management | Garbage-collected | Manual/RAII, smart pointers, or other strategies |
| Pointer access | No explicit pointer arithmetic | Raw pointers available |
| Operator overloading | Generally not user-defined | Supports operator overloading |
| Virtual dispatch | Instance methods are virtual by default unless restricted | Requires `virtual` for runtime dispatch |
| Root class | `Object` | No universal root class |
| Multiple inheritance of type | Multiple interfaces | Multiple base classes |

---

## 26. Common OOP Mistakes

1. Making all fields public and losing control of object state.
2. Using inheritance for code reuse when composition would be cleaner.
3. Forgetting `@Override`, allowing a typo to create a different method.
4. Confusing overloading with overriding.
5. Calling overridable methods from constructors.
6. Assuming `==` compares object contents (it compares reference identity for objects; primitive values are compared directly).
7. Overriding `equals()` but not `hashCode()`.
8. Downcasting without checking the runtime type.
9. Building overly deep inheritance chains.
10. Treating getters/setters as encapsulation even when they expose unrestricted state mutation.

---

## 27. OOP Practice Questions

### Conceptual

1. What is OOP, and why use it?
2. What is the difference between a class and an object?
3. Explain the four pillars of OOP with examples.
4. Encapsulation vs abstraction?
5. Inheritance vs composition?
6. Overloading vs overriding?
7. Abstract class vs interface?
8. What is dynamic method dispatch?
9. Why should `equals()` and `hashCode()` be consistent?
10. What are association, aggregation, and composition?

### Coding

1. Create a `Student` class with private fields, constructor, and methods.
2. Build a bank account that validates deposits and withdrawals.
3. Demonstrate constructor overloading.
4. Create an inheritance hierarchy with overriding.
5. Use an abstract `Shape` class with `Circle` and `Rectangle`.
6. Implement two interfaces in a single class.
7. Demonstrate polymorphism using an array of base-class references.
8. Model composition using `Car` and `Engine`.
9. Override `toString`, `equals`, and `hashCode` for a value class.
10. Create an enum with fields and methods.

---

## Recommended Study Order

1. Classes and objects
2. Fields, methods, and `this`
3. Access modifiers
4. Encapsulation
5. Constructors and constructor chaining
6. `static` and `final`
7. Inheritance and `super`
8. Overloading and overriding
9. Polymorphism
10. Abstract classes
11. Interfaces
12. Composition and object relationships
13. Casting and `instanceof`
14. Object methods (`equals`, `hashCode`, `toString`)
15. Nested classes, records, enums, lifecycle

> **Core principle:** Build classes that protect their own valid state, expose clear behavior, and depend on abstractions where flexibility matters. OOP is not about maximizing classes or inheritance—it is about assigning responsibilities and managing relationships clearly.
