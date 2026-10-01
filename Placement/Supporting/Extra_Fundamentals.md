# Extra fundamental topics

Below are the topics from **today’s session where your answer was incorrect, incomplete, or you explicitly said you didn’t know the concept**. I’m excluding things where you gave a basically correct 9–10/10 answer.

## DBMS

### Optimistic vs pessimistic locking

You initially interpreted optimistic locking as “lock when you actually need the resource.” The important distinction is that **optimistic locking usually does not lock the row while you're working on it**.

With optimistic locking, the system assumes conflicts are uncommon. Multiple users can read/work with the data, and when someone tries to update it, the database/application checks whether the data has changed since it was originally read, often using a version number.

Pessimistic locking assumes a conflict is likely, so it **locks the resource early**, preventing conflicting operations until the lock is released.

Think:

**Optimistic → work freely, detect collision at the end.**

**Pessimistic → prevent the collision from occurring in the first place.**

### Database checkpoint

You described a checkpoint as a database snapshot. That's close conceptually but not precise.

A checkpoint is a **known recovery point** created periodically by the DBMS. Committed in-memory changes are flushed toward persistent storage, allowing crash recovery to start from that checkpoint rather than replaying the entire transaction history from the beginning.

It works alongside mechanisms such as the transaction log/WAL.

Think:

**Checkpoint ≠ full backup/snapshot.**

**Checkpoint = “recovery can safely start from around here.”**

### Partitioning vs sharding

This was one of the more important corrections.

**Partitioning** divides a large table into smaller pieces, typically according to a rule such as date, range, or hash, while remaining within the same database/server in the questionnaire's framing.

**Sharding** distributes subsets of that data across **multiple independent database servers**.

So:

**Partitioning → divide the data.**

**Sharding → divide the data across machines.**

A shard key determines where a particular record belongs in a sharded system.

### Horizontal vs vertical partitioning

You had an intuition about splitting the same data, but the precise distinction is geometric:

**Horizontal partitioning splits rows.** For example, 2025 orders in one partition and 2026 orders in another.

**Vertical partitioning splits columns.** For example, keep `id, name, email` in one place while moving a huge `profile_bio` column elsewhere.

Memory trick:

**Horizontal = rows. Vertical = columns.**

### Connection pooling

You confused this with queuing requests until database load decreases.

Connection pooling means maintaining a **set of already-open database connections** and reusing them across requests.

Without pooling:

`request → open DB connection → query → close connection`

With pooling:

`request → borrow existing connection → query → return connection to pool`

Opening a database connection can be relatively expensive, so pooling reduces repeated setup overhead. If all connections are currently occupied, then requests may wait, but that waiting is not what “connection pooling” itself means.

### BASE

This was completely new for you.

BASE stands for **Basically Available, Soft State, Eventual Consistency**.

It describes a philosophy common in some distributed systems where temporary inconsistency is acceptable in exchange for high availability and scalability.

For example, two servers might temporarily show slightly different social-media like counts, but eventually they converge.

Contrast:

**ACID → strong transactional guarantees.**

**BASE → availability/scalability with possible temporary inconsistency.**

### Composite index

You understood the leftmost-prefix rule well. Your only misconception was connecting composite indexes inherently with uniqueness.

A composite index simply indexes **multiple columns together**:

```sql
INDEX(last_name, first_name, age)
```

That does **not** automatically mean each combination must be unique. It only becomes a uniqueness constraint if it is explicitly a **UNIQUE composite index**.

The leftmost-prefix rule means that this index is naturally useful beginning from its left side, such as `last_name`, then `last_name + first_name`, etc.

### Query suddenly becoming very slow

Your instinct went immediately toward CPU/hardware load. That's possible, but the better database-first diagnostic path is:

Start with the **execution plan (`EXPLAIN`)**. Determine whether an expected index is no longer being used. Then consider table growth/data distribution, missing indexes or stale statistics, lock contention, and finally broader CPU/disk/server load.

The key interview lesson isn't memorizing causes. It's:

**When one particular query became slow, investigate the query/database execution first before blaming hardware.**

### Two-phase commit (2PC)

You had never encountered this. This is also **lower priority for you** than the DBMS topics above.

2PC coordinates one transaction across multiple databases/systems.

**Prepare phase:** all participating systems are asked whether they are capable of committing.

**Commit phase:** if everyone agrees, all participants commit; otherwise the distributed transaction is aborted.

It is essentially:

**“Everyone confirm you're ready before anyone finalizes.”**

---

## Operating Systems

### Polling vs interrupts

You mixed interrupts with process preemption.

**Polling:** the CPU repeatedly asks a device whether something has happened.

```
Ready?
No.
Ready?
No.
Ready?
Yes.
```

This can waste CPU cycles.

**Interrupt:** the CPU does other work, and the device signals the CPU when attention is required.

```
CPU does other work...
Device → "I need attention."
CPU handles interrupt.
```

So:

**Polling = CPU keeps checking.**

**Interrupt = device tells the CPU.**

### Thread pool

You associated this mainly with moving CPU-intensive work away from a main thread. That can be a use case, but it isn't the definition.

A thread pool is a **pre-created group of reusable worker threads**. Tasks are submitted to a queue and available workers execute them.

Instead of:

```
task → create thread → execute → destroy thread
```

repeated thousands of times, you have:

```
worker 1
worker 2
worker 3
...
      ↑
   task queue
```

The important benefit is avoiding constant thread creation/destruction overhead and controlling concurrency.

### File allocation methods

Here you answered **hard links vs symbolic links**, which is a different topic entirely.

The questionnaire asks about how a file's blocks are arranged on storage.

**Contiguous allocation:** blocks are adjacent. Very fast access, but fragmentation can become a problem.

**Linked allocation:** each block points to the next block. Flexible placement, but poor random access.

**Indexed allocation:** a dedicated index contains pointers to the file's blocks. It gives good random access while allowing blocks to be scattered.

Modern file systems commonly use structures related to the indexed approach.

Your explanation of **hard links vs symbolic links itself was mostly correct**, so you don't need to relearn that.

---

## Computer Networks

### Anycast

Completely new for you.

In Anycast, **multiple servers can advertise the same IP address**, and routing sends a user toward an appropriate/topologically nearby instance.

This is useful for globally distributed infrastructure such as DNS/CDNs because one public address can represent many geographically distributed servers.

Memory line:

**Unicast → one address, one destination.**

**Anycast → one address, multiple possible destinations; routing picks one.**

### Piggybacking

Also new.

Suppose A sends data to B, and B needs both to acknowledge A's data and send its own data back.

Instead of sending:

```
B → ACK
B → data
```

B can attach the ACK to its outgoing data:

```
B → data + ACK
```

That's **piggybacking**. It reduces communication overhead.

### MTU and fragmentation

MTU means **Maximum Transmission Unit**. It represents the largest packet/frame size that a particular network link can carry without requiring fragmentation. Ethernet commonly uses an MTU around 1500 bytes.

If an IPv4 packet exceeds what a link can carry, it may have to be split into smaller fragments and later reassembled.

The important concept is simply:

**Every network link has a maximum unit size; oversized packets require special handling.**

### Sliding window

You had studied this before but couldn't recall it.

With stop-and-wait:

```
Send packet
Wait for ACK
Send packet
Wait for ACK
```

The network sits underutilized while waiting.

Sliding window permits **multiple packets to remain unacknowledged/in flight simultaneously**, up to a window limit:

```
send 1
send 2
send 3
send 4
       ← ACKs arrive
```

This substantially improves throughput. It is also central to TCP flow control.

### TTL and traceroute

You knew TTL's purpose, but thought it literally represented elapsed time.

Despite its name, IP TTL operates as a **hop limit**. Every router forwarding the packet decreases TTL by 1. At zero, the packet is discarded.

`traceroute` deliberately exploits this:

```
TTL 1 → expires at first router
TTL 2 → expires at second router
TTL 3 → expires at third router
...
```

The responses reveal the sequence of routers along the path.

Think:

**TTL = how many router hops remain.**

### IPsec VPN vs SSL VPN

You hadn't covered this.

For your level, remember only the broad distinction:

**IPsec VPN:** operates at the network layer and can protect broad IP traffic, often used for full-network or site-to-site connectivity.

**SSL VPN:** typically provides higher-level remote access to particular applications/services and can often work through a browser/client.

Memory line:

**IPsec → network tunnel.**

**SSL VPN → application/remote-user access.**

### Packet-loss diagnosis

You knew what packet loss was and correctly connected TCP with retransmission, but didn't answer the diagnostic half.

The basic interview tools are:

**`ping`** to observe whether packet loss is occurring repeatedly.

**`traceroute` / `mtr`** to inspect the path and help determine where along the route problems appear.

Possible causes include congestion, failing networking equipment, or wireless interference.

---

The **highest-priority gaps** from today are connection pooling, partitioning vs sharding, horizontal vs vertical partitioning, polling vs interrupts, thread pools, file allocation, TTL/traceroute, and sliding window.

Anycast, piggybacking, BASE, MTU, IPsec-vs-SSL-VPN, and especially 2PC are more like **breadth boosters**. You should recognize and explain them, but I would not spend significant study time going deep into them before placements.

## OS PROVE checkpoint — 30 Aug 2026

Closed-book results:

- Process vs thread — **PASS**
- Context switch — **PASS**
- Four deadlock conditions — **PASS**
- Paging vs segmentation — **PASS**
- Thread pool — **PASS**
- Polling vs interrupts — **REPAIR → PASS** after being prompted for the tradeoff

Overall performance was strong enough to move these OS fundamentals to **MAINTAIN**. No broad OS repair session is justified from this round.