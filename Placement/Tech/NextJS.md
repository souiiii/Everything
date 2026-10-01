# Next JS

# Next.js Core Notes — App Router, Components & Routing

> **Mental model:** Next.js is a React framework that adds routing, server-side capabilities, rendering conventions, and backend endpoints around React.
> 
> 
> The important idea from yesterday was not “Next.js is React with extra features.” It was understanding **where code runs and how the App Router organizes an application**.
> 

These notes cover what we actually studied yesterday. The later topics such as data fetching strategies, static/dynamic rendering and streaming belong to the next Next.js block, so I’m not mixing them in here.

---

## 1. The App Router

Modern Next.js uses the **App Router**, where the folder structure inside `app/` defines the routes of the application.

Example:

```
app/
├── page.tsx
├── about/
│   └── page.tsx
└── products/
    └── page.tsx
```

This creates:

```
/           → app/page.tsx
/about      → app/about/page.tsx
/products   → app/products/page.tsx
```

So unlike plain React Router, you usually do not manually write a large route configuration.

> **Folder structure becomes route structure.**
> 

This is called **file-system routing**.

---

# 2. What does `page.tsx` mean?

A `page.tsx` file represents the actual UI for a route.

```tsx
export default function AboutPage() {
    return <h1>About</h1>;
}
```

Placed here:

```
app/about/page.tsx
```

it becomes:

```
/about
```

### Recall

> **Folder = route segment**
> 
> 
> **`page.tsx` = UI shown at that route**
> 

---

# 3. What does `layout.tsx` do?

A layout contains UI that should remain shared across multiple pages.

Example:

```tsx
export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html>
            <body>
                <Navbar />
                {children}
            </body>
        </html>
    );
}
```

The important part is:

```tsx
{children}
```

That is where the current page is inserted.

Layouts are useful for things such as:

```
Navbar
Sidebar
Footer
Dashboard shell
```

that should surround several routes.

### Mental model

```
layout
 ├── Navbar
 ├── page
 └── Footer
```

Instead of every page rebuilding the same surrounding UI.

---

# 4. Server Components

This was one of the most important concepts.

In the App Router, components are **Server Components by default**.

```tsx
export default function Page() {
    return <h1>Hello</h1>;
}
```

You do not have to write anything special to make this a Server Component.

### What does that mean?

The component's logic runs on the **server side**, rather than being treated as interactive browser-side React code.

This has an important architectural benefit:

> If a component does not need browser interaction, there is no reason to turn it into unnecessary client-side JavaScript.
> 

---

# 5. Client Components

Some components genuinely need browser-side behavior.

Examples:

```
button click handlers
useState
useEffect
browser APIs
interactive forms
interactive menus
```

These need to be **Client Components**.

Example:

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button onClick={() => setCount(count + 1)}>
            {count}
        </button>
    );
}
```

The `"use client"` line tells Next.js:

> This component belongs on the client side and may use browser-side React features.
> 

---

# 6. Server vs Client Components

The easiest distinction:

### Server Component

Use when the component mainly:

```
renders content
doesn't need state
doesn't need effects
doesn't need browser APIs
doesn't need event handlers
```

### Client Component

Use when you need:

```tsx
useState()
useEffect()
onClick
onChange
window
localStorage
document
```

### Interview answer

> Server Components are the default in the Next.js App Router and are useful for non-interactive server-side UI. Client Components are required when a component needs browser-side interactivity such as state, effects, event handlers or browser APIs.
> 

---

# 7. `"use client"` is a boundary

A very important detail:

You should **not** blindly put:

```tsx
"use client";
```

at the top of every component.

Doing that throws away much of the advantage of Server Components.

Instead, keep `"use client"` as low in the component tree as practical.

Example:

```
ProductPage          ← Server
│
├── ProductDetails   ← Server
│
└── AddToCartButton  ← Client
```

Only this needs interactivity:

```tsx
"use client";

function AddToCartButton() {
    ...
}
```

So only that interactive portion needs to become client-side.

### Recall

> **Server by default. Move to client only when interactivity requires it.**
> 

That is a much better mental model than:

> “Should this page be server or client?”
> 

Think at the **component boundary** level.

---

# 8. Dynamic Routes

Suppose you need:

```
/products/1
/products/2
/products/500
```

You obviously do not create 500 folders manually.

Next.js supports **dynamic route segments**.

```
app/
└── products/
    └── [id]/
        └── page.tsx
```

Now:

```
/products/10
```

gives:

```
id = "10"
```

and:

```
/products/abc
```

gives:

```
id = "abc"
```

### Recall

> `[name]` means **this part of the URL is dynamic**.
> 

Examples:

```
/users/[id]
/posts/[slug]
/products/[productId]
```

---

# 9. `params`

The dynamic value is received through `params`.

Conceptually:

```
/products/42

        ↓

params.id = "42"
```

One current Next.js detail we specifically discussed is that in modern Server Components, `params` may be provided as a **Promise**, so you may see code shaped like:

```tsx
export default async function Page({
    params,
}: {
    params: Promise<{ id: string }>;
}) {
    const { id } = await params;

    return <div>{id}</div>;
}
```

Don't obsess over memorizing the syntax.

The important interview understanding is:

> Dynamic route values come through `params`.
> 

---

# 10. Route Handlers

Next.js can also expose backend HTTP endpoints.

Suppose you create:

```
app/
└── api/
    └── users/
        └── route.ts
```

You can define handlers such as:

```tsx
export async function GET() {
    // ...
}
```

or:

```tsx
export async function POST(request: Request) {
    // ...
}
```

This gives you an endpoint like:

```
GET /api/users
POST /api/users
```

### Mental model

```
page.tsx   → UI route
route.ts   → HTTP/API route
```

That distinction is worth remembering.

---

# 11. Why Route Handlers Matter

In a normal React SPA you might have:

```
React frontend
       ↓
Express backend
       ↓
Database
```

Next.js allows some backend functionality to exist inside the same application:

```
Next.js UI
    ↓
Route Handler
    ↓
Database / service
```

That does **not** mean every Next.js application must abandon a separate backend.

It simply means Next.js is capable of handling both UI routes and server-side HTTP routes.

---

# 12. Server Actions

We also covered **Server Actions**.

The basic idea is that a function can be explicitly marked to execute on the server:

```tsx
async function createUser(formData: FormData) {
    "use server";

    // server-side work
}
```

A component can then invoke that server-side operation without you manually creating the usual REST-style endpoint for every action.

Conceptually:

```
Client interaction
       ↓
Server Action
       ↓
Server-side logic
```

---

# 13. Server Actions vs Route Handlers

This distinction matters.

### Route Handler

You explicitly create an HTTP endpoint:

```
POST /api/users
```

and call it.

This makes the network/API boundary very obvious.

### Server Action

You invoke a server function through Next.js's server-action mechanism.

This can reduce boilerplate for certain application operations.

---

## Your preferred mental model

You found **explicit API endpoints easier to reason about** than Server Actions.

That is completely reasonable:

```
Frontend
   ↓ POST /api/user
Route Handler
   ↓
Backend logic
```

is a very visible architecture.

Server Actions can make code convenient, but they should not make you forget that there is still a **client → server boundary**.

---

# 14. A Server Function Is Not Automatically Safe

One of the important security points from yesterday:

> **If code can be triggered by the client, treat it as a public-facing server entry point.**
> 

Whether you use:

```
Route Handler
```

or:

```
Server Action
```

you still need things like:

```
authentication
authorization
input validation
permission checks
```

Do not think:

> “It is a server function, therefore the user cannot misuse it.”
> 

The browser may still cause that server-side operation to execute.

### Interview-quality rule

> **Server-side does not mean trusted input.**
> 

---

# 15. Why Next.js Splits Server and Client Components

The split exists because not every part of a React application needs the same environment.

Consider:

```
Product page
```

Most of it may simply display:

```
name
description
price
reviews
```

Only a tiny piece may need interaction:

```
Add to Cart button
```

Making the entire page client-side just because one button needs `onClick` is unnecessary.

Next.js allows:

```
Mostly server-rendered component tree
                 +
Small interactive client islands
```

That is the architectural idea behind the split.

---

# 16. The Next.js App Router Mental Model

Put everything together:

```
app/
│
├── layout.tsx
│
├── page.tsx
│
├── products/
│   │
│   ├── page.tsx
│   │
│   └── [id]/
│   │       └── page.tsx
│   │
│   └── AddToCart.tsx
│       "use client"
│
└── api/
    └── products/
        └── route.ts
```

Interpret it as:

```
layout.tsx
    ↓
shared application UI

page.tsx
    ↓
route UI

[id]
    ↓
dynamic route

"use client"
    ↓
browser interaction required

route.ts
    ↓
HTTP endpoint
```

If this picture is clear, yesterday's block is basically understood.

---

# Interview Recall Sheet

### What is the App Router?

Next.js's file-system based routing system where folders represent route segments and files such as `page.tsx` and `layout.tsx` define route UI and shared layouts.

---

### What is a Server Component?

A component that runs as server-side React code. Server Components are the default in the App Router and are appropriate when browser-side interactivity is unnecessary.

---

### When do you need a Client Component?

When you need things such as:

```
state
effects
event handlers
browser APIs
```

---

### What does `"use client"` do?

It creates a client-component boundary, allowing that component to use browser-side React functionality.

---

### Should you put `"use client"` everywhere?

No.

> Keep the client boundary as low as practical and leave non-interactive components as Server Components.
> 

---

### What does `[id]` mean?

It creates a dynamic route segment.

```
products/[id]
```

can match:

```
/products/123
```

---

### `page.tsx` vs `layout.tsx`?

```
page.tsx   → content for one route
layout.tsx → shared UI around routes
```

---

### `page.tsx` vs `route.ts`?

```
page.tsx  → renders UI
route.ts  → handles HTTP requests
```

---

### Route Handler vs Server Action?

> Route Handlers expose explicit HTTP endpoints. Server Actions let client interactions invoke server-side functions through Next.js's action mechanism.
> 

---

### Are Server Actions automatically secure?

No.

Treat them like any other server entry point:

```
authenticate
authorize
validate
```

---

# The 6 things to remember

If you need to reconstruct yesterday's entire Next.js block before an interview, remember:

1. **`app/` uses file-system routing.**
2. **`page.tsx` = route, `layout.tsx` = shared wrapper.**
3. **Server Components are the default.**
4. **Use `"use client"` only when browser interaction requires it.**
5. **`[id]` creates dynamic routes; values arrive through `params`.**
6. **`route.ts` gives explicit API endpoints; Server Actions provide another server-call mechanism, but both still need security checks.**

That covers the actual scope of yesterday’s **Next.js 1: App Router and rendering/component boundaries** without pulling tomorrow’s data-fetching material forward.