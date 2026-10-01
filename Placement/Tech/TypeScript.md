# TypeScript

# TypeScript — From JavaScript to Interview-Ready

*These notes assume you already know JavaScript. The goal is not to memorise every TypeScript feature, but to understand the small set of ideas that make real TypeScript code predictable: how types describe values, how TypeScript narrows uncertainty, and how reusable types preserve information instead of throwing it away.*

---

## Part 0 · The mental model that makes TypeScript make sense

TypeScript is **JavaScript with a type-checking layer added during development**.

You write:

```tsx
function add(a: number, b: number): number {
  return a + b;
}
```

TypeScript checks whether your assumptions are consistent. After compilation, the type annotations disappear and ordinary JavaScript runs.

```
TypeScript source
      ↓
type checker verifies assumptions
      ↓
JavaScript output
      ↓
runtime
```

This distinction matters because TypeScript does **not** validate arbitrary runtime data for you. If an API sends malformed JSON, the compiler cannot magically stop it. TypeScript protects the code paths it can reason about before execution.

> **Interview answer:** *“TypeScript is a statically typed superset of JavaScript. It catches many type errors during development, improves editor tooling and makes contracts clearer, but its types are erased when the code is compiled to JavaScript.”*
> 

---

## Part 1 · Inference first, annotations where they add information

TypeScript can infer obvious types:

```tsx
let age = 21;          // number
const name = "Shahid"; // literal/string information
```

Writing this everywhere:

```tsx
let age: number = 21;
```

is usually unnecessary. The annotation adds no new information.

Explicit types are most useful when you are defining a **contract**:

```tsx
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}
```

or when the intended type is wider than the initial value:

```tsx
let status: "idle" | "loading" | "success" = "idle";
```

### Practical rule

> Let TypeScript infer local implementation details. Add explicit types at boundaries: function parameters, returned data, component props, API models, shared objects and public library code.
> 

That keeps the code readable without giving up safety.

---

## Part 2 · Describing data: objects, arrays, optional fields and tuples

TypeScript is heavily based on **shape**. If an object has the required properties with compatible types, it can often be used regardless of what class created it.

```tsx
type User = {
  id: string;
  name: string;
  email?: string;
  readonly createdAt: Date;
};
```

Here, `email?` means the property may be absent. `readonly` means TypeScript prevents reassignment through this reference.

```tsx
const user: User = {
  id: "u1",
  name: "Shahid",
  createdAt: new Date()
};

user.createdAt = new Date(); // error
```

### Arrays

```tsx
const scores: number[] = [90, 85, 96];
const names: Array<string> = ["A", "B"];
```

Both forms are valid. `number[]` is usually easier to read.

### Tuples

A tuple describes a fixed sequence where positions have specific meanings:

```tsx
const result: [number, string] = [200, "OK"];
```

This is different from:

```tsx
(number | string)[]
```

which means an array of arbitrary length containing numbers or strings.

---

## Part 3 · Functions are contracts

A function type tells TypeScript what callers must provide and what they can expect back.

```tsx
function greet(name: string): string {
  return `Hello ${name}`;
}
```

Optional parameters use `?`:

```tsx
function greet(name: string, title?: string): string {
  return title ? `Hello ${title} ${name}` : `Hello ${name}`;
}
```

Default parameters are also allowed:

```tsx
function retry(count: number = 3) {
  // ...
}
```

### `void`

```tsx
function logMessage(message: string): void {
  console.log(message);
}
```

`void` means the caller should not expect a useful returned value.

Do not confuse it with `undefined`. A callback typed as returning `void` may technically return something, but the caller is saying, “I am not going to use that result.”

---

## Part 4 · Unions, literal types and narrowing

This is one of the most important parts of TypeScript.

A union says a value can have more than one valid type:

```tsx
function printId(id: string | number) {
  // id might be either
}
```

You cannot safely call string-only methods yet because TypeScript does not know which possibility you currently have.

You **narrow** the union by proving which case you are in:

```tsx
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(0));
  }
}
```

Think:

```
string | number
      ↓ check
specific known type
```

### Literal types

Literal types are useful when only a few states are valid:

```tsx
type Status = "idle" | "loading" | "success" | "error";
```

This is much stronger than:

```tsx
let status: string;
```

because invalid states such as `"finished-ish"` are rejected.

### Discriminated unions — very practical

Suppose an API result can either succeed or fail:

```tsx
type Result =
  | { success: true; data: string }
  | { success: false; error: string };
```

Now TypeScript can narrow using the shared discriminator:

```tsx
function handle(result: Result) {
  if (result.success) {
    console.log(result.data);
  } else {
    console.log(result.error);
  }
}
```

This pattern is common in API responses, reducers and UI state because impossible combinations become harder to represent.

> **Interview answer:** *“A union describes multiple possible types. Narrowing is the process of checking the value so TypeScript can safely determine which member of the union is active.”*
> 

---

## Part 5 · `any` vs `unknown` vs `never`

These three are easy to mix up, but their meanings are very different.

### `any` — turn type checking off

```tsx
let value: any;

value.foo.bar();
value();
value.toUpperCase();
```

TypeScript largely stops protecting you.

`any` is occasionally useful when integrating poorly typed legacy code, but it should usually be treated as an escape hatch rather than a normal design choice.

### `unknown` — the value could be anything, so prove it first

```tsx
let value: unknown;

// value.toUpperCase(); // error
```

You must narrow it:

```tsx
if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

This makes `unknown` ideal for genuinely uncertain data such as parsed input, caught errors or external data before validation.

A useful mental model:

> `any` = “trust me, stop checking.”
> 

> `unknown` = “I do not know yet, make me prove it.”
> 

### `never` — this state cannot happen

A function that never completes normally can return `never`:

```tsx
function fail(message: string): never {
  throw new Error(message);
}
```

A more valuable use is **exhaustive checking**:

```tsx
type Role = "admin" | "user";

function permissions(role: Role) {
  switch (role) {
    case "admin":
      return ["read", "write"];
    case "user":
      return ["read"];
    default: {
      const impossible: never = role;
      return impossible;
    }
  }
}
```

If somebody later adds `"moderator"` to `Role` but forgets to update the switch, TypeScript can expose the missing case.

---

## Part 6 · `type`, `interface` and intersections

Both `type` and `interface` can describe object shapes:

```tsx
type User = {
  name: string;
};

interface UserInfo {
  name: string;
}
```

Interfaces are naturally suited to object contracts and extension:

```tsx
interface Person {
  name: string;
}

interface Employee extends Person {
  salary: number;
}
```

Type aliases are more flexible because they can also represent unions, literals, tuples and primitives:

```tsx
type ID = string | number;
type Status = "loading" | "success" | "error";
type Point = [number, number];
```

### Intersections

An intersection means a value must satisfy **both** types:

```tsx
type Timestamped = {
  createdAt: Date;
};

type User = {
  id: string;
  name: string;
};

type StoredUser = User & Timestamped;
```

`StoredUser` must have `id`, `name` and `createdAt`.

### Interview-safe distinction

> *“Both can model object shapes. I usually use interfaces for extensible object contracts and type aliases when I need unions, tuples or composition. In most application code there is overlap, so consistency within the codebase matters more than treating one as universally better.”*
> 

---

## Part 7 · Generics — reusable code without losing type information

Suppose you write:

```tsx
function identity(value: any): any {
  return value;
}
```

It accepts everything, but the relationship between input and output is lost.

With a generic:

```tsx
function identity<T>(value: T): T {
  return value;
}
```

If the caller passes a string, the result remains a string. If the caller passes a number, the result remains a number.

```tsx
const a = identity("hello"); // string
const b = identity(42);      // number
```

The useful idea is not “`T` means any type.”

It is:

> **A generic lets a type chosen in one place flow into another place.**
> 

That relationship is what `any` throws away.

### Real application example

```tsx
type ApiResponse<T> = {
  success: boolean;
  data: T;
};

type User = {
  id: string;
  name: string;
};

const response: ApiResponse<User> = {
  success: true,
  data: {
    id: "u1",
    name: "Shahid"
  }
};
```

One reusable response shape can safely carry users, files, products or anything else.

### Generic constraints

Sometimes `T` can vary, but it still needs a minimum shape:

```tsx
function printId<T extends { id: string }>(value: T) {
  console.log(value.id);
}
```

Now callers can pass many different objects, as long as each has an `id: string`.

---

## Part 8 · Utility types — transform existing contracts instead of rewriting them

TypeScript includes helpers for deriving new types from existing ones.

Start with:

```tsx
type User = {
  id: string;
  name: string;
  email: string;
  password: string;
};
```

### `Partial<T>`

Makes every property optional:

```tsx
type UserUpdate = Partial<User>;
```

Useful for PATCH-style update objects.

### `Required<T>`

Makes every property required:

```tsx
type CompleteUser = Required<User>;
```

### `Pick<T, K>`

Keep only selected properties:

```tsx
type PublicUser = Pick<User, "id" | "name">;
```

### `Omit<T, K>`

Keep everything except selected properties:

```tsx
type SafeUser = Omit<User, "password">;
```

### `Record<K, V>`

Describe an object whose keys and values follow a known pattern:

```tsx
type Role = "admin" | "user";

const permissions: Record<Role, string[]> = {
  admin: ["read", "write"],
  user: ["read"]
};
```

These are common in real code because they keep related types synchronized. If the original model changes, derived types can change with it instead of silently drifting apart.

---

## Part 9 · `keyof` and `typeof` — useful interview-level type operators

### `keyof`

`keyof` produces a union of property names:

```tsx
type User = {
  id: string;
  name: string;
};

type UserKey = keyof User;
// "id" | "name"
```

A practical generic helper:

```tsx
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

Now invalid keys are rejected while the return type stays accurate.

### `typeof`

In a type position, `typeof` can derive a type from an existing value:

```tsx
const config = {
  port: 3000,
  debug: true
};

type Config = typeof config;
```

This is useful when a runtime value already defines the shape and you do not want to duplicate the type manually.

---

## Part 10 · Type assertions: useful, but they do not validate anything

You may see:

```tsx
const input = value as string;
```

This tells TypeScript:

> “Treat this value as a string.”
> 

It does **not** convert the value and it does **not** verify the value at runtime.

That makes assertions useful when you genuinely know something the compiler cannot infer, but dangerous when used to silence errors.

Bad instinct:

```tsx
const user = apiData as User;
```

If `apiData` is malformed, the assertion does not fix it.

> **Rule:** narrow or validate when possible. Assert only when you have a real reason to know more than the compiler.
> 

---

## Part 11 · Where these ideas appear in real full-stack code

### React props

```tsx
type ButtonProps = {
  label: string;
  disabled?: boolean;
  onClick: () => void;
};

function Button({ label, disabled, onClick }: ButtonProps) {
  // ...
}
```

The type is a contract between the component and every caller.

### API state

```tsx
type RequestState<T> =
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };
```

This is safer than independently tracking several booleans such as `loading`, `hasError` and `hasData`, which can accidentally form contradictory states.

### Update DTO

```tsx
type User = {
  id: string;
  name: string;
  email: string;
};

type UpdateUserInput = Partial<Pick<User, "name" | "email">>;
```

This says exactly what an update endpoint is allowed to modify without duplicating the field types.

---

## Part 12 · Common interview traps

### “TypeScript prevents runtime errors.”

Too broad. It catches many mistakes **before runtime**, but network data, database contents, user input and plain JavaScript can still violate your assumptions.

### “`any` and `unknown` are basically the same.”

They are opposites in safety. `any` disables checking; `unknown` forces checking before use.

### “Generics are just `any` with syntax.”

No. A generic preserves relationships between types. `any` removes those relationships.

### “`interface` is always better than `type`.”

No. They overlap heavily. The useful distinction is extensible object contracts versus the broader composition features of type aliases.

### “`as User` converts data into a User.”

It does not. A type assertion changes what the compiler believes, not the runtime value.

---

## Interview recall sheet

**What is TypeScript?**  

A statically typed superset of JavaScript. It checks types during development and compiles to JavaScript.

**What is type inference?**  

TypeScript derives a type from context instead of requiring an explicit annotation.

**Union vs intersection?**  

`A | B` means the value may satisfy A **or** B. `A & B` means it must satisfy both.

**What is narrowing?**  

Using runtime checks such as `typeof`, property checks or discriminants so TypeScript can reduce a broad type to a specific one.

**`any` vs `unknown`?**  

`any` largely disables type checking. `unknown` accepts any value but requires narrowing before unsafe operations.

**What is `never`?**  

A type representing a value that cannot exist or a code path that never completes normally. It is particularly useful for exhaustive checks.

**Type vs interface?**

Both can model object shapes. Interfaces are natural for extensible object contracts; type aliases can additionally model unions, tuples, primitives and other compositions.

**What is a generic?**  

A type parameter that lets reusable code preserve relationships between input and output types.

**Why utility types?**  

They derive new contracts such as update objects or public views from existing types, reducing duplication and keeping related types synchronized.

**Does TypeScript validate API responses at runtime?**  

No. Runtime data still needs validation if it cannot be trusted.

---

## The seven things worth remembering

1. **TypeScript checks assumptions before JavaScript runs; the types disappear at runtime.**
2. **Inference handles obvious local values; explicit types are most useful at boundaries and contracts.**
3. **Unions model alternatives; narrowing proves which alternative you currently have.**
4. **`unknown` preserves safety, while `any` opts out of it.**
5. **`never` is useful for impossible states and exhaustive checks.**
6. **Generics preserve type relationships; utility types transform existing contracts without duplication.**
7. **A type assertion tells the compiler what to believe — it does not validate or convert the runtime value.**

---

## Practical revision

Do not revise this page by rereading every line. Close it and try to explain these from memory:

- `unknown` vs `any` vs `never`
- union vs intersection
- narrowing with a real example
- `type` vs `interface`
- why a generic is better than `any`
- one real use each of `Partial`, `Pick`, `Omit` and `Record`
- why `as User` is not runtime validation

If you can explain those cleanly and write one small example of each, your TypeScript foundation is interview-ready.