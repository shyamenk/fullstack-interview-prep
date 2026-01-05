# OOPs Concepts Interview Questions & Answers

## Table of Contents

### Four Pillars of OOP
1. [What is Encapsulation?](#q1-what-is-encapsulation)
2. [What is Abstraction?](#q2-what-is-abstraction)
3. [What is Inheritance?](#q3-what-is-inheritance)
4. [What is Polymorphism?](#q4-what-is-polymorphism)

### SOLID Principles
5. [What is the Single Responsibility Principle (SRP)?](#q5-what-is-the-single-responsibility-principle-srp)
6. [What is the Open/Closed Principle (OCP)?](#q6-what-is-the-openclosed-principle-ocp)
7. [What is the Liskov Substitution Principle (LSP)?](#q7-what-is-the-liskov-substitution-principle-lsp)
8. [What is the Interface Segregation Principle (ISP)?](#q8-what-is-the-interface-segregation-principle-isp)
9. [What is the Dependency Inversion Principle (DIP)?](#q9-what-is-the-dependency-inversion-principle-dip)

### Design Patterns
10. [What is the Singleton Pattern?](#q10-what-is-the-singleton-pattern)
11. [What is the Factory Pattern?](#q11-what-is-the-factory-pattern)
12. [What is the Observer Pattern?](#q12-what-is-the-observer-pattern)
13. [What is the Strategy Pattern?](#q13-what-is-the-strategy-pattern)

### Clean Architecture
14. [What is Clean Architecture?](#q14-what-is-clean-architecture)

---

# Four Pillars of OOP

## Q1. What is Encapsulation?

### Concept:

Encapsulation bundles data and methods that operate on that data within a single unit (class), hiding internal state from outside access.

- Protects data from unauthorized access
- Controls how data is modified via getters/setters
- Reduces coupling between components

### Example:

```typescript
class BankAccount {
  private balance: number = 0;

  deposit(amount: number): void {
    if (amount > 0) {
      this.balance += amount;
    }
  }

  withdraw(amount: number): boolean {
    if (amount > 0 && amount <= this.balance) {
      this.balance -= amount;
      return true;
    }
    return false;
  }

  getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount();
account.deposit(100);
// account.balance = -1000; // Error: private property
console.log(account.getBalance()); // 100
```

### Analogy:

Think of a **TV remote**:
- You press buttons (public methods) to change channels
- You can't access the internal circuits directly (private data)
- The remote controls how you interact with the TV

### ⚡ One-liner to remember:
> Encapsulation = Hide the data, expose only what's necessary through methods.

---

## Q2. What is Abstraction?

### Concept:

Abstraction hides complex implementation details and shows only essential features to the user.

- Reduces complexity by hiding unnecessary details
- Defines "what" something does, not "how" it does it
- Implemented via abstract classes and interfaces

### Example:

```typescript
// Abstract class - defines what, not how
abstract class PaymentProcessor {
  abstract processPayment(amount: number): boolean;
  abstract refund(transactionId: string): boolean;
}

// Concrete implementation - defines how
class StripePayment extends PaymentProcessor {
  processPayment(amount: number): boolean {
    // Complex Stripe API logic hidden here
    console.log(`Processing $${amount} via Stripe`);
    return true;
  }

  refund(transactionId: string): boolean {
    // Complex refund logic hidden
    console.log(`Refunding ${transactionId}`);
    return true;
  }
}

// User only sees simple interface
const payment = new StripePayment();
payment.processPayment(99.99); // Don't need to know Stripe internals
```

### Analogy:

Think of a **car steering wheel**:
- You turn the wheel to steer (simple interface)
- You don't need to know about power steering fluid, rack and pinion mechanics
- The complexity is abstracted away

### ⚡ One-liner to remember:
> Abstraction = Show the "what", hide the "how".

---

## Q3. What is Inheritance?

### Concept:

Inheritance allows a class to inherit properties and methods from another class, promoting code reuse.

- **Parent/Base class:** The class being inherited from
- **Child/Derived class:** The class that inherits
- Enables hierarchical relationships

### Example:

```typescript
class Animal {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  eat(): void {
    console.log(`${this.name} is eating`);
  }

  sleep(): void {
    console.log(`${this.name} is sleeping`);
  }
}

class Dog extends Animal {
  private breed: string;

  constructor(name: string, breed: string) {
    super(name); // Call parent constructor
    this.breed = breed;
  }

  bark(): void {
    console.log(`${this.name} says woof!`);
  }

  // Override parent method
  eat(): void {
    console.log(`${this.name} is eating dog food`);
  }
}

const dog = new Dog("Max", "Labrador");
dog.eat();   // "Max is eating dog food" (overridden)
dog.sleep(); // "Max is sleeping" (inherited)
dog.bark();  // "Max says woof!" (own method)
```

### Analogy:

Think of a **family inheritance**:
- A child inherits traits from parents (eye color, height)
- The child can also have their own unique traits
- Some traits can be "overridden" (different hair color)

### ⚡ One-liner to remember:
> Inheritance = Child class gets parent's features and can add or override them.

---

## Q4. What is Polymorphism?

### Concept:

Polymorphism allows objects of different classes to be treated as objects of a common parent class, with each responding differently to the same method call.

- **Compile-time (Method Overloading):** Same method name, different parameters
- **Runtime (Method Overriding):** Child class redefines parent's method

### Example:

```typescript
abstract class Shape {
  abstract calculateArea(): number;
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) {
    super();
  }

  calculateArea(): number {
    return this.width * this.height;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

// Polymorphism in action
function printArea(shape: Shape): void {
  console.log(`Area: ${shape.calculateArea()}`);
}

const shapes: Shape[] = [
  new Rectangle(10, 5),
  new Circle(7)
];

shapes.forEach(shape => printArea(shape));
// Area: 50
// Area: 153.94
```

### Analogy:

Think of a **universal remote**:
- Same "power" button works on TV, AC, and sound system
- Each device responds differently to the same command
- The remote doesn't need to know the internal workings

### ⚡ One-liner to remember:
> Polymorphism = Same interface, different behaviors based on the object.

---

# SOLID Principles

## Q5. What is the Single Responsibility Principle (SRP)?

### Concept:

A class should have **one and only one reason to change** — meaning it should have only one job.

### Bad Example:

```typescript
// Violates SRP - handles user data AND email sending
class User {
  constructor(public name: string, public email: string) {}

  saveToDatabase(): void {
    // Database logic
  }

  sendEmail(message: string): void {
    // Email logic
  }
}
```

### Good Example:

```typescript
class User {
  constructor(public name: string, public email: string) {}
}

class UserRepository {
  save(user: User): void {
    // Only handles database operations
  }
}

class EmailService {
  send(to: string, message: string): void {
    // Only handles email sending
  }
}
```

### Analogy:

Think of a **restaurant kitchen**:
- Chef cooks food (one responsibility)
- Waiter serves food (one responsibility)
- Cashier handles payment (one responsibility)
- If the chef also served and took payments, chaos would ensue

### ⚡ One-liner to remember:
> SRP = One class, one job, one reason to change.

---

## Q6. What is the Open/Closed Principle (OCP)?

### Concept:

Software entities should be **open for extension but closed for modification**.

- Add new functionality by extending, not modifying existing code
- Prevents breaking existing functionality

### Bad Example:

```typescript
// Must modify this class every time we add a new discount type
class DiscountCalculator {
  calculate(type: string, price: number): number {
    if (type === "regular") return price * 0.1;
    if (type === "premium") return price * 0.2;
    if (type === "vip") return price * 0.3; // Added later - modification!
    return 0;
  }
}
```

### Good Example:

```typescript
interface DiscountStrategy {
  calculate(price: number): number;
}

class RegularDiscount implements DiscountStrategy {
  calculate(price: number): number {
    return price * 0.1;
  }
}

class PremiumDiscount implements DiscountStrategy {
  calculate(price: number): number {
    return price * 0.2;
  }
}

// New discount - extension, no modification needed!
class VIPDiscount implements DiscountStrategy {
  calculate(price: number): number {
    return price * 0.3;
  }
}

class DiscountCalculator {
  calculate(strategy: DiscountStrategy, price: number): number {
    return strategy.calculate(price);
  }
}
```

### Analogy:

Think of a **power strip**:
- You can plug in new devices (extend)
- You don't need to rewire the strip (modify)

### ⚡ One-liner to remember:
> OCP = Extend behavior without touching existing code.

---

## Q7. What is the Liskov Substitution Principle (LSP)?

### Concept:

Subtypes must be substitutable for their base types without altering program correctness.

- If S is a subtype of T, objects of T can be replaced with objects of S
- Child classes should not break parent class behavior

### Bad Example:

```typescript
class Bird {
  fly(): void {
    console.log("Flying...");
  }
}

class Penguin extends Bird {
  fly(): void {
    throw new Error("Penguins can't fly!"); // Breaks LSP!
  }
}

function makeBirdFly(bird: Bird): void {
  bird.fly(); // Crashes if penguin is passed
}
```

### Good Example:

```typescript
abstract class Bird {
  abstract move(): void;
}

class FlyingBird extends Bird {
  move(): void {
    console.log("Flying...");
  }
}

class Penguin extends Bird {
  move(): void {
    console.log("Swimming...");
  }
}

function makeBirdMove(bird: Bird): void {
  bird.move(); // Works for all birds!
}
```

### Analogy:

Think of a **substitute teacher**:
- They should be able to teach the class (same behavior)
- If a substitute can't teach at all, they're not a valid substitute
- Students shouldn't notice a difference in capability

### ⚡ One-liner to remember:
> LSP = Child classes must work wherever parent classes are expected.

---

## Q8. What is the Interface Segregation Principle (ISP)?

### Concept:

Clients should not be forced to depend on interfaces they don't use. Prefer many small, specific interfaces over one large interface.

### Bad Example:

```typescript
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Robot implements Worker {
  work(): void { console.log("Working..."); }
  eat(): void { throw new Error("Robots don't eat!"); } // Forced to implement
  sleep(): void { throw new Error("Robots don't sleep!"); } // Forced to implement
}
```

### Good Example:

```typescript
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
  work(): void { console.log("Working..."); }
  eat(): void { console.log("Eating..."); }
  sleep(): void { console.log("Sleeping..."); }
}

class Robot implements Workable {
  work(): void { console.log("Working..."); }
  // No need to implement eat or sleep!
}
```

### Analogy:

Think of a **restaurant menu**:
- A vegetarian shouldn't have to order from a meat-only menu
- Separate menus for appetizers, mains, desserts work better
- Each customer picks only what they need

### ⚡ One-liner to remember:
> ISP = Many small interfaces > one fat interface.

---

## Q9. What is the Dependency Inversion Principle (DIP)?

### Concept:

High-level modules should not depend on low-level modules. Both should depend on abstractions.

- Depend on interfaces, not concrete implementations
- Makes code flexible and testable

### Bad Example:

```typescript
class MySQLDatabase {
  save(data: string): void {
    console.log("Saving to MySQL");
  }
}

class UserService {
  private database = new MySQLDatabase(); // Tightly coupled!

  saveUser(user: string): void {
    this.database.save(user);
  }
}
```

### Good Example:

```typescript
interface Database {
  save(data: string): void;
}

class MySQLDatabase implements Database {
  save(data: string): void {
    console.log("Saving to MySQL");
  }
}

class MongoDatabase implements Database {
  save(data: string): void {
    console.log("Saving to MongoDB");
  }
}

class UserService {
  constructor(private database: Database) {} // Depends on abstraction!

  saveUser(user: string): void {
    this.database.save(user);
  }
}

// Easy to switch databases
const mysqlService = new UserService(new MySQLDatabase());
const mongoService = new UserService(new MongoDatabase());
```

### Analogy:

Think of a **wall outlet**:
- Your laptop charger plugs into any standard outlet (abstraction)
- You don't hardwire your laptop to the electrical grid (concrete dependency)
- Any device with the right plug works

### ⚡ One-liner to remember:
> DIP = Depend on abstractions (interfaces), not concrete implementations.

---

# Design Patterns

## Q10. What is the Singleton Pattern?

### Concept:

Ensures a class has **only one instance** and provides a global point of access to it.

**Use cases:** Database connections, logging, configuration managers

### Example:

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private constructor() {} // Private constructor prevents direct instantiation

  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  query(sql: string): void {
    console.log(`Executing: ${sql}`);
  }
}

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true - same instance
```

### Analogy:

Think of a **country's president**:
- Only one president at a time
- Everyone refers to the same person
- You can't just create another president

### ⚡ One-liner to remember:
> Singleton = One instance to rule them all.

---

## Q11. What is the Factory Pattern?

### Concept:

Creates objects without exposing the creation logic to the client, using a common interface.

**Use cases:** Creating different types of objects based on input, hiding complex instantiation

### Example:

```typescript
interface Notification {
  send(message: string): void;
}

class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`Email: ${message}`);
  }
}

class SMSNotification implements Notification {
  send(message: string): void {
    console.log(`SMS: ${message}`);
  }
}

class PushNotification implements Notification {
  send(message: string): void {
    console.log(`Push: ${message}`);
  }
}

// Factory
class NotificationFactory {
  static create(type: "email" | "sms" | "push"): Notification {
    switch (type) {
      case "email": return new EmailNotification();
      case "sms": return new SMSNotification();
      case "push": return new PushNotification();
    }
  }
}

// Usage - client doesn't know about concrete classes
const notification = NotificationFactory.create("email");
notification.send("Hello!");
```

### Analogy:

Think of a **pizza restaurant**:
- You order "Margherita" (type)
- Kitchen (factory) handles the creation
- You don't need to know the recipe

### ⚡ One-liner to remember:
> Factory = Delegate object creation to a specialized class.

---

## Q12. What is the Observer Pattern?

### Concept:

Defines a one-to-many dependency where when one object (subject) changes state, all dependents (observers) are notified automatically.

**Use cases:** Event systems, pub/sub, real-time updates

### Example:

```typescript
interface Observer {
  update(data: any): void;
}

class Subject {
  private observers: Observer[] = [];

  subscribe(observer: Observer): void {
    this.observers.push(observer);
  }

  unsubscribe(observer: Observer): void {
    this.observers = this.observers.filter(o => o !== observer);
  }

  notify(data: any): void {
    this.observers.forEach(observer => observer.update(data));
  }
}

// Concrete observers
class EmailSubscriber implements Observer {
  update(data: any): void {
    console.log(`Email notification: ${data}`);
  }
}

class SMSSubscriber implements Observer {
  update(data: any): void {
    console.log(`SMS notification: ${data}`);
  }
}

// Usage
const newsChannel = new Subject();
newsChannel.subscribe(new EmailSubscriber());
newsChannel.subscribe(new SMSSubscriber());
newsChannel.notify("Breaking news!"); // Both get notified
```

### Analogy:

Think of a **YouTube channel**:
- You subscribe to a channel (observer)
- When a new video is uploaded (state change)
- All subscribers get notified

### ⚡ One-liner to remember:
> Observer = Subject changes, all subscribers get notified.

---

## Q13. What is the Strategy Pattern?

### Concept:

Defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.

**Use cases:** Payment processing, sorting algorithms, validation strategies

### Example:

```typescript
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
  constructor(private cardNumber: string) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} with Credit Card ${this.cardNumber}`);
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} with PayPal (${this.email})`);
  }
}

class CryptoPayment implements PaymentStrategy {
  constructor(private walletAddress: string) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} with Crypto (${this.walletAddress})`);
  }
}

// Context
class ShoppingCart {
  private paymentStrategy: PaymentStrategy;

  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.paymentStrategy = strategy;
  }

  checkout(amount: number): void {
    this.paymentStrategy.pay(amount);
  }
}

// Usage - switch strategies at runtime
const cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment("1234-5678"));
cart.checkout(100);

cart.setPaymentStrategy(new PayPalPayment("user@email.com"));
cart.checkout(50);
```

### Analogy:

Think of **Google Maps navigation**:
- Same destination, different strategies (car, walk, bike, transit)
- You can switch strategy anytime
- Each calculates the route differently

### ⚡ One-liner to remember:
> Strategy = Swap algorithms at runtime without changing the client code.

---

# Clean Architecture

## Q14. What is Clean Architecture?

### Concept:

Clean Architecture organizes code into concentric layers with dependencies pointing inward. The core business logic is independent of frameworks, databases, and UI.

### Layers (Inside to Outside):

1. **Entities:** Core business objects and rules
2. **Use Cases:** Application-specific business rules
3. **Interface Adapters:** Controllers, presenters, gateways
4. **Frameworks & Drivers:** Database, web framework, external APIs

### Example Structure:

```
src/
├── domain/           # Entities (innermost)
│   └── User.ts
├── application/      # Use Cases
│   └── CreateUser.ts
├── infrastructure/   # Interface Adapters & Frameworks
│   ├── repositories/
│   │   └── UserRepository.ts
│   └── controllers/
│       └── UserController.ts
└── main.ts           # Composition root
```

### Example:

```typescript
// Domain Layer - Entity (no dependencies)
class User {
  constructor(
    public readonly id: string,
    public readonly email: string,
    public readonly name: string
  ) {}
}

// Application Layer - Use Case
interface UserRepository {
  save(user: User): Promise<void>;
  findByEmail(email: string): Promise<User | null>;
}

class CreateUserUseCase {
  constructor(private userRepo: UserRepository) {}

  async execute(email: string, name: string): Promise<User> {
    const existing = await this.userRepo.findByEmail(email);
    if (existing) throw new Error("User already exists");

    const user = new User(crypto.randomUUID(), email, name);
    await this.userRepo.save(user);
    return user;
  }
}

// Infrastructure Layer - Repository Implementation
class PostgresUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // PostgreSQL-specific implementation
  }

  async findByEmail(email: string): Promise<User | null> {
    // PostgreSQL-specific implementation
    return null;
  }
}

// Infrastructure Layer - Controller
class UserController {
  constructor(private createUser: CreateUserUseCase) {}

  async handleCreateUser(req: Request): Promise<Response> {
    const { email, name } = req.body;
    const user = await this.createUser.execute(email, name);
    return { status: 201, body: user };
  }
}
```

### Key Benefits:

| Benefit           | Description                                    |
|-------------------|------------------------------------------------|
| Testability       | Core logic testable without frameworks         |
| Flexibility       | Swap database/framework without changing logic |
| Maintainability   | Changes in one layer don't affect others       |
| Independence      | Business rules don't depend on external tools  |

### Analogy:

Think of an **onion**:
- Core (center) is protected by outer layers
- You can peel/replace outer layers without affecting the core
- Dependencies flow inward, never outward

### ⚡ One-liner to remember:
> Clean Architecture = Business logic at the center, frameworks at the edges, dependencies point inward.

---
