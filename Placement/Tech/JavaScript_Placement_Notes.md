# JavaScript — Placement & Interview Notes

> **Goal of these notes:** understand JavaScript well enough to explain it, predict code output, debug it, and use it in coding/interview questions. The emphasis is on concepts that are repeatedly tested in placements: types, scope, hoisting, closures, functions, `this`, objects/prototypes, arrays, asynchronous JavaScript, promises, the event loop, DOM events, and modern syntax.
>
> Do not memorize isolated definitions. For every topic, be able to answer three things: **what it is, why it behaves that way, and where that behavior matters in real code.**

---

# 1. JavaScript Mental Model

JavaScript is a **high-level, dynamically typed, garbage-collected programming language**. In modern engines such as V8, JavaScript is not simply “interpreted”; code is parsed and then executed with a mix of interpretation and JIT compilation.

JavaScript itself executes code on a **single call stack** within a given execution environment. That does **not** mean the browser or Node.js can only do one thing at a time. The environment provides timers, networking, file I/O, DOM events, and other APIs that can make progress outside the JavaScript call stack. Their callbacks are later scheduled back into JavaScript through queues and the event loop.

A useful mental model is:

```text
JavaScript engine
├── Call Stack      -> currently executing JavaScript
├── Heap            -> objects/functions stored in memory
└── Execution Contexts

Host environment (Browser / Node.js)
├── Timers
├── Network APIs
├── DOM / events
├── File system / OS APIs (Node)
└── Queues + event loop
```

The placement-level takeaway is simple:

> **JavaScript executes one piece of JavaScript at a time, but the environment can handle waiting work elsewhere and schedule callbacks later.**

---

# 2. Values, Types, and Variables

## 2.1 Primitive types

JavaScript has seven primitive types:

```text
string
number
bigint
boolean
undefined
symbol
null
```

Examples:

```js
const name = "Shahid";        // string
const age = 21;               // number
const huge = 123456789n;      // bigint
const placed = true;          // boolean
let city;                     // undefined
const id = Symbol("id");      // symbol
const selected = null;        // null
```

Primitives are **immutable values**. For example, a string cannot be modified character-by-character in place:

```js
let s = "hello";
s[0] = "H";
console.log(s); // "hello"
```

You can make `s` refer to a new string, but the original string value itself is not mutated.

## 2.2 Objects are reference values

Everything that is not a primitive is an object or behaves as an object-backed value: ordinary objects, arrays, functions, dates, maps, sets, etc.

```js
const user = {
  name: "Aman",
  age: 22
};
```

Two separately created objects are different references even if their contents look identical:

```js
{} === {} // false
[] === [] // false
```

But two variables can point to the same object:

```js
const a = { score: 10 };
const b = a;

b.score = 50;
console.log(a.score); // 50
```

This is one of the most important JavaScript ideas: **object assignment copies the reference, not the object itself.**

---

# 3. `var`, `let`, and `const`

Use `const` by default. Use `let` when the variable must be reassigned. Avoid `var` in modern code unless you are explaining legacy behavior or an interview question.

## `let`

`let` is block-scoped and can be reassigned:

```js
let count = 1;
count = 2;
```

## `const`

`const` is block-scoped and cannot be reassigned:

```js
const x = 10;
// x = 20; // TypeError
```

But `const` does **not** make an object immutable. It only prevents the variable from pointing to a different value.

```js
const user = { name: "A" };
user.name = "B"; // valid

// user = {}; // invalid
```

## `var`

`var` is function-scoped rather than block-scoped:

```js
if (true) {
  var x = 10;
}

console.log(x); // 10
```

With `let`:

```js
if (true) {
  let y = 10;
}

console.log(y); // ReferenceError
```

### Interview comparison

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | function | block | block |
| Reassignment | yes | yes | no |
| Redeclaration in same scope | yes | no | no |
| Hoisted | yes | yes | yes |
| Usable before declaration | `undefined` | no, TDZ | no, TDZ |

---

# 4. `typeof`, `null`, `undefined`, and `NaN`

## `undefined`

`undefined` usually means **a value has not been assigned**.

```js
let x;
console.log(x); // undefined
```

A missing object property also returns `undefined`:

```js
const user = {};
console.log(user.age); // undefined
```

## `null`

`null` usually represents an **intentional absence of a value**.

```js
let selectedUser = null;
```

One famous JavaScript historical quirk:

```js
typeof null // "object"
```

Do not infer from that that `null` is an ordinary object. It is a primitive.

## `NaN`

`NaN` means “Not-a-Number”, but its type is still `number`:

```js
typeof NaN // "number"
```

Another classic gotcha:

```js
NaN === NaN // false
```

Use:

```js
Number.isNaN(value)
```

when you need to test specifically for `NaN`.

---

# 5. Equality and Type Coercion

JavaScript can automatically convert values between types. This is called **type coercion**.

## Strict equality: `===`

`===` compares both type and value without performing coercion.

```js
5 === 5    // true
5 === "5"  // false
```

## Loose equality: `==`

`==` may convert values before comparing them.

```js
5 == "5" // true
```

For most application and interview code, prefer `===` because its behavior is easier to reason about.

### Common coercion examples

```js
"5" + 2 // "52"
"5" - 2 // 3
true + 1 // 2
```

Why?

- `+` is also the string-concatenation operator, so a string operand can cause concatenation.
- `-` has no string-concatenation meaning, so JavaScript attempts numeric conversion.

---

# 6. Truthy and Falsy Values

In conditions, JavaScript converts values to booleans.

Falsy values worth memorizing:

```text
false
0
-0
0n
""        // empty string
null
undefined
NaN
```

Almost everything else is truthy, including:

```js
[]  // truthy
{}  // truthy
"0" // truthy
```

This often appears in output questions:

```js
if ([]) {
  console.log("runs");
}
```

It runs because an array object is truthy.

---

# 7. `||`, `&&`, `??`, and Optional Chaining

## `||` returns the first truthy value

```js
const name = inputName || "Guest";
```

The problem is that valid falsy values such as `0` or `""` will also trigger the fallback.

## `??` only falls back for `null` or `undefined`

```js
const count = inputCount ?? 0;
```

This preserves valid values such as `0` and `false`.

## Optional chaining `?.`

Optional chaining safely stops when the left side is `null` or `undefined`:

```js
const city = user?.address?.city;
```

Without it, accessing a missing nested property could throw an error.

---

# 8. Scope and Lexical Scoping

A variable's scope determines where it can be accessed.

JavaScript uses **lexical scoping**, meaning scope is determined by where code is written, not by where a function happens to be called.

```js
const a = 10;

function outer() {
  const b = 20;

  function inner() {
    const c = 30;
    console.log(a, b, c);
  }

  inner();
}
```

`inner()` can access:

- its own scope (`c`)
- the outer function scope (`b`)
- the global scope (`a`)

Variable lookup moves **outward through the scope chain**. An outer function cannot directly access a variable declared only inside an inner function.

---

# 9. Execution Context and the Call Stack

Whenever JavaScript executes code, it does so inside an **execution context**.

At a high level:

- global code gets a global execution context
- every function call creates a function execution context
- active contexts are tracked on the call stack

Example:

```js
function second() {
  console.log("second");
}

function first() {
  second();
}

first();
```

Conceptually:

```text
Global
  -> first()
      -> second()
          -> console.log()
```

Because the stack is LIFO, the most recently called function finishes first.

This becomes important when understanding recursion, errors, stack traces, and asynchronous JavaScript.

---

# 10. Hoisting and the Temporal Dead Zone

“Hoisting” is the convenient term used to describe how declarations are processed before normal line-by-line execution.

The key interview behavior is more important than the metaphor.

## Function declarations

Function declarations are available before the line where they appear:

```js
greet();

function greet() {
  console.log("hello");
}
```

## `var`

A `var` declaration exists before its source line and initially has the value `undefined`:

```js
console.log(x); // undefined
var x = 10;
```

You can think of it approximately as:

```js
var x;
console.log(x);
x = 10;
```

## `let` and `const`

`let` and `const` are also known to the scope before their declaration, but cannot be accessed until execution reaches the declaration.

```js
console.log(x); // ReferenceError
let x = 10;
```

The period from entering the scope until the declaration is executed is called the **Temporal Dead Zone (TDZ)**.

### Placement answer

> `var` is hoisted and initialized to `undefined`. `let` and `const` are hoisted in the sense that their bindings exist, but they remain inaccessible in the TDZ until their declaration is evaluated.

---

# 11. Functions Are First-Class Values

JavaScript treats functions as values. A function can be:

- stored in a variable
- passed to another function
- returned from a function
- stored in an object or array

```js
function greet(name) {
  return `Hello ${name}`;
}

const fn = greet;
console.log(fn("Aman"));
```

This is the basis for callbacks, higher-order functions, closures, event handlers, and functional patterns.

---

# 12. Function Declaration, Expression, and Arrow Function

## Function declaration

```js
function add(a, b) {
  return a + b;
}
```

Declarations are hoisted with their implementation.

## Function expression

```js
const add = function (a, b) {
  return a + b;
};
```

The variable follows normal `let`/`const` initialization rules.

## Arrow function

```js
const add = (a, b) => a + b;
```

Arrow functions are concise, but they are **not merely shorter normal functions**. They differ in several important ways:

- they do not have their own `this`
- they do not have their own `arguments`
- they cannot be used as constructors with `new`
- they do not have a `prototype` property used for construction

Use arrows especially for callbacks where lexical `this` is desirable.

---

# 13. Parameters, Rest Parameters, and Default Values

## Default parameters

```js
function greet(name = "Guest") {
  return `Hello ${name}`;
}
```

The default is used when the argument is `undefined` or omitted.

```js
greet();          // Hello Guest
greet(undefined); // Hello Guest
greet(null);      // Hello null
```

## Rest parameters

Rest parameters collect remaining arguments into an array:

```js
function sum(...numbers) {
  return numbers.reduce((acc, x) => acc + x, 0);
}
```

Rest **collects** values. Spread, discussed later, **expands** values.

---

# 14. Higher-Order Functions and Callbacks

A **higher-order function** either accepts a function as an argument or returns a function.

```js
function run(operation, a, b) {
  return operation(a, b);
}

const add = (a, b) => a + b;
run(add, 2, 3); // 5
```

A **callback** is simply a function passed to another piece of code to be invoked later or under that code's control.

Callbacks are used in:

```js
array.map(callback)
setTimeout(callback, delay)
element.addEventListener("click", callback)
```

A callback is not automatically asynchronous. `map()` uses callbacks synchronously; `setTimeout()` schedules one asynchronously.

---

# 15. Closures

A closure occurs when a function retains access to variables from its lexical environment even after the outer function has finished executing.

```js
function createCounter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const counter = createCounter();

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

Normally, you might expect `count` to disappear when `createCounter()` returns. But the returned function still references it, so the required environment remains accessible.

### Why closures matter

Closures are used for:

- private state
- function factories
- currying and partial application
- memoization
- event handlers
- React hooks and callbacks

### Common interview trap: closure inside loops

With `var`:

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

```text
3
3
3
```

There is one function-scoped `i`. By the time the callbacks run, the loop has finished and `i` is `3`.

With `let`:

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

```text
0
1
2
```

`let` creates a fresh binding for each loop iteration.

---

# 16. `this`: The Rule That Interviewers Actually Test

For regular functions, `this` is primarily determined by **how the function is called**, not where it was written.

A practical set of rules:

## 1. Method call

```js
const user = {
  name: "Aman",
  greet() {
    console.log(this.name);
  }
};

user.greet(); // Aman
```

Because the call is `user.greet()`, `this` is `user`.

## 2. Plain function call

```js
function show() {
  console.log(this);
}

show();
```

In strict mode, `this` is `undefined`. In non-strict browser script code, it may be the global object.

## 3. Explicit binding: `call`, `apply`, `bind`

```js
function greet(city) {
  console.log(`${this.name} from ${city}`);
}

const user = { name: "Riya" };

greet.call(user, "Delhi");
greet.apply(user, ["Delhi"]);

const bound = greet.bind(user);
bound("Delhi");
```

Difference:

- `call()` invokes immediately and receives arguments individually
- `apply()` invokes immediately and receives arguments as an array-like list
- `bind()` returns a new function with `this` fixed

## 4. Constructor call with `new`

```js
function Person(name) {
  this.name = name;
}

const p = new Person("Aman");
```

Inside the constructor, `this` refers to the newly created object.

## Arrow functions

Arrow functions do not create their own `this`. They capture `this` from the surrounding lexical scope.

```js
const user = {
  name: "Aman",
  regular() {
    const arrow = () => console.log(this.name);
    arrow();
  }
};

user.regular(); // Aman
```

### Common mistake

Do not normally use an arrow function as an object method when you expect `this` to refer to the object:

```js
const user = {
  name: "Aman",
  greet: () => console.log(this.name)
};
```

The arrow does not receive `user` as its own `this`.

---

# 17. Objects

Objects store properties as key-value pairs.

```js
const user = {
  name: "Aman",
  age: 22,
  greet() {
    return `Hi ${this.name}`;
  }
};
```

Access properties using:

```js
user.name
user["name"]
```

Bracket notation is useful when the property name is dynamic:

```js
const key = "age";
console.log(user[key]);
```

## Property existence

```js
"name" in user
Object.hasOwn(user, "name")
```

`in` checks the object and its prototype chain. `Object.hasOwn()` checks only the object's own property.

---

# 18. Object Copying: Shallow vs Deep

## Reference copy

```js
const a = { nested: { value: 1 } };
const b = a;
```

`a` and `b` are the same object.

## Shallow copy

```js
const copy = { ...a };
```

The outer object is new, but nested references are still shared:

```js
copy.nested.value = 99;
console.log(a.nested.value); // 99
```

Other common shallow-copy techniques:

```js
Object.assign({}, obj)
[...array]
array.slice()
```

## Deep copy

For many modern values, use:

```js
const deep = structuredClone(obj);
```

`JSON.parse(JSON.stringify(obj))` is sometimes shown in interviews, but it is not a general deep-clone solution because it loses or mishandles values such as `undefined`, `Date`, `Map`, `Set`, functions, and circular references.

---

# 19. Prototypes and the Prototype Chain

JavaScript uses **prototype-based inheritance**.

When you access:

```js
obj.someProperty
```

JavaScript first looks on `obj`. If it is not found, JavaScript looks at `obj`'s prototype, then that prototype's prototype, and so on until it reaches `null`.

```text
object
  ↓
prototype
  ↓
prototype's prototype
  ↓
...
  ↓
null
```

Example:

```js
const animal = {
  eat() {
    console.log("eating");
  }
};

const dog = Object.create(animal);
dog.bark = () => console.log("bark");

dog.eat(); // found through prototype chain
```

## Constructor functions and `.prototype`

```js
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function () {
  return `Hello ${this.name}`;
};

const p = new Person("Aman");
p.greet();
```

Objects created with `new Person()` inherit from `Person.prototype`.

---

# 20. What `new` Does

When you write:

```js
const p = new Person("Aman");
```

Conceptually JavaScript:

1. creates a new empty object
2. connects that object's prototype to `Person.prototype`
3. calls `Person` with `this` set to the new object
4. returns the new object unless the constructor explicitly returns another object

This explanation is a common interview question.

---

# 21. Classes

JavaScript `class` syntax provides a cleaner way to work with the prototype system. It does not replace prototypes with a completely different inheritance mechanism.

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello ${this.name}`;
  }
}

class Student extends Person {
  constructor(name, college) {
    super(name);
    this.college = college;
  }
}
```

Methods declared in a class are placed on the class's prototype rather than copied independently onto every instance.

---

# 22. Arrays

Arrays are ordered, zero-indexed objects specialized for list-like data.

```js
const nums = [10, 20, 30];
```

Useful basics:

```js
nums.length
nums[0]
nums.at(-1) // last element
```

## Mutating methods

These change the original array:

```text
push
pop
shift
unshift
splice
sort
reverse
```

## Non-mutating / returning new values

Common examples:

```text
slice
map
filter
concat
```

Modern JavaScript also provides non-mutating alternatives such as `toSorted()`, `toReversed()`, and `toSpliced()` in supported environments.

---

# 23. `slice()` vs `splice()`

This is repeatedly asked.

## `slice(start, end)`

- does not mutate the original array
- returns a selected portion
- `end` is excluded

```js
const a = [1, 2, 3, 4];
const b = a.slice(1, 3);

console.log(b); // [2, 3]
console.log(a); // unchanged
```

## `splice(start, deleteCount, ...items)`

- mutates the original array
- can remove, insert, or replace elements
- returns the removed elements

```js
const a = [1, 2, 3, 4];
const removed = a.splice(1, 2, 9, 10);

console.log(a);       // [1, 9, 10, 4]
console.log(removed); // [2, 3]
```

---

# 24. Sorting Numbers Correctly

By default, `sort()` converts values to strings and compares them lexicographically.

```js
[2, 10, 3].sort(); // [10, 2, 3]
```

For numeric ascending order:

```js
nums.sort((a, b) => a - b);
```

Descending:

```js
nums.sort((a, b) => b - a);
```

Remember: `sort()` mutates the original array.

---

# 25. Iterating Arrays

## `for...of`

Use when you need values:

```js
for (const value of nums) {
  console.log(value);
}
```

## `for...in`

`for...in` iterates enumerable property keys. It is mainly intended for objects, not normal array iteration.

```js
for (const key in user) {
  console.log(key, user[key]);
}
```

For arrays, prefer `for`, `for...of`, or array methods.

## `forEach`

```js
nums.forEach((value, index) => {
  console.log(index, value);
});
```

`forEach()` always returns `undefined` and cannot be broken using normal `break`.

---

# 26. `map`, `filter`, and `reduce`

These three are heavily used in interviews and frontend work.

## `map()` — transform every item

```js
const doubled = [1, 2, 3].map(x => x * 2);
// [2, 4, 6]
```

`map()` should be used when you want one output value corresponding to each input element.

## `filter()` — keep matching items

```js
const even = [1, 2, 3, 4].filter(x => x % 2 === 0);
// [2, 4]
```

## `reduce()` — combine into one result

```js
const sum = [1, 2, 3, 4].reduce((acc, x) => {
  return acc + x;
}, 0);
```

`reduce()` can produce a number, object, array, map, or essentially any accumulated result.

Example: frequency map

```js
const words = ["a", "b", "a"];

const freq = words.reduce((acc, word) => {
  acc[word] = (acc[word] ?? 0) + 1;
  return acc;
}, {});
```

---

# 27. `find`, `some`, and `every`

These are easy to confuse.

```js
const nums = [2, 4, 7, 8];
```

`find()` returns the first matching element:

```js
nums.find(x => x % 2 !== 0); // 7
```

`some()` asks whether at least one element matches:

```js
nums.some(x => x % 2 !== 0); // true
```

`every()` asks whether all elements match:

```js
nums.every(x => x > 0); // true
```

---

# 28. Strings

Strings are immutable primitives.

Common operations:

```js
const s = "  Hello World  ";

s.length
s.toUpperCase()
s.toLowerCase()
s.trim()
s.includes("World")
s.startsWith("Hello")
s.endsWith("World")
s.indexOf("o")
s.slice(2, 5)
s.replace("World", "JS")
s.split(" ")
```

Template literals make interpolation easy:

```js
const name = "Aman";
const msg = `Hello ${name}`;
```

---

# 29. Destructuring

## Arrays

```js
const [first, second, ...rest] = [10, 20, 30, 40];
```

Results:

```text
first = 10
second = 20
rest = [30, 40]
```

## Objects

```js
const user = { name: "Aman", age: 22 };
const { name, age } = user;
```

Rename during destructuring:

```js
const { name: userName } = user;
```

Default value:

```js
const { city = "Unknown" } = user;
```

Function parameter destructuring:

```js
function printUser({ name, age }) {
  console.log(name, age);
}
```

---

# 30. Spread vs Rest

Both use `...`, but their direction is opposite.

## Spread expands

```js
const a = [1, 2];
const b = [3, 4];
const merged = [...a, ...b];
```

Objects:

```js
const updated = { ...user, age: 23 };
```

Later properties overwrite earlier properties.

## Rest collects

```js
const [first, ...remaining] = [1, 2, 3];

function sum(...numbers) {
  // numbers is an array
}
```

Remember:

> **Spread opens a collection. Rest gathers remaining values.**

---

# 31. `Map` and `Set`

## `Set`

A `Set` stores unique values.

```js
const set = new Set([1, 1, 2, 3]);
console.log([...set]); // [1, 2, 3]
```

Useful operations:

```js
set.add(value)
set.has(value)
set.delete(value)
set.size
```

A common use is removing duplicates:

```js
const unique = [...new Set(array)];
```

## `Map`

A `Map` stores key-value pairs and allows keys of any type.

```js
const map = new Map();
map.set("name", "Aman");
map.set(42, "answer");
```

Useful methods:

```js
map.get(key)
map.has(key)
map.delete(key)
map.size
```

Compared with plain objects, `Map` is particularly useful when keys are dynamic or not just strings/symbols.

---

# 32. Event Handling in the Browser

Use `addEventListener()` rather than assigning inline handlers when you want flexible event management.

```js
const button = document.querySelector("button");

function handleClick(event) {
  console.log(event.target);
}

button.addEventListener("click", handleClick);
```

To remove it:

```js
button.removeEventListener("click", handleClick);
```

You need the **same function reference**. This will not remove the original listener:

```js
button.removeEventListener("click", () => {
  console.log("clicked");
});
```

because that creates a new function object.

---

# 33. Event Bubbling, Capturing, and Delegation

DOM events travel through phases.

A simplified model:

```text
window/document
   ↓ capturing
parent
   ↓
target
   ↑ bubbling
parent
   ↑
window/document
```

By default, event listeners usually participate in the bubbling phase.

```js
parent.addEventListener("click", handler);
```

Use capture explicitly:

```js
parent.addEventListener("click", handler, { capture: true });
```

## Event delegation

Instead of attaching a listener to every child, attach one listener to a common ancestor and inspect `event.target`.

```js
list.addEventListener("click", event => {
  if (event.target.matches("button.delete")) {
    // handle delete
  }
});
```

Why this is useful:

- fewer listeners
- works for dynamically added children
- simpler management

This is a common frontend interview question.

---

# 34. `event.target` vs `event.currentTarget`

`event.target` is the element where the event originated.

`event.currentTarget` is the element whose listener is currently running.

With event delegation, these are often different.

---

# 35. Synchronous vs Asynchronous JavaScript

Synchronous code runs in normal stack order:

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```text
A
B
C
```

Asynchronous APIs allow JavaScript to start some work and continue rather than blocking the stack while waiting.

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

```text
A
C
B
```

`setTimeout(..., 0)` means the callback becomes eligible after at least that delay. It does **not** mean “run immediately.”

---

# 36. The Event Loop

The event loop coordinates execution between the call stack and queued callbacks.

For browser interview questions, distinguish two important categories:

## Tasks / macrotasks

Examples include:

```text
setTimeout callbacks
setInterval callbacks
many DOM event callbacks
```

## Microtasks

Examples include:

```text
Promise .then/.catch/.finally callbacks
queueMicrotask callbacks
```

After the current synchronous stack finishes, JavaScript drains the microtask queue before moving to the next task.

Example:

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

Output:

```text
A
D
C
B
```

Reason:

1. synchronous code runs first: `A`, `D`
2. promise callback is a microtask: `C`
3. timer callback is a later task: `B`

### Important correction for Node.js

`process.nextTick()` is Node-specific and has special scheduling behavior. Do not present it as a normal browser microtask API. For frontend/browser interviews, focus on promises and `queueMicrotask()`.

---

# 37. Callbacks and Callback Hell

Callbacks become difficult when many dependent asynchronous operations are deeply nested:

```js
login(user, result => {
  loadProfile(result.id, profile => {
    loadOrders(profile.id, orders => {
      // ...
    });
  });
});
```

Problems include:

- poor readability
- harder error handling
- difficult composition
- deeply nested control flow

Promises were designed to provide a more composable abstraction for eventual asynchronous results.

---

# 38. Promises

A Promise represents a value that may become available later.

A promise has one of three states:

```text
pending
fulfilled
rejected
```

Once fulfilled or rejected, it is **settled** and does not switch to another state.

## Creating a Promise

```js
const promise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("done");
  } else {
    reject(new Error("failed"));
  }
});
```

The executor passed to `new Promise(...)` runs **synchronously** when the promise is created.

The callbacks registered with `.then()` / `.catch()` are scheduled asynchronously as microtasks.

---

# 39. Promise Chaining

```js
fetchUser()
  .then(user => {
    return fetchOrders(user.id);
  })
  .then(orders => {
    console.log(orders);
  })
  .catch(error => {
    console.error(error);
  })
  .finally(() => {
    console.log("finished");
  });
```

A crucial rule:

> `.then()` returns a **new Promise**.

If a `.then()` callback returns a normal value, the next promise fulfills with that value.

```js
Promise.resolve(5)
  .then(x => x * 2)
  .then(x => console.log(x)); // 10
```

If it returns another promise, the chain waits for that promise.

If it throws, the returned promise becomes rejected.

---

# 40. Promise Static Methods

These are frequent interview questions.

## `Promise.all()`

Waits for all promises to fulfill. Rejects as soon as one input rejects.

```js
const results = await Promise.all([
  fetchA(),
  fetchB(),
  fetchC()
]);
```

Use when **all results are required**.

## `Promise.allSettled()`

Waits for every input to settle and reports each outcome.

```js
const results = await Promise.allSettled([a, b, c]);
```

Use when you want results even if some operations fail.

## `Promise.race()`

Settles with the first input promise that settles, whether fulfilled or rejected.

Useful for timeout-like patterns.

## `Promise.any()`

Fulfills with the first input promise that fulfills. It ignores rejections until all have rejected, in which case it rejects with an `AggregateError`.

### Quick comparison

| Method | Success condition | Failure behavior |
|---|---|---|
| `all` | all fulfill | first rejection rejects result |
| `allSettled` | always waits for all | returns every outcome |
| `race` | first settlement wins | first settlement may be rejection |
| `any` | first fulfillment wins | rejects only if all reject |

---

# 41. `async` / `await`

An `async` function always returns a promise.

```js
async function getValue() {
  return 10;
}
```

Equivalent from the caller's perspective to returning a fulfilled promise:

```js
getValue().then(console.log); // 10
```

`await` pauses **that async function**, not the entire JavaScript thread.

```js
async function load() {
  const user = await fetchUser();
  return user;
}
```

While `load()` is suspended, JavaScript can run other tasks.

## Error handling

```js
async function load() {
  try {
    const result = await fetchData();
    return result;
  } catch (error) {
    console.error(error);
  }
}
```

A rejected awaited promise behaves like a thrown exception at the `await` point.

---

# 42. Sequential vs Concurrent Async Work

This is very important in interviews and real code.

## Sequential

```js
const a = await fetchA();
const b = await fetchB();
```

`fetchB()` starts only after `fetchA()` has completed.

Use this if B depends on A.

## Concurrent independent work

```js
const aPromise = fetchA();
const bPromise = fetchB();

const [a, b] = await Promise.all([aPromise, bPromise]);
```

Both operations are started before waiting for completion.

An even cleaner version:

```js
const [a, b] = await Promise.all([
  fetchA(),
  fetchB()
]);
```

Do not sequentially await independent network requests unless you have a reason.

---

# 43. `fetch()` Interview Notes

```js
const response = await fetch("/api/users");
const data = await response.json();
```

Important gotcha:

> `fetch()` does **not** reject just because the server returned HTTP 404 or 500.

It rejects mainly for network-level failures or aborted requests. You should check the HTTP status yourself:

```js
const response = await fetch(url);

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}
```

---

# 44. Error Handling

Use `try...catch` for synchronous exceptions and awaited promise rejections.

```js
try {
  riskyOperation();
} catch (error) {
  console.error(error.message);
} finally {
  console.log("always runs");
}
```

Create an error with:

```js
throw new Error("Something went wrong");
```

Do not normally throw raw strings because `Error` objects provide stack information and a conventional shape.

---

# 45. Modules: ESM vs CommonJS

## ES Modules (ESM)

```js
// math.js
export function add(a, b) {
  return a + b;
}

// app.js
import { add } from "./math.js";
```

ESM is the standard JavaScript module system and is used by browsers and modern tooling.

## CommonJS

CommonJS is historically common in Node.js:

```js
module.exports = { add };
const { add } = require("./math");
```

Placement answer:

> ESM uses `import`/`export` and is the language-standard module system. CommonJS uses `require`/`module.exports` and originated in the Node.js ecosystem.

---

# 46. Currying

Currying transforms a function that takes several arguments into a sequence of functions that each take one argument.

```js
function add(a) {
  return function (b) {
    return a + b;
  };
}

add(2)(3); // 5
```

Why it matters:

- function reuse
- partial configuration
- functional programming patterns
- frequent interview question about closures

Example:

```js
const multiply = a => b => a * b;
const double = multiply(2);

double(5); // 10
```

The returned function remembers `a` through a closure.

---

# 47. Debouncing and Throttling

These are extremely common frontend interview questions.

## Debounce

Debouncing waits until calls stop for a specified period, then executes once.

Use cases:

```text
search suggestions
resize handling
autosave
```

Basic implementation:

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

Mental model:

> **Debounce: “wait until the noise stops.”**

## Throttle

Throttling limits execution to at most once per interval.

Use cases:

```text
scroll events
mousemove
continuous resize updates
```

One simple implementation:

```js
function throttle(fn, interval) {
  let allowed = true;

  return function (...args) {
    if (!allowed) return;

    allowed = false;
    fn.apply(this, args);

    setTimeout(() => {
      allowed = true;
    }, interval);
  };
}
```

Mental model:

> **Throttle: “keep running, but at a controlled rate.”**

---

# 48. Function Composition and Pure Functions

A **pure function**:

- gives the same output for the same input
- does not produce observable side effects

```js
function add(a, b) {
  return a + b;
}
```

Impure example:

```js
let total = 0;

function addToTotal(x) {
  total += x;
}
```

Pure functions are easier to test and reason about, but real applications naturally need side effects for I/O, DOM updates, logging, etc.

---

# 49. Browser Storage: Quick Placement Notes

## `localStorage`

- persists across browser restarts
- stores strings
- synchronous API
- scoped to an origin

```js
localStorage.setItem("theme", "dark");
localStorage.getItem("theme");
```

## `sessionStorage`

- similar API
- generally lasts for the lifetime of the browser tab/session

## Cookies

- small pieces of data associated with HTTP requests
- can be configured with attributes such as `HttpOnly`, `Secure`, `SameSite`, expiry, path, etc.
- unlike Web Storage, cookies may be sent automatically with matching HTTP requests

For authentication, do not reduce the answer to “localStorage vs cookies” without discussing security requirements and server design.

---

# 50. Garbage Collection and Memory Leaks

JavaScript automatically reclaims memory that is no longer reachable.

You do not manually free objects, but memory leaks can still happen when you accidentally keep references alive.

Common causes:

- forgotten event listeners
- timers that are never cleared
- global references
- caches that grow forever
- closures retaining large objects longer than necessary

Placement-level answer:

> JavaScript uses automatic garbage collection based on reachability. A leak occurs when memory is still reachable through references even though the application no longer needs it.

---

# 51. Common Output Questions

## Hoisting

```js
console.log(a);
var a = 5;
```

Output:

```text
undefined
```

But:

```js
console.log(a);
let a = 5;
```

throws a `ReferenceError` because `a` is in the TDZ.

---

## `this` and detached methods

```js
const user = {
  name: "Aman",
  greet() {
    console.log(this.name);
  }
};

const fn = user.greet;
fn();
```

The method has been detached from `user`. A regular function receives `this` from the call site, so the original object is no longer automatically used.

---

## Promise vs timer

```js
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve().then(() => console.log(3));

console.log(4);
```

Output:

```text
1
4
3
2
```

---

## Object reference

```js
const a = { value: 1 };
const b = a;
b.value = 2;

console.log(a.value); // 2
```

---

## Shallow spread

```js
const a = { nested: { value: 1 } };
const b = { ...a };

b.nested.value = 5;
console.log(a.nested.value); // 5
```

---

# 52. High-Yield Interview Comparisons

## `==` vs `===`

- `==` may coerce types
- `===` compares without coercion
- prefer `===` in most application code

## `null` vs `undefined`

- `undefined`: value missing/not assigned by default
- `null`: intentional empty value chosen by the programmer/API

## `var` vs `let` vs `const`

- scope and TDZ are the core differences
- use `const` by default, `let` for reassignment

## arrow vs regular function

- arrow inherits lexical `this`
- regular function receives `this` from call style
- arrow cannot be used with `new`

## `map` vs `forEach`

- `map` returns a new transformed array
- `forEach` is for side effects and returns `undefined`

## `slice` vs `splice`

- `slice`: non-mutating extraction
- `splice`: mutating insertion/removal/replacement

## shallow vs deep copy

- shallow: nested references shared
- deep: nested structures independently copied where supported

## synchronous vs asynchronous

- synchronous work occupies the call stack until it completes
- asynchronous host APIs allow waiting work to be scheduled later

## microtask vs task

- promise callbacks are microtasks
- timers are tasks
- microtasks are drained before the next task

## debounce vs throttle

- debounce: run after calls stop
- throttle: run at a limited frequency while calls continue

---

# 53. What You Should Be Able to Explain Without Notes

For placement interviews, make sure you can explain these comfortably:

1. Why `let`/`const` behave differently from `var` before declaration.
2. What a closure is and why the counter example keeps its state.
3. How `this` differs in regular functions and arrow functions.
4. What happens when a method is detached from its object.
5. How the prototype chain works.
6. What `new` does internally.
7. Why `{}` is not equal to another `{}`.
8. Shallow copy vs deep copy.
9. `map`, `filter`, `reduce`, `forEach`, `find`, `some`, `every`.
10. `slice` vs `splice`.
11. Spread vs rest.
12. The call stack and event loop.
13. Why promise callbacks run before a `setTimeout(..., 0)` callback.
14. Promise states and chaining.
15. `Promise.all`, `allSettled`, `race`, and `any`.
16. Why `await` does not block the entire JavaScript thread.
17. Sequential vs concurrent async operations.
18. Event bubbling and event delegation.
19. Debounce vs throttle.
20. `call`, `apply`, and `bind`.

If these twenty are strong, your JavaScript placement foundation is already in very good shape.

---

# 54. Recommended Revision Order Before an Interview

If you have only 30–45 minutes, revise in this order:

```text
1. var / let / const + hoisting + TDZ
2. scope + closures
3. functions + this + arrow functions + call/apply/bind
4. objects + references + shallow copy
5. prototypes + classes
6. array methods
7. promises + async/await
8. event loop + microtasks/tasks
9. DOM events + event delegation
10. debounce/throttle + common output questions
```

Do not waste the final hour before an interview memorizing every string method or obscure API. Interviewers usually care more about whether you can reason through JavaScript's execution model than whether you remember a rarely used method name.

---

# 55. Compact Final Cheat Sheet

```text
JavaScript:
- dynamically typed, garbage collected
- one JS call stack per execution context/realm
- host environment provides async APIs

Scope:
- lexical scope
- var = function scoped
- let/const = block scoped
- let/const have TDZ

Functions:
- first-class values
- closures preserve lexical environment
- arrow functions inherit lexical this
- regular function this depends on call site

Objects:
- variables hold references
- spread/Object.assign = shallow copy
- property lookup follows prototype chain

Arrays:
- map -> transform
- filter -> select
- reduce -> accumulate
- forEach -> side effects
- slice -> non-mutating
- splice -> mutating
- sort -> mutating; numeric comparator needed

Async:
- synchronous stack first
- promise handlers -> microtasks
- timers -> tasks
- microtasks drain before next task
- async function always returns a promise
- await pauses only the async function

Promises:
- pending -> fulfilled/rejected
- all = all required, fail fast
- allSettled = collect every outcome
- race = first settlement
- any = first fulfillment

Browser:
- event.target = origin
- event.currentTarget = listener owner
- events bubble by default
- delegation = listener on ancestor

Performance:
- debounce = wait for silence
- throttle = limit rate
```

---

# 56. Final Placement Perspective

The strongest JavaScript candidates are not the ones who memorize the largest number of APIs. They are the ones who can look at a short snippet and explain **why JavaScript executes it that way**.

When revising, repeatedly ask:

```text
Where is this variable stored and visible?
What does this function close over?
What determines this?
Is this value copied or referenced?
Does this operation mutate the original value?
Is this callback synchronous, a microtask, or a task?
Does this await depend on the previous one?
What object will property lookup search next?
```

If you can answer those questions reliably, most placement-level JavaScript questions stop feeling like tricks and start becoming predictable consequences of a small set of rules.
