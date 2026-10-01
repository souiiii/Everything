# System Design for Placements

<aside>
🎯

This is the **running system-design page for placement preparation**. The goal is not to memorize architectures, but to learn a repeatable way to turn vague product requirements into a defensible technical design. Every component on this page should therefore be understood in context: what problem it solves, what trade-off it introduces, and what evidence would justify adding it.

</aside>

# Part I — How to Approach a System Design Problem

## What system design is really testing

System design interviews are rarely about discovering one perfect architecture. They test whether you can take an ambiguous problem, reduce it into clear requirements, identify the important constraints, and make reasonable engineering decisions while explaining the trade-offs. A strong answer therefore looks less like a diagram dump and more like a **structured conversation that gradually earns each component in the design**.

The interviewer is usually evaluating several things at once: whether you can clarify the problem before solving it, whether you understand how traffic and data move through a system, whether your database and API choices follow from real access patterns, and whether you can recognize bottlenecks or failure modes before they become production problems. Depth matters, but only after the basic architecture is coherent.

<aside>
🧭

A useful mental model is: **requirements first, architecture second, optimization last**. Do not begin with Redis, Kafka, sharding, or microservices. Begin by understanding what the system must do and what constraints actually matter.

</aside>

## The repeatable interview flow

A good system-design answer can usually follow this sequence:

```
Clarify scope
→ define functional requirements
→ define non-functional requirements
→ estimate scale where useful
→ define APIs / external contracts
→ model the data and access patterns
→ draw the simplest high-level architecture
→ walk the critical read/write flows
→ identify bottlenecks and failure modes
→ scale or deepen only where justified
→ summarize trade-offs and remaining risks
```

The order matters because every later decision should be supported by something established earlier. If you choose a cache, for example, you should be able to point to a read-heavy access pattern or latency requirement that made the cache useful.

## 1. Clarify the scope before designing

Most system-design prompts are intentionally broad. “Design YouTube,” “design a URL shortener,” or “design a chat system” can each describe dozens of features, so attempting to design everything immediately creates an unfocused answer.

Start by asking which user actions matter for this interview. For a URL shortener, the core scope may be creating a short URL and redirecting it. Analytics, custom aliases, expiration, abuse detection, and QR codes may exist, but they should only enter the design if the interviewer wants them.

A useful opening is to separate **must-have behavior from optional behavior**. This gives both you and the interviewer a shared boundary for the rest of the discussion.

## 2. Define functional requirements

Functional requirements describe **what users or other systems must be able to do**. Keep them concrete and externally visible rather than describing internal technologies.

For example, a URL-shortening service may need to create a shortened link, redirect a short code to its original URL, and optionally allow links to expire. These requirements later determine the APIs, data model, and important request flows.

Do not create a huge feature list. In an interview, two to four core functions are usually enough to anchor the design unless the prompt explicitly requires more.

## 3. Define non-functional requirements

Non-functional requirements describe the qualities the system must maintain while providing those features. Typical concerns include latency, availability, consistency, durability, throughput, scalability, and security.

The important step is to **prioritize rather than list everything**. A redirect service may care heavily about low read latency and high availability, while a payment ledger may prioritize correctness and consistency over serving every request immediately. Different priorities lead to different architectures.

This is where trade-offs begin. If the interviewer says the system must remain available during failures, you may later justify replication or graceful degradation. If stale reads are unacceptable, your caching and replication choices need to respect that.

## 4. Estimate scale only when it changes a decision

Back-of-the-envelope estimation is useful because the same design can be perfectly reasonable for ten thousand requests per day and completely unsuitable for ten million requests per second. Estimate only the quantities that help choose architecture: requests per second, read-to-write ratio, storage growth, object size, bandwidth, or expected concurrency.

You usually do not need precise arithmetic. The goal is to establish order of magnitude and expose design pressure. If reads outnumber writes by 100:1, caching and read replicas become more plausible; if the dataset fits comfortably on one database server, sharding should not appear simply because it is a famous system-design topic.

> **Interview habit:** Say what decision an estimate is helping you make. Numbers without architectural consequences add noise rather than depth.
> 

## 5. Define the external interface

Once the required behavior is clear, define the main APIs or communication contracts. This turns vague product behavior into concrete operations and reveals what information the system must accept and return.

For a REST-style service, this may mean a few endpoints with the important request and response fields. For real-time systems, it may involve WebSocket messages or events rather than ordinary request-response APIs. Authentication, idempotency, pagination, and error behavior should be mentioned where they materially affect the design.

The API is not merely decoration. It helps expose whether the later architecture actually supports the required operations.

## 6. Design the data model from access patterns

Before choosing a database, identify the main entities and the queries the application must perform. A good data model follows **how data will be read and written**, not simply the nouns present in the prompt.

Ask which values are looked up directly, which relationships must be traversed, which queries need ordering or range scans, and which writes must be atomic. Then choose a storage model that supports those operations cleanly. SQL, document databases, key-value stores, and other systems each become useful under different access patterns.

Indexes belong here as part of the design. Once the important queries are known, decide which fields need indexing and what write/storage cost those indexes introduce.

## 7. Draw the simplest high-level architecture first

Only now should the main architecture appear. Start with the minimum set of components required for a correct system, usually something like:

```
Client
  ↓
Load Balancer / API Gateway
  ↓
Application Servers
  ↓
Database
```

Then add components only when a requirement or bottleneck justifies them. A cache may appear because repeated reads are expensive. A queue may appear because some work does not need to block the request path. Object storage may appear because large immutable files do not belong in the primary relational database.

This “simple first, deepen later” approach makes the design easier to defend and prevents architecture-by-buzzword.

## 8. Walk the critical flows end to end

A diagram is not enough. Pick the most important operations and narrate what happens from the client request until the result is returned or persisted.

For a read path, explain where the request enters, whether the cache is checked, when the database is queried, and what is returned. For a write path, explain validation, persistence, cache invalidation, asynchronous work, and acknowledgment. Walking the flow often exposes missing pieces that are invisible in a static architecture diagram.

When the system has both read-heavy and write-heavy behavior, walk at least one of each.

## 9. Find bottlenecks before adding scale

After the basic design works, ask what fails first as traffic grows. The answer might be the application tier, a hot database table, a single partition, connection limits, a large fan-out query, an external API, or network bandwidth.

Scale the bottleneck you actually found. Application servers can be horizontally scaled behind a load balancer, hot reads may justify caching or read replicas, large datasets may eventually require partitioning, and slow background work may justify a queue. Each optimization should solve a specific pressure in the current design.

A strong interview answer often sounds like: **“At the current scale this component is sufficient. If this metric becomes the bottleneck, I would introduce X because it addresses that specific failure mode.”**

## 10. Discuss reliability and failure behavior

Production systems fail partially. A database can become unavailable while application servers remain healthy; a cache can crash; a network call can time out; a worker can process a job and die before acknowledging it.

For important dependencies, ask what happens when they are slow or unavailable. Depending on the system, the answer may involve replication, retries with backoff, timeouts, idempotency, dead-letter queues, health checks, circuit breakers, or graceful degradation. Do not add every reliability mechanism by default; connect each one to a realistic failure.

Also distinguish **durability from availability**. Replicating a service may keep it reachable, while durable storage is about ensuring acknowledged data survives failures.

## 11. Revisit consistency and correctness

Whenever the same logical data exists in more than one place, ask how those copies stay coherent. Caches, database replicas, denormalized views, search indexes, and asynchronous consumers can all introduce temporary disagreement.

State whether the product can tolerate eventual consistency or requires stronger guarantees for particular operations. A social feed may tolerate a short delay before a new post appears everywhere; a payment balance generally cannot tolerate conflicting writes being treated casually.

Consistency decisions should be **operation-specific**, not slogans applied to the entire system.

## 12. Cover security and abuse where relevant

Security is part of system design when it materially affects the product. Mention authentication and authorization boundaries, validation of untrusted input, encryption in transit, secrets management, rate limiting, or abuse prevention when the system exposes those risks.

For public services such as URL shorteners, file-sharing systems, or messaging APIs, abuse and rate limiting can become real capacity and safety concerns. For internal interview exercises, a concise treatment is usually enough unless the interviewer asks for deeper security design.

## 13. Add observability so the design can be operated

A system is difficult to improve if nobody can tell what is slow or broken. Mention the signals that would help operate the design: latency percentiles, error rate, throughput, cache hit rate, queue depth, database saturation, and dependency failures.

Logs explain individual events, metrics expose trends, and traces help follow a request across multiple services. You do not need a full observability platform in every interview, but identifying the measurements that validate your design choices shows practical engineering judgment.

## 14. Close with trade-offs instead of pretending the design is perfect

Before finishing, summarize the major choices and what they cost. Perhaps caching improved latency but introduced staleness risk, or asynchronous processing improved request latency but introduced eventual consistency and retry complexity.

A mature system-design answer makes those compromises explicit. The goal is not to claim the system has no weaknesses; it is to show that the weaknesses are understood and were accepted for reasons tied to the requirements.

## A practical 45-minute pacing guide

| Stage | Approximate time | What should be achieved |
| --- | --- | --- |
| Scope + requirements | 5–7 min | Agree on core features and the most important quality constraints. |
| Scale + APIs + data model | 7–10 min | Establish the workload, contracts, entities, and key access patterns. |
| High-level architecture + flows | 10–12 min | Build the simplest correct system and walk its critical read/write paths. |
| Deep dive + scaling | 10–12 min | Follow the interviewer toward the most important bottleneck or subsystem. |
| Failures + trade-offs + recap | 5 min | Discuss reliability, consistency, major compromises, and remaining risks. |

The exact timings are flexible. If the interviewer pushes deeply into one subsystem, follow that direction rather than mechanically completing every section.

## Questions to keep asking yourself while designing

- **What requirement justifies this component?** If there is no clear answer, the component may be unnecessary complexity.
- **What is the source of truth?** This becomes especially important once caches, replicas, or derived stores appear.
- **What is the hottest read or write path?** The most frequent or latency-sensitive flow usually deserves the most design attention.
- **What breaks first at higher scale?** Scaling becomes meaningful only after the limiting resource is identified.
- **What happens when this dependency is slow or unavailable?** Partial failure should be considered wherever the request crosses a process or network boundary.
- **What consistency does this operation really require?** Different operations inside the same product may justify different guarantees.
- **What trade-off did I just introduce?** Every major optimization usually exchanges simplicity, consistency, cost, latency, or operational burden for some benefit.

## Common interview mistakes

### Designing before clarifying

Jumping directly into databases or microservices often solves a problem the interviewer never asked for. Clarify scope first so the architecture has a target.

### Premature optimization

Sharding, Kafka, Redis, and multi-region deployment are powerful tools, but introducing them before a real bottleneck appears makes the design harder without proving that it is better. Start simple and let scale force complexity.

### Drawing boxes without explaining data flow

A diagram containing ten services can still be shallow if the candidate cannot explain what happens during a request. Always walk the important flows and say where state changes.

### Ignoring write paths

Read performance is easy to discuss, but writes reveal consistency, invalidation, idempotency, and durability problems. Any mutable system should have its write path explained explicitly.

### Treating every requirement as equally important

Real systems optimize for priorities. State which requirements dominate the design so trade-offs can be evaluated against something concrete.

### Memorizing one architecture per product

Interview prompts change details deliberately. Learn the reasoning process and reusable primitives instead of memorizing a diagram for “Twitter,” “Uber,” or “Netflix.”

## How the rest of this page fits into the process

The sections below are the **toolbox used inside this procedure**. Scaling and load balancing help when the application tier becomes a bottleneck. Indexes and connection pools address database-bound work. Caching reduces repeated expensive reads. Later topics such as queues, replication, partitioning, rate limiting, and observability should be added in the same way: first understand the pressure, then understand the primitive that solves it.

This distinction is important. The procedure tells you **how to design**; the remaining sections teach you **what building blocks are available when the design needs them**.

---

# Part II — Core System-Design Building Blocks

## 1. Scaling: what problem are we solving?

Scaling means increasing a system's ability to handle more users, requests, data, or computation while keeping latency and reliability acceptable. The important interview idea is that **scaling is not just “adding more servers”**. First identify the actual bottleneck: CPU, memory, database throughput, network bandwidth, a shared lock, or an external dependency. Scaling the wrong layer only moves more traffic toward the same bottleneck.

### Vertical scaling

**Vertical scaling (scale up)** means making one machine stronger: more CPU cores, RAM, faster storage, or a more powerful instance. It is operationally simple because the application can often remain unchanged, and communication between processes on the same machine can stay very fast.

Its limitation is that a single machine has a hardware ceiling and remains a larger failure domain. At some point the machine cannot be upgraded further, or upgrading becomes disproportionately expensive. Vertical scaling is therefore useful for simplicity and moderate growth, but it is not an unlimited strategy.

### Horizontal scaling

**Horizontal scaling (scale out)** means adding more machines and distributing work across them. Instead of one server handling every request, several servers can process requests in parallel. This improves capacity and can improve fault tolerance because the system does not have to depend on one application server.

Horizontal scaling works best when requests or jobs can be handled relatively independently. It becomes harder when workers require shared mutable state, frequent coordination, or very low-latency communication with one another. Once the application spans machines, communication crosses a network, which introduces serialization, latency, timeouts, retries, and partial failures.

> **Interview framing:** Vertical scaling gives one machine more power. Horizontal scaling adds more machines and distributes work. Horizontal scaling usually offers a higher long-term ceiling, but it introduces distributed-system complexity.
> 

---

## 2. Load balancing

Once multiple application servers exist, clients need a sensible way to reach them. A **load balancer** sits in front of the server pool and distributes incoming traffic so that one server does not receive all the work while others sit idle.

A load balancer is useful for more than traffic distribution. It can stop routing traffic to unhealthy instances, allow servers to be added or removed behind a stable endpoint, and make horizontal scaling practical without clients needing to know the address of every backend server.

A simplified request path is:

```
Client -> Load Balancer -> Server A
                     -> Server B
                     -> Server C
```

Common routing strategies include **round robin**, where requests rotate through servers, and **least connections**, where traffic is sent toward the server currently handling fewer active connections. The correct strategy depends on whether requests have roughly equal cost and whether connection duration matters.

### Load balancing does not remove every bottleneck

Adding ten application servers does not help much if all ten are blocked by the same overloaded database. The load balancer distributes traffic across the application tier; it does not magically increase the capacity of every dependency behind that tier. In an interview, always ask **“what is actually saturated?”** before proposing horizontal scaling.

---

## 3. L4 vs L7 load balancing

Both distribute traffic, but they make routing decisions using information from different networking layers.

| Aspect | Layer 4 Load Balancer | Layer 7 Load Balancer |
| --- | --- | --- |
| Works mainly with | Transport-level information such as IP addresses, ports, TCP or UDP connections | Application-level protocols such as HTTP and HTTPS |
| Understands HTTP path/header? | No. It generally does not need to understand the application request body or URL. | Yes. It can inspect paths, headers, cookies, hostnames, and other HTTP information. |
| Example decision | Forward this TCP connection arriving on port 443 to Server B. | Send `/images` to image servers and `/payments` to payment servers. |
| Main advantage | Simple and efficient because it makes decisions with less application-level inspection. | Much more flexible routing because it understands the application protocol. |

### How to explain the difference in an interview

An **L4 load balancer** routes using transport-layer information. It can distribute TCP or UDP connections without needing to understand what the HTTP request means. An **L7 load balancer** understands application-layer protocols such as HTTP, so it can make content-aware decisions, for example routing requests differently based on URL path, hostname, headers, or cookies.

A useful design question is therefore: **Do I only need to distribute connections, or do I need routing decisions based on the contents of the application request?** The second case points toward L7.

---

## 4. When horizontal scaling does not help much

Horizontal scaling is powerful, but adding machines is useful only when the workload can actually be divided and the limiting resource is being scaled.

### Shared bottleneck

If the database is already saturated, adding more application servers can make the situation worse because even more requests reach the same database. The database, cache, external API, disk, or network link may need to be optimized or scaled instead.

### Heavy cross-server coordination

If workers constantly need to communicate, synchronize, or modify the same shared state, splitting them across machines adds network overhead and distributed coordination. Communication that could have been an inexpensive in-process or shared-memory operation now involves network calls and failure handling.

This does **not** mean that “servers communicating means horizontal scaling is bad.” Most distributed systems communicate. The problem is when the workload is **tightly coupled enough that coordination cost becomes a major part of the work**.

### Sequential or non-parallelizable work

Some workloads contain a large sequential component. If later work fundamentally depends on the completion of earlier work, simply adding more machines cannot proportionally reduce execution time. More workers help only the portion that can actually run in parallel.

### Contention for the same resource

If every worker needs the same lock, file, row, or critical section, adding workers can increase contention rather than throughput. Before scaling out, identify whether the work can proceed independently.

> **Strong interview answer:** “I would not assume horizontal scaling automatically improves throughput. I would first locate the bottleneck. Scale-out gives poor returns when the bottleneck is a shared dependency, when the workload is strongly sequential, or when nodes require heavy coordination or shared mutable state.”
> 

---

## 5. Consistent hashing — context only

Consistent hashing came up while studying load balancing, but it is **not a core topic for this first block**. The only idea worth retaining for now is why it exists.

With a simple mapping such as `hash(key) % numberOfServers`, changing the number of servers changes the modulus and can remap a very large fraction of keys. That is particularly painful for distributed caches or storage because much of the existing placement becomes invalid at once.

**Consistent hashing reduces how many keys must move when a server is added or removed.** It is useful when a particular key, user, or object should map consistently to a node. Do not confuse this with the general purpose of a load balancer: a load balancer distributes incoming traffic, while consistent hashing is a strategy for stable key-to-node placement.

For placements at this stage, remembering the problem it solves is enough. Virtual nodes and deeper ring mechanics can wait until a later system-design block.

---

## 6. Interview-ready checkpoints

You should be able to answer these without notes:

1. **Why do we need a load balancer after horizontal scaling?**
    
    Because clients need a stable entry point that can distribute requests among multiple backend instances. The load balancer also enables health-aware routing and lets servers join or leave the pool without exposing that topology to clients.
    
2. **What is the difference between vertical and horizontal scaling?**
    
    Vertical scaling increases the resources of one machine. Horizontal scaling adds machines and divides work among them. Scale-out has a larger potential ceiling and better redundancy, but introduces network and distributed-state complexity.
    
3. **L4 or L7: when would you choose L7?**
    
    Choose L7 when routing needs knowledge of the application protocol, such as sending requests to different services based on HTTP path, hostname, header, or cookie. If connection-level routing is enough, L4 can be simpler.
    

Because frequent coordination moves from cheap local communication to network communication, adding latency, serialization, retries, synchronization, and partial-failure handling.

1. **Why might adding application servers fail to improve performance?**
    
    Because the application tier may not be the bottleneck. If all instances wait on the same overloaded database, external service, shared lock, or network resource, adding application servers does not remove the limiting factor.
    
2. **Why can tightly coupled workloads scale poorly across machines?**

---

## Resources from this block

- [Gaurav Sen — Horizontal vs. Vertical Scaling](https://www.youtube.com/watch?v=xpDnVSmNFX0)
- [Gaurav Sen — What is Load Balancing?](https://www.youtube.com/watch?v=K0Ta65OqQkY)
- [Cloudflare — Types of Load Balancers: L4 vs L7](https://developers.cloudflare.com/learning-paths/load-balancing/planning/types-load-balancers/)

---

# Session 2 — Database Performance: Indexing + Connection Pooling

<aside>
🧠

The theme of this session is simple: **a system becomes faster not only by adding machines, but also by reducing unnecessary work at the database boundary.** Indexes reduce the amount of data the database must inspect, while connection pools reduce the repeated setup cost of talking to the database in the first place.

</aside>

## 7. Why databases need indexes

Without an index, a database may have to inspect a large portion of a table or collection to find matching rows. That is acceptable for tiny datasets, but as the data grows, repeatedly scanning everything becomes expensive.

An **index is an additional data structure maintained alongside the actual data**. It stores selected column values in an organized form together with enough information to locate the corresponding rows. The database can therefore search the index first instead of blindly checking every row.

A useful analogy is the index at the back of a textbook. If you want the section on "deadlocks," you do not read every page from the beginning. You find "deadlocks" in the index, get the relevant page number, and jump close to the information you need.

### The trade-off: indexes are not free

Indexes improve many reads, but they consume additional storage and must be updated when indexed data is inserted, deleted, or changed. A table with many unnecessary indexes can therefore make writes more expensive.

> **Interview framing:** “I would add an index when an important query repeatedly searches, filters, joins, or sorts using particular fields. I would not index every field blindly because each index has storage and write-maintenance cost.”
> 

## 8. Why B+ trees are useful for database indexes

A database index is commonly implemented using a tree structure such as a **B+ tree**. The important idea is not to memorize every internal rule of the tree, but to understand why it suits storage systems.

Instead of growing into a very tall binary tree, a B+ tree keeps many keys in each node. This gives it a **high branching factor**, so even a very large index can remain relatively shallow. Fewer tree levels usually means fewer storage pages need to be visited during a lookup.

When a node becomes full, it can **split**, moving keys into separate nodes while keeping the tree balanced. The visualizer used in this session makes this useful to see: inserting more keys does not turn the structure into one long chain; the tree reorganizes itself so searches can continue through a small number of levels.

B+ trees are especially useful because their leaf nodes hold the ordered index entries. That ordering helps not only equality lookups such as `id = 42`, but also **range queries** such as `price BETWEEN 500 AND 1000`, because nearby values are stored in ordered leaf pages.

## 9. Composite indexes and the leftmost-prefix idea

A **composite index** contains more than one field. For example, an index on `(a, b)` is ordered first by `a`, and then by `b` among entries that have the same `a` value.

Think of a phone book sorted by **surname first and first name second**. Finding everyone with surname `Sharma` is easy. Finding `Sharma, Amit` is also easy. But finding every person named `Amit` regardless of surname is much harder because all the `Amit` entries are scattered across different surname groups.

That is why an index on `(a, b)` is naturally useful for queries involving:

- `a`
- `a` together with `b`

but it generally cannot provide the same efficient lookup for **`b` alone**. The database does not have one contiguous region where all equal `b` values live, because `a` is the primary ordering key.

This is the practical meaning of the **leftmost-prefix rule**. Column order in a composite index is part of the design decision; `(a, b)` and `(b, a)` are not interchangeable.

> **Placement answer:** “A composite index on `(a, b)` is primarily sorted by `a`, and only then by `b` within each `a` group. Therefore the database can efficiently use the index for `a` or `a + b`, but a query on `b` alone usually cannot jump directly to the matching region.”
> 

## 10. Indexes should follow the query pattern

The correct index depends on the queries the application actually runs. If a marketplace frequently executes a query such as:

```
WHERE category = ? AND price < ?
```

then an index strategy should be designed around how those fields are filtered and sorted in the real workload. An index that looks useful in isolation may provide little benefit if the application's common queries cannot use its ordering effectively.

This leads to an important system-design habit: **start from the access pattern, then design the index**. Do not start by adding indexes simply because columns exist.

## 11. What is connection pooling?

Opening a database connection is not free. Before queries can flow, the application and database may need to establish a network connection, perform protocol setup, authenticate, and allocate resources. Repeating that entire process for every request wastes time and puts unnecessary pressure on the database.

A **connection pool** keeps a limited set of reusable database connections available. When an application request needs the database, it borrows a connection from the pool, executes its work, and returns that connection so another request can reuse it.

```
Request A ─┐
Request B ─┼─> Connection Pool ─> Database
Request C ─┘      [C1 C2 C3]
```

The important distinction is that returning a connection to the pool does **not** normally mean physically closing it. The connection remains alive so the expensive setup can be avoided on the next request.

### Connection pools are similar to thread pools

The analogy with a thread pool is useful:

- a **thread pool** reuses worker threads instead of creating a new thread for every piece of work;
- a **connection pool** reuses database connections instead of opening a new database connection for every query or request.

Both approaches keep a controlled number of expensive resources ready for reuse.

## 12. Connection pooling is also a protection mechanism

Pooling is not only about speed. A pool also places a **bound on concurrency against the database**.

Imagine 10,000 application requests arriving at once. If every request independently opens a database connection, the database may be overwhelmed by connection setup and memory usage before it even begins processing useful queries. With a pool of, say, 50 connections, at most those connections can be actively used by that application instance at one time; other work waits for a connection to become available.

This means pool sizing is a trade-off:

- **too small:** requests may wait even though the database could handle more work;
- **too large:** the application can create excessive concurrency and overload the database.

A connection pool therefore acts as both a **performance optimization and a pressure valve** between the application tier and the database.

> **Interview framing:** “Connection pooling keeps a bounded set of reusable database connections. It avoids repeatedly paying connection-establishment cost and prevents the application from creating an uncontrolled number of simultaneous database connections.”
> 

## 13. How indexing and pooling solve different problems

These two techniques are easy to mix together because both improve database-facing performance, but they optimize different parts of the request path.

| Technique | Main problem solved | Core idea |
| --- | --- | --- |
| Indexing | The database is doing too much work to locate data | Maintain an organized lookup structure so fewer rows/pages need to be examined |
| Connection pooling | The application repeatedly pays the cost of creating database connections | Keep a bounded set of open connections and reuse them |

A slow query is not automatically fixed by increasing the connection pool, and an index does not eliminate the cost of repeatedly establishing connections. **Diagnose which layer is expensive before choosing the optimization.**

## 14. Interview-ready checkpoints from this session

1. **Why does an index speed up a query?**
    
    Because it gives the database an organized structure it can search instead of scanning the entire dataset. The exact benefit depends on whether the query can actually use that index.
    
2. **Why not create an index on every column?**
    
    Because indexes consume storage and every relevant insert, update, or delete may also require index maintenance. Faster reads can come at the cost of slower writes and more storage.
    
3. **Why does a composite index on `(a, b)` not usually help with `b` alone?**
    
    Because the index is ordered primarily by `a`. Values of `b` are only ordered inside each `a` group, so matching `b` values are scattered across the index when `a` is unknown.
    
4. **Why are B+ trees useful for indexes?**
    
    They stay balanced, have a high branching factor, and keep ordered entries at the leaves, which allows efficient lookups and range scans while keeping the tree shallow.
    
5. **What problem does connection pooling solve?**
    
    It avoids repeatedly creating and destroying expensive database connections and limits how many connections the application can use concurrently.
    
6. **How is a connection pool similar to a thread pool?**
    
    Both maintain a bounded set of expensive reusable resources and hand them out temporarily as work arrives instead of constantly creating new ones.
    

## Resources from Session 2

- [Hussein Nasser — Database Indexing](https://youtu.be/-qNSXK7s7_w)
- [B+ Tree Visualizer](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html)
- Connection pooling video used during the session: short conceptual overview of reusable database connections and pooling.

---

# Session 3 — Caching + Redis

<aside>
⚡

Caching is a way of **avoiding repeated expensive work**. Instead of asking the database for the same frequently requested data every time, the application keeps a temporary copy in a faster store such as Redis. The important system-design question is not simply “should I add a cache?” but **what should be cached, when should it be refreshed, and what happens when the cache is wrong or unavailable?**

</aside>

## The basic read path: cache hit and cache miss

A cache normally sits between the application and the slower source of truth. For a request such as `GET /api/listings/:id`, the application can first ask Redis for a key such as `listing:<id>`.

On a **cache hit**, Redis already contains the value, so the application can return it without querying MongoDB. This reduces both response latency and database load. On a **cache miss**, the key is absent or has expired, so the application reads the value from MongoDB, stores the result in Redis with an appropriate TTL, and then returns the response. The database remains the source of truth; Redis is only a faster copy.

```
Request
  ↓
Redis lookup
  ├─ HIT  → return cached value
  └─ MISS → query DB → cache result with TTL → return value
```

This distinction matters because a good cache should improve performance **without becoming a correctness dependency** unless the system deliberately chooses to use Redis for more than caching.

## Cache-aside: the most useful default pattern

In **cache-aside**, also called lazy loading, the application itself controls both Redis and the database. Reads first check the cache; misses go to the database and then populate the cache. Only data that is actually requested enters Redis, which keeps the cache focused on useful values.

The write path is equally important. If a listing changes in MongoDB while Redis still contains the old version, future reads could return stale data. A common cache-aside write sequence is therefore:

```
Update MongoDB
→ delete/invalidate listing:<id> in Redis
→ next read misses
→ fetch fresh value from MongoDB
→ repopulate Redis
```

The key idea is that **the database is updated first and the stale cache entry is explicitly invalidated**. TTL expiry is still useful as a safety net, but waiting only for the TTL after a known write can leave stale data visible unnecessarily.

## Other caching strategies and when they make sense

**Read-through** produces a similar read result to cache-aside, but the responsibility changes: the application talks to the caching layer, and that layer knows how to fetch from the backing database on a miss. In cache-aside, this miss-handling logic lives in application code.

**Write-through** synchronously updates the cache and backing store as part of the write path. This can keep cached data closely aligned with recent writes, but every write pays additional work and latency. It is useful when keeping frequently read cached values fresh is more important than minimizing write cost.

**Write-back or write-behind** accepts the write in the cache first and persists it to the database later. This can make writes extremely fast and absorb bursts, but it introduces durability risk: if the cached write is lost before it reaches the database, data can disappear.

**Write-around** sends writes directly to the database without populating the cache. It is useful for data that is written frequently but rarely read, because immediately caching such values would waste memory. If the value is later requested, the normal read path can load it into the cache.

> **Interview framing:** Cache-aside is usually the easiest pattern to explain and reason about: read from Redis first, fill it on a miss, and invalidate the cached copy after a successful database write.
> 

## TTL, invalidation, and eviction solve different problems

A **TTL (time to live)** gives a cache entry an expiry time. It bounds how long a value can remain cached and is useful when some staleness is acceptable. TTLs are simple, but they do not guarantee that the cached value stays correct during that entire period.

**Active invalidation** removes or updates a cache entry when the underlying data changes. This gives fresher behavior, but every relevant write path must remember to invalidate the correct keys. For important mutable data, a practical design often combines active invalidation with TTL as a fallback.

**Eviction** is different from both. It is about memory pressure rather than freshness. When Redis runs out of space, an eviction policy decides which keys should leave so new data can fit. Policies such as LRU and LFU approximate which cached values are least valuable to retain.

## The failure modes worth knowing

### Cache stampede / thundering herd

A very popular key expires and many requests miss it at almost the same moment. If all of those requests independently rebuild the value, they can hammer the database with identical work. A strong mitigation is **single-flight/request coalescing or a lock**, so one request refreshes the key while the others wait or temporarily use a stale value.

### Cache avalanche

Many keys expire around the same time, causing a large wave of cache misses across the system. One simple defense is **TTL jitter**: instead of assigning every key exactly the same lifetime, add a small random variation so their expirations are spread over time rather than synchronized.

### Cache crash

Redis itself becomes unavailable. If the cache is only an optimization and the database still has enough spare capacity, the application should gracefully bypass Redis and read from the database. If the sudden fallback traffic would overwhelm the database, the system should protect it with short timeouts, rate limiting or load shedding, and circuit breaking rather than allowing unlimited requests to pile up.

### Cache penetration

Clients repeatedly request values that do not exist in either Redis or the database. Every request therefore misses the cache and reaches the database, so caching provides no protection. Common defenses include validating impossible inputs and **negative caching**, where the application briefly caches the fact that a value does not exist. At very large scale, a Bloom filter can also reject many impossible keys before they reach the database.

## Applying caching to Relay

For Relay, Redis would be a sensible optimization for frequently repeated **listing reads**, while MongoDB would remain authoritative. A simplified cached read for `GET /api/listings/:id` would look like this:

```
Browser
→ Next.js API rewrite
→ Express listing route
→ Redis: GET listing:<id>
     ├─ HIT  → return listing
     └─ MISS → Mongoose → MongoDB
                ↓
             listing
                ↓
             Redis SET listing:<id> + TTL
                ↓
             return response
```

For `PUT /api/listings/:id`, Relay would first perform its normal authentication, ownership and validation checks, update MongoDB through Mongoose, and then delete the corresponding Redis key. The next read naturally rebuilds the cache from the fresh database value.

This design keeps the responsibility clear: **MongoDB owns correctness, Redis owns speed**. If Redis disappears, Relay may become slower and MongoDB may receive more traffic, but the cached copy itself should not be the only place where listing data exists.

## Interview-ready mental model

When discussing caching in a system-design interview, start with the access pattern rather than immediately saying “use Redis.” Explain what data is repeatedly read, why caching it reduces latency or database pressure, how a miss is handled, how writes invalidate stale entries, and what the application does when Redis fails. That turns caching from a buzzword into an actual design decision with clear trade-offs.

## Practice checkpoint — 30 Aug 2026

Completed the interview-delivery framework and one **guided URL-shortener design**.

Evidence demonstrated during the guided design:

- separated core functional requirements from secondary analytics;
- identified create-short-link and redirect as the core product contract;
- used `POST /links` and `GET /:shortCode` as the core API shape;
- identified read-heavy behavior and the value of read replicas / caching;
- independently suggested splitting read and write services;
- recognized the need to index the short code;
- understood random/hash generation versus counter → Base62 generation;
- understood why a shared atomic counter is needed across multiple write servers and why range allocation can reduce counter contention.

Current progression state:

**LEARN framework → GUIDED URL shortener complete → INDEPENDENT redesign next.**

Do not count the guided design as independent proof yet.