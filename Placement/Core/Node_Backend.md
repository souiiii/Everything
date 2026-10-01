# Node-Core-Placement-Notes

# Node.js Core — Placement Notes

*Cut down from the 23-section, ~180-question nodejs-basics README to the ~55 that get asked on campus, with every answer rewritten from scratch in say-it-aloud form.*

**How to use this file**
- Read the Q → answer out loud → check → ⭐ star anything your spoken version fumbled. For new topics: read once, close the file, say it back immediately.
- Block mapping — **tonight (Day 1, 6:45 PM):** watch the event-loop animation first, then §1–§3. **Day 2, 2 PM:** §4–§6. **Day 2, 10:30 PM:** §7–§9, then redo stars only.
- Deliberately missing: JWT/auth (Monday’s block), Redis + caching (Tuesday), rate limiting (Wednesday), MongoDB (Sunday). Testing, i18n, profiling, logging, MSSQL — cut, near-zero campus ROI.

---

## §1 · Node fundamentals — your “explain Node in one minute”

**Q. What is Node.js?**
A JavaScript **runtime** — not a language, not a framework — that runs JS outside the browser. It’s Chrome’s **V8** engine (executes the JS) plus **libuv** (a C library providing the event loop and async I/O). Its design bet: **single-threaded, event-driven, non-blocking I/O** — excellent for I/O-heavy servers (APIs, chat, streaming), poor for CPU-heavy work (video encoding, ML).

**Q. How does Node.js work? (the process model)**
One main thread runs your JavaScript. When a request needs I/O (file, DB, network), Node doesn’t wait — it hands the operation to the OS or to **libuv’s thread pool** (4 threads by default), registers a callback, and moves on to the next request. When the I/O finishes, the callback joins a queue, and the **event loop** runs it once the call stack is free. So one thread *executes* JS, but thousands of connections can be *in flight*, because the waiting happens elsewhere.

**Q. What is the V8 engine?**
Google’s open-source JS engine, written in C++. It JIT-compiles JavaScript to machine code — an interpreter (Ignition) starts code fast, an optimizing compiler (TurboFan) recompiles hot paths. It’s why Node is fast for a “scripting language.”

**Q. Why does Node need libuv?**
V8 knows nothing about files, sockets, or timers — libuv supplies all of it. It implements the **event loop**, the **thread pool**, and cross-platform async I/O (wrapping epoll on Linux, kqueue on macOS, IOCP on Windows). One line: *V8 runs my code; libuv talks to the operating system.*

**Q. What is the Reactor Pattern?**
The architecture Node implements: I/O requests go to an **event demultiplexer**; completed operations land in an **event queue**; a single-threaded **event loop** dispatches each to its **handler** (your callback). Fancy name, simple idea — don’t wait on I/O, react to it.

**Q. Process vs thread?**
A **process** owns its memory space; **threads** live inside a process and share its memory. Node runs your JS on one thread in one process. To use more cores you run more processes (**cluster** / PM2) or more threads (**worker_threads**, for CPU work).

> **40-second spoken pitch:** “Node is a JS runtime built on V8 and libuv. My code runs on a single thread and never blocks on I/O — file and network work is delegated to libuv and the OS, and results come back through the event loop as callbacks. That’s why it handles huge numbers of concurrent connections cheaply, and why CPU-heavy work needs worker threads or extra processes.”
> 

---

## §2 · Event loop & async — the most-grilled section

**Q. What are the phases of the event loop?**
Each iteration (“tick”), in order:
1. **Timers** — expired `setTimeout` / `setInterval` callbacks
2. **Pending callbacks** — leftover system-level callbacks (some TCP errors)
3. **Idle / prepare** — internal
4. **Poll** — the workhorse: waits for and runs I/O callbacks (fs, network)
5. **Check** — `setImmediate` callbacks
6. **Close callbacks** — e.g. `socket.on('close')`

Know 1, 4, 5 cold; name-drop the rest.

**Q. Microtasks vs macrotasks?Macrotasks:** timer callbacks, I/O callbacks, `setImmediate`. **Microtasks:** the `process.nextTick` queue and the Promise (`.then`) queue — and these drain **completely after every operation, before the loop moves on**, with nextTick beating promises. That one rule decodes almost every “what’s the output?” question.

**Q. `process.nextTick()` vs `setImmediate()`?** *(near-guaranteed question)*
- `process.nextTick(cb)` — runs **immediately after the current operation**, before the event loop continues. Highest priority in the system.
- `setImmediate(cb)` — runs in the **check phase**, i.e. after poll’s I/O callbacks.

Two traps: (1) inside an I/O callback, `setImmediate` **always** fires before `setTimeout(0)`; (2) at the top level of a script their order is **nondeterministic**. Bonus point: recursive `nextTick` can starve the loop; recursive `setImmediate` can’t. And yes — the names are backwards; nextTick is the immediate one. Saying that earns a smile.

**Q. The classic output question**

```jsx
console.log('1');
setTimeout(() => console.log('2'), 0);
setImmediate(() => console.log('3'));
process.nextTick(() => console.log('4'));
Promise.resolve().then(() => console.log('5'));
console.log('6');
```

Output: **1 6 4 5 2 3** (2/3 may swap — top-level nondeterminism).
Why: all synchronous code first (1, 6) → nextTick queue (4) → promise queue (5) → event-loop phases: timers (2), check (3).

**Q. What is EventEmitter?**
The class behind Node’s entire event system (`require('events')`) — HTTP servers, streams, and `process` all extend it. Core methods: `.on(event, fn)`, `.once`, `.emit(event, ...args)`, `.off` / `.removeListener`, `.listenerCount`.

```jsx
const { EventEmitter } = require('events');
const bus = new EventEmitter();
bus.on('order', id => console.log('got', id));
bus.emit('order', 42);   // listeners run synchronously, in registration order
```

Caveat worth volunteering: an `'error'` event with **no listener attached** throws and crashes the process.

**Q. Callbacks — error-first, callback hell, events vs callbacks**
A callback is a function passed in to run when an async task completes. **Error-first convention:** the first parameter is always the error — `(err, data) => { if (err) return handle(err); ... }` — so “what’s typically the first argument passed to a callback?” = **an error object (or null)**. **Callback hell** = deep nesting for sequential steps; fixed by named functions, Promises, and cleanest of all async/await. **Events vs callbacks:** a callback is one function for one completion; events are pub/sub — many listeners, fired many times, decoupled.

**Q. Timers — and how do you “sleep”?**`setTimeout` (once, timers phase), `setInterval` (repeating), `setImmediate` (check phase), `process.nextTick` (before everything). There’s no blocking sleep; you await a promisified timeout:

```jsx
const sleep = ms => new Promise(r => setTimeout(r, ms));
await sleep(1000);
```

---

## §3 · Single-threaded, yet concurrent — the scaling questions

**Q. Is Node.js entirely single-threaded?**
Spoken answer: “**JavaScript execution** is single-threaded — one call stack. The **process** isn’t: libuv keeps a thread pool (default 4 — used for fs, DNS, some crypto), and I can create real threads with worker_threads. Node is ‘single-threaded’ only in the sense that *my code* runs on one thread.”

**Q. If it’s single-threaded, how does it handle thousands of concurrent requests?**
Because a *waiting* connection costs almost nothing. The JS thread only runs each request’s small synchronous slices; all the waiting (DB, disk, network) is parked with the OS and thread pool. Contrast: one-thread-per-request servers burn a whole thread per waiting request. This is also the honest weakness — one CPU-heavy request blocks **everyone**, which is exactly what worker threads are for.

**Q. How does the cluster module work? / How do you use all CPU cores?**`cluster.fork()` spawns full worker **processes** — one per core is the norm — that **share the same server port**; the primary accepts connections and distributes them (round-robin on Linux). Workers have separate memory, so shared state (sessions, counters) must live outside, e.g. in Redis.

```jsx
const cluster = require('cluster');
const os = require('os');
if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
  cluster.on('exit', () => cluster.fork());   // self-heal on crash
} else {
  require('./server');                        // each worker runs the app
}
```

In production nobody hand-rolls this — **PM2** does it: `pm2 start app.js -i max` (cluster mode, zero-downtime `pm2 reload`). ← This exact line feeds your 10x-traffic mock on Day 8.

**Q. `spawn()` vs `fork()`?**
Both come from `child_process`. **spawn** runs *any* command and streams its stdio — right for big output (ffmpeg, a Python script). **fork** is a specialized spawn for *Node scripts* that opens a built-in **IPC channel** (`child.send()` ↔︎ `process.on('message')`). Bonus: **exec** runs in a shell and buffers the whole output — small outputs only.

**Q. Worker threads — and when vs cluster?**`worker_threads` = real threads **inside one process**, for **CPU-bound** work (image processing, heavy parsing, crypto) so the event loop never freezes; they communicate via message passing and can share memory through SharedArrayBuffer. Rule of thumb to say aloud: “**cluster to scale I/O across cores; worker threads to keep CPU work off the event loop.**”

---

## §4 · Streams

**Q. What are streams, and the four types?**
Streams process data **chunk by chunk** instead of loading it all into memory. Four types: **Readable** (`fs.createReadStream`, an incoming HTTP request), **Writable** (`fs.createWriteStream`, the HTTP response), **Duplex** (both directions, independent — a TCP socket), **Transform** (a duplex that modifies data flowing through — gzip, encryption).

**Q. How do you handle a large file / large data?**
Never `fs.readFile` a huge file — that loads all of it into RAM. Stream it:

```jsx
app.get('/video', (req, res) => {
  fs.createReadStream('big.mp4').pipe(res);
});
```

Say the two wins: **constant memory** no matter the file size, and the client starts receiving **immediately**.

**Q. What is backpressure?**
When the readable side produces faster than the writable side can consume (fast disk → slow network connection). Mechanics if pressed: `writable.write()` returns `false` → pause the readable → resume on the writable’s `'drain'` event. In practice the sentence to say is: **“pipe and pipeline handle backpressure automatically.”**

**Q. `pipe()` vs `pipeline()`?**`a.pipe(b)` connects streams but **doesn’t forward errors** — a failure mid-chain can leak file handles and memory. `pipeline(a, b, c, cb)` (from the `stream` module) propagates errors, destroys every stream on failure, and signals completion. Interview line: *“pipe for a quick demo, pipeline in production.”*

---

## §5 · Promises & async patterns

**Q. What is a Promise?**
An object representing a value that will exist later. States: **pending → fulfilled or rejected** — once settled, permanent. Consumed with `.then / .catch / .finally`. It replaced callback nesting with flat, chainable error handling.

**Q. Promise.all vs allSettled vs race vs any** *(asked constantly)*
- **all** — waits for every promise; **fail-fast** — one rejection rejects the lot. Use: parallel fetches where you need all results.
- **allSettled** — never rejects; returns `{status, value|reason}` per item. Use: batch work where partial failure is acceptable.
- **race** — settles with the **first to settle**, success *or* failure. Use: timeouts — `Promise.race([fetchData(), timeout(3000)])`.
- **any** — first to **fulfill**; ignores rejections; rejects only if *all* fail (AggregateError). Use: fastest mirror wins.

**Q. Promises vs async/await?**
Same machinery — async/await is **syntax over promises**. An `async` function always returns a promise; `await` pauses *that function* (never the thread) until the promise settles. Wins: reads top-to-bottom, normal `try/catch`, easy loops and conditionals. Trap to volunteer: `await` inside a for-loop runs **sequentially** — independent work should be `await Promise.all([...])`.

**Q. Asynchronous vs non-blocking?Non-blocking:** the call returns immediately instead of holding the thread. **Asynchronous:** the result arrives later via callback/promise/event. Node’s I/O is both at once — non-blocking calls whose completions are delivered asynchronously through the event loop.

**Q. How does Node prevent blocking code — and how do people block it anyway?**
Prevention: every I/O API is async by default, so the thread keeps moving. But *you* can still block it — and one blocked thread stalls **every user**: `fs.readFileSync` or `crypto.pbkdf2Sync` inside a request handler, `JSON.parse` on a giant payload, an accidental heavy loop. Fixes: async APIs, worker threads for CPU work, chunking. Favorite “spot the bug” question — sync call in a route handler.

---

# Node.js Core + Express — Updated Placement Notes

These keep the structure and spoken-answer style of your original notes, while correcting the event-loop/microtask nuances and adding the missing Express middleware section. Your original file already had the right placement-focused scope.

---

# §1 · Node Fundamentals

## Q. What is Node.js? ⭐

**Node.js is a JavaScript runtime**, not a language and not a framework.

It combines:

- **V8** → executes JavaScript.
- **libuv** → provides the event loop and much of Node's asynchronous I/O infrastructure.
- Node APIs → filesystem, networking, streams, processes, etc.

Its central design is **event-driven, non-blocking I/O**, which makes it particularly effective for applications that spend a lot of time waiting for network, database, or filesystem operations.

### Interview answer

> “Node.js is a JavaScript runtime built around V8 and an event-driven, non-blocking I/O model. JavaScript normally executes on one main thread, while Node can keep many I/O operations in flight without blocking that thread.”
> 

---

## Q. How can Node handle many requests with one JavaScript thread? ⭐

Suppose three requests arrive:

```
Request A → waiting for DB
Request B → waiting for network
Request C → executing JavaScript
```

Node does **not** keep the JavaScript thread blocked while A and B wait.

Instead, asynchronous work is handled by the operating system and, for certain operations, libuv's worker pool. When work completes, Node schedules the corresponding callback so JavaScript can process the result later.

Important nuance:

> **Not every asynchronous operation uses the libuv thread pool.**
> 

Network I/O is generally driven by operating-system event mechanisms, while operations such as some filesystem, DNS, crypto and compression work can use libuv workers.

### The important distinction

```
Concurrency ≠ Parallel JavaScript execution
```

Node can have thousands of operations **in progress concurrently**, even though ordinary JavaScript callbacks on the main thread execute one at a time.

---

## Q. What is V8?

V8 is Google's JavaScript engine.

Its job is essentially:

```
JavaScript
    ↓
V8
    ↓
Machine-executable code
```

Node uses V8 to execute JavaScript outside the browser.

---

## Q. Why does Node use libuv?

V8 executes JavaScript, but a runtime also needs things such as:

```
timers
filesystem I/O
network I/O
event-loop machinery
async operating-system interaction
```

libuv provides much of this cross-platform infrastructure.

Good interview line:

> **“V8 executes my JavaScript; libuv helps Node coordinate asynchronous work with the operating system.”**
> 

---

## Q. Process vs Thread

A **process** has its own memory space and resources.

A **thread** executes inside a process, and threads within the same process can share memory.

In the normal Node execution model:

```
Node process
│
├── Main JavaScript thread
├── libuv infrastructure
└── optional Worker Threads
```

For scaling across cores, Node can also run multiple processes.

---

# §2 · Event Loop and Async Execution

## Q. What is the event loop? ⭐

The event loop is the mechanism that allows Node to execute callbacks when asynchronous work becomes ready, without blocking the main JavaScript thread while waiting.

The important phases to recognize are:

```
Timers
Pending callbacks
Idle / prepare
Poll
Check
Close callbacks
```

For interviews, know these three especially well:

### Timers

Handles callbacks whose timer threshold has been reached:

```jsx
setTimeout()
setInterval()
```

A delay such as:

```jsx
setTimeout(fn, 1000);
```

means:

> run `fn` **no earlier than approximately 1000 ms**, once the event loop is able to execute it.
> 

It does not guarantee execution at exactly 1000 ms.

### Poll

The central I/O phase.

This is where Node processes many callbacks associated with completed I/O.

### Check

This is where:

```jsx
setImmediate()
```

callbacks execute. Node documents `setImmediate()` as scheduling a callback after I/O callbacks in the current event-loop turn. ([Node.js](https://nodejs.org/api/timers.html?utm_source=chatgpt.com))

You do **not** need to recite obscure libuv implementation details in a normal interview.

---

# §3 · `process.nextTick`, Promises and `setImmediate`

This section is highly interviewable.

## `process.nextTick()`

```jsx
process.nextTick(callback);
```

schedules a callback to run after the current JavaScript operation finishes, **before Node continues with the regular event loop**. Recursive use can therefore starve the event loop. Current Node documentation classifies `process.nextTick()` as Legacy and recommends `queueMicrotask()` for most ordinary userland deferral. ([Node.js](https://nodejs.org/api/process.html?utm_source=chatgpt.com))

---

## Promise callbacks

Callbacks registered through:

```jsx
Promise.resolve().then(...)
```

are scheduled in the JavaScript microtask queue.

---

## `setImmediate()`

```jsx
setImmediate(callback);
```

runs through the event loop's **check phase**. ([Node.js](https://nodejs.org/api/timers.html?utm_source=chatgpt.com))

So conceptually:

```
Current JavaScript
       ↓
nextTick / microtask processing
       ↓
event loop continues
       ↓
I/O / timers / check etc.
```

---

## `nextTick` vs Promise — important nuance

For a typical **CommonJS** example, Node drains the next-tick queue before the Promise microtask queue:

```jsx
process.nextTick(() => console.log("nextTick"));

Promise.resolve().then(() => {
    console.log("promise");
});
```

CommonJS normally gives:

```
nextTick
promise
```

But **do not say this ordering is universal**.

With top-level ES Modules, execution itself occurs in the microtask machinery, and Promise/`queueMicrotask()` callbacks can run before `process.nextTick()`. Node's current documentation explicitly distinguishes CommonJS and ESM here. ([Node.js](https://nodejs.org/api/process.html?utm_source=chatgpt.com))

### Interview-safe answer

> “`process.nextTick` schedules work before the event loop continues, while `setImmediate` runs in the check phase. Promise callbacks use the microtask queue. In CommonJS, nextTick normally runs before Promise microtasks, although ESM has an ordering nuance.”
> 

That is more accurate than memorizing one output sequence.

---

## Classic output problem

```jsx
console.log("1");

setTimeout(() => console.log("2"), 0);

setImmediate(() => console.log("3"));

process.nextTick(() => console.log("4"));

Promise.resolve().then(() => console.log("5"));

console.log("6");
```

The guaranteed beginning is:

```
1
6
```

because synchronous JavaScript runs first.

In a typical CommonJS context:

```
4
5
```

then follow.

The relative ordering of the zero-delay timer and `setImmediate` at the top level should **not be blindly relied upon**.

The original notes correctly warned against assuming deterministic top-level ordering for those two.

---

# §4 · EventEmitter

Node uses an event-driven model extensively.

`EventEmitter` allows objects to publish named events and other code to listen for them.

```jsx
import { EventEmitter } from "node:events";

const bus = new EventEmitter();

bus.on("order", id => {
    console.log("Order:", id);
});

bus.emit("order", 42);
```

Important methods:

```
on()       → listen repeatedly
once()     → listen once
emit()     → trigger event
off()      → remove listener
```

Listeners for a given `emit()` are invoked synchronously in registration order. Your original notes also correctly flag the special `'error'` event: emitting it without an appropriate listener can cause the process to terminate.

---

# §5 · Callbacks

A callback is simply a function passed to another function so it can be invoked later.

Traditional Node APIs commonly use the **error-first callback convention**:

```jsx
readFile("a.txt", (err, data) => {
    if (err) {
        return handleError(err);
    }

    console.log(data);
});
```

The first argument is:

```
error or null
```

The second usually contains the result.

---

## Callback Hell

Sequential asynchronous work can become deeply nested:

```jsx
login(() => {
    getUser(() => {
        getOrders(() => {
            sendResponse(() => {});
        });
    });
});
```

This becomes difficult to:

- read
- maintain
- reason about
- handle errors in

Promises and `async/await` provide cleaner control flow.

---

# §6 · Why Node is “Single-Threaded Yet Concurrent” ⭐

This is one of the most important Node interview questions.

Saying:

> “Node is single-threaded.”
> 

is incomplete.

More accurate:

> **JavaScript execution on the main event loop is normally single-threaded. The Node process itself is not limited to one thread.**
> 

There may be:

- operating-system asynchronous I/O
- libuv worker threads
- Worker Threads
- multiple Node processes

The main event loop therefore does not have to sit idle waiting for every external operation.

---

## Why is Node strong for I/O-heavy applications?

Imagine 10,000 users waiting for network or database responses.

A waiting request does not require the JavaScript thread to continuously execute work for it.

```
Request
   ↓
Start I/O
   ↓
Main thread moves on
   ↓
I/O completes
   ↓
Callback becomes runnable
   ↓
Event loop executes callback
```

That is why Node works well for:

- APIs
- web servers
- real-time systems
- network services
- streaming systems

---

## What is Node bad at?

Long-running synchronous CPU work on the main thread.

Example:

```jsx
app.get("/heavy", (req, res) => {
    for (let i = 0; i < 10_000_000_000; i++) {
        // CPU work
    }

    res.send("done");
});
```

During that loop, the event loop cannot serve ordinary JavaScript callbacks efficiently for other users.

So:

> **Never confuse asynchronous I/O concurrency with unlimited CPU parallelism.**
> 

---

# §7 · Cluster vs Worker Threads ⭐

## Cluster / Multiple Processes

Cluster allows multiple Node **processes** to serve workloads, including workers that can share a server port. Each process has its own memory space.

Conceptually:

```
          Requests
             ↓
   ┌─────────┼─────────┐
   ↓         ↓         ↓
Process 1 Process 2 Process 3
```

Useful when you want to scale a server across CPU cores.

Because processes have separate memory, application state that must be shared generally needs to live somewhere external, such as a database or distributed cache.

---

## Worker Threads

Worker Threads execute JavaScript in parallel **inside the same Node process** and can communicate through message passing; they can also share certain memory objects.

They are primarily useful for **CPU-intensive JavaScript**, not ordinary I/O work. Node's current documentation explicitly recommends them for CPU-intensive operations and notes that built-in async I/O is usually better for I/O-bound workloads. ([Node.js](https://nodejs.org/download/release/latest/docs/api/worker_threads.html?utm_source=chatgpt.com))

Examples:

```
large data parsing
CPU-heavy calculations
image processing logic
compression algorithms
computational work
```

### Interview rule

> **Multiple processes/cluster → scale server execution across cores.**
> 
> 
> **Worker Threads → move CPU-intensive JavaScript off the main event loop.**
> 

---

# §8 · `spawn()` vs `fork()` vs `exec()`

All belong to:

```jsx
node:child_process
```

### `spawn()`

Starts another process and gives you streaming access to its input/output.

Good for:

```
ffmpeg
Python script
large command output
```

### `fork()`

A specialized mechanism for launching another **Node.js process**, with an IPC communication channel available between parent and child.

```
parent
  ↕ IPC
child Node process
```

### `exec()`

Runs a command through a shell and buffers its output.

Useful for relatively small command outputs.

### Interview answer

> “`spawn` is general-purpose and streams process output. `fork` is specialized for Node child processes and provides IPC. `exec` runs through a shell and buffers the command output.”
> 

This preserves the distinction in your original notes.

---

# §9 · Express Middleware ⭐

This was the main missing piece in the original notes.

Express is essentially built around a **middleware pipeline**.

A middleware function receives:

```jsx
(req, res, next)
```

Example:

```jsx
app.use((req, res, next) => {
    console.log(req.method, req.url);
    next();
});
```

A middleware can:

- execute code
- inspect or modify `req`
- inspect or modify `res`
- terminate the request-response cycle
- call `next()` to continue to the next matching middleware

If it neither sends a response nor calls `next()`, the request remains hanging. ([Express.js](https://expressjs.com/en/guide/using-middleware/?utm_source=chatgpt.com))

---

## Middleware pipeline

Suppose:

```jsx
app.use(logger);
app.use(auth);

app.get("/profile", controller);
```

A request conceptually flows through:

```
Request
   ↓
logger
   ↓ next()
auth
   ↓ next()
controller
   ↓
Response
```

Middleware executes sequentially, so **registration order matters**. ([Express.js](https://expressjs.com/en/5x/api/application/?utm_source=chatgpt.com))

---

## What does `next()` do? ⭐

`next()` says:

> “I have finished my work; continue to the next matching middleware or route handler.”
> 

Example:

```jsx
function auth(req, res, next) {
    if (!req.user) {
        return res.status(401).json({
            error: "Unauthorized"
        });
    }

    next();
}
```

If authentication fails:

```
response sent
→ chain stops
```

If authentication succeeds:

```
next()
→ continue
```

This maps directly to middleware such as the authentication logic you already use in backend projects.

---

## Application vs Router Middleware

Application-level:

```jsx
app.use(logger);
```

Router-level:

```jsx
router.use(auth);
```

You can also mount middleware at a path:

```jsx
app.use("/api", middleware);
```

Then it applies to matching paths beneath `/api`. Express supports application-level, router-level, built-in, third-party and error-handling middleware. ([Express.js](https://expressjs.com/en/guide/using-middleware/?utm_source=chatgpt.com))

---

# §10 · Error Middleware ⭐

Normal middleware:

```jsx
(req, res, next)
```

Error middleware:

```jsx
(err, req, res, next)
```

Example:

```jsx
app.use((err, req, res, next) => {
    console.error(err);

    res.status(500).json({
        error: "Internal server error"
    });
});
```

The **four-argument signature matters** because Express uses it to recognize error-handling middleware. ([Express.js](https://expressjs.com/en/guide/using-middleware/?utm_source=chatgpt.com))

In Express 5, an async middleware/handler that returns a Promise and rejects or throws automatically forwards that failure into the error-handling flow. ([Express.js](https://expressjs.com/en/4x/guide/writing-middleware/?utm_source=chatgpt.com))

---

# §11 · Streams

Streams process data **incrementally** instead of loading the entire value into memory first.

Four fundamental types:

### Readable

Produces data.

```jsx
fs.createReadStream(...)
```

### Writable

Consumes data.

```jsx
fs.createWriteStream(...)
```

### Duplex

Both readable and writable.

Example:

```
TCP socket
```

### Transform

A Duplex stream that changes the data passing through it.

Examples:

```
gzip
encryption
compression
```

Your original notes summarize these distinctions well.

---

## Why use streams for large files?

Bad:

```jsx
const file = await fs.promises.readFile("5GB-video.mp4");
```

That approach attempts to load the whole file before processing it.

Instead:

```jsx
fs.createReadStream("video.mp4").pipe(res);
```

Now data moves approximately as:

```
disk
 ↓ chunk
Node
 ↓ chunk
client
```

Benefits:

- lower memory requirements
- processing can begin before the entire file is loaded

---

# §12 · Backpressure

Suppose:

```
Fast producer
     ↓↓↓↓↓↓↓
Slow consumer
```

Example:

```
fast disk
   ↓
slow network
```

If the producer continues indefinitely, memory usage can grow.

**Backpressure** is the mechanism that lets the consumer effectively tell the producer:

> “Slow down; I cannot consume data this quickly.”
> 

Node streams are designed to coordinate this flow. Your original notes correctly identify `pipe()`/`pipeline()` as the practical mechanisms that manage stream flow and backpressure.

---

## `pipe()` vs `pipeline()`

```jsx
readable.pipe(writable);
```

is simple and convenient.

For production chains where robust error propagation and cleanup matter, `pipeline()` is safer.

Interview line:

> **“`pipe` connects streams; `pipeline` gives me coordinated error handling and cleanup across the stream chain.”**
> 

---

# §13 · Promises

A Promise represents the eventual result of an asynchronous operation.

States:

```
pending
   ↓
fulfilled

OR

pending
   ↓
rejected
```

Once settled, a Promise does not return to pending.

---

## Promise combinators ⭐

### `Promise.all()`

```jsx
await Promise.all([a(), b(), c()]);
```

Use when **all operations must succeed**.

If one rejects, the returned Promise rejects.

---

### `Promise.allSettled()`

Waits for every Promise regardless of success or failure.

Useful when:

> “Process everything and tell me what succeeded and failed.”
> 

---

### `Promise.race()`

Returns the outcome of the **first Promise to settle**.

That first result can be either:

```
fulfilled
OR
rejected
```

Typical use:

```
operation vs timeout
```

---

### `Promise.any()`

Returns the first **successful** result.

Rejections are ignored until every candidate has failed.

---

# §14 · `async` / `await`

`async/await` is syntax built on Promises.

```jsx
async function loadUser() {
    const user = await getUser();
    return user;
}
```

An `async` function returns a Promise.

`await` pauses **that async function's continuation**; it does **not block the JavaScript thread** while an asynchronous operation is pending.

---

## Common performance mistake

This:

```jsx
for (const id of ids) {
    await fetchUser(id);
}
```

runs the requests sequentially:

```
A finishes
↓
B starts
↓
B finishes
↓
C starts
```

If they are independent:

```jsx
await Promise.all(
    ids.map(id => fetchUser(id))
);
```

allows them to be in flight concurrently.

---

# §15 · Asynchronous vs Non-Blocking

These terms are related but not identical.

### Non-blocking

The call does not keep the execution thread waiting until the operation completes.

### Asynchronous

The result becomes available later through some mechanism such as:

```
callback
Promise
event
```

Node frequently combines both:

> Start an I/O operation without blocking the main thread, then process its completion asynchronously later.
> 

---

# §16 · What Can Block Node? ⭐

Asynchronous Node APIs do not magically prevent **your own JavaScript** from blocking the event loop.

Examples:

```jsx
fs.readFileSync(...)
```

inside a request handler,

a huge:

```jsx
JSON.parse(...)
```

or:

```jsx
while (...) {
    // millions/billions of operations
}
```

If the main event-loop thread is occupied with CPU work:

```
Request A → heavy CPU work
                  ↓
Request B ─────── waits
Request C ─────── waits
Request D ─────── waits
```

Common solutions:

```
async I/O APIs
Worker Threads for CPU-heavy JS
separate services/processes
breaking/chunking expensive work where appropriate
```

The original notes correctly identify this as Node's main concurrency weakness.

---

# The 8 Answers You Should Be Able to Say Cold

### 1. What is Node.js?

> “Node.js is a JavaScript runtime built around V8 and an event-driven, non-blocking I/O model.”
> 

### 2. How is Node concurrent if JavaScript is single-threaded?

> “Only the main JavaScript execution is single-threaded. Node can delegate or monitor asynchronous I/O elsewhere and run its callback later through the event loop.”
> 

### 3. What does the event loop do?

> “It coordinates execution of callbacks when asynchronous operations, timers and other events become ready.”
> 

### 4. `nextTick` vs `setImmediate`?

> “`nextTick` runs before the event loop continues; `setImmediate` runs in the check phase.”
> 

### 5. Cluster vs Worker Threads?

> “Multiple processes scale server execution across cores; Worker Threads are mainly for CPU-intensive JavaScript.”
> 

### 6. What is Express middleware?

> “A function in the request-response pipeline that receives `req`, `res` and `next`; it can process the request, terminate it, or call `next()` to continue.”
> 

### 7. Why streams?

> “Streams process data chunk by chunk, avoiding the need to keep an entire large payload in memory.”
> 

### 8. Why is CPU-heavy work dangerous in Node?

> “Because long synchronous computation occupies the event-loop thread, preventing it from serving other JavaScript callbacks.”
> 

---

For **today's Backend 1 block**, §§1–10 are the priority. Streams and Promises are already solid material in your notes and can be revised later rather than expanding today's session.

## §6 · Express essentials

**Q. Why Express, over the raw http module?**`http` gives you one request handler and nothing else — you’d hand-parse URLs, methods, and bodies. Express adds **routing**, **middleware**, and helpers (`res.json`, `req.body`, status chaining) in a minimal, unopinionated layer. That’s the whole pitch — say it in two sentences, not ten.

**Q. What is middleware? What does `next()` do?**
Functions with the signature `(req, res, next)` that run **in registration order**, forming a pipeline in front of your route handler. Each either ends the response or calls `next()` to pass control on; `next(err)` skips straight to the **error middleware** (the 4-argument one). Forget both → the request hangs forever. Kinds worth naming: built-in (`express.json`, `express.static`), third-party (cors, morgan, helmet), custom (auth checks, logging), and error-handling `(err, req, res, next)`.

**Q. Why separate `app` from `server`?**
Define routes/middleware on `app` and export it; a separate `server.js` calls `app.listen()`. The reason to say: **testability** — Supertest can exercise the exported app without opening a real port — plus reuse and cleaner startup.

**Q. `req.params` vs `req.query` (vs `req.body`)?**
For `GET /users/42?page=2` on route `/users/:id` → `req.params.id === '42'` (path segments), `req.query.page === '2'` (after the `?`). `req.body` is the JSON payload of POST/PUT/PATCH — and only exists after `app.use(express.json())`. All arrive as **strings**.

**Q. PUT vs PATCH (and vs POST)?PUT** replaces the entire resource — you send the full object; idempotent (same call twice, same result). **PATCH** updates only the fields you send. **POST** creates — not idempotent (twice = two records). Full verb list if asked: GET, POST, PUT, PATCH, DELETE, plus HEAD and OPTIONS.

**Q. Explain RESTful services in three sentences.**
Resources are nouns in URLs (`/orders/7`); HTTP methods are the verbs; status codes report outcomes (200, 201, 400, 401, 404, 500). The server holds **no client state** between requests — each request is self-contained (“stateless”). JSON in, JSON out.

**Q. How do you implement pagination?**
Offset-based: `GET /products?page=3&limit=20` → `.skip((page-1)*limit).limit(limit)` in Mongo, returning metadata (`total`, `page`, `totalPages`). Then name the upgrade: **cursor-based** (`?after=<lastId>`) — stays fast on huge collections and doesn’t skip or duplicate items when data changes mid-scroll, whereas a large offset skip gets slower and slower.

**Q. The package glossary** *(easy-to-fail opener — one line each)*
- **nodemon** — dev-only auto-restart on file change
- **pm2** — production process manager: cluster mode, crash restarts, logs
- **morgan** — HTTP request logger middleware
- **cors** — sets the headers that let a browser on another origin call your API
- **dotenv** — loads `.env` into `process.env`
- **body-parser** — request-body parsing; now built in as `express.json()` / `express.urlencoded()`
- **cookie-parser** — populates `req.cookies`
- **helmet** — sets a dozen security headers in one line
- **multer** — handles `multipart/form-data`, i.e. file uploads
- **bcrypt** — salted, deliberately slow password hashing

---

## §7 · Error handling

**Q. How do you handle errors in async/await Express routes?** *(the one they actually ask)*
try/catch and forward to the error middleware:

```jsx
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (err) { next(err); }              // → error middleware
});

app.use((err, req, res, next) => {          // central error handler — registered LAST
  console.error(err);
  res.status(err.status || 500).json({ error: err.message });
});
```

One extra sentence for depth: in Express 4 an uncaught promise rejection **hangs the request** — teams wrap handlers in a tiny `asyncHandler` — while Express 5 forwards rejections to the error middleware automatically.

**Q. How do you create custom errors?**
Extend `Error` so it can carry a status:

```jsx
class NotFoundError extends Error {
  constructor(msg) { super(msg); this.status = 404; }
}
```

Then `next(new NotFoundError('no such user'))` and the central handler reads `err.status`. Add the distinction interviewers like: **operational errors** (bad input, missing record — handle them) vs **programmer errors** (bugs — crash loudly, fix the code).

**Q. What about uncaught exceptions and unhandled rejections?**
Last-resort hooks: `process.on('uncaughtException')` and `process.on('unhandledRejection')` → **log it, close connections, `process.exit(1)`**, and let PM2 restart the process. Spoken principle: *“after an uncaught exception the process is in an unknown state — you restart it, you don’t limp on.”*

---

## §8 · Modules & Buffers

**Q. CommonJS vs ES Modules?CJS:** `require()` / `module.exports` — Node’s original system; loads synchronously at runtime; works everywhere. **ESM:** `import` / `export` — the language standard; enabled with `"type": "module"` or `.mjs`; imports are static (analyzable, tree-shakable) and top-level `await` works. Interview line: *“CJS resolves at runtime, ESM at parse time.”* Interop footnote: `import`-ing a CJS package works; `require`-ing an ES module doesn’t (historically), which is why some npm packages ship both.

**Q. How does module caching work?**
The first `require('./db')` executes the file and caches `module.exports` keyed by resolved path; every later require returns **the same object** without re-running anything. Two consequences to say: exporting an object gives you a free **singleton** (one DB connection shared app-wide), and mutating an imported object is visible to every other importer.

**Q. How do Buffers work?**
Raw binary memory allocated **outside V8’s heap** — Node’s answer to JavaScript having no binary type. You meet them in fs, TCP, and crypto: streams emit Buffer chunks. API to cite: `Buffer.from('hi')`, `Buffer.alloc(16)`, `buf.toString('utf8' | 'hex' | 'base64')`.

---

## §9 · One-liners (Tier 2 — don’t study, just don’t go blank)

- **npm vs npx** — npm installs packages; npx runs one without a permanent install (`npx create-react-app`).
- **package.json vs package-lock.json** — declared dependencies + scripts vs the exact resolved version tree; the lock file makes installs reproducible, so it gets committed.
- **Semver `^4.2.1` vs `~4.2.1`** — major.minor.patch; `^` accepts minor+patch updates, `~` patch only.
- **Environment variables / NODE_ENV** — config and secrets live in `process.env`, never in code; `NODE_ENV=production` turns off dev behavior and makes Express faster.
- **readFile vs readFileSync** — async (thread pool) vs blocking; sync versions are fine in startup scripts, never inside request handlers.
- **bcrypt** — passwords are salted and hashed with a deliberately slow algorithm; you compare hashes, you never decrypt.
- **helmet** — one middleware, ~a dozen security headers (CSP, X-Frame-Options, …).
- **CORS** — browsers block cross-origin API calls by default; the `cors` middleware sends `Access-Control-Allow-*` headers permitting your frontend’s origin.
- **PM2** — `pm2 start app.js -i max` = cluster mode; `pm2 reload` = zero-downtime deploy; auto-restarts on crash.

---

## Self-test — the 12 you must not fumble

Cover the file. Say each aloud; anything shaky goes back on the star list.

1. Node in one minute (V8 + libuv + event loop)
2. Event-loop phases, in order
3. nextTick vs setImmediate (+ the inside-an-I/O-callback trap)
4. The output-order question (sync → nextTick → promises → timers → check)
5. “Single-threaded, yet thousands of concurrent requests” — why
6. Cluster module + `pm2 start -i max`
7. Worker threads vs cluster (CPU-bound vs I/O scaling)
8. Streams: four types, backpressure, pipeline over pipe
9. Promise.all / allSettled / race / any
10. The sequential-await trap → Promise.all
11. Middleware order, `next(err)`, central error handler
12. CJS vs ESM + module caching = free singleton

*Friday 10:30 PM block = §7–§9 fresh, then stars only. Nothing unstarred gets re-read.*

# MongoDB — Placement Notes

*Rebuilt from the learning-zone MongoDB question bank. Roughly 45 questions cut to the ~20 that get asked, rewritten in say-it-aloud form and explained from first principles rather than assuming you know the internals.*

**How to use this**

- Same method as the Node notes: read the Q → answer aloud → check → ⭐ star the fumbles.
- Today's afternoon block (3:00–4:30) is §1–§4. The evening block is the harder Devinterview set plus writing five pipelines from your own project's schemas.
- Everything here is one of three things: asked directly in campus interviews, needed to understand something that is, or ammunition for *"why MongoDB in your project?"* — which for a MERN candidate is close to guaranteed.
- Deliberately cut: profiler levels, journaling internals, storage-engine comparison, ObjectId byte layout, capped collections, `snapshot()`, compact, splitting, namespaces, disk-write intervals, the 32-bit limit, the full SQL-Server comparison table. Real Mongo administration, zero placement ROI.

---

## §1 · The model — documents, collections, BSON

**Q. What is MongoDB?**
A document-oriented NoSQL database. Instead of tables and rows it stores **documents** (JSON-like objects) inside **collections**. There's no fixed schema — two documents in the same collection can have different fields. Because a document already looks like the object your application works with, there's no translation layer between your code and storage; in a MERN app the shape you `res.json()` is essentially the shape on disk.

**The vocabulary map** — say this when asked to compare with SQL:

| SQL | MongoDB |
| --- | --- |
| Database | Database |
| Table | **Collection** |
| Row | **Document** |
| Column | **Field** |
| Primary key | **`_id`** (auto-created, auto-indexed, unique) |
| JOIN | `$lookup`, or embed the data instead |

**Q. What is BSON, and why not plain JSON?**
BSON = **Binary JSON**, the format MongoDB actually stores. It's JSON's data model plus (a) binary encoding, so it's faster to scan and skip fields, and (b) **more types than JSON has** — proper dates, 32/64-bit integers, doubles, binary blobs, and ObjectId. JSON only has strings and one number type, which is why a date in raw JSON is just a string. You write JSON, Mongo stores BSON.

**Q. Is MongoDB really schema-less?**
Careful, this is a trap question — the accurate answer is **"dynamically typed schema, not no schema."** The *database* doesn't enforce structure, but your application always assumes one. In practice you enforce it in the app layer — which is exactly what **Mongoose schemas** do in your projects. Good line: *"Mongo doesn't enforce a schema; Mongoose does, at the application level, and that's usually where you want it."*

**Q. What are the types of NoSQL databases?** *(one line each, then move on)***Document** (MongoDB, CouchDB) · **Key-value** (Redis, DynamoDB) · **Column-family** (Cassandra, HBase) · **Graph** (Neo4j). Know which bucket MongoDB and Redis fall into — that's the whole ask, and it connects to Tuesday's Redis block.

---

## §2 · Indexing — the highest-yield topic in this file

**Q. What is an index and why does it exist?**
Without an index, a query forces a **collection scan** — MongoDB opens every single document to see if it matches. Fine for 100 documents, fatal for a million. An index is a separate **B-tree** structure holding one field's values in sorted order, each pointing back to its document. Sorted order means the database can binary-search instead of scanning: roughly O(log n) instead of O(n).

Say it as: *"An index trades write speed and disk space for read speed."*

**Q. What's the cost? Why not index everything?** *(the follow-up that separates candidates)*
Every index must be **updated on every insert, update, and delete** — ten indexes means ten extra B-tree writes per insert. They also consume RAM and disk. So you index the fields you actually **query, sort, or join on**, and nothing else.

**Q. What index types should I know?**

- **Single field** — `db.users.createIndex({ age: 1 })`. `1` = ascending, `1` = descending. For a single-field index the direction rarely matters; Mongo can walk a B-tree either way.
- **Compound** — an index on several fields: `createIndex({ city: 1, age: 1 })`. **Field order matters** — see below.
- **Multikey** — what you get automatically when you index an **array** field. Mongo creates one index entry *per element*, so `{ tags: ["node","mongo"] }` indexed on `tags` is findable by either tag.
- **Text** — for string search (`$text` operator); tokenizes words and strips stop words.
- **Hashed** — used for hash-based sharding, spreads writes evenly.
- *Geospatial exists; name it only if asked.*

**Q. Why does field order matter in a compound index?** *(favourite follow-up)*
Think of a phone book sorted by **(last name, first name)**. You can look up "everyone named Khan" and "Khan, Shadab" — but you *cannot* efficiently find "everyone named Shadab" regardless of surname, because first names are only sorted *within* each last name. Same rule for indexes: an index on `{city, age}` serves queries on `city` alone or `city + age`, but **not** `age` alone. This is called the **prefix rule** — a compound index supports any leading prefix of its fields.

**Q. What is a covered query?**
A query that MongoDB answers **entirely from the index, never touching the documents** — possible when every field in the filter *and* every field returned lives in that index. Fastest possible read. Usually needs `_id: 0` in the projection, since `_id` comes back by default and isn't in your index.

**Q. Index properties worth naming**

- **Unique** — rejects duplicate values (`_id`'s index is unique automatically). This is how you enforce "one account per email."
- **TTL** — auto-deletes documents after N seconds; the field must be a date. Real use: sessions, OTPs, password-reset tokens, cached results. Very citable in a project discussion.
- **Sparse / partial** — index only documents that have the field, or only those matching a filter. Smaller index, less overhead.

**Q. What if the index doesn't fit in RAM?**
Mongo falls back to reading index pages from disk, which is orders of magnitude slower — the practical answer to "why did my query get slow at scale."

---

## §3 · Schema design — embed vs reference

This is the section that makes you sound like you've built something, because it's a *judgement* topic, not a memorization one.

**Q. Embedding vs referencing — explain both.Embedded (denormalized):** the related data lives *inside* the parent document — a user with an `addresses: [...]` array. One query gets everything; no join.
**Referenced (normalized):** the child lives in its own collection and the parent stores its `_id` — two queries, or a `$lookup`.

**Q. How do you choose?** *(this is the real question)*
Embed when: the relationship is **one-to-few**, the child is **always read with the parent**, and it **doesn't grow unboundedly**. Reference when: the child is **large or unbounded** (a post's comments), **shared between parents** (an author referenced by many books), or **queried on its own**.

The failure mode to name: an embedded array that grows forever eventually hits the **16 MB document limit**, and every read drags the whole thing into memory. *"Comments on a post get referenced, not embedded"* is the canonical example.

**Q. Should I normalize data in MongoDB?**
Mostly **no** — and this is the deliberate inversion of everything DBMS teaches. Relational design normalizes to eliminate redundancy because joins are cheap. MongoDB has no cheap joins, so it **denormalizes for read performance** and accepts controlled duplication. The tradeoff to state honestly: duplicated data must be updated in multiple places, so you're trading write complexity for read speed.

**Q. How do you do primary key / foreign key relationships?**
There are no foreign keys and **no referential integrity enforcement** — nothing stops you from storing an `_id` that points to a deleted document. You either embed, or store the `_id` and resolve it with `$lookup` / a second query / Mongoose's `.populate()`. Naming that missing guarantee as a real tradeoff scores well.

**Q. What is `_id`?**
The primary key, unique, indexed, auto-created if you don't supply one. Default value is an **ObjectId**: 12 bytes containing a 4-byte creation timestamp plus random and counter bytes. The only useful consequence: **ObjectIds are roughly time-ordered**, so sorting by `_id` approximates sorting by creation time — and `getTimestamp()` extracts the date. That's all you need; the byte layout is trivia.

---

## §4 · Aggregation

**Q. What is the aggregation pipeline?**
A sequence of **stages**; documents flow through, each stage transforming them and passing results to the next. It's how you do the work `GROUP BY`, `JOIN`, and computed columns do in SQL. Mental model: *Unix pipes for documents.*

js

```jsx
db.orders.aggregate([
  { $match:  { status: "A" } },                                  // filter    ≈ WHERE
  { $group:  { _id: "$cust_id", total: { $sum: "$amount" } } },  // group     ≈ GROUP BY
  { $sort:   { total: -1 } },                                    // sort      ≈ ORDER BY
  { $limit:  5 }                                                 // cap       ≈ LIMIT
]);
```

**The stages you must be able to use** — `$match` (filter), `$group` (aggregate; `_id` is the grouping key), `$sort`, `$limit`, `$skip`, `$project` (choose/rename/compute fields), `$lookup` (left outer join to another collection), `$unwind` (split an array into one document per element).

**Accumulators inside `$group`:** `$sum`, `$avg`, `$min`, `$max`, `$count`, `$push` (collect into an array), `$addToSet` (collect, no duplicates), `$first`, `$last`.

**The optimization point worth volunteering:** put `$match` **first**. Filtering early means later stages process far fewer documents, and a leading `$match` can use an index — mid-pipeline stages generally can't. Same instinct as pushing a WHERE clause down.

- Evening block: write five of these against your own project's collections, from memory. Suggested set — count orders per user, average rating per product, `lookup‘joiningtwoofyourcollections,atop−10with‘sort` + `limit‘,anda‘project` reshaping documents for an API response. This doubles as Aug 7 project-deep-dive rehearsal.*
    
    lookup‘joiningtwoofyourcollections,atop−10with‘lookup` joining two of your collections, a top-10 with `
    
    limit‘,anda‘limit`, and a `
    

---

## §5 · Scaling — replication vs sharding

These two get conflated constantly. **Replication = copies of the same data. Sharding = different data on different machines.** Availability versus capacity.

**Q. What is replication / a replica set?**
A **replica set** is a group of mongod processes holding the same data: one **primary** (takes all writes) and several **secondaries** (copy the primary's changes). If the primary dies, the secondaries hold an **election** and one is promoted — automatic failover, no manual intervention. Purpose: **redundancy and high availability**, plus optional read scaling by routing reads to secondaries.

**Q. How does replication actually propagate data?**
Through the **oplog** (operations log) — a rolling capped collection on the primary recording every data-modifying operation. Secondaries continuously read the primary's oplog and replay it. Worth knowing: oplog entries are **idempotent**, so replaying one twice is harmless — that's what makes recovery safe.

**Q. What is sharding?Horizontal scaling**: split one collection's documents across multiple machines by a **shard key**, so each shard holds a subset. Used when data or throughput exceeds what one server can hold. Three components: **shards** (each usually a replica set), **mongos** (the router the app talks to — it knows which shard has what), and **config servers** (metadata).

**Q. Vertical vs horizontal scaling?**
Vertical = a bigger machine (more RAM/CPU) — simple, but there's a ceiling and cost climbs steeply. Horizontal = more machines — no ceiling, but real complexity in infrastructure. MongoDB is built for horizontal; traditional SQL leans vertical. ← Connects straight to Wednesday's scaling block and your 10x-traffic mock.

---

## §6 · Transactions & ACID

**Q. Does MongoDB support ACID?**
Yes, with nuance — give the two-part answer. **Single-document operations have always been atomic**, and because Mongo encourages embedding, one document often contains everything a relational design would spread over several tables — so many apps never need more. Since **v4.0**, MongoDB also supports **multi-document ACID transactions** (replica sets in 4.0, sharded clusters in 4.2).

**ACID itself** — worth having crisp since your DBMS list asks it too: **Atomic** (all or nothing), **Consistent** (valid state to valid state), **Isolated** (concurrent transactions don't see each other's partial work), **Durable** (committed = survives a crash).

**Q. When would you use a transaction — and why sparingly?**
Use it when a single logical operation must span multiple documents or collections — the standard example being transfer money: debit one account, credit another, both or neither. Use sparingly because transactions **cost performance** (holding a snapshot pressures the cache), have a **default 60-second limit**, and are usually a hint that your schema should have embedded that data instead. The mature line: *"If I need transactions everywhere, my schema is fighting the document model."*

---

## §7 · The question you will actually be asked

**"Why did you choose MongoDB for your project?"**

Structure the answer in three beats — reason, tradeoff, alternative:

1. **Reason** — the schema was still moving while I built it, and the data fit the document shape: things I read together, I stored together, so most reads are a single query with no joins.
2. **Tradeoff I accepted** — no enforced referential integrity and no cheap cross-collection joins. I handle relationships in the application layer with Mongoose, and duplicated data has to be updated in more than one place.
3. **When I'd choose otherwise** — if the data were highly relational with many-to-many joins, or if multi-entity transactions were central (payments, ledgers, bookings), I'd take PostgreSQL for the enforced integrity and mature transaction support.

Naming a genuine weakness scores higher than defending the choice. Interviewers are testing judgement, not loyalty.

**Related, if pressed on SQL vs NoSQL generally:** SQL = fixed schema, strong integrity, powerful joins, vertical scaling, ACID across tables. NoSQL = flexible schema, denormalized reads, horizontal scaling, weaker cross-document guarantees. Neither is "better" — they optimize for different access patterns.

---

## Self-test — the 12 that matter

Cover the file, say each aloud.

1. Document / collection / field, and the SQL vocabulary map
2. BSON vs JSON — and one type BSON has that JSON lacks
3. "Schema-less" — the accurate version, and where Mongoose fits
4. Why an index makes reads fast (B-tree, sorted, no collection scan)
5. Why not index everything
6. Compound index field order — the phone book
7. Covered query
8. Embed vs reference, with the rule of thumb and the 16 MB failure mode
9. A four-stage pipeline out loud, and why `$match` goes first
10. Replication vs sharding in one sentence
11. Replica set failover and the oplog
12. "Why MongoDB for your project" — all three beats

⭐ Stars go into tomorrow's queue. And after the evening block: five pipelines written from memory against your own schemas — no tutorial data.

## §8 · Mongoose vs the Mongo shell

*Added because your projects are written in Mongoose, and interviewers question you on your code. The shell is what the database understands; Mongoose is what you actually typed.*

### The one idea to get first

The **shell** (and the raw `mongodb` Node driver) talks to the database directly. No schema, no validation, no types — whatever you send is what gets stored.

**Mongoose is an ODM** (Object Document Mapper): a library that sits *in your Node process*, in front of the driver. It adds a **schema layer that MongoDB itself doesn't have** — structure, types, validation, defaults, hooks, relationship helpers. Nothing Mongoose enforces exists in the database; it's all enforced in your application before the query is sent.

Spoken answer to *"What does Mongoose give you over the native driver?"* — **schemas and type casting, validation, middleware hooks, `populate()` for references, and a chainable query builder.** Say those five and you're done.

### Setup — no shell equivalent, this is Mongoose-only

js

```jsx
const mongoose = require('mongoose');
await mongoose.connect(process.env.MONGO_URI);

const userSchema = new mongoose.Schema({
  name:  { type: String, required: true, trim: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  age:   { type: Number, min: 0, default: 18 },
  posts: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Post' }]
}, { timestamps: true });          // adds createdAt / updatedAt automatically

const User = mongoose.model('User', userSchema);   // collection becomes "users"
```

Note the pluralize-and-lowercase rule: model `User` → collection `users`. A common "why is my collection empty?" bug.

### The command map

| Task | Mongo shell | Mongoose |
| --- | --- | --- |
| Insert one | `db.users.insertOne({...})` | `User.create({...})` or `new User({...}).save()` |
| Insert many | `db.users.insertMany([...])` | `User.insertMany([...])` |
| Find all | `db.users.find({...})` | `User.find({...})` |
| Find one | `db.users.findOne({...})` | `User.findOne({...})` |
| Find by id | `db.users.findOne({_id: ObjectId("...")})` | `User.findById(id)` |
| Update one | `db.users.updateOne(f, {$set:{...}})` | `User.updateOne(f, {...})` |
| Update many | `db.users.updateMany(f, {$set:{...}})` | `User.updateMany(f, {...})` |
| Update + return doc | `db.users.findOneAndUpdate(f, u, {returnDocument:"after"})` | `User.findByIdAndUpdate(id, u, { new: true })` |
| Delete one | `db.users.deleteOne(f)` | `User.deleteOne(f)` / `User.findByIdAndDelete(id)` |
| Delete many | `db.users.deleteMany(f)` | `User.deleteMany(f)` |
| Count | `db.users.countDocuments(f)` | `User.countDocuments(f)` |
| Sort / limit / skip | `db.users.find().sort({age:-1}).limit(10)` | `User.find().sort({age:-1}).limit(10)` |
| Pick fields | `db.users.find({}, {name:1, _id:0})` | `User.find().select('name -_id')` |
| Aggregate | `db.users.aggregate([...])` | `User.aggregate([...])` — **identical** |
| Create index | `db.users.createIndex({email:1},{unique:true})` | declared in the schema: `unique: true` |
| Join | `$lookup` stage | `.populate('posts')` |

Names mostly match — the shape of a query is the same object either way. `$set`, `$push`, `$inc`, `$match`, `$group` are all identical, because Mongoose passes them straight through.

### The seven differences that actually matter

**1.**  `set` is implied.** Shell: `updateOne(f, { $set: {name:"x"} })` — omit `
set` and you *replace the whole document*. Mongoose wraps top-level fields in `$set` for you, so `User.updateOne(f, {name:"x"})` is safe. Good thing to know; dangerous habit if you move to the shell.

**2. `{ new: true }` — the classic gotcha.** `findByIdAndUpdate` returns the document **as it was before the update** by default. If your API returns the "updated" user and it looks stale, this is why. Pass `{ new: true }`.

**3. Validation doesn't run on updates.** Schema rules (`required`, `min`, `enum`) fire on `.save()` and `.create()` — **not** on `updateOne` / `findByIdAndUpdate`, because those go straight to the database. Opt in with `{ runValidators: true }`. Frequently asked, rarely known.

**4. Automatic type casting.** The shell needs `ObjectId("650a...")`; Mongoose casts the string `"650a..."` to an ObjectId itself using the schema, and casts `"25"` to the number 25. Convenient — and it throws a **CastError** on garbage, which is often your 500-instead-of-400 bug.

**5. Strict mode silently drops unknown fields.** Send `{ name: "x", hacker: true }` where `hacker` isn't in the schema, and Mongoose discards it. The shell would happily store it. This is a *feature* — it's schema enforcement doing its job.

**6. Documents aren't plain objects.** Mongoose returns hydrated Documents carrying `.save()`, getters, virtuals, and change tracking. That costs memory and time. For read-only endpoints, `.lean()` returns plain JS objects and is **noticeably faster**. Worth volunteering in a performance conversation.

**7. Queries are thenables, not executed.** `User.find({...})` builds a Query object — it doesn't run until you `await` it (or call `.exec()`). That's what makes the chaining work: each `.sort().limit().select()` refines the query before it's sent as one command.

### `populate()` vs `$lookup`

MongoDB has no foreign keys, so a reference is just an ObjectId sitting in a field. Resolving it:

js

```jsx
// Mongoose — reads ref:'Post' from the schema, then fetches the posts
const user = await User.findById(id).populate('posts');

// Shell / aggregation equivalent
db.users.aggregate([
  { $match: { _id: ObjectId("...") } },
  { $lookup: { from: "posts", localField: "posts", foreignField: "_id", as: "posts" } }
]);
```

**The difference that gets asked:** `$lookup` is a **single database operation** — the join happens server-side. `populate()` issues a **second query** from your app and stitches the results together in Node. Convenient, but it's an extra round trip, and inside a loop it becomes the N+1 query problem. If asked which is faster: `$lookup`.

### Mongoose-only features with no shell equivalent

- **Middleware / hooks** — `userSchema.pre('save', fn)` runs before every save. **This is where password hashing lives in your project** — hash in a `pre('save')` hook so it happens no matter which route saves the user. Excellent thing to cite in an auth discussion.
- **Instance & static methods** — `userSchema.methods.comparePassword`, `userSchema.statics.findByEmail`.
- **Virtuals** — computed fields that aren't stored (`fullName` from first + last).
- **`__v`** — the version key Mongoose adds for internal concurrency tracking. If asked what that mystery field is: that's it.

### Two extra self-test lines

1. Five things Mongoose adds over the native driver
2. `populate()` vs `$lookup` — where the work happens, and why `$lookup` is faster

# Authentication — Placement Notes

*Written for what actually gets asked. Structured around the ByteByteGo two-parter you're reading, but the explanations here stand alone — and several sections are anchored to your own project's code, because that's where your interview answers should come from.*

**How to use this**

- Aloud-first, stars only, same as every other file.
- This afternoon's block: §1–§5. Tonight's 22:15 block: §6–§9 plus the flow drawing (§6 *is* the drawing exercise). One-liners in §10 are read-once material.
- Deliberately cut: bcrypt internals (your `pre('save')` discussion covered it), SAML, Kerberos, biometrics, session fixation deep-dives. Low campus ROI.

---

## §1 · The problem auth solves

HTTP is **stateless** — every request arrives with amnesia. The server has no built-in memory that request #2 came from the same person as request #1. Authentication is the machinery for re-proving identity on every request without re-typing a password every time.

Every scheme answers one design question: **who keeps the memory?**

- **The server remembers** → sessions. The client carries only a reference number.
- **The client carries proof** → tokens. The server verifies but stores nothing.

Every tradeoff in this file falls out of that single choice. Say that in an interview and you've framed the whole topic before answering anything.

---

## §2 · Session-cookie authentication (the pre-JWT world)

**The flow:**

1. User logs in with credentials
2. Server verifies, then **creates a session record server-side** — in memory, Redis, or a DB — holding the user's identity
3. Server sends back only the **session ID** in a `Set-Cookie` header
4. Browser automatically attaches that cookie to every later request to this domain
5. Server looks the ID up in the session store → knows who you are

**Properties to be able to state:**

- **Stateful** — the server holds a record per logged-in user.
- **Revocation is trivial** — delete the session record and that user is logged out *instantly*, everywhere. This is sessions' superpower; remember it for §4.
- **The cookie contains nothing** — just an opaque random ID. Stealing the cookie is still bad (it *is* the session), but reading it reveals nothing.
- **Scaling is the weakness** — with multiple servers behind a load balancer, server 2 doesn't have the session server 1 created. Fixes: a **shared session store (Redis)** — the standard answer — or **sticky sessions** (LB pins a user to one server; fragile, breaks on server death). This is the bridge to Wednesday's scaling block.

---

## §3 · JWT — JSON Web Token

**Anatomy — three base64url parts joined by dots:** `header.payload.signature`

- **Header** — algorithm + type: `{"alg":"HS256","typ":"JWT"}`
- **Payload** — the **claims**: registered ones (`sub` subject/userId, `exp` expiry, `iat` issued-at) plus anything custom (role, email)
- **Signature** — `HMAC-SHA256(header + "." + payload, secret)`

**The single most-asked JWT fact:** the payload is **encoded, not encrypted**. Anyone who obtains the token can base64-decode it and read every claim — paste one into jwt.io and watch. The signature doesn't hide anything; it only proves **the content wasn't altered** and was issued by someone holding the secret. Consequences you should volunteer: never put passwords or secrets in a JWT, and assume the user can read their own token.

**Why it scales:** verification is pure computation — recompute the signature, compare, check `exp`. **No database, no session store.** Any server holding the secret can verify any token, which is why JWTs suit horizontally-scaled and microservice systems.

**HS256 vs RS256** (one-liner depth): HS256 = one shared secret signs *and* verifies. RS256 = private key signs, **public key verifies** — so twenty microservices can verify tokens without any of them being able to mint one. That sentence is the whole answer.

**Expiry:** `exp` is the only self-defense a stateless token has. Once issued, a JWT is valid until it expires — which sets up the next section.

---

## §4 · Session vs JWT — *the* auth interview question

|  | Session | JWT |
| --- | --- | --- |
| State lives | Server (session store) | Inside the token, on the client |
| Server lookup per request | Yes (store hit) | No — verify signature |
| **Revocation** | **Instant** — delete the record | **Hard** — valid until expiry |
| Horizontal scaling | Needs shared store / sticky LB | Free — any server verifies |
| Client can read contents | No (opaque ID) | Yes (decoded payload) |
| Size per request | Tiny cookie | Larger (whole payload travels every request) |

**The mature framing, which scores more than the table:** each design's strength *is* its weakness. JWT's statelessness is why it scales — and why you can't log someone out early: there's no record to delete. Sessions' server-side state is the scaling burden — and exactly what makes revocation instant. You don't pick a winner; you pick which failure you can live with.

**The follow-up you will get: "so how do you revoke a JWT?"** Three honest answers, in escalating order: (1) keep expiry short so the damage window is small; (2) keep a **blocklist** of revoked token IDs and check it on each request — which quietly reintroduces state, and you should *say so*; (3) verify the user against the DB on each request — which is **exactly what your `checkAuthHard` does**. Your project is a deliberate hybrid: JWT for the transport, a DB check for revocation, trading back some statelessness for control. Present it as a choice, not an accident — that's the difference between "I used JWT because tutorials did" and an engineering answer.

---

## §5 · Where the client stores the token

The question is really "which attack are you exposed to" — and there are exactly two:

**localStorage** — readable by any JavaScript on the page. One successful **XSS** injection and the attacker runs `localStorage.getItem('token')` and exfiltrates it. Convenient, and the least safe common option.

**httpOnly cookie** — the browser attaches it automatically, and **JavaScript cannot read it at all**. XSS can no longer *steal* the token. The cost: because the browser auto-sends it, a malicious site can make your browser fire authenticated requests without reading anything — **CSRF**. Mitigations: the **`sameSite`** cookie attribute (`Lax` is the modern default and kills most CSRF; `Strict` is tighter; `None` requires `Secure`) and CSRF tokens for the remaining cases.

The one-breath version: **"XSS steals what JavaScript can read; CSRF rides what the browser auto-sends. localStorage loses to XSS; cookies need sameSite for CSRF; httpOnly + sameSite + Secure is the sane default."**

Your project already made this choice — `req.cookies?.token` means httpOnly cookie. On Aug 7, present the *reasoning*, not just the fact: "I kept the token in an httpOnly cookie so script injection can't exfiltrate it, and sameSite covers the CSRF side."

(Third option for completeness: in-memory only — safest, dies on refresh, which is why it pairs with refresh tokens in §7.)

---

## §6 · Your login flow, end to end — the drawing

This is the exercise, and the drawing is the Aug 7 answer. Sketch it as boxes and arrows, then narrate it aloud:

1. **Signup** → password never stored raw — hashed in the **Mongoose `pre('save')` hook**, so hashing happens no matter which route creates a user
2. **Login** → `bcrypt.compare` against the hash → on success, **sign a JWT** carrying the userId
3. **Set-Cookie**: httpOnly (+ `secure`, `sameSite` in production) — token leaves via a channel JS can't touch
4. Every request → **`checkAuthSoft`**: read cookie, verify signature; invalid → clear cookie, continue as anonymous (`req.user = null`)
5. Protected routes → **`checkAuthHard`**: require `req.user`, validate the ObjectId, **fetch the user from Mongo** — deleted/banned accounts die here despite holding a valid token — attach the fresh document
6. **Logout** → `clearCookie`. (Say the honest part: this clears *this browser's* cookie; the token itself remains cryptographically valid until `exp` — which is why the DB check matters.)

**The two weaknesses you name before they ask** — volunteering flaws reads as seniority:

- **No refresh tokens** — one long-ish-lived access token; a stolen one lives until expiry
- **Stateless-in-name-only** — the DB check per request re-adds a lookup; at scale you'd cache the user in **Redis with a short TTL**, or check DB only on writes

Being able to say "here's my flow, here are its two weaknesses, here's the fix I'd ship next" is a complete interview answer.

---

## §7 · Refresh tokens — the fix you'd ship next

**The problem:** one token doing everything must be long-lived (or users re-login hourly), and long-lived + stolen = long compromise.

**The split:** two tokens with different jobs.

- **Access token** — short-lived (5–15 min), sent on every request, verified statelessly
- **Refresh token** — long-lived (days/weeks), stored httpOnly, sent **only** to one endpoint (`/refresh`), whose job is minting new access tokens

**Why this wins:** a stolen access token expires in minutes. The refresh token is the crown jewel, but it travels rarely, lives httpOnly, and — the key part — **the refresh endpoint is a checkpoint**: the server verifies it against a stored record, so revocation happens there. Ban a user → refuse the refresh → they're out within one access-token lifetime. You get near-stateless request handling *and* real revocation, paying state costs only once per ~15 minutes instead of per request.

**Rotation** (one-liner for depth): each refresh issues a *new* refresh token and invalidates the old — a replayed old token signals theft and kills the whole session family.

---

## §8 · Passwords — the two-sentence version

Stored as **salted, slow hashes** (bcrypt): one-way, so you `compare`, never decrypt. The **salt** (random, per-user, stored beside the hash) kills rainbow tables; the **cost factor** makes brute force expensive on purpose. If asked why not SHA-256: it's *fast*, and fast is exactly wrong for passwords.

---

## §9 · OAuth 2.0 — one honest paragraph

OAuth is **delegation, not a token format**: letting site A act on your behalf at site B without giving A your B password. "Login with Google" in one breath: your app redirects the user to Google → user consents *on Google's page* (your app never sees the password) → Google redirects back with a one-time **authorization code** → your **backend** exchanges code + client secret for tokens. The four role names, since they get asked: **resource owner** (user), **client** (your app), **authorization server** (Google's login), **resource server** (the API holding the data). OIDC = the identity layer on top of OAuth that standardizes "who is this user." Campus depth ends here.

---

## §10 · Rapid one-liners

- **Authentication vs authorization** — who you are vs what you may do. Auth*n* precedes auth*z*.
- **401 vs 403** — 401: not (validly) logged in, try credentials. 403: logged in, still forbidden. Your `checkAuthHard` returns 401; a role check would return 403.
- **Bearer token** — `Authorization: Bearer <jwt>` header; "bearer" literally means possession = power, which is the whole threat model.
- **Why HTTPS is non-negotiable** — every scheme here transmits a secret (cookie or token); plain HTTP means anyone on the path reads it.
- **Cookie flags** — `httpOnly` (no JS), `secure` (HTTPS only), `sameSite` (CSRF). Know all three by name.
- **CORS + cookies** — cross-origin requests don't send cookies unless the client sets `credentials: 'include'` *and* the server answers with `Access-Control-Allow-Credentials: true` plus an explicit origin (not ). In a MERN app with a separate frontend origin, this is the bug you've likely already fought — claim it.
- **Why not roll your own crypto/auth** — the acceptable answer is "signed libraries, boring algorithms, my code only decides *policy*."

---

## Self-test — aloud, tonight

1. Who keeps the memory — the one-question framing
2. Session flow, five steps
3. Why sessions revoke instantly and scale awkwardly
4. JWT's three parts and what the signature does / doesn't do
5. "Encoded, not encrypted" — and its two consequences
6. HS256 vs RS256 in one sentence
7. The session-vs-JWT table from memory, then the strength-is-the-weakness framing
8. Three ways to revoke a JWT, and which one your project uses
9. XSS vs CSRF, and which storage choice each attacks
10. Your login flow end to end, with the two named weaknesses
11. Refresh token split — why 15 minutes changes everything
12. OAuth in one breath, four role names

# Backend Flows — Placement Notes

*API design, pagination, real-time communication, and message queues. Four topics that share one theme: how services talk to each other, and what happens when one of them is slow. This is the block where backend stops being "how do I write an endpoint" and starts being "how does this behave at scale" — which is exactly the ground campus interviewers probe when they ask about your project.*

**How to use this**

- Aloud-first, star the fumbles, same as always.
- §2 and §4 connect directly to your own project — those are the ones to be able to answer *from* your code, not from theory.
- Tonight's Redis block sits on top of §4; tomorrow's rate-limiter block sits on top of §1.

---

## §1 · API design

**REST in three sentences.** Resources are nouns in the URL (`/orders/7`), HTTP methods are the verbs, status codes report the outcome. Every request is **stateless** — it carries everything the server needs, and the server remembers nothing between calls. JSON in, JSON out.

**Resource naming conventions**, since sloppy URLs are the fastest way to look junior:

```
GET    /users            list
GET    /users/42         one
POST   /users            create
PUT    /users/42         replace entirely
PATCH  /users/42         update some fields
DELETE /users/42         remove

GET    /users/42/orders  nested resource — orders belonging to user 42
```

Plural nouns, no verbs in paths. `POST /users` — never `POST /createUser`. The method already *is* the verb; putting it in the path repeats yourself.

**Status codes worth knowing cold:**

| Code | Meaning | When |
| --- | --- | --- |
| 200 | OK | Successful GET/PUT/PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Malformed input |
| 401 | Unauthorized | Not authenticated — *your `checkAuthHard`* |
| 403 | Forbidden | Authenticated but not permitted |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate, version clash |
| 429 | Too Many Requests | Rate limited — *tomorrow's block* |
| 500 | Internal Server Error | You broke |

**Idempotency** — the concept that ties this together and gets asked. An operation is idempotent if performing it twice has the same effect as once. GET, PUT, DELETE are idempotent; **POST is not**. Why it matters practically: a client whose request times out doesn't know whether the server processed it. Retrying a PUT is safe. Retrying a POST might charge the card twice — which is why payment APIs make you send an **idempotency key**, a client-generated ID the server uses to recognise and ignore duplicates.

You've already met this idea twice: in your Cloudinary delete, where treating "not found" as success made deletion idempotent, and in the OS chapter, where oplog entries are idempotent so replay is safe. Same principle, three places.

**REST vs GraphQL**, one honest paragraph. REST gives you fixed endpoints with fixed shapes, so you frequently **over-fetch** (getting fields you don't need) or **under-fetch** (needing three calls to build one screen). GraphQL gives you one endpoint where the client specifies exactly which fields it wants — solving both, at the cost of a more complex server, harder caching (every query is a different shape), and the risk of expensive nested queries. REST is the default and the safe answer; know GraphQL exists and why someone would reach for it.

---

## §2 · Pagination

Never return an unbounded list. A `GET /files` that returns every row works beautifully on your laptop with 40 records and falls over in production with 400,000.

**Offset-based** — the obvious one, and what you've almost certainly written:

js

```jsx
GET /files?page=3&limit=20
db.files.find().skip((page - 1) * limit).limit(limit)
```

Simple, lets users jump to any page, easy to show "Page 3 of 47." Two flaws, and interviewers ask about both:

**It gets slower as the offset grows.** `skip(100000)` doesn't teleport — the database walks and discards 100,000 documents before returning anything. Page 1 is instant; page 5,000 crawls.

**It's unstable when data changes mid-scroll.** You read page 1 (items 1–20). Someone inserts a new item at the top. You request page 2 — which is now items 21–40 of a *shifted* list, so the old item 20 appears again. Delete instead of insert, and an item vanishes unseen. Users experience this as duplicated or skipped rows in infinite scroll.

**Cursor-based** — the fix:

js

```jsx
GET /files?after=650a3f...&limit=20
db.files.find({ _id: { $gt: lastSeenId } }).sort({ _id: 1 }).limit(20)
```

Instead of "skip 100,000," you say "give me what comes after *this specific item*." The database seeks straight there using the index — **constant time regardless of depth** — and because the cursor names a real item rather than a position, inserts and deletes elsewhere can't shift your window.

The cost: **no random access.** You can't jump to page 47, only forward (and backward) from where you are. Which is exactly why infinite-scroll feeds use cursors and admin tables with page numbers use offsets.

The one-line verdict: **offset for numbered pages over small stable data; cursor for feeds, infinite scroll, and anything large.** And note the natural cursor in Mongo is `_id` itself — ObjectIds are roughly time-ordered, so sorting by `_id` approximates sorting by creation time for free.

---

## §3 · Polling vs webhooks vs WebSockets

All three answer one question: **how does a client find out that something happened?** Choosing between them is a standard interview question, usually disguised as "how would you build live notifications?"

**Polling** — the client asks repeatedly. "Anything new? Anything new?"

```
Client → GET /notifications  → "no"
    (wait 5s)
Client → GET /notifications  → "no"
    (wait 5s)
Client → GET /notifications  → "yes, 1 new"
```

Simple, works everywhere, needs no special infrastructure. But it's wasteful — most requests return nothing — and latency is bounded by the interval: poll every 30 seconds and news can be 30 seconds stale. **Long polling** is the refinement: the server holds the request open until something happens or a timeout hits, then the client immediately re-requests. Fewer empty responses, near-instant delivery, but you're holding connections open.

**Webhooks** — inversion of control. Instead of you asking, **the other server calls you** when the event occurs. You register a URL; they POST to it.

```
Stripe → POST https://yourapp.com/webhooks/payment  { "event": "payment.succeeded" }
```

Efficient (zero wasted requests), real-time, and the standard for **server-to-server** integration — payments, CI pipelines, GitHub events. The catches: your endpoint must be **publicly reachable**, you must **verify the signature** so anyone can't forge events, and you must handle **retries** — providers resend on failure, so your handler has to be idempotent (§1 again). A browser can't receive webhooks; it has no public address.

**WebSockets** — a persistent, **full-duplex** connection. One HTTP request upgrades into a long-lived channel where either side can send at any time, with no request/response ceremony.

```
Client ⇄ Server   (one connection, open, both directions)
```

The right tool for genuine real-time: chat, live cursors, multiplayer, streaming dashboards. The cost is stateful connections — each open socket consumes server memory, and scaling across multiple servers means a shared backplane (Redis pub/sub, typically) so a message arriving at server 2 reaches a user connected to server 1.

**How to choose:**

| Need | Use |
| --- | --- |
| Occasional updates, simplest possible | **Polling** |
| Another *server* must notify you | **Webhook** |
| Continuous two-way, low latency | **WebSocket** |
| Server → browser only, one direction | **SSE** (Server-Sent Events) |

That last row is worth one sentence: SSE is a one-way server-to-client stream over plain HTTP — lighter than WebSockets and auto-reconnecting, perfect for live feeds and progress bars where the client never needs to talk back.

---

## §4 · Message queues

**The problem.** A user uploads a file. You need to generate a thumbnail, scan it, and email them. Done synchronously, the HTTP request stays open for eight seconds while all three run — and if the email service is down, the whole upload fails, even though the *upload* worked perfectly.

**The fix.** Do the essential work, put the rest on a queue, respond immediately.

```
Request → save file → enqueue "process-upload" job → 201 Created   (fast)

                            ↓  (independently, later)

Worker → dequeue → thumbnail → scan → email
```

The shape is always the same: **producer → queue → consumer(s)**. The producer doesn't know or care who does the work, or when.

**What this buys you, and these four are the answer to "why use a queue?":**

**Decoupling** — the producer and consumer don't need to know about each other, or even be running at the same time. Deploy the worker separately, rewrite it in another language, whatever.

**Resilience** — if the email service is down, the job waits in the queue rather than failing the user's request. When the service recovers, the backlog drains.

**Load smoothing** — a traffic spike enqueues 10,000 jobs; workers chew through them at their own pace. Without a queue, the spike hits your database directly.

**Independent scaling** — one worker is slow? Run ten. The web servers are untouched.

**The vocabulary to have ready:**

- **Message queue vs pub/sub** — a queue delivers each message to **one** consumer (work distribution). Pub/sub broadcasts each message to **every** subscriber (notification). Different jobs.
- **Dead letter queue** — where messages go after failing repeatedly, so one poison message doesn't block the line forever.
- **At-least-once delivery** — most queues guarantee a message is delivered *at least* once, not exactly once. So a job can run twice, which means **your consumers must be idempotent**. Third appearance of that word today, and this is why it's worth internalising.
- **Names**: RabbitMQ and SQS for general queues, Kafka for high-throughput event streaming (it's a durable log, not just a queue — consumers can replay from any point), **BullMQ** for Node, which is Redis-backed and the one you'd actually reach for.

**Your project angle**, and this is the interview-usable part: anything you currently do synchronously inside a request that isn't strictly necessary for the response is a queue candidate. In your file-storage app that's post-upload processing — thumbnailing, virus scanning, notification emails. Being able to say *"right now my upload route does X synchronously; I'd move it to a BullMQ job so the response returns immediately and a failed email doesn't fail the upload"* is a strong answer to "what would you improve?"

---

## Self-test — aloud

1. Resource naming — why `POST /users` and never `POST /createUser`
2. 401 vs 403, and 201 vs 204
3. Idempotency — the definition, which methods have it, and why POST needs an idempotency key
4. Offset pagination's two failure modes
5. Cursor pagination — what it fixes and what it gives up
6. Polling vs long polling
7. Webhooks — the inversion, plus the three things you must handle (public URL, signature, retries)
8. WebSockets — what "full-duplex" buys you, and why scaling them needs a backplane
9. The four-row choosing table, from memory
10. Producer → queue → consumer, and the four benefits
11. Queue vs pub/sub
12. At-least-once delivery → why consumers must be idempotent
13. Where a queue belongs in *your* project, and what it fixes

# Redis & Caching — Placement Notes

*The last backend topic before drives. Short by design — this subject has about twenty minutes of genuinely interview-relevant content, and padding it would waste time you don't have.*

**How to use this:** aloud-first, stars only. §5 is the deliverable — everything else exists to let you say §5 convincingly.

---

## §1 · What Redis is

An **in-memory key-value store**. That single phrase carries the whole answer to "why is Redis fast": data lives in RAM, so reads and writes are served without touching disk, no query planner, no joins, no table scans. Sub-millisecond latency isn't clever engineering — it's a consequence of where the data sits.

Two clarifications that separate a real answer from a memorised one:

**It isn't only a cache.** Redis persists to disk through **RDB** (periodic point-in-time snapshots — fast restart, may lose the last few minutes) and **AOF** (append-only log of every write — more durable, slower, larger). So "isn't Redis just a cache?" has a real answer: it can be a durable database, and many teams use it as one.

**It's single-threaded**, and that's a feature. One command executes at a time, so **every Redis command is atomic without locks** — no race conditions, no mutexes. Same event-loop model as Node, and worth saying aloud because the parallel shows you understand both.

**Data structures**, one line each: strings (counters, cached JSON), hashes (objects — fields you can update individually), lists (queues), sets (unique membership), **sorted sets** (leaderboards and sliding-window rate limiters — the one that earns points), plus bitmaps and JSON.

---

## §2 · Caching strategies

*Do not treat these as five unrelated definitions. First ask who owns the read-miss logic, then ask what happens when data is written.*

### Cache-aside (lazy loading) — the default application pattern

The **application owns the cache logic**. On a read, it checks Redis first. A cache hit returns immediately; a miss falls back to the database, then the fresh result is placed in Redis for later requests.

```
read:   check cache → hit? return cached value
                    → miss? read DB → populate cache → return

write:  update DB → invalidate/delete the stale cache key
```

For the write path, **DB write + cache invalidation** is the clean default. The next read misses, fetches the new database value, and repopulates the cache. Updating both DB and cache directly is possible, but now the application must keep two writes consistent if one succeeds and the other fails.

**When to use it:** general application caching where you want simple control and only want frequently requested data to occupy cache memory.

### Read-through — same read result, different owner

With **read-through**, the application asks the caching layer for the data and does not implement the miss-to-database logic itself. On a miss, the caching layer loads the value from the database and stores it before returning it.

The distinction worth remembering is ownership:

```
Cache-aside:  application handles cache miss → DB → cache
Read-through: caching layer handles cache miss → DB → cache
```

### Write strategies

These answer a separate question: **when the underlying data changes, where does the write go and when does the database receive it?**

**Write-through** — the write is propagated through the cache to the database synchronously before success is returned. This keeps cached data closely aligned with the database, but every write pays the extra latency of updating both layers.

**Write-back (write-behind)** — the application writes to the cache first and the database is updated asynchronously later. This gives very fast writes and can absorb bursts, but introduces durability risk: if the cache fails before pending writes reach the database, data can be lost.

**Write-around** — writes bypass the cache and go directly to the database. This avoids filling the cache with data that may never be read. If that data is requested later, the first read will miss and load it into the cache.

### Placement memory

- **Cache-aside:** app manages reads; DB write → invalidate cache; next read refills it.
- **Read-through:** cache layer manages read misses for the app.
- **Write-through:** cache + DB updated synchronously; fresher cache, slower writes.
- **Write-back:** cache first, DB later; fast writes, greater durability/consistency risk.
- **Write-around:** DB only on write; useful for write-heavy, rarely-read data.

> **Interview rule:** Do not say one strategy is universally best. Cache-aside is the common simple default; the write strategy depends on how much the system values write latency, freshness, durability, and cache efficiency.
> 

---

## §3 · Invalidation and eviction

> *"There are only two hard things in computer science: cache invalidation and naming things."*
> 

The problem: the cache holds a copy, the copy goes stale, and the cache has no idea. Two mechanisms:

**TTL** — every key expires after a set time. Simple, and it **bounds staleness** rather than eliminating it: a 60-second TTL means data can be at most 60 seconds old. Usually good enough, and it's what you'd reach for first.

**Active invalidation** — when the underlying data changes, explicitly delete or update the key. Precise, and it requires every write path to remember to do it — which is exactly where bugs live.

**Eviction** kicks in when memory fills, and it's a different question from invalidation: not "is this stale?" but "which key do I drop?" **LRU** evicts the least *recently* used, **LFU** the least *frequently* used. LRU is the common default — and you already know it cold from Chapter 10, including why real implementations approximate it rather than tracking exact recency.

---

## §4 · The four failure modes

*Keep the trigger, the failure, and the fix separate. These names are easy to mix up in interviews.*

**1. Cache stampede (thundering herd / cache breakdown)** — **one popular key expires**, and many requests for that same key arrive at once. They all miss the cache and may query the database simultaneously, turning one expired hot key into a sudden database spike. **Fix:** use a lock or single-flight mechanism so only one request rebuilds the value while the others wait; another option is stale-while-revalidate, where slightly stale data is served while one request refreshes the cache.

**2. Cache avalanche** — **many keys expire around the same time**, causing a large wave of unrelated cache misses to reach the database together. Unlike a stampede, this is not about one hot key; it is about synchronized expiration across many keys. **Fix:** stagger expiration times by adding random **TTL jitter**, for example using roughly 60 minutes ± a random few minutes, so the keys do not all disappear at once.

**3. Cache crash** — the **cache service itself becomes unavailable** because of a node, process, network, or cluster failure. Requests can no longer use the cache and may suddenly fall back to the database, which can then become overloaded. **Fix:** use replication/failover for the cache and protect the database with measures such as rate limiting, circuit breakers, and graceful degradation rather than allowing unlimited fallback traffic.

**4. Cache penetration** — requests repeatedly ask for keys that **do not exist in either the cache or the database**, such as invalid IDs. Every request misses the cache and still performs a useless database lookup, so caching provides no protection. **Fix:** cache the negative result for a short TTL; input validation or a Bloom filter can also reject obviously impossible lookups before they reach the database.

> **Interview memory:** one hot key expires → **stampede**; many keys expire together → **avalanche**; Redis/cache goes down → **cache crash**; requested data does not exist anywhere → **penetration**.
> 

**Separate concern — data consistency:** stale cache data is not one of the four failure modes above, but it is a permanent caching trade-off. The cache and database can temporarily disagree, so the design must define how entries are invalidated or refreshed and how much staleness the feature can tolerate.

---

## §5 · Where Redis goes in your project

*The exit test — say this aloud, it's a direct upgrade to an answer you already have.*

> "Right now `checkAuthHard` hits MongoDB on every protected request to confirm the account still exists and hasn't been banned. That's correct, but it's a database read per request — I've made a stateless JWT effectively stateful.
> 
> 
> I'd cache the user document in Redis keyed by user ID with a **60-second TTL**. Auth then costs one Mongo read per user per minute instead of one per request, and I keep the revocation behaviour I wanted — worst case a banned user stays valid for another 60 seconds. On an explicit ban or account deletion I'd delete the key immediately, so even that window closes."
> 

That's cache-aside with a TTL, applied to your own code, with the staleness tradeoff named. Second candidate if pressed: cache the file-listing endpoint per user, invalidated on upload or delete.

**Two connections worth having ready**, since they close loops from today's other blocks: **BullMQ runs on Redis** — the message queue from the flows notes is a Redis list underneath. And **Redis pub/sub is the standard WebSocket backplane**, which is how a message arriving at server 2 reaches a user connected to server 1.

---

## Self-test — aloud

1. Why Redis is fast — the one-sentence answer
2. RDB vs AOF, and why "isn't it just a cache?" has a real answer
3. Single-threaded → commands are atomic without locks
4. Cache-aside, as a read path and a write path
5. Write-through vs write-back — the safety/speed trade
6. When write-around is right
7. TTL vs active invalidation
8. LRU vs LFU
9. Thundering herd, and one mitigation
10. Cache penetration, and why caching the negative result fixes it
11. Your project's Redis answer, in full — §5

⭐ Stars go to tomorrow. This closes the backend track before drives; what's left is rate limiting and scaling tomorrow, then project prep.

# Rate Limiting — Placement Notes

*Short by design. For campus interviews this is a two-minute topic — you need the why, two algorithms, and the Redis connection. Everything past that is system-design-round material.*

---

## Why

Prevent abuse and DoS, control infrastructure cost, keep usage fair between clients. The response is **429 Too Many Requests**, ideally with a `Retry-After` header telling the client when to try again.

Identity is usually by user ID, API key, or IP address. It sits in **middleware or an API gateway**, before your business logic — you reject early, cheaply.

---

## The algorithms

**Fixed window** — count requests per fixed interval, reset each period.
Simple, two lines with Redis. One real flaw, and it's the asked one: **boundary bursts.** With a limit of 100/minute, a client can send 100 at 11:59:59 and 100 more at 12:00:01 — 200 requests in two seconds, and both windows are technically legal.

**Token bucket** — tokens refill at a steady rate; each request spends one; no tokens means rejection.
Allows **controlled bursts** (a client who's been quiet has tokens saved up) while capping the long-run rate. This is what most production systems use, and the right default answer.

*Two more to name, not study:* **sliding window log** — store a timestamp per request in a Redis sorted set, drop old ones, count the rest. Exact, but costs memory per request. **Sliding window counter** — weighted blend of the current and previous window; approximate but cheap.

---

## Why Redis

The counter must live **outside your app servers**. Three servers each keeping their own count means each allows the full limit, and you've rate-limited nothing — the same shared-state argument as sessions in the auth notes.

Fixed window is two Redis commands:

```
INCR   user:123:requests      → returns the new count
EXPIRE user:123:requests 60   → set on first request only
```

Both are atomic because **Redis is single-threaded**, so no locking is needed even under concurrent requests. Sorted sets (`ZADD` / `ZREMRANGEBYSCORE` / `ZCARD`) implement the sliding window log.

---

## The answer to give

> "I'd use token bucket, because it caps the long-run rate while still allowing legitimate bursts from a client who's been idle. I'd keep the counters in Redis rather than in-process, since with multiple app servers each one would otherwise enforce the limit separately. Fixed window is simpler — just INCR and EXPIRE — but it lets a client double the limit across a window boundary."
> 

---

## Self-test

1. Why rate limit, and what status code
2. Fixed window's boundary-burst flaw
3. Token bucket — what "controlled burst" means and why it's the usual choice
4. Why the counter can't live in your app process
5. `INCR` + `EXPIRE`, and why they're atomic

# Scaling — Placement Notes

*The last backend topic. Everything here exists to support one answer: "your app gets 10× traffic tomorrow — what breaks, and what do you do?" That question, or a version of it, is how campus interviewers probe whether you understand systems or only endpoints.*

Deliberately short. Most of this you already have scattered across the other files — this assembles it.

---

## §1 · The two directions

**Vertical scaling** — a bigger machine. More RAM, faster CPU, more disk. Simple, requires no code changes, and has a hard ceiling: eventually no machine is big enough, and cost climbs faster than capability.

**Horizontal scaling** — more machines. No ceiling, cheaper hardware, and survives a single machine dying. The cost is complexity: your application must become **stateless**, because a user's second request may land on a different server than the first.

That last consequence is the whole reason sessions moved to Redis, and it's the same argument as rate-limiter counters. **Anything held in one server's memory breaks the moment there are two servers.**

---

## §2 · The pieces

**Load balancer** — sits in front of your servers and distributes incoming requests. Algorithms worth naming: **round robin** (in turn), **least connections** (to whoever's least busy), **IP hash** (same client to the same server). It also health-checks, and stops sending traffic to a dead instance.

**CDN** — a network of geographically distributed servers caching static assets: images, CSS, JS, video. Two wins: users are served from somewhere physically near them, and your origin server stops handling traffic that never needed application logic.

**Cache** — Redis in front of the database, for reads that repeat. Covered fully in the Redis notes; the scaling-relevant point is that it takes load off the database, which is almost always the first thing to break.

**Replication** — copies of the same data. Primary takes writes, secondaries take reads. Buys **availability** and read capacity. The cost is replication lag: a read immediately after a write may hit a secondary that hasn't caught up.

**Sharding** — different data on different machines, split by a shard key. Buys **capacity** beyond one machine. The costs are shard-key choice (a bad one creates hotspots) and cross-shard queries (any query without the shard key must hit every shard and merge).

*Replication is availability; sharding is capacity. Production systems do both — each shard is itself a replica set.*

---

## §3 · CAP theorem

Under a **network partition** — machines unable to talk to each other — a distributed system must choose:

- **Consistency** — every read sees the latest write, or fails. Refuse to answer rather than answer wrongly.
- **Availability** — every request gets an answer, possibly stale.

**Partition tolerance isn't a choice** — networks fail, so any distributed system must survive partitions. The real trade is C versus A *during* a partition.

The framing that gets it right: *"CAP isn't 'pick two of three.' Partitions happen to you. The design question is what you do when one occurs."*

Banking picks consistency — better to decline a transaction than to allow a double spend. A social feed picks availability — a slightly stale timeline beats an error page.

---

## §4 · The 10× traffic answer

*Say this aloud until it's one fluent paragraph, not six facts. It's tomorrow's backend mock and a near-certain interview question.*

> **"First I'd measure rather than guess** — find the actual bottleneck with logs and metrics. In most apps it's the database, so I'd start there.
> 
> 
> **Database:** add indexes on the fields I'm actually querying and sorting on, since without them every query is a collection scan. Then cache hot reads in Redis to keep repeated queries off the database entirely. If reads dominate, add read replicas and route reads to them. If the data itself outgrows one machine, shard it.
> 
> **Application layer:** run multiple Node instances behind a load balancer — PM2 cluster mode uses every core, since a single Node process is one thread. That forces the app to be stateless, so sessions and rate-limit counters move to Redis rather than process memory.
> 
> **Request path:** static assets to a CDN so they never touch my servers. Anything slow that isn't needed for the response — thumbnails, emails, processing — moves to a background queue so requests return immediately.
> 
> **And I'd add rate limiting**, so one abusive client can't consume the capacity I just added."
> 

Every element of that is already in your other notes: PM2 cluster from the Node file, Redis from Tuesday, queues and CDN from the flows file, sharding from Mongo, rate limiting from this afternoon. This paragraph is the assembly, not new material.

---

## §5 · Applied to your project — the stronger version

Generic scaling answers are fine. **Answers about your own code are better**, because they prove you've thought about the system you actually built:

> "In my file-storage app the first thing to break would be auth — `checkAuthHard` hits MongoDB on every protected request. I'd cache the user document in Redis with a short TTL, so that becomes one read per user per minute instead of one per request.
> 
> 
> Second would be the upload path — Cloudinary work happens inside the request. I'd move post-upload processing to a queue so the response returns immediately.
> 
> Third, file listings are read constantly and change rarely, so those get cached and invalidated on upload or delete."
> 

Three specific bottlenecks in code you wrote, each with a named fix. That's a better answer than any list of technologies.

---

## Self-test — aloud

1. Vertical vs horizontal, and the ceiling on each
2. Why horizontal scaling forces statelessness — and the two things that must move to Redis
3. Load balancer algorithms, three by name
4. What a CDN removes from your origin
5. Replication vs sharding — availability vs capacity
6. Replication lag, and when it bites
7. CAP — why partition tolerance isn't optional, and what the real trade is
8. **The full 10× answer, in one breath**
9. **The three bottlenecks in your own project, with fixes**

*Backend track closed. What remains is rehearsing your projects out loud — which is the only thing an interviewer will definitely ask about.*

# Docker Fundamentals — Short Notes

## 1. What is Docker?

**Docker is a platform for packaging and running applications in isolated environments called containers.**

An application often depends on a particular runtime, libraries, system packages, and configuration. Docker packages these requirements together so that the application behaves consistently across different machines.

A useful way to think about it:

> **Docker makes the application carry its environment with it.**
> 

This greatly reduces the classic **“works on my machine”** problem.

---

## 2. What is a Container?

A **container** is an isolated environment in which an application process runs.

Unlike a virtual machine, a container does **not** normally contain an entire operating system and kernel. Containers share the host system's kernel while isolating processes, filesystems, networking, and other resources.

```
Application
    ↓
Container
    ↓
Docker Engine
    ↓
Host OS / Kernel
```

### Container vs Virtual Machine

| Container | Virtual Machine |
| --- | --- |
| Shares the host kernel | Has its own guest OS/kernel |
| Lightweight | Heavier |
| Starts quickly | Usually takes longer to boot |
| Typically uses fewer resources | Requires more CPU/RAM/storage |

**Interview line:**

> “A container isolates an application and its environment while sharing the host kernel, whereas a VM virtualizes an entire operating system.”
> 

---

## 3. What is a Docker Image?

A **Docker image** is a read-only package containing everything required to create a container, such as:

- application files
- runtime
- libraries
- dependencies
- configuration

An image itself is **not a running application**.

A container is created **from** an image.

```
Image
  ↓ docker run
Container
```

### Image vs Container

The easiest analogy is:

> **Image = blueprint**
> 
> 
> **Container = instance created from that blueprint**
> 

One image can be used to create many containers.

---

## 4. Docker Images and Layers

Docker images are made of **layers**.

Each layer represents a change to the image, such as installing software, adding application files, or changing configuration.

Layers are reusable, so different images can share existing layers instead of storing identical data repeatedly.

This helps Docker with:

- faster image building
- efficient storage
- faster downloads

---

## 5. Docker Registry

A **registry** stores Docker images.

The most common public registry is **Docker Hub**.

For example:

```bash
docker run nginx
```

If the `nginx` image is not already available locally, Docker can download it from a registry first.

The flow is:

```
docker run nginx
       ↓
Check local images
       ↓
Image missing
       ↓
Pull nginx from Docker Hub
       ↓
Store image locally
       ↓
Create container
       ↓
Start nginx
```

That is why Docker printed:

```
Unable to find image 'nginx:latest' locally
```

It was **not an error**. Docker was simply telling you that it needed to download the image first.

---

## 6. What Does `docker run` Do?

Conceptually:

```bash
docker run IMAGE
```

means:

```
Find/pull image
      ↓
Create container
      ↓
Configure container
      ↓
Start its main process
```

For example:

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

means:

```
-d
Run in the background

--name my-nginx
Give the container the name "my-nginx"

-p 8080:80
Host port 8080 → Container port 80

nginx
Use the nginx image
```

So:

```
Browser
localhost:8080
      ↓
Host port 8080
      ↓
Docker
      ↓
Container port 80
      ↓
nginx
```

---

## 7. Essential Commands Covered

```bash
docker images
```

Shows **images stored locally**.

```bash
docker ps
```

Shows **currently running containers**.

```bash
docker ps -a
```

or:

```bash
docker ps --all
```

Shows **all containers**, including running, stopped/exited, and created containers.

A useful distinction:

```
docker images   → What templates do I have?

docker ps       → What containers are running?

docker ps -a    → What containers exist?
```

---

## 8. Docker Desktop

**Docker Desktop is not Docker itself.** It is a convenient application that bundles Docker tooling and provides a GUI.

On Linux, Docker Desktop includes its own Docker Engine environment.

In Docker Desktop you can visually inspect:

```
Images
Containers
Logs
Ports
Running/stopped state
```

The terminal and Docker Desktop are simply two ways of interacting with the same Docker environment when using the `desktop-linux` context.

---

## Interview Summary

> **Docker packages applications and their dependencies into images. Images are used to create isolated running containers. Containers are lightweight compared with virtual machines because they share the host kernel. Images can be stored in registries such as Docker Hub and are composed of reusable layers. Commands such as `docker run`, `docker images`, and `docker ps` are used to create and manage containers and images.**
> 

# Docker 2 — Containerising Velvet

## 1. Dockerfile

A **Dockerfile** is the recipe Docker uses to build an image.

```docker
FROM node:24-bookworm-slim

WORKDIR /velvet-doc

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### What each line means

**`FROM node:24-bookworm-slim`**

Uses Node.js 24 on a lightweight Debian base.

**`WORKDIR /velvet-doc`**

Creates/sets `/velvet-doc` as the working directory **inside the image**.

**`COPY package*.json ./`**

Copies dependency files first so Docker can cache the dependency-installation layer.

**`RUN npm ci`**

Installs dependencies during image creation. Prefer `npm ci` when `package-lock.json` exists.

**`COPY . .`**

Copies the project from the **build context** into the current image directory.

```
Host project → /velvet-doc inside image
```

**`EXPOSE 3000`**

Documents that the application listens on port `3000`.

**`CMD ["npm", "start"]`**

Command executed when the container starts.

---

## 2. Build Context

When running:

```bash
docker build -t velvet .
```

the final `.` means:

> “Use the current directory as the Docker build context.”
> 

Therefore:

```docker
COPY . .
```

means:

> Copy everything from the current project directory into the current `WORKDIR` inside the image.
> 

The container's `/velvet-doc` directory does **not** appear on your laptop filesystem.

---

## 3. Image vs Container

```
Dockerfile
    ↓ docker build
Image
    ↓ docker run
Container
```

**Image** → packaged, read-only blueprint of the application.

**Container** → running instance of that image.

Build:

```bash
docker build -t velvet .
```

Run:

```bash
docker run -d -p 3000:3000 velvet
```

Here:

```
-d             → run in background
-p 3000:3000   → host port 3000 → container port 3000
velvet         → image name
```

---

## 4. Environment Variables

Secrets should **not** be copied into the Docker image.

Add `.env` to:

```
.dockerignore
```

Example:

```
node_modules
.env
.git
```

Then inject environment variables when the container starts:

```bash
docker run \
  --env-file .env \
  -d \
  -p 3000:3000 \
  velvet
```

Inside Node, they remain available normally:

```jsx
process.env.SECRET_KEY
```

### Interview line

> **“Secrets are injected at runtime instead of being baked into the Docker image.”**
> 

---

## 5. Velvet's Current Container Shape

Velvet's production React build is served by Express, so for now:

```
Docker Container
│
├── Node / Express API
└── React production build
        ↓
External MongoDB + Cloudinary
```

You do **not** need separate React and Node containers for this architecture.

Compose and containerising MongoDB are separate topics for the later Docker session.

---

## 6. Useful Commands

```bash
docker image ls
```

List images.

```bash
docker ps
```

Show running containers.

```bash
docker ps -a
```

Show all containers.

```bash
docker logs <container-id>
```

Inspect application output/errors.

```bash
docker run --rm -it velvet sh
```

Open a shell inside the image/container.

---

## Interview Summary

> **“I containerised Velvet using a Node LTS Debian-slim image. The Dockerfile installs dependencies in a cache-friendly layer, copies the application, exposes the server port and starts Express. The React production build is served by Express, while MongoDB and Cloudinary remain external services. Secrets are injected at runtime rather than stored in the image.”**
> 

# Docker Compose — Compact Placement Notes

## 1. Why Docker Compose?

A **Dockerfile** describes how to build an image for one application.

For example, our Node backend has its own Dockerfile:

```docker
FROM node:24-bookworm-slim

WORKDIR /NODE

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 8000

CMD ["npm", "start"]
```

This answers:

> **How should my Node application image be created?**
> 

But our application does not contain only Node. It also needs MongoDB.

Instead of manually starting each container separately, Docker Compose lets us describe the **complete set of services** in one file.

```yaml
services:
  node:
    build: .
    ports:
      - "8000:8000"

  mongo:
    image: mongo
```

The mental distinction is:

```
Dockerfile
   ↓
How to build ONE image

compose.yaml
   ↓
Which services make up the application
and how they should run together
```

---

## 2. Image vs Container vs Service

An **image** is the packaged blueprint.

A **container** is a running instance of that image.

```
Dockerfile
    ↓ build
Node Image
    ↓ run
Node Container
```

Compose introduces the idea of a **service**.

```yaml
services:
  node:
    build: .

  mongo:
    image: mongo
```

Here we have two services:

```
node service  → Node container
mongo service → MongoDB container
```

A project is therefore not restricted to one Dockerfile or one container.

Different services may:

- use their own Dockerfiles, or
- use ready-made images such as `mongo`

For MongoDB, we do not need to create a Mongo Dockerfile:

```yaml
mongo:
  image: mongo
```

Docker simply uses the existing Mongo image.

---

# 3. Why separate Node and Mongo?

Instead of putting everything into:

```
One container
├── Node
└── MongoDB
```

we use:

```
Node container
      +
Mongo container
```

Each container has one clear responsibility.

This also allows the services to be started, replaced, and managed independently.

---

# 4. Understanding `build: .`

Our Node service contains:

```yaml
node:
  build: .
```

The `.` means:

> Use the current directory as the build context.
> 

Compose finds the Dockerfile there and builds the image required for the `node` service.

Conceptually:

```
compose.yaml
     |
     | build: .
     ↓
Dockerfile
     ↓
Node image
     ↓
Node container
```

Mongo is different:

```yaml
mongo:
  image: mongo
```

There is nothing to build ourselves.

---

# 5. Port Mapping

Suppose Compose contains:

```yaml
ports:
  - "8000:3000"
```

The format is:

```
HOST_PORT : CONTAINER_PORT
```

So:

```
8000 : 3000
```

means:

```
Your computer
localhost:8000
       ↓
Container
port 3000
```

If Express runs inside the container on port `3000`:

```jsx
app.listen(3000);
```

you access it from your computer using:

```
http://localhost:8000
```

If Express itself listens on `8000`, then the mapping can instead be:

```yaml
ports:
  - "8000:8000"
```

---

# 6. Why the YAML dash matters

This is wrong:

```yaml
ports: -"8000:3000"
```

`ports` is a **list**.

Correct:

```yaml
ports:
  - "8000:3000"
```

The indentation and `-` are YAML structure, not Docker-specific decoration.

---

# 7. How Node reaches Mongo

This was the most important practical concept from today's session.

Suppose:

```yaml
services:
  node:
    build: .

  mongo:
    image: mongo
```

Docker Compose places these services on a network where they can identify each other using their **service names**.

Therefore, from Node:

```
mongo
```

acts as the hostname of the Mongo service.

So Mongoose can connect using:

```jsx
mongoose.connect("mongodb://mongo:27017/todo");
```

Break it down:

```
mongodb://mongo:27017/todo
          │       │    │
          │       │    └─ database name
          │       └────── MongoDB port
          └────────────── Compose service name
```

---

# 8. Why not `localhost`?

This is an important Docker interview question.

Inside the Node container:

```
localhost
```

means:

> **The Node container itself.**
> 

It does not mean your Mongo container.

So this would be wrong for two separate Compose services:

```jsx
mongoose.connect("mongodb://localhost:27017/todo");
```

Because Node would effectively look for MongoDB inside itself.

Instead:

```jsx
mongoose.connect("mongodb://mongo:27017/todo");
```

gives:

```
Node container
      |
      | mongo:27017
      ↓
Mongo container
```

---

# 9. Where did `todo` come from?

MongoDB connection strings follow the pattern:

```
mongodb://HOST:PORT/DATABASE
```

Therefore:

```
mongodb://mongo:27017/todo
```

means:

```
Host     → mongo
Port     → 27017
Database → todo
```

`todo` is simply a database name chosen by us.

It could just as easily be:

```
mongodb://mongo:27017/shop
mongodb://mongo:27017/users
mongodb://mongo:27017/myapp
```

We do not need to create an empty `todo` database manually before connecting. MongoDB can create it once data is actually written.

---

# 10. `docker build`

A normal Docker build:

```bash
docker build -t todo-node .
```

takes a Dockerfile and produces an image.

```
Dockerfile
    ↓
docker build
    ↓
Image
```

It does **not start the application**.

---

# 11. `docker compose build`

```bash
docker compose build
```

reads `compose.yaml` and builds the services that contain:

```yaml
build:
```

For:

```yaml
services:
  node:
    build: .

  mongo:
    image: mongo
```

Compose builds the Node image.

Mongo already uses an image, so our own Mongo image does not need to be built.

---

# 12. `docker compose build --no-cache`

Docker normally caches previous build steps so repeated builds are faster.

For example:

```docker
COPY package*.json ./
RUN npm ci
```

may be reused if Docker determines that the relevant input has not changed.

Running:

```bash
docker compose build --no-cache
```

means:

> Rebuild the Compose services from scratch without reusing cached build layers.
> 

We used this when debugging our earlier Dockerfile problem because we wanted to ensure Docker was actually using the **latest Dockerfile**, rather than an older cached build.

This is mainly a debugging/fresh-build tool, not something needed every time.

---

# 13. `docker compose up`

```bash
docker compose up
```

means:

> Start the application described by `compose.yaml`.
> 

With:

```yaml
services:
  node:
    build: .

  mongo:
    image: mongo
```

Compose starts both services:

```
docker compose up
       ↓
 ┌─────────────┐
 │ Node        │
 │ container   │
 └─────────────┘

 ┌─────────────┐
 │ Mongo       │
 │ container   │
 └─────────────┘
```

The logs of both services appear together:

```
node-1  | server started
mongo-1 | Waiting for connections
```

That is exactly what we saw when our setup worked.

---

# 14. `docker compose up --build`

This combines two steps:

```
BUILD
  +
START
```

So:

```bash
docker compose up --build
```

essentially means:

> Make sure the required images are rebuilt, then start the Compose application.
> 

Useful after changing:

- the Dockerfile
- installed dependencies
- other image-building configuration

---

# 15. `docker compose down`

When finished:

```bash
docker compose down
```

stops and removes the containers created for that Compose application, along with its normal Compose network.

Think:

```
docker compose up
        ↓
Application running

docker compose down
        ↓
Application containers removed
```

It does **not mean "delete my Dockerfile" or "delete my source code."**

Your project files remain untouched.

---

# 16. `build` vs `up` vs `down`

This distinction should be completely clear:

```
Dockerfile
    |
    | BUILD
    ▼
  Image
    |
    | UP
    ▼
Running Container(s)
    |
    | DOWN
    ▼
Containers stopped/removed
```

### Build

```bash
docker compose build
```

> Prepare the images.
> 

### Up

```bash
docker compose up
```

> Run the application.
> 

### Down

```bash
docker compose down
```

> Tear down the running Compose application.
> 

---

# 17. What happened in our application?

Our setup was essentially:

```
       Your Computer
            |
      localhost:8000
            |
            ▼
    Node Container
    Express + Mongoose
            |
            |
mongodb://mongo:27017/todo
            |
            ▼
    Mongo Container
        MongoDB
```

Compose was responsible for running both containers and allowing the Node service to reach Mongo using the hostname:

```
mongo
```

That is the central idea to take away from today's practical session.

---

# Placement-Level Distinctions

### Dockerfile vs Compose

> A Dockerfile defines how one image is built. Docker Compose defines which services form an application and how those containers should run together.
> 

### Image vs container

> An image is the packaged blueprint; a container is a running instance of that image.
> 

### `docker build` vs `docker compose build`

> `docker build` explicitly builds an image from a Dockerfile. `docker compose build` builds the services marked with `build:` in the Compose configuration.
> 

### `build` vs `up`

> Build creates images. Up creates and starts containers using those images.
> 

### Why `mongo` instead of `localhost`?

> Each container has its own `localhost`. Compose service names provide network addresses between containers, so the Node container reaches Mongo using `mongo:27017`.
> 

---

# Final Mental Model

For what we learned today, keep this picture:

```
             compose.yaml
                  |
       ┌──────────┴──────────┐
       │                     │
       ▼                     ▼
 Node Service           Mongo Service
  build: .              image: mongo
       │                     │
       ▼                     ▼
  Node Image            Mongo Image
       │                     │
       ▼                     ▼
Node Container  ─────→ Mongo Container
                 mongo:27017
```

And remember:

```
Dockerfile   → How is my image built?

Compose      → What services make up my application?

build        → Create the images.

up           → Start the application.

down         → Tear down the running application.

mongo:27017  → Reach the Mongo service from another
               container on the Compose network.
```

That is the full scope of what we actually practiced today.

## Backend PROVE checkpoint — 30 Aug 2026

Closed-book results:

- Offset vs cursor pagination — **REPAIR → understood**
- Session vs JWT — **PASS**
- Cache-aside read/write path — **PASS**
- WebSockets vs polling — **PASS**
- Backpressure — **PASS**

The only meaningful repair was cursor pagination: use the last ordered key as the continuation point (for example `WHERE id > cursor ORDER BY id ASC LIMIT n`), and use a compound cursor such as `(created_at, id)` when the primary sort key is non-unique. Backend reasoning remains strong-intermediate for fresher interviews; no beginner backend review is needed.