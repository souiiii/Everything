# Velvet — Interview Preparation Notes

*An exhaustive map of what an interviewer can ask about this project, and what to answer. Every section is anchored to real code. This is your flagship — it contains the single best technical story you have, in §6.*

**Repository:** `souiiii/Velvet` · React (Vite) + Express + MongoDB + Cloudinary · a secure file-sharing platform built around **controlled, expiring, revocable public links**.

---

## §1 · The 90-second pitch

*The opening question is always some version of "tell me about this project." Rehearse this. Don't recite the README.*

> "Velvet is a file-sharing platform, but the interesting part isn't the uploading — it's the link governance. When you share a file, you generate a public link that you can put a password on, give an expiry date, cap at a number of downloads, and revoke at any moment. Every one of those rules is enforced server-side on each download request, not just hidden in the UI.
> 
> 
> React on the front, Express and MongoDB behind it, with the actual bytes living in Cloudinary. Three collections: users, files, and links — a file can have many links with different rules.
> 
> The parts I found most interesting were the storage quota, which turned out to be a real concurrency problem, and the download route, which streams the file through the server rather than redirecting — because the moment you hand out the storage URL, every rule you just set becomes unenforceable."
> 

Then stop. That last sentence is bait, and a good interviewer will bite.

---

## §2 · Architecture

**"Walk me through the structure."**

```
index.js               Express entry: static client, cookies, soft auth, routers, SPA fallback
connection.js          mongoose.connect wrapper
middlewares/user.js    checkAuthSoft (global) · checkAuthHard (per-route)
utils/auth.js          setUser (sign) · getUser (verify)
models/                User · File · Link
routes/                user.js (auth) · file.js (uploads, links, downloads)
client/                React SPA — Vite, React 19, React Router v7
```

**"How are the frontend and backend deployed?"** As one unit. Express serves the built client:

```jsx
app.use(express.static(path.join(__dirname, "client", "dist")));
...
app.use((req, res) => res.sendFile(path.join(__dirname, "client", "dist", "index.html")));
```

**"Why the catch-all at the end?"** This is the **SPA fallback**, and it's worth explaining properly:

> "React Router handles routing on the client, so a URL like `/link/abc123` only exists in the browser. If a user refreshes that page, the browser asks the *server* for `/link/abc123` — which has no matching Express route. Without the fallback that's a 404. Sending `index.html` instead lets the SPA boot and resolve the route itself. It has to be registered last, after the API routers, or it would swallow the API requests too."
> 

**"And in development?"** Vite's dev proxy:

```jsx
server: { proxy: { "/api": "http://localhost:8000" } }
```

Same effect for a different reason — the browser only sees `localhost:5173`, so requests are **same-origin, no CORS, and the auth cookie stays first-party**. Third-party cookies are increasingly blocked, so an architecture that avoids them is more durable than one that fights for them.

**"Explain your middleware order."**

```jsx
app.use(express.static(...));   // assets short-circuit before anything else
app.use(cookieParser());        // must precede anything reading req.cookies
app.use(checkAuthSoft);         // decode JWT → req.user, or null
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
```

The load-bearing constraint is that **`cookieParser` precedes `checkAuthSoft`**, because the latter reads `req.cookies?.token`. Static first means asset requests never run auth. One thing worth noticing and owning: `checkAuthSoft` sits *before* the body parsers, which is unusual — it works because it only touches cookies, but the conventional order puts parsers first.

**"Why soft and hard auth as separate middleware?"**

> "They answer different questions. `checkAuthSoft` answers *who is this* and never rejects — the public link page needs to work for anonymous visitors, so an unauthenticated request just gets `req.user = null`. `checkAuthHard` answers *are they allowed* and returns 401. Identity is ambient; permission is explicit and applied per route."
> 

---

## §3 · The data model

**User** — `fullName`, `email` (unique), `password` (bcrypt hash), **`storageUsed`** (Number, default 0).

**File** — `userId` ref, `fileName`, `mimeType`, `size`, and a nested **`storage`** subdocument:

```jsx
storage: {
  provider:     { default: "cloudinary", enum: ["cloudinary"], required: true },
  publicId:     { required: true },
  secureUrl:    { required: true },
  resourceType: { enum: ["raw", "image"], required: true },
}
```

**Link** — `fileId` ref, `userId` ref (indexed), `expiresAt`, `maxDownloads`, `downloads` (default 0), `password`, `isPassEnabled`, **`publicId` (unique)**, `isRevoked`, `isAnonymous`.

**"Why is `storage` a nested object rather than flat fields?"**

> "It groups everything that's provider-specific. The `provider` field is an enum with one value today, but the shape means adding S3 later is a schema change in one place rather than scattered columns — and the rest of the code only ever asks for `file.storage.secureUrl`, which stays true regardless of who's storing it."
> 

That's a **deliberate extensibility seam**, and it's worth naming as one.

**"Link stores both `fileId` and `userId`. Isn't `userId` derivable from the file?"**

Yes — and this is the denormalisation question:

> "It's redundant, deliberately. The dashboard needs all of a user's links, and without `userId` on the link that query means fetching the user's files first and then querying links by `fileId: { $in: [...] }` — two round trips. With it, one indexed query. The cost is that the two could disagree if a file ever changed owner, which it can't in this application.
> 
> 
> Notice the code doesn't fully trust it, though: ownership checks in `edit-link` and `revoke-link` populate the file and compare `link.fileId.userId`, not `link.userId`. The denormalised field is for querying; the authoritative field is for authorisation."
> 

That distinction — *denormalise for reads, authorise against the source of truth* — is a strong thing to be able to articulate.

**"Why `publicId` unique?"** It's the entire security of a public link — the URL is the credential. Uniqueness is enforced at the database level so two links can never collide, and it's the field every public route looks up by.

**"Why is `downloads` on the link rather than counted from a downloads collection?"** Because the only question ever asked is "how many so far," never "who and when." A counter is one field and one atomic `$inc`; an events collection would be correct if you wanted analytics, and is the change you'd make if download history became a feature.

---

## §4 · The upload pipeline

`POST /api/file/add-file`. Read this sequence carefully — most of the interview lives here.

```jsx
const storage = multer.memoryStorage();
const upload = multer({ storage });
```

**"Why memory storage rather than disk?"**

> "The file never needs to exist on my server — it's going straight to Cloudinary. Writing it to disk first means a write and a read I don't need, plus cleanup on every failure path, plus a filesystem that's ephemeral on most hosts anyway. Keeping it in a buffer and streaming it onward is one hop instead of three.
> 
> 
> The tradeoff is memory: each in-flight upload holds its whole file in RAM. That's why the 10 MB cap matters — it bounds the exposure. If files were larger or concurrency higher, disk storage or a direct browser-to-Cloudinary signed upload would be the right call."
> 

That tradeoff, stated in both directions, is exactly the answer they want.

**"Explain this block."**

```jsx
const stream = Readable.from(buffer);
const uploadResult = await new Promise((resolve, reject) => {
  const uploadStream = cloudinary.uploader.upload_stream({...}, (error, result) => {
    if (error) reject(error); else resolve(result);
  });
  stream.pipe(uploadStream);
});
```

Two concepts stacked:

- **Promisifying a callback API.** Cloudinary's uploader is error-first callback-based; wrapping it in `new Promise` lets it sit in an `async/await` flow with everything else. It's the same bridge `util.promisify` provides, done by hand because the API also needs a stream piped into it.
- **`Readable.from(buffer)`** converts an in-memory buffer into a stream so it can be piped. Cloudinary's `upload_stream` wants a stream; Multer gave a buffer.

**"What's `resource_type` doing?"**

```jsx
resource_type: imageTypes.includes(mimetype) ? "image" : "raw"
```

Cloudinary treats images as media it can transform and everything else as opaque bytes. Getting it wrong means a PDF gets processed as an image and corrupts. The chosen type is stored on the file document because **deletion needs it too** — `destroy` requires the same `resource_type` used at upload.

**"Why scope the folder by user?"** `folder: velvet/users/${req.user._id}` — organisational, and it makes a per-user cleanup or audit a single prefix operation rather than a scan.

**"Why `randomBytes(12).toString('hex')` for the public id?"** Cryptographically random rather than derived from the filename — no collisions, no leaking of what the file was called, and no path-traversal surface from an attacker-controlled name.

---

## §5 · The storage quota — your single best answer

*This is the strongest technical moment in any of your projects. Make sure it's fluent.*

```jsx
const updatedUser = await User.findOneAndUpdate(
  { _id: req.user._id, storageUsed: { $lte: MAX_STORAGE - uploadedBytes } },
  { $inc: { storageUsed: uploadedBytes } },
  { new: true },
);
if (!updatedUser) {
  await cloudinary.uploader.destroy(uploadResult.public_id, { resource_type: uploadResult.resource_type });
  return res.status(400).json({ err: "Insufficient space" });
}
```

**"Walk me through what this does and why it's written this way."**

> "It's a check-and-set done atomically. The quota condition lives in the *filter* — `storageUsed` must be at most the maximum minus what I'm about to add — and the increment is the update. MongoDB guarantees atomicity on a single document, so no other write can interleave between the check and the increment.
> 
> 
> The naive version is read the user, add in Node, write back. That has a lost-update race: two uploads at 90 MB both read 90, both compute 95, both write 95, and the user ends at 95 having stored 100. Putting the condition in the filter and using `$inc` rather than `$set` means the *database* does the arithmetic against the current value, not a value I read a round trip ago.
> 
> And the failure signal is the absence of a document. If no user matches the filter — meaning they're over quota — `findOneAndUpdate` returns `null`. That's the rejection."
> 

**"What's the `destroy` call for?"**

This is the part that elevates the answer:

> "It's a **compensating action**. The file is already in Cloudinary by the time I check the quota, and Cloudinary and MongoDB are two separate systems with no shared transaction. So when the quota check fails, I have to undo the side effect myself — delete the uploaded asset and report the failure. It's the manual version of a rollback across a service boundary."
> 

**"Why upload before checking the quota? Isn't that backwards?"**

The obvious follow-up, and there's a real reason:

> "Because I don't know the true size until Cloudinary reports it. The browser's reported size and the stored size differ — Cloudinary may transform or re-encode, and `uploadResult.bytes` is what I'm actually charging against. Checking first would mean charging an estimate and then reconciling anyway. So I upload, get the authoritative number, then check — and compensate if it fails."
> 

**The gap to volunteer:** `File.create(...)` runs *after* the quota increment. If that insert fails, `storageUsed` has been charged for a file that doesn't exist. The user's quota drifts upward with no file to show for it. The fix is either a transaction across both writes, or a reconciliation job — and notice the delete route already does exactly that kind of reconciliation, which is §6.

**Interview framing:** *"Two writes to two systems with no shared transaction is the distributed-transaction problem in miniature. I handled the Cloudinary side with a compensating action and left one window open on the Mongo side, which I'd close with a transaction if I rebuilt it."*

---

## §6 · Deletion — increment on write, recompute on delete

```jsx
// idempotent Cloudinary delete
if (result.result === "ok" || result.result === "not found") resolve(result);
else reject(new Error("Cloudinary deletion failed"));

await Promise.all([
  File.deleteOne({ _id: file._id }),
  Link.deleteMany({ fileId: file._id }),
]);

const aggregation = await File.aggregate([
  { $match: { userId: req.user._id } },
  { $group: { _id: null, total: { $sum: "$size" } } },
]);
await User.findByIdAndUpdate(req.user._id, { storageUsed: aggregation[0]?.total || 0 });
```

Three things here, each worth a question.

**"Why does `not found` count as success?"**

> "Because deletion should be **idempotent** — the desired end state is 'this file is gone', and if it's already gone, that state is achieved. Treating a missing asset as an error would mean a retry after a partial failure blows up. This matters because network calls get retried, and an operation that breaks on the second attempt is fragile."
> 

**"Why `Promise.all` for the two Mongo deletes?"** They're independent, so awaiting them sequentially costs the sum of their latencies rather than the max. Same reasoning as any parallel independent I/O — and the trap worth naming is that `await` inside a loop runs sequentially, which is why independent work belongs in `Promise.all`.

**"Why recalculate storage with an aggregation instead of `$inc`-ing it down?"**

This is a deliberate asymmetry and a genuinely good design decision:

> "Upload increments; delete recomputes from scratch. The increment is fast and atomic, which is what a hot path needs. But increments can drift — if a write fails halfway, as in the gap I mentioned, the counter and reality diverge with no way to notice.
> 
> 
> Delete is a rare operation, so I can afford to sum every file the user owns and write the authoritative total. That makes deletion **self-healing**: any drift accumulated by the fast path gets corrected the next time a file is removed. Fast where it matters, correct where it's affordable."
> 

That framing — *fast path increments, slow path reconciles* — is a real engineering pattern and stating it in those terms is worth a lot.

**The cost to acknowledge:** the aggregation is O(number of files owned). Fine for a hundred files; a scan you'd want indexed and bounded at a hundred thousand.

---

## §7 · The link system — the domain logic

A file can have many links, each with independent rules. Four controls, all optional and composable:

| Control | Field | Enforced where |
| --- | --- | --- |
| Password | `password` (bcrypt) + `isPassEnabled` | Every download |
| Expiry | `expiresAt` | Metadata fetch and download |
| Download cap | `maxDownloads` vs `downloads` | Atomically, on download |
| Revocation | `isRevoked` | Metadata fetch and download |

**"Why is the link password hashed?"**

> "Same reason a user password is. If the database leaked, a plaintext link password would grant access to whatever that link protects. It's compared with `bcrypt.compare` on each download, so it's never stored or transmitted in a recoverable form."
> 

Most people would store this in plaintext because "it's just a share password." You didn't — say so.

**"How is `publicId` generated, and is it guessable?"** `randomBytes(10).toString("base64url")` — 80 bits of entropy, URL-safe encoding. Not enumerable, and `base64url` matters because standard base64 contains `+` and `/`, which would need escaping in a URL.

**"Explain these two rules in `edit-link`."**

```jsx
if (expiresAt && link.expiresAt && new Date(expiresAt) < link.expiresAt)
  return res.status(400).json({ err: "Expiry can only be extended" });

if (maxDownloads && (... || maxDownloads < link.downloads))
  return res.status(400).json({ err: "Invalid max downloads" });
```

These are **business invariants**, and being able to explain *why* rather than *what* is the point:

> "Both prevent retroactively tightening a promise. If I share a link saying 'valid for a week' and then quietly shorten it to a day, the recipient's expectation breaks silently. Same with the download cap — setting it below the count already used would make a link that's already exceeded its limit, which is an incoherent state. If the owner wants to cut access off early, that's what revocation is for — it's explicit and it's a different action."
> 

**"What does `isAnonymous` do?"** Suppresses the uploader's name on the public page:

```jsx
if (link.isAnonymous) link.userId = null;
```

Worth volunteering: **the field exists in the schema and is honoured on read, but no route ever sets it to true.** It's a half-built feature. Better to name it than to be caught claiming a feature that isn't wired up.

---

## §8 · Downloads — and the architectural decision behind them

```jsx
if (link.maxDownloads !== undefined) {
  const updated = await Link.findOneAndUpdate(
    { publicId, downloads: { $lt: link.maxDownloads } },
    { $inc: { downloads: 1 } },
    { new: true },
  );
  if (!updated) return res.redirect(`...?error=Download%20limit%20reached`);
} else {
  await Link.updateOne({ publicId }, { $inc: { downloads: 1 } });
}
```

**"This looks familiar."** It should — it's the **same atomic pattern as the quota**: condition in the filter, `$inc` as the update, `null` means rejected. Two students clicking a last-download link simultaneously can't both succeed, because the condition is evaluated inside the same operation as the increment.

Being able to say *"I used the same pattern in two places for the same reason"* shows you recognised a general technique rather than copying a fix.

**"Why does the file stream through your server? Why not just redirect to the Cloudinary URL?"**

This is the best architectural question in the project:

> "Because a redirect hands out the storage URL, and that URL has no expiry, no password, no download cap and no revocation. Every rule the owner set becomes decorative the moment someone shares the raw link. Proxying keeps my server as the enforcement point — the Cloudinary URL never reaches the browser.
> 
> 
> The cost is real: every byte of every download crosses my server, so bandwidth and connection time scale with usage rather than staying flat. The middle ground is **signed URLs with short expiry**, where the storage provider enforces a time limit on my behalf. That gives me back the bandwidth but not the download counting, so it's a genuine trade rather than a free win."
> 

**"Explain the streaming."**

```jsx
const cloudStream = request.get(cloudinaryUrl);
cloudStream.on("response", (cloudRes) => {
  res.setHeader("Content-Disposition", `attachment; filename="${asciiName}"; filename*=UTF-8''${encodedName}`);
  res.setHeader("Content-Type", link.fileId.mimeType || "application/octet-stream");
  pipeline(cloudRes, res, (err) => { ... });
});
```

> "The file is piped from Cloudinary through to the client without ever being fully buffered. Memory stays constant regardless of file size, and the client starts receiving bytes immediately rather than after a full download-then-forward. For a 10 MB file the difference is noticeable; for a 1 GB file, buffering simply wouldn't work."
> 

**"Why `pipeline` rather than `pipe`?"** `pipe` doesn't forward errors or clean up — a failure mid-transfer can leak the source stream and leave the response hanging. `pipeline` propagates errors, destroys every stream in the chain, and gives a completion callback. **Then volunteer the inconsistency:** the *private* download route uses plain `cloudRes.pipe(res)`. Two routes doing the same job with different error safety — an easy fix and a good thing to have noticed.

**"What's the double `filename` in Content-Disposition?"**

```
attachment; filename="report_2024.pdf"; filename*=UTF-8''report%202024%20%E2%9C%93.pdf
```

> "`filename=` is the ASCII fallback for old clients, and I strip everything non-ASCII to build it. `filename*=UTF-8''` is the RFC 5987 encoded form that modern browsers prefer, which preserves the original name including spaces and non-Latin characters. Sending both means every client gets something sensible.
> 
> 
> The stripping also serves a security purpose — an unfiltered filename in a header is a **response-splitting** vector if it contains newlines."
> 

**"What is `res.headersSent` guarding?"** Once headers are flushed you cannot send a status code — attempting it throws. Since streaming starts before the transfer completes, an error partway through has to be handled differently from an error before any bytes moved. Checking the flag is what makes that safe.

---

## §9 · Authentication

**The flow:** signup validates presence, types, email format and a password regex (`8+ chars, upper, lower, digit`), hashes with **bcrypt at 10 rounds**, and creates the user. Login finds by email, `bcrypt.compare`s, signs a JWT carrying only `{ _id }`, and sets it as an httpOnly cookie. `checkAuthSoft` decodes it on every request; `checkAuthHard` requires it, validates the ObjectId, **fetches the user from MongoDB**, and attaches the fresh document.

**"Why fetch the user from the database on every protected request?"**

> "A signed token proves what was true when it was issued — it can't tell me the account still exists or hasn't been disabled. It's also load-bearing here for a second reason: `req.user.storageUsed` has to be current, and a day-old token can't carry that.
> 
> 
> The cost is that a stateless token becomes effectively stateful — one read per protected request. At scale I'd cache the user in Redis with a short TTL, so it's one read per user per minute rather than per request, and delete the key on any change so the cache can't serve stale data."
> 

**"What does `null` from `checkAuthHard`'s user lookup mean?"** Deleted account with a still-valid token. It 401s, which is the correct behaviour — and it's exactly the case a pure-JWT approach would miss.

**"Why is only `_id` in the token payload?"** Minimal claims. Anything else — name, email, quota — is mutable, so putting it in a token that lives for a day guarantees staleness. Since the middleware fetches the user anyway, the token only needs to answer *who*.

**"How does the client know it's logged in?"** `AuthContext` calls `/api/user` on mount and then **every 60 seconds**:

```jsx
fetchUser(true);
const interval = setInterval(() => fetchUser(false), 60000);
return () => clearInterval(interval);
```

The `initial` flag means only the first call flips the loading state — subsequent polls refresh the data silently without flashing a loader. And the returned cleanup function clears the interval on unmount, without which the timer would keep firing against a dead component.

**"Is `ProtectedPages` real security?"** No — and say so plainly:

```jsx
if (values.loading) return <PageLoader show={true} />;
else if (values.user) return <Outlet />;
else return <Navigate to="/login" />;
```

> "It's a UX affordance. The security boundary is `checkAuthHard` on every protected endpoint. Client-side guards can be bypassed with devtools, and they don't need to be secure because they aren't what's protecting the data. What they do is stop a logged-out user seeing an empty dashboard flash before an API call fails."
> 

Note the three-way branch handles the state most people forget: **loading**. Rendering `Navigate` before auth resolves would bounce a logged-in user to the login page on every refresh.

---

## §10 · The React client

**"What's in the stack?"** React 19, Vite, React Router v7, `motion/react` for animation, Luxon and dayjs for dates, lucide-react for icons. Notably, the **React Compiler** is enabled:

```jsx
react({ babel: { plugins: ["babel-plugin-react-compiler"] } })
```

**"What does the React Compiler do?"** It auto-memoises — it analyses components and inserts the equivalent of `useMemo` and `useCallback` where they'd help, so you don't hand-tune re-renders. Worth being honest: it's new, and enabling it was partly to try it.

**"Explain the code splitting."**

```jsx
const HomePage = React.lazy(() => import("./pages/HomePage"));
...
<Suspense fallback={<PageLoader show={true} />}>
```

> "`React.lazy` with a dynamic import tells the bundler to split each page into its own chunk, so the initial load only ships what's needed for the first screen. `Suspense` provides the fallback while a chunk is fetched. It matters here because the public link page is the most-visited route and doesn't need the dashboard's code."
> 

**"Why a custom `useAuth` hook rather than `useContext` directly?"**

```jsx
function useAuth() {
  const value = useContext(AuthContext);
  if (value === undefined) throw new Error("using context outside the scope");
  return value;
}
```

It centralises the context reference and fails loudly if a component is rendered outside the provider — a clear error rather than a confusing `undefined` crash deep in a render.

**"Why are uploads sequential rather than parallel?"**

```jsx
for (let i = 0; i < files.length; i++) {
  setUploading({ name: file.name, index: i + 1 });
  await uploadSingleFile(file);
}
```

> "Three reasons. Progress is meaningful — I can show 'uploading 2 of 5' with the actual filename. Failures are isolated, so one rejected file doesn't take the batch down; the loop counts successes and errors separately and reports both. And it doesn't fire five concurrent multipart uploads at a server holding each one in memory. `Promise.all` would be faster, and would be right if the files were small and the server streamed to disk."
> 

**"What's `await new Promise(requestAnimationFrame)` doing?"** Yielding to the browser so the "uploading file N" state actually paints before the next upload blocks. Without it, React may batch the update and the user sees nothing change until the whole loop finishes.

---

## §11 · Security posture

**Defended:**

- **Passwords** — bcrypt, 10 rounds, for both user accounts *and* link passwords
- **Token theft via XSS** — httpOnly cookie; React escapes rendered content by default
- **CSRF** — `sameSite: "lax"` plus a JSON API, so a cross-site form can't produce a valid request
- **Unguessable share URLs** — 80 bits of entropy, base64url
- **IDOR** — every mutating route checks `file.userId.toString() !== req.user._id.toString()` and returns 403
- **ObjectId validation** — `mongoose.Types.ObjectId.isValid` before every lookup
- **Type confusion / NoSQL injection** — explicit `typeof` checks on auth inputs
- **Response-splitting via filenames** — non-ASCII and whitespace stripped before entering the header
- **Race conditions** — atomic conditional updates on both quota and download count
- **File type** — mimetype whitelist, 10 MB per file, 100 MB per user

**Not defended, and worth volunteering:**

- **`application/octet-stream` is in the whitelist**, which is the generic "unknown binary" type — it substantially weakens the list, since a client can label anything that way. The stronger approach is magic-number sniffing rather than trusting the declared mimetype.
- **`image/svg+xml` is in the image list.** SVG can carry embedded scripts. It's served from Cloudinary's domain rather than yours, which limits the damage, but it's a known footgun.
- **No rate limiting** — the public download endpoint and the login route are both unprotected. A Redis-backed limiter returning 429 is the fix.
- **No `helmet`** — a dozen security headers in one line, absent.
- **The `request` package is deprecated** and unmaintained. `node-fetch` or native `fetch` with `Readable.fromWeb` is the modern replacement.

---

## §12 · Known weaknesses — volunteer these

*Naming your own flaws with fixes is the strongest move available. Pick two or three.*

1. **`File.create` runs after the quota increment** — a failure there charges storage for a file that doesn't exist. Fix: a transaction, or lean on the delete-path reconciliation. *(Best one — it pairs with your best answer.)*
2. **`pipe` vs `pipeline` inconsistency** — the public download handles stream errors, the private one doesn't.
3. **`application/octet-stream` in the mimetype whitelist** effectively defeats the whitelist.
4. **The download counter increments before the stream succeeds** — a failed transfer still consumes a download.
5. **`isAnonymous` is honoured on read but never set** — a half-wired feature.
6. **Storage recalculation is O(files owned)** — fine now, unbounded later.
7. **No pagination on `/api/file/all`** — returns every file and link a user owns.
8. **No rate limiting anywhere.**
9. **`request` is deprecated.**
10. **Memory storage caps concurrency** — ten simultaneous uploads is 100 MB of RAM.

---

## §13 · "What breaks at 10× traffic?"

> "I'd measure first, but the predictable failures are:
> 
> 
> **Memory**, because every in-flight upload sits in RAM. Ten concurrent 10 MB uploads is 100 MB, and it scales linearly with concurrency. The fix is to stop routing bytes through my server at all — issue a **signed upload URL** so the browser uploads directly to Cloudinary and only tells my API the result.
> 
> **Download bandwidth**, because every downloaded byte crosses my server to keep the link rules enforceable. Short-lived signed URLs move that to the CDN, at the cost of exact download counting — which is a real trade, not a free win.
> 
> **Auth**, because `checkAuthHard` does a database read on every protected request. Redis with a short TTL turns that into one read per user per minute.
> 
> **The dashboard**, because `/all` returns every file and link with no limit. Pagination, and an index on `{ userId, createdAt }`.
> 
> Structurally the API is stateless — the session is a signed cookie — so it scales horizontally behind a load balancer with PM2 cluster mode, since one Node process is one thread. Then rate limiting, so one client can't consume the capacity I just added."
> 

---

## §14 · Comparative questions

You have three projects with three architectures. Use the contrast.

|  | **Eventually** | **Velvet** | **Firebase project** |
| --- | --- | --- | --- |
| Rendering | SSR, EJS | React SPA, Vite | React |
| Backend | Express, same app | Express + Cloudinary | BaaS |
| Auth | JWT + role, DB-verified | JWT, DB-verified | Managed provider |
| Hard problem | Capacity race, idempotent attendance | Atomic quota, streaming downloads | — |

**"What did you learn across them?"** The strongest answer you have:

> "Concurrency, specifically. In Eventually I checked event capacity by counting registrations and then inserting — two operations, so two students could both take the last seat. I half-caught it: a unique index stops one user registering twice, but nothing protected the capacity limit itself.
> 
> 
> By Velvet I understood the shape of the problem. The storage quota is the same bug in different clothes, and there I put the condition into the filter of a single `findOneAndUpdate` with `$inc`, so the check and the write can't be interleaved. Then I recognised it a third time in the download cap and used the same pattern. Same class of problem, and by the third time I saw it coming."
> 

That is a real engineering arc, and it answers "tell me about a time you improved" without needing a separate story.

**"Why three different auth approaches?"** Because the requirements differ. Eventually has roles, so a stale role claim is a privilege-escalation risk. Velvet has no roles but needs live `storageUsed`, so it fetches for a different reason. A managed provider trades control for speed. Being able to say *why* each was right is better than having used one three times.

---

## §15 · Rapid-fire bank

**Node/Express** — Why Express over raw `http` · What `next()` does · Middleware order and why `cookieParser` is first · The SPA fallback and why it must be last · Why Node is single-threaded yet concurrent · Streams and why they beat buffering · `pipe` vs `pipeline` · Promisifying a callback API

**Mongo/Mongoose** — `.lean()` and what it gives up · `populate` vs `$lookup` · Why `runValidators` on updates · What `{ new: true }` changes · Single-document atomicity · `$inc` vs `$set` and why it matters here · Aggregation pipeline stages · Embed vs reference

**Auth** — JWT structure · Why the payload is readable, not secret · What the signature proves · httpOnly / sameSite / secure · Session vs JWT · How you'd revoke a JWT · Why bcrypt is deliberately slow · 401 vs 403

**Concurrency** — Race conditions and lost updates · TOCTOU · Why the check must be in the filter · Idempotency, and where it appears in this project · Compensating actions across services

**React** — `useContext` and why the custom hook wrapper · `useEffect` cleanup · `React.lazy` and Suspense · Why client-side route guards aren't security · Sequential vs parallel async in a loop · What the React Compiler does

**Web** — CORS and why the proxy avoids it · Content-Disposition and RFC 5987 · Response splitting · XSS, CSRF · Why streaming matters for large files

---

## How to use this

**Rehearse §1 and §13 aloud** until they're paragraphs rather than lists.

**Then rehearse §5 until it's automatic.** The atomic quota check — the lost-update race, the condition in the filter, `null` as the rejection signal, and the Cloudinary compensating action — is the single best technical answer you have across all your projects. Most candidates cannot produce a concurrency bug from their own code, let alone the fix.

**Pick three from §12 to volunteer.** The `File.create` ordering gap, the `pipe`/`pipeline` inconsistency, and the `octet-stream` whitelist hole are the strongest — each has a clear mechanism and a clear fix.

**Keep §8's redirect question loaded.** "Why not just redirect to Cloudinary?" is the question that lets you explain that the server is the enforcement point for every rule the product is built on. It's the architectural heart of the project, and the answer is genuinely yours.