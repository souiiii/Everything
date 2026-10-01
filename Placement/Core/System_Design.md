# System Design for Placements

<aside>
🎯

This is the **running system-design page for placement preparation**. The goal is not to memorize architectures, but to learn a repeatable way to turn vague product requirements into a defensible technical design. Every component on this page should therefore be understood in context: what problem it solves, what trade-off it introduces, and what evidence would justify adding it.

</aside>

# Part I — How to Approach a System Design Problem

## What system design is really testing

System design is not mainly a test of how many technologies you can name. It tests whether you can take an incomplete problem, identify what actually matters, and make engineering choices that follow from those facts. A strong design therefore has a visible chain of reasoning: **requirements create pressures, pressures become decision criteria, and those criteria justify architectural choices**.

The interviewer is usually evaluating several abilities at the same time. You need to clarify an ambiguous product, distinguish essential behavior from optional features, understand the important quality goals, recognize constraints, make sensible assumptions when information is missing, and then build the simplest architecture that satisfies those needs. Only after the basic design is coherent should you deepen into scale, failure handling, consistency, or specialized infrastructure.

> **Core mental model:** requirements first, architecture second, optimization last. Redis, Kafka, sharding, microservices, replicas, and queues are not starting points. They are tools that become justified only when a requirement, constraint, workload, or failure mode gives you a reason to use them.

## The repeatable interview flow

A good system-design discussion can usually follow this order:

1. Clarify the scope and the important stakeholders or users.
2. Define the core functional requirements.
3. Identify the most important quality requirements and make them concrete.
4. Separate hard constraints from assumptions you are making.
5. Turn those requirements and constraints into decision criteria.
6. Estimate scale only where the numbers may change a design choice.
7. Define the main APIs or external contracts.
8. Model the data and the important access patterns.
9. Draw the simplest architecture that can work correctly.
10. Walk the critical read and write flows end to end.
11. Identify bottlenecks and realistic failure modes.
12. Add scaling or reliability mechanisms only where they solve a demonstrated problem.
13. Revisit consistency, security, and operability where they matter.
14. Explain the important architectural decisions and their consequences.
15. Close by summarizing the major trade-offs and remaining risks.

The order matters. Each later choice should be traceable to something established earlier. If you add Redis, for example, you should be able to point to a latency target or repeated read pattern that made a cache useful. If you introduce a queue, you should be able to explain why that work does not belong on the synchronous request path.

## 1. Clarify the scope before designing

Most prompts are intentionally broad. “Design YouTube,” “design a URL shortener,” or “design a chat system” can each describe dozens of different products. Trying to design everything immediately creates a large but unfocused answer.

Start by deciding which user actions are inside the problem. For a URL shortener, the core scope might be creating a short link and redirecting it. Analytics, custom aliases, expiration, QR codes, moderation, and account management may all be real features, but they should not silently become part of the design unless they matter to the prompt.

Also identify whose expectations actually influence the architecture. In a placement interview, you do not need a formal stakeholder table, but you should recognize that different people care about different outcomes. End users may care about latency and availability, operators about recoverability and observability, the business about cost and delivery speed, and a regulated product about retention or data-location rules. A stakeholder matters when their expectation changes a design decision.

A useful opening is therefore to establish **core behavior, excluded behavior, and the few expectations that will drive the design**. That shared boundary prevents you from solving a different problem from the one the interviewer intended.

## 2. Define functional requirements

Functional requirements describe **what the system must allow a user or another system to do**. They should be expressed as externally visible behavior, not as implementation choices.

For a URL shortener, good functional requirements are:

- a user can submit a long URL and receive a short URL;
- opening the short URL redirects to the original URL;
- a link may optionally support expiry if that feature is in scope.

“Use Redis,” “store data in MongoDB,” or “build microservices” are not functional requirements. Those are possible design choices that must be justified later.

In an interview, two to four core functions are normally enough to anchor the architecture. Separate **must-have behavior** from secondary features. This keeps the core request paths obvious and prevents optional product ideas from dictating unnecessary infrastructure.

## 3. Define the quality requirements that actually drive architecture

Functional requirements tell you what the system does. Quality requirements describe **how well the system must do it**. Typical qualities include latency, availability, durability, consistency, throughput, security, and ease of change.

The common mistake is to list generic words such as “fast, scalable, reliable, secure.” Those words are too vague to guide architecture. Instead, prioritize the small number of qualities that matter most and make them concrete enough that you could tell whether the system satisfies them.

For example:

| Vague statement | Useful architectural requirement |
| --- | --- |
| “Redirects should be fast.” | “Under normal load, 95% of redirect requests should complete within about 100 ms.” |
| “The service should be reliable.” | “Failure of one application instance should not make the redirect service unavailable.” |
| “Links must not disappear.” | “Once creation is acknowledged, the short-code mapping must survive an application restart and ordinary instance failure.” |
| “The system should scale.” | “The design should support a large read-heavy workload by adding application capacity without changing the public API.” |

The exact numbers in an interview may be assumptions rather than real product requirements. What matters is that the quality goal becomes **specific enough to influence a choice**.

A useful quality scenario contains three ideas: the situation, the event that occurs, and a measurable response. For example: “Under normal production load, when a user requests a valid short URL, the service returns the redirect within the agreed latency target.” This is much more useful than simply writing “low latency.”

Two forms of quality scenarios are worth knowing. A **runtime or usage scenario** describes how the live system should react to an event, such as a latency or failure-recovery target. A **change scenario** describes how easily the system should accommodate a future modification, such as adding another storage backend or authentication provider. In placements, runtime scenarios are more common, but changeability matters when maintainability or extensibility is part of the discussion.

Do not try to maximize every quality simultaneously. Strong consistency can increase coordination cost; extremely high availability can make the design more expensive; aggressive caching can improve latency while increasing staleness risk. The point of quality requirements is to establish which compromises are acceptable.

## 4. Separate constraints from requirements

A **requirement** describes an outcome the system must provide. A **constraint** limits the freedom you have while choosing how to provide it.

This distinction matters because a constraint can remove otherwise reasonable designs from consideration. If the system must run in a particular cloud, integrate with an existing PostgreSQL database, remain within a strict infrastructure budget, use a mandated protocol, or keep data inside a particular region, those facts shape the solution before performance optimization even begins.

Useful constraint categories include:

- **technical constraints**, such as an existing database, protocol, runtime, or external system that cannot be replaced;
- **organizational constraints**, such as the technologies a team can realistically operate or a fixed deployment environment;
- **business or legal constraints**, such as budget, licensing, retention, or data-location rules.

Some constraints are truly fixed and some are negotiable. Clarify which is which. A team preference for PostgreSQL is not automatically the same as “the company platform requires PostgreSQL.” Treating preferences as hard constraints can eliminate better solutions for no real reason.

A good design makes the solution space explicit: **where am I free to choose, and where am I not?**

## 5. Make assumptions explicit when the prompt is incomplete

System-design prompts almost never provide every fact you need. When an important detail is missing, make a reasonable assumption and say it aloud rather than silently building the architecture around it.

An assumption is not a requirement and not a constraint. It is a provisional belief used to keep the design moving until better information is available.

For example:

- “I’ll assume redirects are roughly 100 times more frequent than link creation.”
- “I’ll assume analytics can be eventually consistent.”
- “I’ll assume a newly created short link should be usable immediately.”
- “I’ll assume we are designing for one region first unless multi-region availability is required.”

Explicit assumptions improve the design in two ways. First, the interviewer can correct them before you build too much on top of them. Second, when a later choice depends on one of those assumptions, the reasoning stays visible.

A useful interview sentence is: **“This detail is unspecified, so I’ll make X assumption for now. If that assumption changes, this part of the design would change as well.”**

Hidden assumptions are dangerous because they look like facts. Explicit assumptions are useful because they can be challenged.

## 6. Turn requirements into decision criteria

Architecture is the process of choosing among alternatives. Those alternatives should be compared using **decision criteria derived from the requirements, constraints, and assumptions**, not personal preference.

Suppose you are choosing storage for a particular subsystem. “SQL vs NoSQL” is not a useful question by itself. The useful questions are: Do we need transactions across related records? What are the dominant access patterns? Is horizontal write scale a real requirement? How important is schema flexibility? What operational expertise does the team have? What consistency is required?

Decision criteria can include:

- correctness and consistency guarantees;
- latency or throughput needs;
- durability and availability requirements;
- access patterns and data relationships;
- expected growth;
- operational complexity;
- cost;
- team expertise and delivery speed;
- security, licensing, or compliance constraints.

Not every criterion has equal importance. Separate **must-have criteria** from preferences. If an option violates a must-have property, a minor convenience elsewhere should not rescue it. Numerical scoring can help when several options are genuinely close, but interviews rarely need a formal weighted matrix. Clear priorities and explicit trade-offs are usually better.

A useful reasoning chain is:

**Requirement or constraint → decision criterion → chosen approach → consequence**

For example:

**Low redirect latency + very read-heavy traffic → repeated database reads are expensive → add a cache for hot mappings → accept invalidation and staleness complexity.**

The architecture is now defensible because the component has a reason to exist.

## 7. Estimate scale only when it changes a decision

Back-of-the-envelope estimation matters because the same design can be sensible at one scale and inappropriate at another. Estimate the quantities that create architectural pressure: requests per second, read-to-write ratio, storage growth, object size, bandwidth, or expected concurrency.

You usually do not need precise arithmetic. The goal is to understand the order of magnitude. If reads outnumber writes by 100:1, caching and read replicas become more plausible. If the entire dataset comfortably fits on one database server, sharding should not appear just because it is a famous system-design topic.

> **Interview habit:** always say what decision an estimate is helping you make. A number with no architectural consequence is usually noise.

## 8. Define the external interface

Once the behavior is clear, define the main APIs or communication contracts. This turns product language into concrete operations and exposes what information must enter and leave the system.

A REST-style service may need only a few endpoints with the important request and response fields. A real-time product may require WebSocket messages or events instead. Authentication, idempotency, pagination, and error behavior should be mentioned only where they materially affect the design.

The API is not decoration. It is a test of whether the architecture actually supports the required behavior. If the design cannot clearly explain how each core operation is served, the architecture is probably still too vague.

## 9. Design the data model from access patterns

Before choosing a database, identify the important entities and, more importantly, **how the application will read and write them**.

Ask which values are looked up directly, which relationships must be traversed, which queries require ordering or range scans, which writes must be atomic, and which fields will be used to filter or join. Then choose a storage model that supports those operations cleanly.

This is why database choice should follow access patterns rather than fashion. SQL, document stores, key-value stores, search engines, and object stores each solve different shapes of problem.

Indexes belong in this discussion as well. If short-code lookup is the critical read path, the short code must be efficiently searchable. If a query frequently filters by category and orders by time, the index strategy should follow that actual query pattern. Every index also has write and storage cost, so indexing everything is not a design strategy.

## 10. Draw the simplest high-level architecture first

Only after the requirements, constraints, workload, API, and data are understood should the main architecture appear.

Start with the minimum number of components needed for a correct system. A first version might be:

Client → Load Balancer / API Gateway → Application Servers → Database

Then add components only when a known pressure justifies them. A cache may appear because repeated reads dominate latency. A queue may appear because work can happen asynchronously. Object storage may appear because large immutable files are a poor fit for the transactional database.

This “simple first, deepen later” approach prevents architecture-by-buzzword. It also makes later optimizations easier to explain, because the interviewer can see what problem each new component solves.

## 11. Walk the critical flows end to end

A box diagram is not enough. Choose the most important operations and narrate exactly what happens from request arrival to response or persistence.

For a read path, explain where the request enters, whether a cache is checked, when the source of truth is queried, and what is returned. For a write path, explain validation, persistence, acknowledgment, cache invalidation, event publication, or asynchronous work where relevant.

Walking the flow exposes missing decisions. A diagram can hide questions such as “when is the write considered successful?”, “what happens if cache invalidation fails?”, or “can the same request be processed twice?”

If the product has meaningful reads and writes, walk at least one of each.

## 12. Find the bottleneck before scaling

After the basic design is correct, ask what fails first as traffic grows. The bottleneck might be CPU in the application tier, database throughput, a hot partition, connection limits, an expensive query, network bandwidth, or an external dependency.

Scale the bottleneck you actually found. Application servers can be replicated behind a load balancer. Repeated hot reads may justify caching or read replicas. A dataset that outgrows one storage node may eventually require partitioning. Slow non-critical work may move to a queue.

A strong answer sounds like: **“At the current scale this component is sufficient. If this metric becomes the bottleneck, I would introduce X because it addresses that specific pressure.”**

That is better than adding infrastructure pre-emptively.

## 13. Design for realistic failure behavior

Production systems fail partially. A database can become unavailable while application servers remain healthy. A worker can process a job and die before acknowledging it. A remote call can time out even though the remote service eventually completed the operation.

For important dependencies, ask what should happen when they are slow, unavailable, or return an uncertain result. Depending on the system, the answer may involve replication, timeouts, bounded retries with backoff, idempotency, dead-letter handling, circuit breaking, or graceful degradation.

Do not add every reliability mechanism by default. Match the mechanism to a real failure mode.

Also keep **availability and durability** separate. Availability is about whether the service can continue responding. Durability is about whether acknowledged data survives failure. Replicating application servers can improve availability without making stored data durable.

## 14. Revisit consistency and correctness

Whenever the same logical data exists in more than one place, ask how those copies stay coherent. Caches, replicas, denormalized views, search indexes, and asynchronous consumers can all temporarily disagree.

State what each important operation actually requires. A social feed may tolerate a short delay before a new post appears everywhere. A payment balance normally cannot treat conflicting writes casually. Even inside the same product, different operations can justify different consistency guarantees.

Do not label an entire system “strongly consistent” or “eventually consistent” without explaining which data and which operation you mean.

## 15. Cover security and abuse where they affect the architecture

Security belongs in the design when it changes system boundaries or behavior. Relevant concerns can include authentication, authorization, validation of untrusted input, encryption in transit, secret handling, rate limiting, auditability, or abuse prevention.

Public services such as URL shorteners, file-sharing platforms, and messaging APIs often need rate limiting or abuse controls because malicious traffic affects both safety and capacity. Internal systems may need strict authorization boundaries or audit logs instead.

Keep this section proportional to the prompt. A concise, relevant security discussion is stronger than mechanically listing every security concept you know.

## 16. Add observability so the design can be operated

A system cannot be improved reliably if nobody can tell what is slow, saturated, or failing. Identify the measurements that would validate the design: latency percentiles, error rate, throughput, database saturation, cache hit rate, queue depth, retry rate, or dependency failures.

Logs explain individual events, metrics reveal aggregate behavior and trends, and traces help follow a request across service boundaries. You do not need to design a full observability platform in every interview. The important skill is knowing **which signals would tell you whether your architectural assumptions are correct**.

For example, if Redis was added to reduce database pressure, cache hit rate and database query volume are direct evidence of whether that decision is working.

## 17. Make important architectural decisions traceable

Not every coding choice deserves architectural documentation. Focus on decisions that significantly affect structure, quality characteristics, important dependencies, interfaces, cost, or future flexibility.

A lightweight Architecture Decision Record, or ADR, captures the reasoning behind such a choice. The useful structure is:

| Field | What it should explain |
| --- | --- |
| Context | What problem, requirement, constraint, or trade-off forced a decision? |
| Decision | What approach was chosen? |
| Status | Is the choice proposed, accepted, superseded, or no longer used? |
| Consequences | What positive, negative, and neutral effects follow from the choice? |

The most important part is not the format. It is the **rationale**. Code may reveal that Redis is used, but it does not automatically explain why Redis was introduced, what alternatives were rejected, or what complexity the team accepted in exchange.

In an interview, you normally express the ADR logic verbally rather than writing a formal record. For example:

**Context:** redirect traffic is read-heavy and the latency target is difficult to meet with repeated database lookups.  
**Decision:** cache frequently accessed short-code mappings using cache-aside.  
**Consequences:** redirect latency and database load improve, but invalidation, cache failure, and temporary staleness now have to be handled.

That is a complete architectural explanation: not just **what**, but **why and at what cost**.

## 18. Close with trade-offs rather than pretending the design is perfect

Before finishing, summarize the major choices and what they cost.

Caching may reduce latency while creating staleness and invalidation complexity. Asynchronous processing may reduce request latency while introducing retries and eventual consistency. Replication may improve availability while making failover and consistency harder. Stronger consistency may improve correctness while increasing coordination and latency.

A mature design does not claim to eliminate trade-offs. It shows that the important trade-offs are understood and that the chosen compromises match the priorities established at the beginning.

## Putting the reasoning together — URL shortener example

Suppose the prompt is simply “Design a URL shortener.”

**Scope:** support creating a short link and redirecting it. Treat detailed analytics as secondary unless requested.

**Functional requirements:** create a mapping from a long URL to a short code, and resolve a short code back to its destination.

**Quality requirements:** redirects should be low-latency and highly available; acknowledged mappings should be durable. A newly created link should be usable promptly.

**Constraints:** assume a normal HTTP-facing service and no mandated database technology unless the interviewer provides one.

**Assumptions:** redirect traffic is much heavier than creation traffic, while detailed analytics can be eventually consistent.

**Decision criteria:** the redirect path should perform a cheap key lookup, the durable mapping must have a clear source of truth, short codes must be unique, and the first version should remain operationally simple.

Those facts lead naturally to a design rather than a memorized diagram. The durable store holds the short-code mapping and indexes the lookup key. Stateless application servers can scale horizontally when request volume grows. A cache can be introduced for frequently requested mappings because the workload is read-heavy and latency-sensitive. Analytics can be moved off the critical redirect path if they do not need to be immediately consistent.

Notice the sequence: **the components appear because the requirements and workload earn them**.

## A practical 45-minute pacing guide

| Stage | Approximate time | What should be achieved |
| --- | --- | --- |
| Scope, requirements, constraints, assumptions | 7–9 min | Agree on core behavior, the most important quality goals, and any facts that limit the solution. |
| Scale, APIs, and data model | 7–9 min | Establish the workload, external contract, entities, access patterns, and source of truth. |
| High-level architecture and flows | 10–12 min | Build the simplest correct system and walk the critical read and write paths. |
| Deep dive, bottlenecks, and failures | 10–12 min | Follow the strongest design pressure and add complexity only where justified. |
| Decisions, trade-offs, and recap | 4–6 min | Explain why the main choices were made, what they cost, and what would change under different assumptions. |

The timings are flexible. If the interviewer pushes deeply into one subsystem, follow that direction rather than mechanically completing every section.

## Questions to keep asking yourself while designing

- **What is the smallest product I am actually designing?** This prevents optional features from taking over the architecture.
- **Which three to five quality goals matter most, and how would I know whether they are met?** Vague qualities do not provide useful design pressure.
- **Is this fact a requirement, a constraint, or an assumption?** Mixing them together hides where the design is flexible.
- **What criterion justifies this technology or pattern?** If there is no clear answer, the component may be unnecessary.
- **What is the source of truth?** This becomes critical once caches, replicas, derived stores, or queues appear.
- **What is the hottest or most correctness-sensitive path?** That path usually deserves the deepest reasoning.
- **What breaks first at higher scale?** Scaling becomes meaningful only after the limiting resource is identified.
- **What happens if this dependency is slow, unavailable, or returns an uncertain result?** Distributed systems fail partially.
- **What consistency does this operation actually require?** Different operations in one product may need different guarantees.
- **What consequence did this decision introduce?** Every meaningful optimization normally exchanges simplicity, cost, consistency, latency, or operational effort for some benefit.
- **What assumption would make me redesign this?** This keeps the architecture adaptable rather than dogmatic.

## Common interview mistakes

### Designing before clarifying

Jumping directly into databases, caches, or microservices often solves a problem the interviewer never asked for. Clarify the scope first so the architecture has a target.

### Using vague non-functional requirements

Saying “highly scalable and reliable” sounds complete but gives no direction. Prioritize the qualities that matter and make them concrete enough to influence a decision.

### Hiding important assumptions

If you silently assume a read-heavy workload, eventual consistency, one region, or a particular traffic level, later choices can look arbitrary. State the assumption so it can be corrected.

### Treating implementation choices as requirements

“Use Kafka” is not normally a product requirement. First identify the behavior or pressure that needs asynchronous messaging, then decide whether Kafka is an appropriate solution.

### Choosing technology before defining decision criteria

“I prefer MongoDB” or “Redis is fast” is not sufficient reasoning. Compare alternatives against the system’s actual access patterns, guarantees, constraints, operational needs, and cost.

### Premature optimization

Sharding, Kafka, Redis, and multi-region deployment are powerful tools, but introducing them before a real pressure appears makes the design harder without proving it is better. Start simple and let requirements or bottlenecks force complexity.

### Drawing boxes without explaining data flow

A diagram containing ten services can still be shallow if you cannot explain what happens during a request. Walk the important flows and state where data is read, changed, acknowledged, cached, or queued.

### Ignoring write paths

Reads are often easier to optimize, but writes reveal consistency, invalidation, idempotency, and durability problems. Any mutable system should have its important write path explained explicitly.

### Treating every quality as equally important

Real systems optimize around priorities. If consistency, availability, latency, cost, and flexibility are all described as equally critical, you have no basis for making trade-offs.

### Memorizing one architecture per product

Interview prompts deliberately change assumptions. Learn the reasoning process and reusable primitives instead of memorizing a diagram for “Twitter,” “Uber,” or “Netflix.”

## How the rest of this page fits into the process

The sections below are the **toolbox used inside this reasoning process**. Scaling and load balancing help when the application tier becomes a bottleneck. Indexes and connection pools address different forms of database-bound work. Caching reduces repeated expensive reads. Queues, replication, partitioning, rate limiting, and observability solve other specific pressures.

The procedure in Part I tells you **how to decide**. The remaining sections teach you **which building blocks are available once the design has earned them**.

This distinction is the foundation of good system design: do not begin with a tool and search for a place to use it. Begin with a requirement, constraint, assumption, or failure mode, and choose the simplest tool that addresses it.

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