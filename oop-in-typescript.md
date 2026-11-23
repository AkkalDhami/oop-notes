# Object-Oriented Programming (OOP) in TypeScript

> A detailed OOP study note focused on TypeScript: classes, access control, interfaces, inheritance, polymorphism, abstract classes, generics, composition, and how TypeScript's type system supports object-oriented design.

---

## 1. What is Object-Oriented Programming?

**Object-Oriented Programming (OOP)** organizes programs around objects that combine state (data) and behavior (methods).

TypeScript builds on JavaScript's runtime object model and adds a static type system. It supports classes, interfaces, access modifiers, abstract classes, and structural typing.

### Why OOP?
- Organize state and behavior into focused units.
- Reuse behavior and extend types.
- Encapsulate implementation and protect invariants.
- Make systems easier to maintain and test.

### Four pillars
| Pillar | Meaning |
|---|---|
| Encapsulation | Protect internal state and control how it changes. |
| Abstraction | Expose useful contracts and hide implementation details. |
| Inheritance | Specialize a base type and reuse behavior. |
| Polymorphism | Use a shared contract for different implementations. |

---

## 2. Objects and Types

An object has properties and methods.

```ts
type User = {
  name: string;
  age: number;
  greet(): void;
};

const user: User = {
  name: "Akkal",
  age: 20,
  greet() {
    console.log(`Hello, ${this.name}`);
  },
};

user.greet();
```

TypeScript checks that the object conforms to the declared type. The type does not generally exist at runtime.

### State and behavior
- **State:** `name`, `age`
- **Behavior:** `greet()`

### Structural typing
TypeScript usually compares object types by their shape (the members they have), not by an explicit declared relationship.

```ts
type Point = { x: number; y: number };

function printPoint(point: Point) {
  console.log(point.x, point.y);
}

const position = { x: 10, y: 20, label: "origin-ish" };
printPoint(position); // valid: it has x and y
```

---

## 3. Classes

A **class** defines instance state, constructors, and methods.

```ts
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): void {
    console.log(`Hi, I'm ${this.name}`);
  }
}

const person = new Person("Akkal", 20);
person.greet();
```

### Type annotations
- `name: string` — field type
- `age: number` — field type
- `: void` — method does not return a meaningful value

### Shorthand parameter properties
TypeScript can declare and initialize fields from constructor parameters:

```ts
class Person {
  constructor(
    public name: string,
    private age: number,
  ) {}
}
```

This is equivalent to declaring the fields and assigning them in the constructor, with the indicated access modifiers.

---

## 4. Fields, Methods, and `this`

### Instance fields
Each object has its own instance fields.

### Methods
Methods define behavior.

```ts
class Counter {
  count = 0;

  increment(): void {
    this.count++;
  }
}
```

### `this`
`this` refers to the current object for a normal method call, but its value can depend on how the function is invoked.

```ts
class User {
  constructor(public name: string) {}

  greet(): void {
    console.log(this.name);
  }
}

const user = new User("Akkal");
user.greet();
```

When extracting methods, preserve or bind the receiver if the method relies on `this`.

---

## 5. Access Modifiers

TypeScript provides member access modifiers.

| Modifier | Meaning |
|---|---|
| `public` | Accessible anywhere (default for class members). |
| `private` | Accessible only within the declaring class (TypeScript compile-time enforcement). |
| `protected` | Accessible within the class and subclasses. |
| `readonly` | Prevents reassignment after initialization according to TypeScript rules. |

```ts
class BankAccount {
  public owner: string;
  private balance: number;
  protected accountId: string;
  readonly currency = "USD";

  constructor(owner: string, balance: number, accountId: string) {
    this.owner = owner;
    this.balance = balance;
    this.accountId = accountId;
  }
}
```

**Important:** TypeScript's `private` and `protected` are generally enforced by the type checker and erased at runtime. JavaScript `#private` fields provide runtime privacy.

---

## 6. Encapsulation

**Encapsulation** protects an object's internal state and provides controlled operations.

```ts
class BankAccount {
  private balance: number;

  constructor(initialBalance: number) {
    if (initialBalance < 0) {
      throw new Error("Balance cannot be negative");
    }
    this.balance = initialBalance;
  }

  deposit(amount: number): void {
    if (amount <= 0) {
      throw new Error("Deposit must be positive");
    }
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

### Runtime-private fields
Use `#balance` when runtime privacy is required:

```ts
class SecureAccount {
  #balance = 0;

  deposit(amount: number): void {
    if (amount > 0) this.#balance += amount;
  }

  get balance(): number {
    return this.#balance;
  }
}
```

Encapsulation should maintain valid state; it is not simply about generating getters and setters for every field.

---

## 7. Getters and Setters

Getters and setters provide property-like access to computed values or validated updates.

```ts
class Temperature {
  private _celsius: number;

  constructor(celsius: number) {
    this._celsius = celsius;
  }

  get celsius(): number {
    return this._celsius;
  }

  set celsius(value: number) {
    if (!Number.isFinite(value)) throw new Error("Invalid temperature");
    this._celsius = value;
  }

  get fahrenheit(): number {
    return (this._celsius * 9) / 5 + 32;
  }
}

const temperature = new Temperature(25);
console.log(temperature.fahrenheit);
temperature.celsius = 30;
```

Access them like properties: `temperature.celsius`, not `temperature.celsius()`.

---

## 8. Abstraction

**Abstraction** provides a simple contract for using a type while hiding its implementation details.

In TypeScript, abstraction is often expressed with interfaces or abstract classes.

```ts
interface PaymentMethod {
  pay(amount: number): void;
}

class CardPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Paid ${amount} by card`);
  }
}
```

The caller depends on `PaymentMethod`, not on card-processing implementation details.

---

## 9. Constructors

A constructor initializes a new instance.

```ts
class Rectangle {
  constructor(
    public width: number,
    public height: number,
  ) {}

  area(): number {
    return this.width * this.height;
  }
}

const rectangle = new Rectangle(5, 4);
console.log(rectangle.area()); // 20
```

### Constructor overloading
TypeScript supports multiple constructor signatures but only one implementation.

```ts
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number);
  constructor(coords: { x: number; y: number });
  constructor(xOrCoords: number | { x: number; y: number }, y?: number) {
    if (typeof xOrCoords === "number") {
      this.x = xOrCoords;
      this.y = y ?? 0;
    } else {
      this.x = xOrCoords.x;
      this.y = xOrCoords.y;
    }
  }
}
```

---

## 10. `static` Members

Static members belong to the class, not instances.

```ts
class Counter {
  static total = 0;

  constructor() {
    Counter.total++;
  }

  static getTotal(): number {
    return Counter.total;
  }
}

new Counter();
new Counter();
console.log(Counter.getTotal()); // 2
```

Use static members for class-wide data, utilities, or factories. Avoid static state when it creates hidden dependencies or complicates testing.

---

## 11. `readonly`

`readonly` prevents assignment to a property after its allowed initialization points.

```ts
class User {
  constructor(
    public readonly id: number,
    public name: string,
  ) {}
}

const user = new User(1, "Akkal");
// user.id = 2; // TypeScript error
user.name = "Updated";
```

`readonly` is a compile-time restriction, not a general runtime freeze. It is also shallow for object-valued properties.

---

## 12. Inheritance (`extends`)

Inheritance allows a subclass to reuse or specialize behavior from a superclass.

```ts
class Animal {
  constructor(public name: string) {}

  speak(): void {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  bark(): void {
    console.log(`${this.name} barks`);
  }
}

const dog = new Dog("Buddy");
dog.speak();
dog.bark();
```

TypeScript classes support single class inheritance.

---

## 13. `super`

`super` calls the parent constructor or a parent method.

```ts
class Animal {
  constructor(public name: string) {}

  speak(): string {
    return "Some sound";
  }
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name);
  }

  speak(): string {
    return `${super.speak()} — woof`;
  }
}
```

A derived constructor must call `super()` before accessing `this`.

---

## 14. Method Overloading

TypeScript supports overload signatures that describe different accepted argument combinations, but the implementation must handle them.

```ts
class Calculator {
  add(a: number, b: number): number;
  add(a: string, b: string): string;

  add(a: number | string, b: number | string): number | string {
    if (typeof a === "number" && typeof b === "number") {
      return a + b;
    }
    if (typeof a === "string" && typeof b === "string") {
      return a + b;
    }
    throw new Error("Arguments must have matching types");
  }
}

const calculator = new Calculator();
calculator.add(2, 3);
calculator.add("Hello, ", "world");
```

Overload signatures are compile-time declarations. JavaScript output contains only the implementation.

---

## 15. Method Overriding

A subclass can override a base-class method.

```ts
class Animal {
  speak(): void {
    console.log("Animal sound");
  }
}

class Dog extends Animal {
  override speak(): void {
    console.log("Woof");
  }
}
```

The `override` keyword (when `noImplicitOverride` is enabled, it can be required for overrides) helps ensure the method actually exists in the base class. It is a type-checking aid, not a runtime feature.

---

## 16. Polymorphism

**Polymorphism** means different implementations can be used through a shared contract.

```ts
interface Animal {
  speak(): void;
}

class Dog implements Animal {
  speak(): void {
    console.log("Woof");
  }
}

class Cat implements Animal {
  speak(): void {
    console.log("Meow");
  }
}

function makeAnimalSpeak(animal: Animal): void {
  animal.speak();
}

makeAnimalSpeak(new Dog());
makeAnimalSpeak(new Cat());
```

TypeScript uses structural typing, so unrelated classes can satisfy the same interface by having the required members.

---

## 17. Abstract Classes

An **abstract class** cannot be instantiated directly and can require subclasses to implement abstract methods.

```ts
abstract class Shape {
  abstract area(): number;

  describe(): void {
    console.log(`Area: ${this.area()}`);
  }
}

class Circle extends Shape {
  constructor(public radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

Abstract classes can contain state, constructors, abstract methods, and concrete methods.

---

## 18. Interfaces

An **interface** defines a type contract.

```ts
interface Printable {
  print(): void;
}

interface Scannable {
  scan(): string;
}

class MultiFunctionPrinter implements Printable, Scannable {
  print(): void {
    console.log("Printing");
  }

  scan(): string {
    return "Scanned document";
  }
}
```

### Interface features
- Describes object shapes.
- Can be implemented by classes.
- Can extend one or more interfaces.
- Supports optional properties (`?`), readonly properties, method signatures, index signatures, and generics.
- Does not produce runtime JavaScript by itself.

### Interface vs abstract class
| Interface | Abstract class |
|---|---|
| Defines a type contract | Defines a base class with shared implementation/state |
| Can be extended by multiple interfaces | A class can extend only one class |
| No runtime representation by default | Exists as a JavaScript class at runtime |
| A class can implement many interfaces | Useful for shared fields/constructor logic |

---

## 19. Type Aliases vs Interfaces

Both can describe object shapes, but they have different capabilities.

```ts
interface User {
  name: string;
}

type Admin = User & {
  permissions: string[];
};
```

| `interface` | `type` |
|---|---|
| Often preferred for extendable object contracts | Can describe unions, intersections, primitives, tuples, and more |
| Can be declaration-merged | Cannot be declaration-merged in the same way |
| Extends interfaces with `extends` | Combines shapes with intersection (`&`) |

There is no universal rule that interfaces are always better. Choose based on the type shape and desired extensibility.

---

## 20. Composition

Composition builds a type from smaller objects/components rather than using inheritance.

```ts
class Engine {
  start(): void {
    console.log("Engine started");
  }
}

class Car {
  constructor(private engine: Engine) {}

  start(): void {
    this.engine.start();
  }
}

const car = new Car(new Engine());
car.start();
```

Composition promotes replaceable dependencies and simpler inheritance structures.

---

## 21. Association, Aggregation, and Composition

| Relationship | Meaning | Ownership |
|---|---|---|
| Association | Objects reference/use each other | No ownership implied |
| Aggregation | Whole-part with independently existing parts | Weak/shared ownership |
| Composition | Whole owns its parts as a strong lifecycle/design relationship | Strong ownership |

These are conceptual design relationships; TypeScript does not enforce them automatically.

---

## 22. Generics in OOP

Generics allow classes, interfaces, and methods to operate on typed values without losing type information.

```ts
class Box<T> {
  constructor(public value: T) {}

  getValue(): T {
    return this.value;
  }
}

const numberBox = new Box<number>(42);
const stringBox = new Box<string>("Hello");

numberBox.getValue(); // number
stringBox.getValue(); // string
```

Generics can also constrain acceptable types:

```ts
interface Identifiable {
  id: string | number;
}

class Repository<T extends Identifiable> {
  constructor(private items: T[]) {}

  findById(id: T["id"]): T | undefined {
    return this.items.find((item) => item.id === id);
  }
}
```

---

## 23. Accessors and Parameter Properties

TypeScript supports concise property declaration in constructor parameters.

```ts
class Product {
  constructor(
    public name: string,
    private price: number,
    readonly sku: string,
  ) {}

  getPrice(): number {
    return this.price;
  }
}
```

The modifiers create class properties and assign constructor arguments to them.

---

## 24. Enums and Literal Unions

Enums represent named constants; unions of literals are also commonly used.

```ts
enum Status {
  Pending,
  Approved,
  Rejected,
}

type Theme = "light" | "dark" | "system";
```

Use literal unions for simple closed sets of values when they fit naturally with TypeScript's type system. Enums have runtime output (depending on enum form and compiler settings), unlike many type-only constructs.

---

## 25. `instanceof`, Type Guards, and Narrowing

`instanceof` can test class instances at runtime, and TypeScript narrows the type in the guarded branch.

```ts
class Dog {
  bark(): void {
    console.log("Woof");
  }
}

class Cat {
  meow(): void {
    console.log("Meow");
  }
}

function handleAnimal(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

### Custom type guard
```ts
type User = { name: string };

function isUser(value: unknown): value is User {
  return typeof value === "object"
    && value !== null
    && "name" in value
    && typeof (value as { name?: unknown }).name === "string";
}
```

Type guards allow TypeScript to narrow union types. Type assertions do not validate data at runtime.

---

## 26. Object Methods: `toString`, Equality, and Identity

JavaScript objects compare by reference identity with `===`; TypeScript does not change that runtime behavior.

```ts
const a = { value: 1 };
const b = { value: 1 };
const c = a;

console.log(a === b); // false
console.log(a === c); // true
```

Classes can define a `toString()` method, but TypeScript does not have Java's automatic `equals()`/`hashCode()` contract. Implement equality explicitly when the domain requires value equality.

---

## 27. `readonly` Objects and Utility Types

TypeScript provides built-in utility types that transform types.

```ts
interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
type UserPreview = Pick<User, "id" | "name">;
type UserWithoutId = Omit<User, "id">;
```

These transformations are type-level only (unless you separately freeze or transform values at runtime).

### Common utility types
| Utility | Purpose |
|---|---|
| `Readonly<T>` | Makes properties readonly at type level |
| `Partial<T>` | Makes properties optional |
| `Required<T>` | Makes properties required |
| `Pick<T, K>` | Selects a subset of properties |
| `Omit<T, K>` | Excludes selected properties |
| `Record<K, V>` | Maps keys to a value type |

---

## 28. Dependency Injection

Dependency injection means providing an object's dependencies from outside instead of hard-coding their creation.

```ts
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(message);
  }
}

class Application {
  constructor(private logger: Logger) {}

  run(): void {
    this.logger.log("Application started");
  }
}

const app = new Application(new ConsoleLogger());
app.run();
```

Benefits:
- Easier to test with mock/fake implementations.
- Easier to replace infrastructure.
- Makes dependencies explicit.

---

## 29. SOLID Principles (OOP Design Guidelines)

SOLID are guidelines for designing maintainable object-oriented systems.

| Principle | Meaning |
|---|---|
| S — Single Responsibility | A class/module should have one cohesive responsibility. |
| O — Open/Closed | Extend behavior without repeatedly modifying stable code. |
| L — Liskov Substitution | Subtypes should be usable wherever the base type is expected without breaking correctness. |
| I — Interface Segregation | Prefer small, focused interfaces over large ones. |
| D — Dependency Inversion | High-level code should depend on abstractions rather than concrete details. |

SOLID are not rules to apply mechanically; use them to diagnose coupling and design problems.

---

## 30. TypeScript vs JavaScript OOP

| Feature | TypeScript | JavaScript |
|---|---|---|
| Static type checking | Yes (compile time) | No built-in static checker |
| Interfaces | Yes (type-level) | No `interface` syntax |
| `private` / `protected` | Type-checking access controls | No corresponding TS modifiers |
| `#private` fields | Runtime private fields supported | Runtime private fields supported |
| Abstract class | Compile-time restrictions, emitted class | No native `abstract` keyword |
| Generics | Compile-time type parameters | No native generic syntax |
| Structural typing | Type checker | Runtime duck typing |
| Runtime | JavaScript | JavaScript |

Remember: TypeScript types and interfaces are generally erased during compilation. Runtime validation is needed for untrusted data.

---

## 31. Common OOP Mistakes in TypeScript

1. Treating TypeScript `private` as runtime privacy (use `#private` for that).
2. Using `any` excessively, bypassing type safety.
3. Overusing inheritance instead of composition.
4. Creating large classes with unrelated responsibilities.
5. Assuming interfaces exist at runtime.
6. Using type assertions as if they validate values.
7. Adding getters/setters without real validation or abstraction.
8. Forgetting `override` when project conventions/compiler settings require it.
9. Assuming `readonly` deeply freezes an object.
10. Ignoring null/undefined in strict mode.

---

## 32. OOP Practice Questions

### Conceptual
1. What is OOP? Explain its four pillars.
2. What is the difference between a class and an object?
3. Explain encapsulation and TypeScript access modifiers.
4. Abstract class vs interface?
5. Inheritance vs composition?
6. Overloading vs overriding in TypeScript?
7. What is structural typing?
8. What are generics, and how do they support reusable classes?
9. Explain `readonly`, `private`, `protected`, and `public`.
10. What are the SOLID principles?
11. Why are interfaces erased at runtime?
12. What is dependency injection?

### Coding
1. Create a `Student` class with typed fields and methods.
2. Implement a bank account with private balance and validation.
3. Create a base `Animal` class and derived classes.
4. Demonstrate polymorphism using an interface.
5. Create an abstract `Shape` class and concrete shapes.
6. Build a generic `Box<T>` class.
7. Use composition to build a `Car` with an injected `Engine`.
8. Create a `Repository<T>` constrained by an interface.
9. Implement a custom type guard for a union.
10. Refactor a class to follow Single Responsibility and Dependency Inversion.

---

## Recommended Study Order

1. Objects and object types
2. Classes, fields, methods, and constructors
3. `this` and parameter properties
4. Access modifiers and encapsulation
5. Getters, setters, and `readonly`
6. Static members
7. Inheritance and `super`
8. Overloading, overriding, and polymorphism
9. Abstract classes and interfaces
10. Composition and object relationships
11. Generics and constraints
12. Type guards, narrowing, and `instanceof`
13. Utility types and equality
14. Dependency injection and SOLID

> **Core principle:** TypeScript adds compile-time contracts to JavaScript's flexible object model. Use types to make contracts explicit, classes to manage state and behavior where appropriate, and composition/interfaces to keep implementations replaceable. Do not mistake type annotations for runtime validation or classes for the only way to organize code.
