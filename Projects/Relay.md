# Relay—Interview Preparation Notes

# Relay — Interview Notes

## 1. What is Relay?

**Relay is an intra-campus marketplace built for students to buy and sell used college essentials** such as textbooks, lab equipment, drafters, bicycles, electronics, and hostel items. The idea is to keep the entire transaction within the campus: a seller creates a listing, another student discovers it through search or filters, contacts the seller through Relay's messaging system, and they complete the physical exchange themselves.

A good 30-second explanation is:

> Relay is a full-stack campus marketplace. The frontend is built with Next.js and React, while a separate Express backend exposes REST APIs backed by MongoDB through Mongoose. Users can authenticate, create listings with images, search and filter products, save favorites, message sellers, and manage their listings through a dashboard. Authentication uses JWTs stored in httpOnly cookies.
> 

---

# 2. Overall Architecture

Relay is essentially split into two applications:

```
Browser
   ↓
Next.js frontend
   ↓  /api/* rewrite
Express backend
   ↓
Mongoose
   ↓
MongoDB
```

The frontend uses **Next.js 15 with the App Router, React 19 and Tailwind CSS 4**. The backend is an independent **Node.js + Express** service using **MongoDB with Mongoose**.

One particularly useful architectural decision is that the browser does not normally call the Express server directly. Next.js rewrites:

```
/api/*     → Express /api/*
/uploads/* → Express /uploads/*
```

The browser therefore sees requests as going to the same frontend origin. This keeps the authentication cookie **first-party** and avoids most cross-origin cookie/CORS complexity. The deployment documentation explicitly describes this as the reason the frontend can communicate with the backend without needing normal cross-origin configuration.

### Interview question: Why Next.js if the backend is Express?

Next.js is being used primarily for the **frontend application structure**: App Router routing, dynamic routes such as `/listing/[id]`, navigation APIs, layouts, query parameters and deployment. Business APIs remain in Express.

This is not a purely server-rendered Next.js application. Many pages are explicitly client components because they contain interactive state, effects and browser APIs.

---

# 3. Database Design

There are five important MongoDB models:

```
User
Listing
Favorite
Conversation
Message
```

### User

A user stores identity/profile information such as name, email, password hash, hostel, department and year. The password field has `select: false`, so ordinary user queries do not accidentally return password hashes. A `toPublic()` method explicitly constructs the safe user object returned to the frontend.

### Listing

A listing contains:

```
title
description
price
category
condition
images[]
seller → User ObjectId
status → active | sold
timestamps
```

The seller is stored as a reference to the User collection rather than embedding the whole user document. Mongoose's `populate()` is then used when seller information is actually required.

### Favorite

Instead of putting an array of listing IDs inside every user, favorites are modeled as a separate relation:

```
User ← Favorite → Listing
```

A compound unique index on `{ user, listing }` guarantees that the same user cannot favorite the same listing twice.

### Conversation and Message

A conversation represents **who is talking about which listing**:

```
listing
buyer
seller
lastMessageAt
lastMessageBody
```

The individual messages live separately and reference the conversation. A unique compound index on `(listing, buyer)` ensures one buyer gets one conversation per listing rather than creating a fresh thread every time they send a message.

**Why separate Conversation and Message?** Because one conversation can contain an arbitrary number of messages. Keeping messages separate prevents the conversation document from continually growing and makes messages easy to query chronologically.

`lastMessageBody` and `lastMessageAt` are deliberately also stored on the Conversation. This is a small amount of duplicated data, but it makes the inbox cheap to render and sort without first finding the newest Message document for every conversation.

---

# 4. Authentication Flow

Authentication is one of the most interviewable parts of Relay.

### Registration

When a user registers:

```
React form
→ POST /api/auth/register
→ server validates fields
→ checks whether email already exists
→ bcrypt hashes password
→ User saved to MongoDB
→ JWT created
→ JWT placed in cookie
→ public user object returned
```

The password is hashed with bcrypt before storage. A JWT containing the user's ID is then signed and returned inside the `token` cookie.

### Login

Login retrieves the user including the normally-hidden password hash and runs:

```jsx
bcrypt.compare(password, user.password)
```

If valid, another signed JWT is placed in the cookie.

### Why store JWT in a cookie instead of localStorage?

The cookie is configured as:

```
httpOnly: true
sameSite: lax
secure: true in production
7-day expiry
```

`httpOnly` means client-side JavaScript cannot directly read the token, which reduces token theft through XSS. `secure` makes it HTTPS-only in production, while `SameSite=Lax` provides some protection against cross-site request scenarios.

The frontend's `api()` helper always uses:

```jsx
credentials: "include"
```

so the browser automatically includes the authentication cookie.

### How are protected APIs protected?

This is important:

**The React frontend is not the real security boundary.**

Express has `requireAuth` middleware which verifies the JWT, extracts its user ID and stores it as:

```jsx
req.userId
```

Invalid or missing authentication receives HTTP 401. `optionalAuth` performs the same check without rejecting anonymous users, which is useful for public pages that can show additional information when someone happens to be logged in.

On the frontend, `useRequireAuth()` merely improves UX by redirecting anonymous users to login while preserving their intended destination.

**Interview trap:** If asked, “Can I bypass `useRequireAuth()` and edit someone else's listing?” the answer is **no**, because the backend independently checks both authentication and ownership.

---

# 5. Global Authentication State

Relay uses React Context for authentication.

When the application mounts, the global `Providers` component calls:

```
GET /api/auth/me
```

The server checks the cookie and returns either the current user or `null`. The result is stored in the AuthContext and exposed through `useAuth()`. Login can therefore call `setUser(user)`, while logout clears both the cookie and client state.

The same Provider component also manages toast notifications and theme state.

### Likely question: Why Context here?

Authentication is required in unrelated parts of the component tree: navbar, listing actions, dashboard, login flow and protected pages. Passing the user manually through each intermediate component would create unnecessary prop drilling.

---

# 6. Creating and Editing Listings

The listing form handles both creation and editing.

It stores normal form fields with React state and keeps two separate image collections:

```
existingImages → already stored on server
newFiles       → newly selected File objects
```

For newly selected images, `URL.createObjectURL()` creates temporary local preview URLs so the user sees the picture before uploading it. Those object URLs are revoked when an image is removed.

### Why FormData?

Images are binary files, so the form cannot simply send JSON.

The component builds:

```jsx
const form = new FormData();
```

and appends text fields plus every image file. The same reusable form performs either:

```
POST /api/listings       → create
PUT /api/listings/:id    → edit
```

depending on whether `listingId` exists.

The generic API helper deliberately does **not** manually set `Content-Type` when the body is `FormData`, allowing the browser to generate the correct multipart boundary.

### Image upload implementation

The Express backend uses **Multer**. It allows at most four images, each up to 5 MB, and only accepted image MIME types. Locally, files are stored on disk with randomized filenames. If `CLOUDINARY_URL` is configured, the same upload pipeline instead stores them in Cloudinary.

Validation occurs on **both frontend and backend**. Frontend validation improves UX, but backend validation is authoritative because requests can bypass the frontend. The backend validates title, description, price, category and condition before inserting the listing.

There is also thoughtful cleanup logic: Multer writes files before route validation finishes, so rejected requests remove those already-uploaded files when using local storage.

---

# 7. Browse, Search, Filtering and Pagination

The Browse page stores filters in the **URL query string**:

```
/browse?q=calculator
       &category=Electronics
       &min=200
       &max=1000
       &sort=price-asc
       &page=2
```

This is better than keeping everything only in component state because the search becomes refreshable and shareable.

The frontend reads these values using `useSearchParams()` and updates them using `router.replace()`. Every relevant change triggers the listings request.

On the backend, the route builds a Mongoose query dynamically:

- category and condition become exact filters;
- minimum/maximum prices become `$gte` and `$lte`;
- search checks title or description;
- a predefined map chooses sorting;
- `skip()` and `limit()` implement pagination;
- `countDocuments()` calculates the total number of pages.

`Promise.all()` obtains the page of listings and total count concurrently rather than waiting for one query before starting the other.

### Interesting implementation detail: stale requests

The Browse frontend increments a `requestRef` for each request. A response updates state only if its ID still matches the newest request.

That prevents this:

```
search "book"       → slow request
search "calculator" → fast request returns
old "book" response arrives afterward
```

from incorrectly replacing the newer results. The implementation is visible in the browse-page request handling.

### Good interviewer follow-up

**“You have a MongoDB text index. Are you actually using it?”**

Not in the current search route. The schema defines a text index on title and description, but the active search implementation uses escaped case-insensitive regex queries.

For a larger dataset, improving search would be a reasonable next step.

---

# 8. Favorites

Favoriting is intentionally simple.

```
POST /api/favorites/:listingId
```

acts as a **toggle**:

```
favorite exists    → delete it
favorite absent    → create it
```

The unique `(user, listing)` database index guarantees duplicates cannot exist. The route even catches MongoDB duplicate-key error `11000`, which protects against two concurrent attempts racing to create the same favorite.

This is a good example if asked about **database constraints vs frontend checks**: the database constraint is the final guarantee.

---

# 9. Messaging System

Relay's messaging is **not WebSocket-based**. That distinction is important.

When a buyer messages a seller:

```
POST /api/conversations
→ validate listing
→ prevent seller messaging themselves
→ find existing conversation for this buyer + listing
→ create one if absent
→ create Message
→ update conversation preview information
```

The server explicitly reuses an existing conversation rather than creating a duplicate.

Only the buyer and seller are allowed to retrieve or post messages in that conversation. The backend checks the authenticated user's ID against the conversation participants before returning data.

### How are new messages received?

The frontend uses **polling every five seconds**:

```jsx
const POLL_MS = 5000;
setInterval(load, POLL_MS);
```

The interval is cleaned up when the component unmounts.

This is simpler than WebSockets and perfectly understandable for a small campus marketplace, but it makes a very good interview tradeoff question:

> For a larger real-time chat system, I would replace polling with WebSockets or another push-based mechanism so clients receive new messages immediately without repeatedly querying the server.
> 

---

# 10. Dashboard and Listing Ownership

The seller dashboard requests:

```
/api/listings?seller=me&status=all
```

and derives active listings, sold listings and total earned on the client. Sellers can edit, delete, mark an item sold or relist it. After a successful status/update API call, React updates the relevant item locally rather than re-fetching the entire dashboard.

The backend never trusts the UI's claim of ownership. Before status updates, edits or deletion, it verifies:

```jsx
listing.seller.toString() === req.userId
```

Unauthorized users receive HTTP 403.

### What happens when a listing is deleted?

MongoDB references do not automatically behave like SQL `ON DELETE CASCADE` here. Relay performs manual cleanup:

```
delete listing images
delete its messages
delete its conversations
delete its favorites
delete listing
```

That is an excellent implementation detail to know because it shows you understand **referential cleanup in MongoDB**.

---

# 11. Deployment

The intended production architecture is:

```
Vercel
  → Next.js frontend

Render
  → Express backend

MongoDB Atlas
  → persistent database

Cloudinary
  → optional persistent image storage
```

The project deliberately supports local development without Atlas by starting an embedded MongoDB instance using `mongodb-memory-server`; production uses `MONGODB_URI` instead.

Local disk uploads work during development, but the repository explicitly notes that disk storage on many free hosts does not survive restarts, which is why Cloudinary support exists.

---

# 12. Questions You Should Be Ready For

### “Walk me through login from browser to database.”

Explain React form → API helper → Next rewrite → Express login route → MongoDB user lookup → bcrypt comparison → JWT generation → httpOnly cookie → AuthContext update.

### “How do you protect private routes?”

Mention both layers. `useRequireAuth()` handles frontend redirection and UX; `requireAuth` middleware verifies the JWT server-side. **Backend middleware is what actually provides security.**

### “How do listings with images work?”

Explain `FormData` → multipart request → Multer → validation → disk/Cloudinary → image URLs stored in MongoDB.

### “Why MongoDB/Mongoose?”

The application's entities map naturally to documents, while ObjectId references support relationships such as seller/listing and conversation/messages. Mongoose supplies schemas, validation, indexes, query APIs and `populate()`.

### “How does search work?”

URL query parameters on the client become dynamically constructed Mongoose filters on the server, with regex search, category/condition filters, price ranges, sorting and `skip/limit` pagination.

### “How is chat real-time?”

**It isn't truly real-time.** It polls the conversation endpoint every five seconds. This is simpler but less scalable than WebSockets.

### “What prevents two conversations between the same buyer and seller?”

More precisely, the restriction is **one conversation per buyer per listing**. The `(listing, buyer)` unique index enforces that at the database level.

### “How do you prevent someone from modifying another seller's product?”

JWT middleware establishes `req.userId`, and every sensitive listing operation compares it with the listing's stored seller ID before modification.

### “What would you improve if the application scaled?”

The strongest answers based on the existing implementation are:

- move messaging from polling to WebSockets;
- improve search beyond regex for larger datasets;
- rely on persistent object storage such as Cloudinary rather than local disk;
- introduce stronger caching/pagination strategies as data grows;
- move appropriate Next.js pages/data fetching toward server components where doing so would reduce unnecessary client-side JavaScript.

---

## Final Mental Model

If you remember only one flow, remember this:

```
Next.js/React UI
        ↓
central api() helper
        ↓
Next.js rewrite
        ↓
Express REST route
        ↓
auth/validation middleware
        ↓
Mongoose model/query
        ↓
MongoDB
        ↓
JSON response
        ↓
React state/context updates UI
```

And remember the five feature stories:

**Auth:** bcrypt + JWT + httpOnly cookie.

**Marketplace:** Mongoose filtering + URL query parameters + pagination.

**Listings:** FormData + Multer + MongoDB + optional Cloudinary.

**Messaging:** Conversation + Message models + 5-second polling.

**Security:** server-side JWT verification + ownership/participant checks.

If you can explain those five properly and trace one request through the architecture, you know the part of Relay an interviewer is most likely to probe.