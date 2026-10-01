# Eventually — Interview Preparation Notes

*An exhaustive map of what an interviewer can ask about this project, and what to answer. Every section references the actual code. Read it once end to end, then rehearse §1 and §14 aloud until they're fluent — those two carry the interview.*

**Repository:** `souiiii/Eventually` · Node + Express 5 + MongoDB/Mongoose + EJS · role-based event management for a college.

---

## §1 · The 90-second pitch

*The opening question is always some version of "tell me about this project." Have this rehearsed. Do not read the README aloud — that reads as memorised.*

> "Eventually is a college event management system with two roles. Admins create and edit events, view who's registered, and mark attendance live at the venue. Students browse upcoming events, register subject to capacity and deadline, and get a unique registration code that acts as their ticket.
> 
> 
> It's a server-rendered Express app — EJS templates rather than a SPA, because every page is content the server already has and I wanted the pages to arrive complete rather than fetch after load. MongoDB with three collections: users, events, and a registrations collection that joins them.
> 
> The parts I found most interesting were the concurrency around capacity limits, generating collision-free registration codes, and making attendance marking safe to retry — an admin scanning codes at a door will inevitably double-submit."
> 

Then stop. Let them pick the thread.

---

## §2 · Architecture and stack decisions

**"Why server-side rendering instead of React?"**

The honest answer, and it's a good one because you've built both:

> "Every page here is data the server already has — an event list, a registration table. There's no client-side state worth managing, no offline behaviour, no partial updates. SSR means the page arrives complete, there's no loading spinner, and I don't ship a framework to render static content. My other project, Velvet, is a React SPA because it has genuine client state — file selection, upload progress, panel layout — and that's where the framework pays for itself."
> 

**"Walk me through the layers."**

```
app.js                 entry: middleware chain, route mounting, DB connect, listen
connection.js          mongoose.connect wrapper
middlewares/user.js    checkAuth (global, soft) · checkAuthorization(roles) (per-route, hard)
routes/                common · user · events · dashboard · admin
services/              auth (JWT) · registration (codes, register flows) · message (redirect helpers)
models/                User · Event · Registration
views/                 EJS templates, partials for header/navbar/footer
public/                CSS and images, served statically
```

**"Why a services layer?"** Because `registerNew` and `registerAgain` contain retry logic and error classification that has nothing to do with HTTP. Keeping it out of the route handler means the route reads as *policy* — check deadline, check capacity, then register — while the service handles *mechanism*. It's also the code most likely to be reused or tested independently.

**"Explain your middleware order."** From `app.js`:

```jsx
app.use(json());                  // parse JSON bodies
app.use(urlencoded({ extended: false }));  // parse form posts
app.use(cookieParser());          // populate req.cookies
app.use(express.static("public"));// serve assets without touching routes
app.use(checkAuth);               // decode JWT into req.user, or leave null
```

Order matters and you should be able to say why: **`cookieParser` must precede `checkAuth`**, because `checkAuth` reads `req.cookies?.uid`. **`express.static` sits early** so asset requests short-circuit before auth work happens. Body parsers precede routes because handlers read `req.body`.

**"Why is `checkAuth` global but `checkAuthorization` per-route?"**

> "They answer different questions. `checkAuth` answers *who is this*, and it never rejects — an anonymous visitor just gets `req.user = null`, which the homepage and login pages need. `checkAuthorization` answers *are they allowed here*, which is route-specific. So identity is ambient, permission is explicit."
> 

That's a genuinely clean separation and worth stating in those terms.

---

## §3 · The data model

Three collections. The interesting one is the third.

**User** (`models/User.js`) — `fullName`, `email` (unique), `password` (hashed), `role` (enum `ADMIN`/`STUDENT`, default `STUDENT`), timestamps.

**Event** (`models/Event.js`) — title, two descriptions, organiser, venue, `capacity` (min 1), `deadline`, `startTime`, `endTime`, author/position with defaults, timestamps.

**Registration** (`models/Registration.js`) — `userId` ref user, `eventId` ref event, `registrationCode` (unique), `status` (`REGISTERED`/`CANCELLED`), `attendanceStatus` (`PENDING`/`ATTENDED`), timestamps.

**"Why is Registration its own collection instead of an array on Event or User?"**

This is the schema-design question, and there's a strong answer:

> "It's a many-to-many relationship — a user registers for many events, an event has many registrants — and MongoDB has no native join table, so the relationship needs its own documents. Embedding fails in both directions: an array of registrants on Event grows unboundedly toward the 16 MB document limit for a popular event, and an array of events on User means every capacity count has to scan every user. Registration as its own collection also lets the relationship *carry its own state* — status, attendance, the code — which an array of ObjectIds can't."
> 

That last point is the one that separates a real answer from a memorised one. **The join carries data**, so it must be an entity.

**"Why store `capacity` on Event but not a `registeredCount`?"**

Honest answer with the tradeoff named:

> "I derive the count with `countDocuments` rather than maintaining a counter, so there's no chance of the counter drifting from reality. The cost is a query every time I need the number, and — more seriously — it's why my capacity check isn't atomic. If I rebuilt this I'd keep a counter on the event and increment it conditionally in the same operation, which is exactly what I did for storage quota in my other project."
> 

That flows straight into §6, which is the best part of this project to discuss.

**"Why does the JWT payload carry `role` if you re-fetch the user anyway?"** Convenience for rendering — templates read `user.role` to decide what to show. But **authorisation never trusts it**: `checkAuthorization` fetches the user and checks `validUser.role`. Say that explicitly, because the alternative — trusting a role claim in a token — is a real vulnerability, and knowing the difference is the point.

---

## §4 · Indexing strategy

`Registration` carries five indexes, and each defends a specific query. Be ready to justify all of them:

```jsx
{ userId: 1, eventId: 1 }              // unique — one registration per user per event
{ eventId: 1, status: 1 }              // capacity counts, admin registration list
{ userId: 1, status: 1 }               // student dashboard: registered / cancelled
{ eventId: 1, attendanceStatus: 1 }    // attendance views
{ registrationCode: 1, eventId: 1 }    // unique — attendance lookup by code
```

**"Why is the first one unique?"** This is the strongest single point in the schema:

> "It enforces a business rule at the database level rather than in application code. Even if two requests race past my application-level check, the second insert fails with duplicate key error 11000. The database is the last line of defence, and it's the only one that can't be raced."
> 

**"Why compound rather than separate single-field indexes?"** Because of the **prefix rule** — an index on `{userId, eventId}` serves queries on `userId` alone and on both fields together, but not on `eventId` alone. Compound indexes are ordered, and only leading prefixes are usable. That's why `{eventId, status}` exists separately: the dashboard filters on `userId + status`, the capacity count on `eventId + status`, and neither compound serves the other.

**"What's the cost?"** Every index must be updated on every insert, update, and delete, and each consumes RAM and disk. Five indexes on a write-heavy collection is a real write tax. Justification: registrations are read far more than written — every event page, every dashboard load, every capacity check reads them; a registration is written once.

**"Is `registrationCode` unique globally or per event?"** Both, and that's worth noticing: the field carries `unique: true` in the schema *and* appears in a compound unique index with `eventId`. Globally unique is the stronger constraint, so the compound one is arguably redundant. Volunteer that — noticing redundancy in your own schema reads well.

---

## §5 · Authentication and authorisation

**The flow**, end to end, which you should be able to narrate without notes:

1. **Signup** (`routes/user.js`) — validate presence, then types, then trim and lowercase, then `validator.isEmail`, then a password regex requiring 8+ chars with upper, lower and digit. Hash with **bcrypt, 10 rounds**. Create the user. Duplicate email surfaces as error 11000 → 409.
2. **Login** — same validation, `User.findOne({ email })`, `bcrypt.compare`. On success sign a JWT with `{ _id, fullName, role }`, one-day expiry, and set it as cookie `uid` with `httpOnly: true, sameSite: "strict"`.
3. **Every request** — `checkAuth` reads the cookie, verifies the signature, and attaches `req.user`, or nulls it and clears the cookie on any failure.
4. **Protected routes** — `checkAuthorization([...roles])` validates the shape of `req.user`, checks the ObjectId is well-formed, **fetches the user from the database**, and compares `validUser.role` against the allowed list. 401 redirect if unauthenticated, 403 if authenticated but wrong role.
5. **Logout** — `res.clearCookie("uid", ...)`.

**"Why bcrypt and not SHA-256?"** Because bcrypt is **deliberately slow** and **salted per user**. Fast hashes are exactly wrong for passwords: a GPU can compute billions of SHA-256 hashes per second. The salt kills rainbow tables; the cost factor makes brute force expensive. You never decrypt — you `compare`.

**"Why `httpOnly`?"** JavaScript cannot read the cookie, so a successful XSS injection can't exfiltrate the session token. The tradeoff is that the browser sends it automatically, which opens CSRF — mitigated here by `sameSite: "strict"`, which stops the cookie travelling on cross-site requests entirely.

**"Why hit the database on every authorised request?"**

The tradeoff question, and the answer is the same shape as Velvet's:

> "A signed token proves what was true when it was issued. It can't tell me the account still exists, or that the role hasn't changed, or that the user hasn't been removed. Since role decides what an admin can do, I verify against the database rather than trusting a day-old claim. The cost is a read per protected request, which makes a stateless token effectively stateful. At scale I'd cache the user in Redis with a short TTL — one read per user per minute instead of one per request — or move to short-lived access tokens with a refresh endpoint as the checkpoint."
> 

**"What's missing from your cookie configuration?"** Volunteer it: **the `secure` flag**. Without it the cookie travels over plain HTTP if the site is ever served that way. Velvet sets `secure: process.env.NODE_ENV === "production"`; Eventually doesn't. Knowing your own inconsistency is a good look.

**"How does `checkAuthorization` work as a function?"** It's a **middleware factory** — a function returning a middleware, so the role list is captured in a closure:

```jsx
app.use("/admin", checkAuthorization(["ADMIN"]), adminRouter);
```

If asked to explain closures, this is your concrete example: `roles` isn't a parameter of the middleware, it's captured from the enclosing scope and lives as long as the returned function does.

---

## §6 · The registration flow — the deep dive

*This is the richest area in the project. Expect the interviewer to live here if they're any good.*

**The happy path** (`routes/events.js`, `POST /register/:id`):

```
validate ObjectId
→ fetch event
→ check deadline hasn't passed
→ check for existing registration (REGISTERED → reject; CANCELLED → allow re-register)
→ countDocuments({eventId, status: REGISTERED}) and compare to capacity
→ registerNew or registerAgain
```

### The race condition — own this before they find it

**There is a TOCTOU (time-of-check to time-of-use) bug in the capacity check**, and the strongest thing you can do is raise it yourself:

> "The capacity check isn't atomic. I count registrations, then insert — two separate operations. If the event has one seat left and two students submit simultaneously, both counts return 'capacity-1', both pass the check, and both insert. The event goes over capacity.
> 
> 
> The compound unique index protects against a *user* double-registering, but nothing protects the *capacity* invariant, because it's a property of the collection rather than of any one document."
> 

Then the fix, and this is where you connect it to your other project:

> "The fix is to make the check and the write one atomic operation. In Velvet I hit exactly this with storage quotas and solved it by putting the condition in the filter of a `findOneAndUpdate` and using `$inc` for the change — `{ _id, storageUsed: { $lte: MAX - size } }` with `{ $inc: { storageUsed: size } }`. MongoDB guarantees atomicity on a single document, so the check and the increment can't be interleaved. Here I'd add a `registeredCount` field to Event and do the same: `findOneAndUpdate({ _id: eventId, registeredCount: { $lt: capacity } }, { $inc: { registeredCount: 1 } })`. If it returns null, the event is full — no race, no separate count."
> 

That is an unusually strong answer: a named concurrency bug in your own code, the mechanism behind it, and a working fix you've already implemented elsewhere. Most candidates cannot produce a race condition from their own project at all.

**Related follow-up: "Isn't the deadline check racy too?"** Technically yes, but the window is milliseconds and the consequence is one late registration — far less serious than breaching capacity. Being able to rank your own bugs by impact is itself a signal.

### Why two functions, `registerNew` and `registerAgain`?

Because they're different operations against the database: one **inserts**, one **updates an existing CANCELLED row back to REGISTERED**. The compound unique index makes this necessary — a cancelled registration still occupies the `{userId, eventId}` slot, so re-registering can't insert a second document. It must revive the old one.

**"Why not delete on cancel instead?"** Then re-registration would be a clean insert. But you'd lose the history — you couldn't show a cancelled-events dashboard, and you couldn't audit who cancelled what. **Status transitions preserve history; deletes destroy it.** That's a design choice you can defend.

---

## §7 · Registration codes and collision handling

```jsx
export function generateCode(length) {
  const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  const bytes = crypto.randomBytes(length);
  for (let i = 0; i < length; i++) result += chars[bytes[i] % chars.length];
  return result;
}
```

**"Why `crypto.randomBytes` and not `Math.random`?"**

> "`Math.random` is a pseudo-random generator seeded from predictable state — it's fine for shuffling a UI, not for anything guessable-matters. The registration code is effectively a ticket: if someone can predict codes, they can mark attendance for a student who never showed. `crypto.randomBytes` is cryptographically secure."
> 

**"How likely is a collision?"** 36⁹ ≈ 1.0 × 10¹⁴ possible codes. For realistic registration volumes the probability is negligible — but *negligible isn't zero*, which is why the retry loop exists.

**"Explain the retry loop."**

```jsx
for (let i = 0; i < 10; i++) {
  try { ... await Registration.create({...}); return; }
  catch (err) {
    if (err?.code === 11000) {
      if (err.keyPattern?.userId && err.keyPattern?.eventId) throw new Error("Already registered");
      continue;
    }
    throw err;
  }
}
```

The interesting part is **error classification**. Error 11000 is a duplicate key, but *which* key matters enormously:

- Duplicate `{userId, eventId}` → the user is already registered → **fail permanently**, retrying will never help
- Duplicate `registrationCode` → an astronomically unlucky collision → **retry with a new code**

`err.keyPattern` tells you which index rejected the write, and the code branches on it. That's a genuinely sophisticated bit of error handling and worth pointing at.

**"Why 10 attempts?"** Bounded retry: a loop that retries forever turns a persistent failure into a hung request. Ten attempts against a 10⁻¹⁴ collision probability is effectively infinite headroom while still terminating.

**A flaw to volunteer:** `bytes[i] % chars.length` introduces **modulo bias**. A byte is 0–255, and 256 isn't divisible by 36 — values 0–3 map to four extra characters, so `A`–`D` are marginally more likely than the rest. The bias is tiny and irrelevant at this scale, but knowing it exists is a real cryptographic detail. The unbiased fix is rejection sampling: discard bytes ≥ 252.

**Second flaw:** in `registerAgain`, the result of `findOneAndUpdate` is assigned to `registration` and never checked. If no CANCELLED registration matched, it returns `null` and the function reports success regardless. The route guards against this by checking first, but the service shouldn't depend on its caller.

---

## §8 · Attendance marking — the part you got right

`routes/admin.js`, `POST /mark-attendance/:eventId`:

```jsx
const registration = await Registration.findOneAndUpdate(
  { attendanceStatus: "PENDING", status: "REGISTERED", eventId, registrationCode: finalCode },
  { $set: { attendanceStatus: "ATTENDED" } },
  { runValidators: true, new: true }
).populate("userId", "_id fullName");
```

**"Why is this one atomic when the capacity check isn't?"** Because every condition lives in the **filter** of a single document operation. Four things are verified — the registration exists, belongs to this event, matches the code, is REGISTERED, and is still PENDING — and the write happens in the same indivisible step. Nothing can interleave.

**"What happens if an admin scans the same code twice?"** The second call finds no document matching `attendanceStatus: "PENDING"` — it's now ATTENDED — so it returns `null` and the route reports "already marked present." **This is idempotent in effect**: double submission can't corrupt state. That matters in the real deployment, where an admin is scanning codes at a door on a flaky connection and *will* double-submit.

**"Why check `startTime <= now <= endTime`?"** Attendance is only meaningful during the event. It prevents an admin marking attendance days early or long after.

**Also worth mentioning:** `{ runValidators: true }`. Mongoose validators fire on `save()` but **not on update operations** by default, so schema rules like the `attendanceStatus` enum would otherwise be skipped entirely on this write. Most people don't know that, and volunteering it is a strong signal.

**And the code format.** Codes are stored as 9 characters, displayed as `XXX-XXX-XXX` (11 with hyphens) via `.match(/.{1,3}/g).join('-')` in the view. The route strips hyphens and uppercases before lookup, so admins can type it in either form. **Store canonical, display formatted, normalise on input** — that's the principle.

---

## §9 · Query patterns and performance

**"Why `Promise.all` here?"** In `/discover` and `/events/:id`:

```jsx
const [events, registeredEvents] = await Promise.all([
  Event.find({ endTime: { $gt: now } }).sort(sortObject).lean(),
  Registration.find({ userId: req.user._id }).lean(),
]);
```

The two queries are independent, so awaiting them sequentially would take the sum of their latencies; `Promise.all` makes it the maximum. Standard, but be ready to state *why* — and to note the trap: **`await` inside a loop runs sequentially**, and independent work should always be `Promise.all`.

**"Why `.lean()` everywhere?"** Mongoose normally returns hydrated documents carrying `save()`, getters, virtuals and change tracking. These are read-only render paths — none of that is needed, and hydration costs memory and time. `.lean()` returns plain objects and is measurably faster. Using it deliberately on read paths is a small mark of care.

**"How does sorting work, and why the Map?"**

```jsx
const sortParameter = new Map();
sortParameter.set("nearest", "startTime");
sortParameter.set("deadline", "deadline");
...
const field = sortParameter.get(sortt) ?? "startTime";
const sortObject = { [field]: direction };
```

This is a **whitelist**, and the security reason is the answer:

> "The sort field comes from the query string. If I passed it straight into the sort object, a user could sort by any field in the collection — including ones I don't index, which is a cheap denial of service, or fields I don't intend to expose ordering on. Mapping a small set of allowed keys to actual field names means unrecognised input falls back to a default instead of reaching the database."
> 

Same principle as parameterised queries: **never let user input become a database identifier.**

**"What's the N+1 problem, and where does it appear here?"** `.populate()` issues a **second query** and stitches results in Node — it's not a server-side join. In `admin/student-registrations` you populate both `userId` and `eventId`, so one page load is three round trips. The server-side equivalent is `$lookup` in an aggregation pipeline, which does the join in one operation. For a single page with a bounded result set, populate is fine and far more readable; inside a loop it becomes N+1 and you'd switch.

**Inefficiency worth volunteering:** in `GET /events/:id` you fetch *every* registration for the event just to count the REGISTERED ones with a `reduce`. `countDocuments({ eventId, status: "REGISTERED" })` does that in the database and returns a number instead of a document array. On a 500-person event you're pulling 500 documents to compute one integer.

**"Why no pagination anywhere?"** Honest answer: college-scale data, so lists are naturally bounded. But it's the first thing that breaks at scale — an unbounded `find()` on a large collection will eventually time out or exhaust memory. You'd add offset pagination for numbered pages, or cursor-based (`_id > lastSeen`) for infinite scroll, since `skip` degrades as the offset grows.

**The dashboard's in-memory re-sort.** In `routes/dashboard.js` you sort in MongoDB, then re-sort in JavaScript to float *live* events to the top. Why not in the database? Because "live" depends on `now` compared against fields on the **populated** event, which the initial query can't see. You could do it with an aggregation pipeline using `$lookup` and `$addFields` — that would be the scalable version.

**The defensive filter.** `.filter((r) => r.eventId)` after populate discards registrations whose event no longer exists. Deleting an event does cascade `Registration.deleteMany`, so orphans shouldn't occur — but the two deletes aren't transactional, so a failure between them leaves orphans. The filter means a stale row degrades to an omitted list item rather than a crash on `r.eventId.title`.

---

## §10 · Validation and error handling

**"Why validate on the server when the HTML form already has `required`?"** Because client-side validation is a **user experience feature, not a security control**. Anyone can post directly to the endpoint with curl or Postman. Every check must exist server-side; the client version just avoids a round trip for honest users.

**"Walk me through your validation order."** Presence → type → normalise (trim, lowercase) → format (`validator.isEmail`, `isISO8601`) → business rules (`startTime < endTime`, `deadline <= startTime`, capacity a positive integer, dates not in the past). **Cheap checks first, expensive checks last** — no reason to run a regex on a field that's missing.

**"Why `typeof body.email !== 'string'`?"** Because JSON bodies can contain anything. `{"email": {"$gt": ""}}` is valid JSON, and if it reached a Mongo query unchecked it would be a **NoSQL injection** — an operator object where a string was expected. Type checking closes it. (Mongoose's schema casting also helps, but defence in depth is right.)

**"How do you surface errors?"** Two patterns, and it's worth naming the difference:

- **Re-render with an error** — signup and login re-render the same view with `error` and the submitted `body`, so the user doesn't retype everything
- **Redirect with a query param** — `redirectWithSuccess` / `redirectWithError` in `services/message.js` encode the message into the URL

**"Why redirect rather than render on success?"** The **POST/Redirect/GET** pattern: after a successful POST, redirecting means the browser's final navigation is a GET, so a refresh doesn't resubmit the form. Without it, refreshing after registering would attempt to register again. That's the answer they're looking for, and few candidates name the pattern.

**A flaw to acknowledge:** `views/common/server-error.ejs` is an empty file. Every 500 path renders a blank page. Easy fix, and noticing it shows you've read your own code recently.

---

## §11 · Time and time zones

An area with more depth than it looks.

**"How do you store dates?"** As JS `Date` objects, which Mongo stores as BSON dates in **UTC**. Storage is unambiguous; only display is local.

**"How do you display them?"** The views emit `<span class="client-local-time" data-iso="...">` and let the **browser** format the ISO string in the user's own zone. That's the right architecture: the server has no reliable way to know the viewer's zone, and the browser knows it exactly.

**"What's `toDatetimeLocalUTC` for?"** HTML `<input type="datetime-local">` needs a `YYYY-MM-DDTHH:mm` string with no zone suffix, so editing an event requires converting a Date into that format. Worth volunteering: **the function is misnamed** — it uses `getFullYear`, `getMonth`, `getHours` and so on, which are *local* getters, not UTC. And the edit POST handler uses a different conversion, `d.toISOString().slice(0, 16)`, which *is* UTC. **Two different conversions for the same job, and they disagree by the server's UTC offset.** That's a real bug worth naming before they find it.

**"What breaks if the server and user are in different zones?"** Comparisons like `event.deadline < now` are fine — both are absolute instants. The problems are all at the presentation boundary: an admin in IST editing an event on a UTC server would see times shifted by five and a half hours in the edit form.

---

## §12 · Security posture

Walk this as a checklist — what's defended, what isn't.

**Defended:**

- **Password storage** — bcrypt, 10 rounds, per-user salt
- **XSS** — EJS `<%= %>` escapes HTML by default; `<%- %>` (unescaped) is used only for including partials, never for user data. Say that explicitly, because it's the distinction that matters.
- **Token theft via XSS** — `httpOnly` cookie
- **CSRF** — `sameSite: "strict"` means the cookie isn't sent on cross-site requests at all
- **NoSQL injection** — explicit `typeof` checks plus Mongoose casting
- **Sort-field injection** — Map whitelist
- **Privilege escalation** — role is re-read from the database, never trusted from the token
- **IDOR on ObjectIds** — `mongoose.Types.ObjectId.isValid` before every lookup

**Not defended, and worth volunteering:**

- **No `secure` cookie flag** — the cookie can travel over plain HTTP
- **No CSRF tokens** — `sameSite: strict` covers most of it, but a token per form is the belt-and-braces answer, and older browsers ignore `sameSite`
- **No rate limiting** — the login endpoint can be brute-forced. The fix is a Redis-backed limiter keyed by IP and email, returning 429; token bucket if you want to allow bursts
- **No account lockout or CAPTCHA**
- **No `helmet`** — a dozen security headers in one line, absent here
- **Registration codes are not tied to identity at scan time** — anyone holding a code can be marked present. Acceptable for the threat model (it's a college event), and being able to say "acceptable for this threat model" is better than pretending it's secure

**"Is your admin route protected?"** Yes — `app.use("/admin", checkAuthorization(["ADMIN"]), adminRouter)` mounts the guard at the router level, so every admin route inherits it and there's no way to add a new one and forget the check. Guarding at the mount point rather than per-handler is the safer pattern, and it's worth saying so.

---

## §13 · Known weaknesses — volunteer these

*Naming your own flaws, with fixes, is the single strongest move available in a project interview. It reads as engineering maturity rather than defensiveness. Pick two or three; don't recite all of them.*

1. **Capacity race condition** — count-then-insert isn't atomic. Fix: `registeredCount` on Event with a conditional `$inc`. *(The best one. Lead with it.)*
2. **Event deletion isn't transactional** — `findByIdAndDelete` then `deleteMany` are two operations; a failure between them orphans registrations. Fix: a MongoDB multi-document transaction, or soft-delete the event and filter it out.
3. **Two conflicting date conversions** — `toDatetimeLocalUTC` uses local getters, the edit handler uses `toISOString`. Pick one.
4. **`registerAgain` ignores its return value** — a `null` result reports success.
5. **`GET /events/:id` fetches all registrations to compute one count** — should be `countDocuments`.
6. **Empty `server-error.ejs`** — every 500 renders blank.
7. **Auth hits the database on every protected request** — correct, but expensive. Fix: Redis cache with a short TTL, or refresh tokens.
8. **No pagination** — fine at college scale, first thing to break beyond it.
9. **Modulo bias in code generation** — real, negligible here, fixable with rejection sampling.
10. **`registrationCode` is both globally unique and in a compound unique index** — the compound is redundant.

---

## §14 · "What breaks at 10× traffic?"

*Rehearse this. It's the standard scaling question and you have the material for a strong answer.*

> "First I'd measure rather than guess, but the predictable failures are:
> 
> 
> **Authorisation**, because every protected request does a `User.findById`. I'd cache the user in Redis with a short TTL — one read per user per minute rather than per request — and delete the key on role change so admin revocation is still immediate.
> 
> **Registration**, because the capacity race that's theoretical at college scale becomes routine when hundreds register at once. That has to become an atomic conditional `$inc`.
> 
> **The event listing**, because `find()` with no limit returns everything. Pagination, and an index on `endTime` since that's the filter field.
> 
> **The event details page**, which pulls every registration to compute one count — that becomes `countDocuments`, or a cached counter.
> 
> Structurally: multiple Node instances behind a load balancer with PM2 cluster mode, since a single Node process is one thread. That's already safe here because the app is stateless — the session lives in a signed cookie, not in server memory. Static assets to a CDN. And rate limiting, so one abusive client can't consume the capacity I just added."
> 

---

## §15 · Comparative questions

You have two Node projects, and interviewers will probe the differences. That's an opportunity.

**"Why EJS here and React in Velvet?"** — §2. Content-driven pages versus genuine client state.

**"Which is better code, and why?"** Velvet's concurrency handling is better — the quota check is atomic where Eventually's capacity check isn't. Eventually's schema is more interesting — a real many-to-many with a join collection carrying state, plus a considered index strategy. **Say which one you'd rewrite and how**; that's the answer they want.

**"What did you learn between them?"** The honest and most valuable answer:

> "Eventually taught me the shape of the problem — I hit the capacity race and half-solved it with a unique index, which protects one invariant but not the other. By the time I built Velvet I understood that the check and the write have to be one operation, so the quota logic went into the filter of a `findOneAndUpdate` with `$inc`. Same class of bug, and the second time I recognised it before shipping."
> 

That is a genuine learning arc across two projects, and it's exactly what "tell me about a time you improved" is fishing for.

---

## §16 · Rapid-fire bank

Answer each in one or two sentences.

**Node/Express** — Why Express over raw `http`? · What does `next()` do? · Middleware execution order · Error-handling middleware signature · Why `express.static` early? · Why is Node single-threaded yet concurrent? · What does `dotenv` do and why aren't secrets in code?

**Mongo/Mongoose** — Mongoose vs the native driver · What `.lean()` gives up · `populate` vs `$lookup` · Why `runValidators` on updates · What `{ new: true }` returns without it · What error 11000 means · Compound index prefix rule · Embed vs reference

**Auth** — JWT structure · Why the payload is readable · What the signature proves · `httpOnly` vs `sameSite` vs `secure` · Session vs JWT · How you'd revoke a JWT · Why bcrypt is slow on purpose

**Concurrency** — What a race condition is · TOCTOU · Why single-document operations are atomic in Mongo · How a unique index enforces an invariant · What idempotency means and where it appears here

**Web** — POST/Redirect/GET · XSS and how EJS escaping defends it · CSRF and how `sameSite` defends it · 401 vs 403 · Why 429 exists

---

## How to use this

**Tonight or in the morning:** read once end to end, then rehearse **§1** (the pitch) and **§14** (10× traffic) aloud until they're fluent paragraphs rather than lists.

**Then pick three from §13** to volunteer — the capacity race, the transaction gap on deletion, and the date-conversion inconsistency are the strongest three, because each has a clear mechanism and a clear fix.

**The single best moment available in this interview** is §6: a named concurrency bug in your own code, the reason it happens, and a working fix you've already shipped in another project. Have that ready and steer toward it.