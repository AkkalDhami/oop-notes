# Object-Oriented Programming (OOP) in JavaScript — Detailed Notes

> A detailed OOP study note focused on JavaScript's object model, prototypes, classes, inheritance, encapsulation, polymorphism, and practical patterns.

---

## 1. What is Object-Oriented Programming?

**Object-Oriented Programming (OOP)** organizes programs around objects that combine data (state) and functions (behavior).

JavaScript supports OOP, but it is not exclusively class-based. Its underlying inheritance model is **prototype-based**.

### Why OOP?
- Organize related state and behavior.
- Reuse behavior without duplicating code.
- Protect object state and establish clear APIs.
- Model entities and relationships.
- Make code easier to extend and maintain when responsibilities are well-defined.

### Four OOP pillars
| Pillar | Meaning |
|---|---|
| Encapsulation | Bundle state and behavior; control access to internal data. |
| Abstraction | Provide a clear interface while hiding implementation details. |
| Inheritance | Reuse or specialize behavior from another type/object. |
| Polymorphism | Different objects can be used through a shared interface but behave differently. |

---

## 2. Objects

An **object** is a collection of properties. A property can contain data or a function (a method).

```js
const user = {
  name: "Akkal",
  age: 20,
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  },
};

user.greet();
```

### State and behavior
- **State:** `name`, `age`
- **Behavior:** `greet()`

### Accessing properties
```js
console.log(user.name);       // dot notation
console.log(user["age"]);     // bracket notation

user.age = 21;                // update
user.role = "developer";       // add
delete user.role;              // remove
```

Use bracket notation when the property name is dynamic or contains characters unsuitable for dot notation.

---

## 3. Object Creation Patterns

### Object literal
Best for simple, one-off objects.

```js
const point = { x: 10, y: 20 };
```

### Factory function
A function that returns objects; useful when you want private state with closures.

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count++;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
counter.increment();
console.log(counter.getCount()); // 1
```

### Constructor function
Historically used with `new` before `class` syntax.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function () {
  console.log(`Hi, ${this.name}`);
};

const person = new Person("Akkal");
person.greet();
```

### `Object.create()`
Create an object with a chosen prototype.

```js
const animal = {
  speak() {
    console.log("Some sound");
  },
};

const dog = Object.create(animal);
dog.speak();
```

---

## 4. `this` Keyword

`this` is determined primarily by **how a function is called**, not where it was defined.

```js
const person = {
  name: "Akkal",
  greet() {
    console.log(this.name);
  },
};

person.greet(); // "Akkal"
```

### Common `this` rules
| Call form | Typical `this` value |
|---|---|
| `obj.method()` | `obj` |
| Plain function call (`fn()`) | `undefined` in strict mode; global object in non-strict scripts |
| `new Fn()` | Newly created instance |
| `fn.call(obj)` / `fn.apply(obj)` | Explicitly provided object |
| Arrow function | Lexically inherited `this`; no own `this` |

### Arrow-function caveat
Arrow functions are usually unsuitable as object methods when the method needs the object's `this`.

```js
const user = {
  name: "Akkal",
  greet: () => console.log(this.name), // not user.name
};
```

---

## 5. Prototypes and Prototype Chain

JavaScript objects can inherit properties from another object through their internal prototype link.

### Prototype
A **prototype** is an object from which another object can inherit properties and methods.

```js
const animal = {
  eat() {
    console.log("Eating");
  },
};

const dog = Object.create(animal);
dog.bark = () => console.log("Barking");

dog.eat();  // inherited
dog.bark(); // own
```

### Prototype chain
When a property is accessed, JavaScript:
1. Checks the object itself.
2. If not found, checks its prototype.
3. Continues upward through the prototype chain.
4. Stops when found or when the chain reaches `null`.

<pre>
dog
  ↓ [[Prototype]]
animal
  ↓ [[Prototype]]
Object.prototype
  ↓ [[Prototype]]
null
</pre>

### Useful prototype methods
```js
Object.getPrototypeOf(dog);
Object.setPrototypeOf(dog, animal); // generally avoid changing it repeatedly
Object.hasOwn(dog, "bark");        // own property?
"eat" in dog;                     // own OR inherited?
```

Prefer `Object.create()` for deliberate prototype setup; avoid frequent runtime prototype mutation because it can harm performance and readability.

---

## 6. Classes

JavaScript `class` syntax provides a structured way to create objects and set up inheritance. Class bodies run in strict mode.

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hi, I'm ${this.name}`);
  }
}

const person = new Person("Akkal", 20);
person.greet();
```

### Class components
- `constructor()` initializes the instance.
- Instance methods are stored on the class prototype, not copied onto every instance.
- `new` creates an instance and calls the constructor.
- Classes cannot be called without `new`.

### Instance fields
```js
class User {
  name = "Guest";
  active = true;
}
```

---

## 7. Constructor and `new`

A constructor initializes the new instance.

```js
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

const rect = new Rectangle(5, 4);
console.log(rect.area()); // 20
```

Conceptually, `new`:
1. Creates a new object.
2. Links its prototype to the constructor's prototype.
3. Calls the constructor with the new object as `this`.
4. Returns the new object (unless the constructor explicitly returns a suitable object).

A class constructor must not be invoked as a normal function.

---

## 8. Encapsulation

**Encapsulation** combines state and behavior and limits direct access to internal state.

JavaScript offers several approaches.

### Convention-based privacy
```js
class BankAccount {
  constructor(balance) {
    this._balance = balance; // underscore is only a convention
  }
}
```

`_balance` is not actually private; callers can still access it.

### Private fields (`#`)
Private fields are enforced by the language and can only be accessed inside the class body (and appropriate lexical class contexts).

```js
class BankAccount {
  #balance;

  constructor(initialBalance) {
    if (initialBalance < 0) throw new Error("Invalid balance");
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
// account.#balance; // SyntaxError outside the class
```

Encapsulation is more than making fields inaccessible: the class should maintain its invariants and expose meaningful operations.

---

## 9. Getters and Setters

Getters and setters provide property-like access to computed values or validated updates.

```js
class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }

  get celsius() {
    return this._celsius;
  }

  set celsius(value) {
    if (typeof value !== "number") throw new TypeError("Must be numeric");
    this._celsius = value;
  }

  get fahrenheit() {
    return (this._celsius * 9) / 5 + 32;
  }
}

const temp = new Temperature(25);
console.log(temp.fahrenheit); // 77
temp.celsius = 30;
```

Getters/setters should add useful semantics, validation, or computed behavior—not simply exist for every field automatically.

---

## 10. Abstraction

**Abstraction** exposes the essential operations while hiding the implementation.

JavaScript does not have a built-in `abstract` class keyword. You can enforce abstract behavior with a base class convention or design an interface-like contract with documentation, TypeScript, or runtime checks.

```js
class Shape {
  area() {
    throw new Error("area() must be implemented by subclass");
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius ** 2;
  }
}
```

This is a runtime convention, not compile-time abstract enforcement.

---

## 11. Inheritance (`extends`)

Inheritance allows a class to extend another class and reuse or specialize its behavior.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks`);
  }
}

const dog = new Dog("Buddy");
dog.speak(); // Buddy barks
```

### JavaScript inheritance facts
- A class can extend one class.
- A subclass inherits methods through the prototype chain.
- A subclass can override inherited methods.
- `extends` works with classes and constructor functions that are constructible.

---

## 12. `super`

`super` refers to the superclass constructor or superclass methods.

### Call superclass constructor
A derived class constructor must call `super()` before using `this`.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
}
```

### Call superclass method
```js
class Animal {
  speak() {
    return "Some sound";
  }
}

class Dog extends Animal {
  speak() {
    return `${super.speak()} — woof!`;
  }
}
```

---

## 13. Polymorphism

**Polymorphism** allows different objects to be used through a common expected interface while each object provides its own behavior.

JavaScript is dynamically typed, so this is often achieved through **duck typing**: if an object supports the required method, it can be used.

```js
class Dog {
  speak() {
    console.log("Woof");
  }
}

class Cat {
  speak() {
    console.log("Meow");
  }
}

function makeAnimalSpeak(animal) {
  animal.speak();
}

makeAnimalSpeak(new Dog());
makeAnimalSpeak(new Cat());
```

No common superclass is required. The objects simply satisfy the behavior expected by `makeAnimalSpeak`.

---

## 14. Method Overriding

A subclass can provide a method with the same name to specialize behavior.

```js
class Notification {
  send() {
    console.log("Sending notification");
  }
}

class EmailNotification extends Notification {
  send() {
    console.log("Sending email");
  }
}

const notification = new EmailNotification();
notification.send(); // Sending email
```

JavaScript has no `@Override` annotation. A method can replace inherited behavior without a compiler verifying that the base method exists.

---

## 15. Static Members

Static fields and methods belong to the class itself, not to individual instances.

```js
class MathHelper {
  static add(a, b) {
    return a + b;
  }
}

console.log(MathHelper.add(2, 3)); // 5
```

### Static fields
```js
class User {
  static count = 0;

  constructor() {
    User.count++;
  }
}
```

Static members are often useful for utility methods, shared counters, or factory methods.

---

## 16. `instanceof`

The `instanceof` operator tests whether a constructor's prototype appears in an object's prototype chain.

```js
class Animal {}
class Dog extends Animal {}

const dog = new Dog();

console.log(dog instanceof Dog);    // true
console.log(dog instanceof Animal); // true
```

`instanceof` can be misleading across realms (for example, objects from different browser windows) or when prototypes are manually changed. For behavior-based checks, consider checking capabilities instead.

---

## 17. Composition (Has-a Relationship)

Composition creates objects that contain or delegate to other objects rather than inheriting from them.

```js
class Engine {
  start() {
    console.log("Engine started");
  }
}

class Car {
  constructor(engine) {
    this.engine = engine;
  }

  start() {
    this.engine.start();
  }
}

const car = new Car(new Engine());
car.start();
```

`Car` has an `Engine`.

Composition often gives more flexibility than a deep inheritance hierarchy because components can be replaced or configured.

---

## 18. Association, Aggregation, and Composition

These are conceptual relationships between objects.

| Relationship | Meaning | Ownership |
|---|---|---|
| Association | Objects interact or reference each other | No ownership implied |
| Aggregation | Whole-part relationship with independently existing parts | Weak ownership/shared parts |
| Composition | Whole owns its parts as a strong lifecycle/design relationship | Strong ownership |

JavaScript does not enforce these relationship types; they are design descriptions.

---

## 19. Object Destructuring and Methods

Destructuring extracts property values into variables.

```js
const user = {
  name: "Akkal",
  role: "Developer",
};

const { name, role } = user;
```

Be careful when extracting methods that depend on `this`:

```js
const person = {
  name: "Akkal",
  greet() {
    console.log(this.name);
  },
};

const greet = person.greet;
// greet(); // `this` no longer points to person

const boundGreet = person.greet.bind(person);
boundGreet();
```

---

## 20. Object Property Descriptors

Properties can have metadata that controls enumerability, writability, and configurability.

```js
const user = {};

Object.defineProperty(user, "id", {
  value: 101,
  writable: false,
  enumerable: true,
  configurable: false,
});

console.log(user.id);
```

- `writable`: whether the value can be reassigned.
- `enumerable`: whether it appears in common enumeration operations.
- `configurable`: whether descriptor settings can be changed or property deleted.

These are property-level controls, distinct from private fields.

---

## 21. Copying Objects: Shallow vs Deep

### Shallow copy
Copies own enumerable properties, but nested object references remain shared.

```js
const original = {
  name: "Akkal",
  address: { city: "Kathmandu" },
};

const copy = { ...original };
copy.address.city = "Pokhara";

console.log(original.address.city); // "Pokhara" (same nested object)
```

### Deep copy
For supported structured data, `structuredClone()` can create a deep copy.

```js
const clone = structuredClone(original);
```

Deep copying has limitations for some values (e.g., functions); not every object can be cloned with it. Choose a cloning approach based on the data you need to preserve.

---

## 22. Object Equality

Objects are compared by identity using `===`, not by comparing their properties.

```js
const a = { value: 1 };
const b = { value: 1 };
const c = a;

console.log(a === b); // false (different objects)
console.log(a === c); // true (same object)
```

For structural equality, compare fields or use a carefully chosen utility/library. JSON string comparison is not a general equality solution (ordering, unsupported values, cycles, and other edge cases).

---

## 23. Mixins

JavaScript does not have built-in multiple class inheritance. Mixins can add behavior by copying methods or composing classes.

```js
const CanWalk = (Base) =>
  class extends Base {
    walk() {
      console.log("Walking");
    }
  };

class Person {}

class WalkingPerson extends CanWalk(Person) {}

new WalkingPerson().walk();
```

Mixins can be useful but can also cause name collisions and unclear method resolution. Prefer small, well-defined capabilities.

---

## 24. `Object.freeze`, `Object.seal`, and `Object.preventExtensions`

These operations restrict changes to objects.

| Operation | Add properties | Delete properties | Change existing values |
|---|---|---|---|
| `Object.preventExtensions()` | No | Yes | Yes |
| `Object.seal()` | No | No | Yes (if writable) |
| `Object.freeze()` | No | No | No (for own data properties) |

These are shallow operations; nested objects are not automatically frozen.

```js
const settings = Object.freeze({ theme: "dark" });
settings.theme = "light"; // ignored in sloppy mode; throws in strict mode
```

---

## 25. Factory Pattern and Dependency Injection (OOP Design)

Factory functions or static factory methods can hide object creation details.

```js
class Logger {
  log(message) {
    console.log(message);
  }
}

class App {
  constructor(logger) {
    this.logger = logger;
  }

  run() {
    this.logger.log("App started");
  }
}

const app = new App(new Logger());
app.run();
```

**Dependency injection** means supplying dependencies from outside rather than constructing them inside the class. It helps testing and replacement of implementations.

---

## 26. Object Lifecycle

JavaScript does not provide deterministic destructors like C++.

- Objects are created through literals, constructors, factories, or built-in operations.
- References allow code to access objects.
- The garbage collector reclaims memory for unreachable objects.
- Garbage-collection timing is not guaranteed.
- Resource cleanup (files, sockets, locks) should use explicit cleanup APIs, `try/finally`, or supported resource-management patterns rather than relying on garbage collection.

---

## 27. JavaScript OOP vs Java/C++ (Quick Comparison)

| Feature | JavaScript | Java | C++ |
|---|---|---|---|
| Core inheritance model | Prototype-based | Class-based | Class-based |
| `class` syntax | Prototype-based syntax layer | Native class model | Native class model |
| Multiple inheritance | No class multiple inheritance; mixins/prototypes | Multiple interfaces | Multiple base classes |
| Type system | Dynamic | Static | Static |
| Method dispatch | Dynamic | Virtual instance dispatch | Depends on virtual declarations |
| Encapsulation | Closures, `#private`, conventions | Access modifiers | Access modifiers |
| Memory management | Garbage-collected | Garbage-collected | Multiple ownership strategies |
| Operator overloading | Not user-defined | Not user-defined (except built-in `+` behavior) | Supported |
| Interface keyword | No native `interface` in JS | Yes | Abstract classes/contracts via patterns |

---

## 28. Common OOP Mistakes in JavaScript

1. Thinking `_field` is truly private (it is only a naming convention).
2. Using arrow functions as object methods when `this` is required.
3. Forgetting `super()` in a derived constructor.
4. Confusing prototype inheritance with copying properties.
5. Using inheritance only to reuse code when composition is clearer.
6. Assuming `instanceof` is a universal type test.
7. Extracting a method and losing its `this` context.
8. Assuming object spread performs a deep copy.
9. Comparing separate objects with `===` and expecting structural equality.
10. Making a huge class responsible for too many unrelated operations.

---

## 29. OOP Practice Questions

### Conceptual
1. What is OOP and what are its benefits?
2. Explain the four pillars of OOP.
3. What is the difference between an object and a class?
4. What is the prototype chain?
5. How does `this` work in JavaScript?
6. Class syntax vs prototype-based inheritance?
7. Encapsulation with `_` vs `#private`?
8. What is polymorphism and duck typing?
9. Inheritance vs composition?
10. What is the difference between shallow and deep copy?
11. What do `static`, `instanceof`, `super`, and `extends` do?
12. What are mixins?

### Coding
1. Create a `Student` class with private fields and methods.
2. Create a bank account with validated deposits and withdrawals.
3. Demonstrate constructor function and prototype methods.
4. Build a base `Animal` class and override `speak()` in subclasses.
5. Demonstrate polymorphism with duck typing.
6. Use composition to build `Car` from an `Engine`.
7. Write a static factory method.
8. Demonstrate method binding with `bind()`.
9. Implement a mixin for a capability like `walk()`.
10. Show shallow copy versus deep copy.

---

## Recommended Study Order

1. Objects and object literals
2. `this` and methods
3. Object creation patterns
4. Prototypes and prototype chain
5. Classes and constructors
6. Encapsulation, private fields, getters/setters
7. Abstraction
8. Inheritance, `extends`, `super`
9. Overriding and polymorphism
10. Static members
11. Composition and object relationships
12. `instanceof`, equality, copying
13. Property descriptors and object restriction
14. Mixins, factories, dependency injection
15. Lifecycle and garbage collection

> **Core principle:** JavaScript OOP is flexible: objects can share behavior through prototypes, classes, or composition. Choose the simplest design that gives each object a clear responsibility and protects the invariants it owns.
