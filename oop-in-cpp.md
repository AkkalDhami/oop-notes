# Object-Oriented Programming (OOP) in C++

> A focused study note on OOP concepts in C++.

## 1. What is OOP?

**Object-Oriented Programming (OOP)** is a programming paradigm that organizes software around **objects**—entities that combine data (state) and functions (behavior). In C++, a class defines the structure and behavior that its objects will have.

### Procedural programming vs. OOP

| Procedural approach | OOP approach |
| --- | --- |
| Organizes code around functions and procedures | Organizes code around objects and classes |
| Data and functions are often separate | Data and related operations are bundled together |
| Works well for straightforward, function-driven programs | Helps model larger systems and manage complexity |

OOP is useful for modularity, code reuse, maintainability, and representing real-world or domain concepts. It is not automatically better for every program; simple tasks may be clearer with procedural code.

## 2. Class and Object

### Class

A **class** is a user-defined type that describes the data and operations its objects can have. It is a blueprint, not an object itself.

```cpp
class Student {
public:
    string name;
    int age;

    void introduce() {
        cout << "I am " << name << " and I am " << age << " years old.\n";
    }
};
```

### Object

An **object** is an instance of a class. Each object has its own instance data.

```cpp
Student s1;
s1.name = "Asha";
s1.age = 20;
s1.introduce();
```

### Creating objects

```cpp
Student s1;             // automatic (stack) object
Student* s2 = new Student; // dynamically allocated object
// Use s2->name and s2->introduce()
delete s2;               // release manually allocated object
```

Prefer automatic objects or smart pointers (`std::unique_ptr`, etc.) over raw `new`/`delete` in modern C++.

## 3. Access specifiers

Access specifiers control where class members can be accessed.

| Specifier | Within class | In derived class | Outside class |
| --- | ---: | ---: | ---: |
| `public` | Yes | Yes (subject to inheritance mode) | Yes |
| `protected` | Yes | Yes (subject to inheritance mode) | No |
| `private` | Yes | No direct access | No |

- **public:** the class's accessible interface.
- **protected:** accessible to the class and derived classes, but not ordinary outside code.
- **private:** accessible only by the class itself and its friends.

A `class` defaults to `private` access; a `struct` defaults to `public` access. In C++, `struct` can also have methods, constructors, inheritance, and private/protected members.

```cpp
class Account {
private:
    double balance = 0;
public:
    void deposit(double amount) {
        if (amount > 0) balance += amount;
    }
    double getBalance() const { return balance; }
};
```

## 4. The four pillars of OOP

### 4.1 Encapsulation

**Encapsulation** bundles data and operations in one class and restricts direct access to internal state. It helps enforce rules and prevents invalid changes.

```cpp
class Temperature {
private:
    double celsius;
public:
    explicit Temperature(double c) : celsius(c) {}
    void setCelsius(double c) { celsius = c; }
    double getCelsius() const { return celsius; }
};
```

**Why it matters:** the class can validate inputs, preserve invariants, and change its implementation without exposing internal details.

### 4.2 Abstraction

**Abstraction** exposes essential functionality while hiding implementation details. Users of a class rely on its interface rather than needing to understand its internals.

```cpp
class RemoteControl {
public:
    void powerOn() { /* internal hardware steps hidden */ }
};
```

Encapsulation is about protecting/bundling the internal state; abstraction is about presenting a simpler interface. They are related but not identical.

### 4.3 Inheritance

**Inheritance** allows a derived class to extend or specialize a base class. It can reuse interface and implementation, and can support polymorphism when designed appropriately.

```cpp
class Animal {
public:
    void eat() { cout << "Eating\n"; }
};

class Dog : public Animal {
public:
    void bark() { cout << "Barking\n"; }
};
```

```cpp
Dog d;
d.eat(); // inherited
 d.bark();
```

Inheritance should express a genuine **is-a** relationship. If a class merely needs to use another class's services, composition may be a better choice.

### 4.4 Polymorphism

**Polymorphism** means an interface or operation can take different forms depending on the types involved.

- **Compile-time polymorphism:** function overloading, operator overloading, and templates (templates are a broader compile-time mechanism, not necessarily an OOP feature).
- **Runtime polymorphism:** virtual functions dispatched according to the object's dynamic type, typically through a base pointer or reference.

Runtime polymorphism is covered in detail in Section 11.

## 5. Member functions and `this`

Member functions operate on objects of their class. Inside a non-static member function, `this` is a pointer to the current object.

```cpp
class Box {
    int width;
public:
    Box& setWidth(int width) {
        this->width = width; // distinguishes member from parameter
        return *this;        // returns current object by reference
    }
};
```

`this` is not available in static member functions because those functions are not called on a particular object.

## 6. Constructors

A **constructor** is a special member function invoked when an object is initialized. It has the class name, no return type, and may be overloaded.

### 6.1 Default constructor

A constructor callable with no arguments.

```cpp
class Point {
public:
    Point() { cout << "Point created\n"; }
};
```

If you declare another constructor, the compiler generally does not automatically provide a no-argument constructor. Add one explicitly if needed.

### 6.2 Parameterized constructor

```cpp
class Point {
    int x, y;
public:
    Point(int xValue, int yValue) : x(xValue), y(yValue) {}
};
```

### 6.3 Member initializer list

Initializes members before the constructor body. It is required for `const` members, reference members, and members without a default constructor; it is also usually the preferred way to initialize data members.

```cpp
class User {
    const int id;
    string name;
public:
    User(int userId, string userName)
        : id(userId), name(std::move(userName)) {}
};
```

Members initialize in the order they are **declared in the class**, not the order written in the initializer list.

### 6.4 Copy constructor

Creates a new object from an existing object of the same class.

```cpp
class Point {
    int x, y;
public:
    Point(int a, int b) : x(a), y(b) {}
    Point(const Point& other) : x(other.x), y(other.y) {}
};
```

A copy constructor has the general form `ClassName(const ClassName& other)`. If a class owns a resource (heap memory, file, handle, etc.), a default member-wise copy may be incorrect; implement proper ownership semantics or use resource-managing members.

### 6.5 Move constructor (modern C++)

A move constructor can transfer resources from a temporary or expiring object rather than copy them. It commonly takes `ClassName&&`.

```cpp
ClassName(ClassName&& other) noexcept; // example signature
```

Use standard resource-managing types where possible; move operations are most useful for resource-owning types.

### 6.6 Conversion constructors and `explicit`

A constructor that accepts one argument can sometimes be used for implicit conversions. Mark it `explicit` to prevent unintended conversions.

```cpp
class Distance {
    double meters;
public:
    explicit Distance(double m) : meters(m) {}
};
```

## 7. Destructor

A **destructor** runs when an object’s lifetime ends. It has the form `~ClassName()` and takes no parameters.

```cpp
class Logger {
public:
    Logger() { cout << "Start\n"; }
    ~Logger() { cout << "End\n"; }
};
```

Destructors are used for cleanup, but modern C++ usually relies on RAII: resource lifetime is tied to object lifetime, and resource-managing members perform cleanup automatically.

### Destruction order

- Local objects are destroyed in reverse order of construction.
- Class members are destroyed in reverse declaration order.
- Derived objects are destroyed before their base subobjects (derived destructor, then base destructor).
- For polymorphic base classes deleted through a base pointer, the base destructor should normally be `virtual`.

## 8. `const` member functions and const objects

A member function marked `const` promises not to modify the object's observable non-`mutable` state.

```cpp
class Circle {
    double radius;
public:
    explicit Circle(double r) : radius(r) {}
    double area() const { return 3.14159 * radius * radius; }
};
```

A const object can call const member functions, but cannot call ordinary non-const member functions (unless those functions are otherwise callable through the interface).

## 9. Static members

A **static data member** belongs to the class as a whole, not to each individual object. A **static member function** can be called without an object and has no `this` pointer.

```cpp
class Counter {
    inline static int count = 0; // C++17 inline static member
public:
    Counter() { ++count; }
    static int getCount() { return count; }
};
```

Call with `Counter::getCount()`. Static functions can directly access static members, but not non-static members without an object.

## 10. Friend function and friend class

A `friend` declaration grants a specified function or class access to private and protected members. Friendship is not inherited, not reciprocal, and not automatically transitive.

```cpp
class Vault {
    int secret = 42;
    friend void reveal(const Vault& v);
};

void reveal(const Vault& v) {
    cout << v.secret << '\n';
}
```

Use friendship sparingly because it increases coupling.

## 11. Polymorphism in detail

### 11.1 Function overloading (compile-time)

Functions can share a name if their parameter lists differ in a way that creates distinct overloads. Return type alone cannot distinguish overloads.

```cpp
void print(int value);
void print(double value);
void print(const string& value);
```

### 11.2 Operator overloading

An operator can be defined for a user-defined type, allowing natural syntax. It should preserve intuitive meaning.

```cpp
class Vector2 {
public:
    double x, y;
    Vector2(double x, double y) : x(x), y(y) {}
    Vector2 operator+(const Vector2& rhs) const {
        return Vector2(x + rhs.x, y + rhs.y);
    }
};
```

Some operators cannot be overloaded; precedence and associativity cannot be changed. At least one operand for an overloaded operator must be a user-defined type.

### 11.3 Virtual functions (runtime)

A virtual function enables dynamic dispatch when called through a base-class pointer or reference. The implementation chosen is based on the object's dynamic type.

```cpp
class Shape {
public:
    virtual double area() const { return 0.0; }
    virtual ~Shape() = default;
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double width, double height) : w(width), h(height) {}
    double area() const override { return w * h; }
};

void showArea(const Shape& s) {
    cout << s.area() << '\n';
}
```

`override` tells the compiler that a derived member is intended to override a virtual function; use it to catch signature mismatches.

### 11.4 Pure virtual function and abstract class

A pure virtual function uses `= 0`. A class with at least one pure virtual function is abstract and cannot be instantiated directly.

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};
```

Abstract classes define an interface for derived concrete classes.

### 11.5 Virtual destructor

If a class is intended to be used polymorphically and objects may be deleted through a base pointer, make its destructor virtual.

```cpp
class Base {
public:
    virtual ~Base() = default;
};
```

Without a virtual destructor, deleting a derived object through a base pointer is undefined behavior.

### 11.6 `override` and `final`

- `override`: confirms that the function overrides a base virtual function.
- `final`: prevents a virtual function from being overridden further, or prevents a class from being derived from.

### 11.7 Object slicing

Assigning or passing a derived object **by value** as a base object copies only the base subobject, losing the derived portion (object slicing).

```cpp
Derived d;
Base b = d; // sliced: b is a Base object
```

Use references or pointers to base types when you intend polymorphic behavior.

## 12. Inheritance types and access modes

### Inheritance forms

- **Single:** one base class → one derived class.
- **Multilevel:** a class derives from a derived class (A → B → C).
- **Multiple:** one derived class has multiple base classes.
- **Hierarchical:** multiple classes derive from one base class.
- **Hybrid:** combination of inheritance patterns.

### Inheritance access modes

| Base member | `public` inheritance | `protected` inheritance | `private` inheritance |
| --- | --- | --- | --- |
| Base `public` | public in derived | protected in derived | private in derived |
| Base `protected` | protected in derived | protected in derived | private in derived |
| Base `private` | inaccessible directly | inaccessible directly | inaccessible directly |

```cpp
class Base { public: void f(); protected: void g(); private: void h(); };
class PublicChild : public Base {};    // f stays public; g stays protected
class ProtectedChild : protected Base {}; // f and g become protected
class PrivateChild : private Base {};  // f and g become private
```

Private base members still exist in the derived object but are not directly accessible by derived-class code.

## 13. Constructor/destructor order in inheritance

When creating a derived object:

1. Virtual base classes (when present) are initialized first.
2. Direct base classes initialize in declaration order.
3. Members initialize in declaration order.
4. Derived constructor body executes.

Destruction proceeds in reverse: derived destructor body, members, base subobjects, virtual bases. For ordinary single inheritance, the base constructor executes before the derived constructor, and the base destructor after the derived destructor.

## 14. Multiple inheritance and ambiguity

Multiple inheritance allows a class to inherit from more than one base class.

```cpp
class Printer { public: void print() {} };
class Scanner { public: void scan() {} };
class MultiFunction : public Printer, public Scanner {};
```

If two bases provide same-named members, a call may be ambiguous. Qualify the intended base member (`BaseA::method()`) or redesign the interface when appropriate.

### Diamond problem and virtual inheritance

When two intermediate classes inherit from the same base, a class inheriting from both may contain two copies of that common base. **Virtual inheritance** shares one common base subobject.

```cpp
class Person { public: int id; };
class Student : virtual public Person {};
class Employee : virtual public Person {};
class TeachingAssistant : public Student, public Employee {};
```

The `TeachingAssistant` object has one shared `Person` base subobject. The most-derived class is responsible for initializing virtual bases.

## 15. Association, aggregation, and composition

These describe object relationships; they are not special C++ keywords.

- **Association:** objects know about or interact with one another, without implying ownership.
- **Aggregation:** a whole contains parts, but parts can exist independently of the whole (often modeled with references/pointers).
- **Composition:** strong ownership; the part's lifetime is tied to the whole (often represented by a direct member object).

```cpp
class Engine { /* ... */ };
class Car {
    Engine engine; // composition: Engine is a member; lifetime follows Car
};
```

Prefer composition over inheritance when the relationship is **has-a** rather than **is-a**.

## 16. Copying, assignment, and the Rule of 3 / 5 / 0

- **Copy constructor:** creates a new object from another object.
- **Copy assignment:** replaces the state of an already-existing object with another object's state.
- **Move constructor / move assignment:** transfer resources from an expiring object where appropriate.

### Rule of 3

If a class manually manages a resource and needs a destructor, it often also needs a copy constructor and copy assignment operator.

### Rule of 5

In modern C++, also consider move constructor and move assignment operator when resource ownership is managed manually.

### Rule of 0

Prefer members that already manage resources correctly (e.g. `std::string`, containers, smart pointers) so the class needs no custom destructor, copy, or move operations.

## 17. Dynamic memory and object pointers

```cpp
class Demo { public: void hello() { cout << "Hi\n"; } };
Demo obj;
Demo* ptr = &obj;
ptr->hello(); // arrow operator accesses a member through a pointer
```

`obj.member` is used with an object; `ptr->member` is used with a pointer. A reference is an alias, while a pointer stores an address and can be reassigned or null.

For polymorphic ownership, use smart pointers such as `std::unique_ptr<Base>` (with a virtual base destructor when deleting derived objects through the base interface) rather than owning raw pointers.

## 18. Common OOP design guidelines

- Keep data private unless exposing it is intentional.
- Use constructors to establish valid initial state.
- Keep class responsibilities focused.
- Use `const` member functions for read-only operations.
- Use `override` for overrides and `virtual` destructors for polymorphic base classes.
- Prefer composition when reuse does not represent a true subtype relationship.
- Avoid inheritance solely to reuse a few functions; it can make APIs rigid and increase coupling.
- Avoid owning raw pointers; use RAII and smart pointers.
- Do not make every method virtual by default—virtual dispatch should be intentional.

## 19. Frequent mistakes and misconceptions

1. **Class vs object:** a class is a type/blueprint; an object is an instance.
2. **Private vs protected:** derived classes cannot directly access base `private` members.
3. **Constructor return type:** constructors do not declare a return type.
4. **Overload vs override:** overloads share a name but differ in parameters; overrides replace inherited virtual behavior with a matching signature.
5. **Virtual call misconception:** runtime dispatch requires a virtual function and normally a base pointer/reference; calling a non-virtual function through a base interface does not dynamically dispatch.
6. **Missing virtual destructor:** can cause undefined behavior when deleting derived instances through a base pointer.
7. **Slicing:** base objects passed/returned by value can lose derived state and polymorphic behavior.
8. **Initializer order:** actual initialization follows member declaration order.
9. **Friendship:** friend access does not make a function a member, and friendship is not inherited.
10. **Inheritance misuse:** “has-a” is not the same as “is-a.”

## 20. Short example combining core OOP concepts

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Animal { // abstraction/interface base
public:
    virtual void sound() const = 0; // pure virtual
    virtual ~Animal() = default;     // safe polymorphic destruction
};

class Dog : public Animal { // inheritance
public:
    void sound() const override { cout << "Woof\n"; } // polymorphism
};

class Cat : public Animal {
public:
    void sound() const override { cout << "Meow\n"; }
};

int main() {
    unique_ptr<Animal> pet = make_unique<Dog>();
    pet->sound(); // runtime dispatch: Dog::sound

    pet = make_unique<Cat>();
    pet->sound(); // runtime dispatch: Cat::sound
}
```

**What this demonstrates:** abstraction through `Animal`, inheritance through `Dog` and `Cat`, runtime polymorphism through virtual `sound()`, `override` for correctness, and safe ownership through `unique_ptr` plus a virtual destructor.

## 21. Quick revision table

| Concept | One-line meaning |
| --- | --- |
| Class | Defines a user-defined type and its members |
| Object | Instance of a class |
| Encapsulation | Bundle state/behavior and control access |
| Abstraction | Expose essential interface, hide implementation detail |
| Inheritance | Derive a type from another to specialize or reuse behavior |
| Polymorphism | One interface/operation can behave in different forms |
| Constructor | Initializes an object when it is created |
| Destructor | Runs as an object's lifetime ends |
| Overloading | Same function/operator name with different parameter signatures |
| Overriding | Derived class provides a new implementation of inherited virtual function |
| Virtual function | Enables dynamic dispatch through base interface |
| Pure virtual | Virtual function declared `= 0`; makes class abstract (if not otherwise overridden) |
| Abstract class | Cannot be instantiated directly; defines an interface or partial implementation |
| `friend` | Grants designated function/class access to private/protected members |
| `static` member | Belongs to the class, not an individual instance |
| Composition | Whole owns parts through member objects |
| Object slicing | Derived object copied into base value, losing derived portion |

## 22. Practice / exam questions

1. Define OOP and explain how it differs from procedural programming.
2. What is the difference between a class and an object?
3. Explain `private`, `protected`, and `public` with examples.
4. Explain the four pillars of OOP.
5. What is a constructor? Describe default, parameterized, and copy constructors.
6. What is a destructor? When should a base-class destructor be virtual?
7. Explain constructor overloading and function overloading.
8. Differentiate overloading, overriding, and hiding.
9. Explain compile-time and runtime polymorphism.
10. What is a pure virtual function? What is an abstract class?
11. Explain inheritance modes and types of inheritance.
12. What is multiple inheritance? What is the diamond problem? How does virtual inheritance help?
13. Explain static members and friend functions.
14. What are association, aggregation, and composition?
15. Explain object slicing and how to avoid it.
16. Describe the Rule of 3, Rule of 5, and Rule of 0.
17. Why is composition often preferred over inheritance?

---

## Suggested study order

1. Class, object, member functions, and access specifiers
2. Encapsulation and abstraction
3. Constructors, initializer lists, destructors, `this`, and `const`
4. Static and friend members
5. Inheritance and inheritance modes
6. Overloading and operator overloading
7. Virtual functions, abstract classes, and runtime polymorphism
8. Multiple inheritance, ambiguity, and virtual inheritance
9. Relationships (association/aggregation/composition), object slicing, and resource ownership
