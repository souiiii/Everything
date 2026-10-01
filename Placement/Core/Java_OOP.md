# Java OOPS

<aside>
🎯

**Goal of these notes:** understand Java OOP well enough to explain it clearly in interviews, reason about code, and answer the common follow-up questions. The focus is on placement-level depth, not obscure language trivia.

</aside>

---

# 1. The OOP mental model

Object-oriented programming organizes a program around **objects**. An object combines **state** (data) and **behaviour** (methods that operate on that data).

A **class** is the blueprint. An **object** is an actual instance created from that blueprint.

```java
class Car {
    String model;
    int speed;

    void accelerate() {
        speed += 10;
    }
}

Car c = new Car();
c.model = "M340i";
c.accelerate();
```

Here, `Car` describes what every car object contains. `c` is a reference variable that refers to one actual `Car` object in memory.

## Reference variable vs object

```java
Student s1;              // only a reference variable
s1 = new Student();      // an object is now created
Student s2 = s1;         // s1 and s2 refer to the same object
```

Assigning `s1` to `s2` does **not** copy the object. Both references point to the same object, so a change made through one reference is visible through the other.

## The `new` keyword

For normal class instances, `new` creates an object and returns a reference to it.

```java
Car c = new Car();
```

Conceptually:

```
new Car()
   ↓
object created
   ↓
reference returned
   ↓
stored in c
```

---

# 2. Constructors, `this`, `static`, and `final`

These are not the four pillars of OOP, but interviewers frequently use them to test whether you actually understand Java objects.

## Constructors

A constructor initializes an object when it is created. It has the same name as the class and has no return type.

```java
class User {
    String name;
    int age;

    User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

User u = new User("Ava", 22);
```

If you declare **no constructor at all**, Java supplies a no-argument default constructor. Once you declare your own constructor, Java does not automatically add that default one.

Constructors are **not inherited** and cannot be overridden.

### Constructor chaining

`this(...)` calls another constructor in the same class.

`super(...)` calls a constructor of the parent class.

```java
class Employee {
    String name;

    Employee() {
        this("Unknown");
    }

    Employee(String name) {
        this.name = name;
    }
}
```

If used, `this(...)` or `super(...)` must be the first statement in the constructor.

## `this`

`this` refers to the current object.

```java
class Account {
    int balance;

    Account(int balance) {
        this.balance = balance;
    }
}
```

The left side, `this.balance`, is the object's field. The right side, `balance`, is the constructor parameter.

## `static`

A static member belongs to the **class**, not to each individual object.

```java
class Employee {
    static int count = 0;

    Employee() {
        count++;
    }
}
```

All `Employee` objects share the same `count`.

A static method has no current object, so it cannot directly use instance fields or `this`.

```java
class Demo {
    int x = 10;

    static void print() {
        // System.out.println(x);   // invalid without an object
    }
}
```

Use static members for behaviour or data that logically belongs to the class as a whole.

## `final`

`final` means different things depending on where it is used:

- **final variable:** cannot be assigned again.
- **final method:** cannot be overridden.
- **final class:** cannot be extended.

```java
final int x = 10;
```

For an object reference, `final` prevents the reference from pointing to another object. It does **not** automatically make the object immutable.

```java
final ArrayList<Integer> list = new ArrayList<>();
list.add(10);               // allowed
// list = new ArrayList<>(); // not allowed
```

---

# 3. The four pillars of OOP

The four ideas interviewers expect you to know clearly are:

1. **Encapsulation**
2. **Abstraction**
3. **Inheritance**
4. **Polymorphism**

They are related, but they solve different problems.

---

# 4. Encapsulation

Encapsulation means keeping an object's state and the operations that manage that state together, while controlling how outside code can access the internal data.

A common implementation is to keep fields private and expose meaningful methods.

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

Outside code cannot directly force the balance into an invalid state. It must go through the behaviour defined by the class.

<aside>
💡

**Interview line:** Encapsulation is not just "getters and setters." It is about protecting an object's state and exposing controlled operations around that state.

</aside>

## Access modifiers

| Modifier | Accessible from |
| --- | --- |
| `private` | Only the same class |
| default | The same package |
| `protected` | The same package, plus subclasses in other packages |
| `public` | Everywhere |

Use the least permissive access that still allows the design to work.

---

# 5. Abstraction

Abstraction means exposing **what an object can do** while hiding unnecessary implementation details.

When you call:

```java
list.add("Java");
```

you care that an element is added. You do not need to know every internal array resize or memory operation performed by `ArrayList`.

A cleaner example is a payment contract:

```java
interface PaymentService {
    void pay(double amount);
}
```

Different implementations can perform the work differently:

```java
class CardPayment implements PaymentService {
    public void pay(double amount) {
        System.out.println("Paid using card");
    }
}

class UpiPayment implements PaymentService {
    public void pay(double amount) {
        System.out.println("Paid using UPI");
    }
}
```

Code using `PaymentService` depends on the capability, not on the internal payment mechanism.

## Encapsulation vs abstraction

| Encapsulation | Abstraction |
| --- | --- |
| Controls access to internal state. | Hides unnecessary implementation complexity. |
| Often achieved using classes and access modifiers. | Often expressed through interfaces and abstract classes. |
| Focuses on protecting how an object is represented. | Focuses on exposing only what the user of the object needs. |

A useful way to remember it:

> **Encapsulation:** "You cannot directly touch my internals."
**Abstraction:** "You do not need to know how my internals work."
> 

---

# 6. Inheritance

Inheritance allows one class to reuse and specialize another class.

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle started");
    }
}

class Car extends Vehicle {
    void openBoot() {
        System.out.println("Boot opened");
    }
}
```

A `Car` receives accessible members of `Vehicle` and can add its own behaviour.

Inheritance models an **IS-A** relationship:

```
Car IS-A Vehicle
Dog IS-A Animal
SavingsAccount IS-A Account
```

If that sentence feels unnatural, inheritance is probably the wrong design.

## `super`

`super` refers to the parent-class part of the current object.

```java
class Car extends Vehicle {
    Car() {
        super();
    }
}
```

It is commonly used to call a parent constructor or access a parent implementation.

## Java and multiple inheritance

A Java class can extend only **one class**.

```java
class C extends A { }      // valid
```

Java does not allow one class to extend multiple classes. The main problem is ambiguity when parent classes provide conflicting implementation.

Java supports multiple **interfaces**, which lets a class implement multiple behavioural contracts.

---

# 7. Polymorphism

Polymorphism means that the same operation can behave differently depending on the situation or actual object involved.

## Compile-time polymorphism: method overloading

Overloading means using the same method name with different parameter lists.

```java
class Printer {
    void print(String text) {
        System.out.println(text);
    }

    void print(int number) {
        System.out.println(number);
    }
}
```

The compiler determines which method to call from the arguments.

Changing only the return type does **not** create a valid overload.

## Runtime polymorphism: method overriding

Overriding happens when a child class provides its own implementation of an inherited instance method.

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
```

Now consider:

```java
Animal a = new Dog();
a.sound();
```

The reference type is `Animal`, but the actual object is a `Dog`, so the output is `Bark`.

This is **dynamic method dispatch**. For overridden instance methods, Java chooses the implementation at runtime based on the actual object.

## Reference type vs object type

```java
Animal a = new Dog();
```

The **reference type** determines what members the compiler allows you to access.

The **actual object type** determines which overridden instance method runs.

## Important overriding rules

- The child method must have the same method signature.
- The return type must be the same or covariant.
- The child cannot make the method less accessible.
- A `final` method cannot be overridden.
- Constructors cannot be overridden.
- Static methods are resolved by the class/reference and are **hidden**, not runtime-overridden.

---

# 8. Upcasting, downcasting, and `instanceof`

## Upcasting

Treating a child object as a parent reference is safe and happens automatically.

```java
Animal a = new Dog();
```

This is useful because code can work with a general abstraction instead of one specific implementation.

## Downcasting

Going from a parent reference back to a child reference requires an explicit cast.

```java
Dog d = (Dog) a;
```

This is only safe if the object really is a `Dog`.

```java
if (a instanceof Dog) {
    Dog d = (Dog) a;
}
```

A bad downcast causes `ClassCastException`.

---

## 9. Abstract class vs interface

Both help you program against abstractions, but the easiest mental model is:

> **Abstract class:** "These classes belong to the same family and may share state or implementation."  
> **Interface:** "These classes may be unrelated, but they promise that they can do the same capability."

This distinction matters in placements because interviewers usually care less about memorizing syntax and more about **when you would choose one over the other**.

## Abstract class

An abstract class is a **partially implemented base class**. Use it when several closely related classes share some common state or behaviour, but some behaviour must still be supplied by each subclass.

An abstract class can contain state, constructors, normal methods, and abstract methods.

Think of it as saying: **"I know what all members of this family have in common, but I cannot completely define every member yet."**

```java
abstract class Shape {
    String color;

    Shape(String color) {
        this.color = color;
    }

    abstract double area();

    void describe() {
        System.out.println("Color: " + color);
    }
}
```

You cannot instantiate an abstract class directly because it represents an incomplete/general concept.

For example, `Shape` can define common state such as `color` and common behaviour such as `describe()`, while leaving `area()` for each specific shape to implement.

You can still use an abstract class as a reference type:

```java
Shape s = new Circle("red", 5);
```

This is useful for runtime polymorphism: code can work with the general `Shape` type while the actual subclass provides the implementation.

Important placement points:

- If a class contains an abstract method, the class itself must be abstract.
- An abstract class does **not** have to contain an abstract method.
- It can have constructors and normal instance fields.
- A subclass uses `extends`, so Java's single-class-inheritance rule still applies.

## Interface

An interface defines a **behavioural contract or capability**.

Instead of saying what an object **is**, an interface often says what an object **can do**.

For example, a bird and a drone are not naturally part of the same class hierarchy, but both may be `Flyable`. A card-payment class and a UPI-payment class may be unrelated internally, but both can satisfy a `PaymentService` contract.

That lets other code depend on the capability rather than a specific implementation.

```java
interface Flyable {
    void fly();
}
```

A class implements it:

```java
class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("Bird is flying");
    }
}
```

A class can implement multiple interfaces. This is one major reason interfaces are useful: one class can promise several independent capabilities.

```java
class SmartPhone implements Camera, GPS, MusicPlayer {
    // implementations
}
```

Code can then depend on the interface:

```java
void checkout(PaymentService payment) {
    payment.pay(1000);
}
```

`checkout()` does not care whether `payment` is a card, UPI, or a future implementation. It only cares that the object follows the `PaymentService` contract. This reduces coupling and makes code easier to extend.

Interface fields are implicitly `public static final`.

Traditional interface methods are implicitly `public abstract`. Modern Java also allows `default` and `static` methods with implementations.

<aside>  
💡

**Placement answer:** Use an **abstract class** when related classes share common state or implementation. Use an **interface** when you mainly want to define a capability/contract that possibly unrelated classes can implement.

</aside>

## When should you use which?

|Abstract class|Interface|
|---|---|
|Use when related classes share common state or implementation.|Use when different classes should promise the same capability.|
|A class can extend only one.|A class can implement multiple interfaces.|
|Can have constructors and instance fields.|Primarily defines a contract.|

Example:

```
Dog extends Animal          → strong IS-A relationship
Bird implements Flyable    → Bird has the capability to fly
Drone implements Flyable   → Drone also has that capability
```

---

# 10. Composition vs inheritance

The simplest mental model is:

```
Inheritance  → IS-A
Composition  → HAS-A
```

**Inheritance** means one class is a specialized version of another:

```
Car IS-A Vehicle
Dog IS-A Animal
```

**Composition** means one object contains or uses another object to do part of its work:

```
Car HAS-A Engine
OrderService HAS-A PaymentService
```

Do not use inheritance just because you want to reuse some methods. First ask whether the IS-A relationship is actually true.

Example:

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }

    void start() {
        engine.start();
    }
}
```

The `Car` does not inherit from `Engine`. It **contains** an `Engine` and delegates engine-specific work to it:

```java
void start() {
    engine.start();
}
```

This delegation is the key idea behind composition.

A real backend-style example is:

```java
class OrderService {
    private PaymentService paymentService;
    private EmailService emailService;
    private OrderRepository repository;
}
```

`OrderService` **has** these dependencies. It should not extend `PaymentService` just because it needs payment functionality.

## Why composition is often preferred

Composition usually creates looser coupling. A component can often be replaced without redesigning an inheritance hierarchy.

For example, if a class depends on a `MessageSender`, you can supply an `EmailSender` today and an `SmsSender` later while keeping the main class unchanged.

This is what interviewers mean by **"prefer composition over inheritance"**. It does not mean inheritance is bad. It means you should not invent an inheritance hierarchy merely for code reuse.

Use **inheritance** when:

- there is a genuine **IS-A** relationship
- the child is a specialization of the parent
- treating the child as the parent makes logical sense
- runtime polymorphism through that hierarchy is useful

Use **composition** when:

- one object simply **HAS-A / uses** another object
- you want components to be replaceable
- you want the functionality without tightly coupling classes through inheritance

<aside>  
💡

**Placement answer:** Inheritance represents an **IS-A** relationship and creates a stronger parent-child coupling. Composition represents a **HAS-A** relationship and builds an object using other objects. Composition is usually more flexible because those components can be replaced independently, while inheritance is best when there is a genuine hierarchy and polymorphism is useful.

</aside>

<aside>  
⚠️

A common bad design is using inheritance only to reuse code. Code reuse alone is not enough reason for an IS-A relationship.

</aside>

---

# 11. The `Object` class, `==`, `equals()`, and `hashCode()`

Every normal Java class ultimately inherits from `Object`.

Important methods include `equals()`, `hashCode()`, `toString()`, and `getClass()`.

## `==` vs `equals()`

For object references, `==` checks whether both references point to the **same object**.

`equals()` is a method. The default implementation inherited from `Object` also behaves like identity comparison, but classes can override it to define **logical equality**.

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);       // false
System.out.println(a.equals(b));  // true
```

`String` overrides `equals()` to compare content.

## The equals/hashCode contract

If you override `equals()`, you normally must override `hashCode()` consistently.

The critical rule is:

> If `a.equals(b)` is true, then `a.hashCode()` and `b.hashCode()` must be equal.
> 

The reverse is not guaranteed. Two unequal objects can still have the same hash code.

Hash-based collections such as `HashMap` and `HashSet` use the hash code to narrow down where to search, then use equality to distinguish keys.

A broken contract can cause logically equal objects to behave like different keys.

---

# 12. How HashMap works at interview level

A `HashMap` stores key-value pairs.

```java
Map<String, Integer> ages = new HashMap<>();
ages.put("Ava", 22);
```

Conceptually, insertion works like this:

```
key
 ↓
hashCode()
 ↓
hash transformed into bucket index
 ↓
entry stored in that bucket
```

On lookup:

```
get(key)
 ↓
calculate hash
 ↓
go to likely bucket
 ↓
use equals() to find the correct key
```

## Collision

A collision happens when different keys map to the same bucket.

Modern Java stores multiple entries in that bucket. With sufficiently heavy collisions, the bucket structure can be converted from a linked structure into a balanced tree to avoid very poor lookup performance.

Average-case lookup is approximately **O(1)**, but poor hash distribution can make it slower.

<aside>
💡

**Interview connection:** `hashCode()` helps HashMap choose where to look. `equals()` confirms whether the key it found is actually the key you asked for.

</aside>

---

# 13. ArrayList vs LinkedList: the real trade-off

Both implement `List`, but their internal structures are very different.

## ArrayList

`ArrayList` uses a resizable array.

```
[A][B][C][D]
```

Advantages:

- Random access by index is **O(1)**.
- Appending is usually **O(1) amortized**.
- Elements are stored compactly, which tends to be CPU-cache friendly.

When the backing array runs out of capacity, a larger array is allocated and elements are copied. That resize operation is O(n), but it does not happen on every insertion.

## LinkedList

A `LinkedList` stores nodes connected through references.

```
[A] ⇄ [B] ⇄ [C]
```

Random access is **O(n)** because Java must walk through nodes.

Insertion/removal can be cheap once the exact node position is already known, but finding an arbitrary position is itself O(n).

So the common statement "LinkedList is faster for insertion" is incomplete. In real application code, `ArrayList` is often the better default because indexing and iteration are much cheaper.

| Operation | ArrayList | LinkedList |
| --- | --- | --- |
| Get by index | O(1) | O(n) |
| Append | O(1) amortized | O(1) |
| Insert/remove in middle by index | O(n) | O(n) to locate the position |
| Memory overhead | Lower | Higher because each node stores links |

---

# 14. Immutability

An immutable object cannot change its observable state after construction.

`String` is the classic Java example.

A typical immutable class:

```java
final class User {
    private final String name;

    User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

Common rules:

1. Keep fields private.
2. Make state final where appropriate.
3. Do not expose mutating setters.
4. Prevent unsafe subclass mutation, often by making the class final.
5. Make defensive copies of mutable objects when necessary.

Immutability helps reasoning, thread safety, and safe use as map keys.

---

# 15. Shallow copy vs deep copy

A **shallow copy** creates a new outer object but keeps references to the same nested mutable objects.

```
original ─┐
          ├──> same Address object
copy ─────┘
```

Changing the shared nested object can affect both.

A **deep copy** also creates independent copies of the nested mutable objects.

```
original → Address A
copy     → Address B
```

For interview answers, focus on the ownership difference rather than memorizing the old `Cloneable` API.

---

# 16. Java is pass-by-value

Java is always pass-by-value.

For primitives, the value itself is copied.

For objects, the **value of the reference** is copied.

```java
void changeName(User u) {
    u.name = "Ava";
}
```

The method receives another copy of the reference, but both references still point to the same object, so mutating that object is visible outside.

However:

```java
void replace(User u) {
    u = new User();
}
```

reassigning the local reference does not replace the caller's reference.

This is why saying "Java passes objects by reference" is incorrect.

---

# 17. Common interview traps

## Can an abstract class have a constructor?

Yes. A child constructor can call it through `super(...)`.

## Can an abstract class have normal methods?

Yes. It can contain both implemented and abstract methods.

## Can an interface have method bodies?

Yes. Modern Java interfaces can have `default` and `static` methods with bodies.

## Can static methods be overridden?

Not in the runtime-polymorphism sense. A child can declare another static method with the same signature, but that is **method hiding**.

## Can private methods be overridden?

No. They are not visible to the child class.

## Can constructors be overridden?

No. Constructors are not inherited.

## Can a final reference object's fields change?

Yes. `final` prevents reassignment of the reference; it does not automatically freeze the object.

## Is `hashCode()` a memory address?

No. It is an integer used by hash-based structures. Its contract matters more than its exact implementation.

## If two objects have the same hash code, are they equal?

No. Hash collisions are allowed.

## If two objects are equal, must their hash codes match?

Yes.

---

# 18. Five answers you should be able to say cleanly

## What are the four pillars of OOP?

> Encapsulation controls access to an object's state, abstraction exposes essential behaviour while hiding unnecessary implementation, inheritance lets a child specialize a parent through an IS-A relationship, and polymorphism allows the same abstraction or method call to behave differently depending on the actual implementation.
> 

## Overloading vs overriding?

> Overloading is compile-time polymorphism: methods share a name but have different parameter lists. Overriding is runtime polymorphism: a child provides a new implementation of an inherited instance method, and dynamic dispatch selects the method from the actual object at runtime.
> 

## Interface vs abstract class?

> I use an abstract class when closely related classes need shared state or common implementation. I use an interface when I want to define a capability or contract that potentially unrelated classes can implement. A class can extend only one class but can implement multiple interfaces.
> 

## Composition vs inheritance?

> Inheritance models an IS-A relationship, while composition models a HAS-A relationship. I prefer composition when I only need to reuse or delegate behaviour because it creates less coupling and makes components easier to replace.
> 

## Why must equals and hashCode agree?

> Hash-based collections first use the hash code to find the likely bucket and then use equality to identify the correct key. Therefore, objects that are logically equal must produce the same hash code; otherwise structures like HashMap and HashSet can behave incorrectly.
> 

---

# 19. What to prioritize before an interview

If revision time is short, know these especially well:

1. The four pillars and how they differ.
2. Overloading vs overriding and dynamic dispatch.
3. Reference type vs actual object type.
4. Abstract class vs interface.
5. Composition vs inheritance.
6. `==` vs `equals()`.
7. The `equals()` / `hashCode()` contract.
8. HashMap lookup and collision handling.
9. ArrayList vs LinkedList real costs.
10. `static`, `final`, constructors, `this`, and `super`.
11. Upcasting/downcasting and `instanceof`.
12. Java pass-by-value.

<aside>
✅

If you can explain these topics with the examples above without memorized one-liners, your Java OOP coverage is strong enough for most fresher/backend placement interviews.

</aside>