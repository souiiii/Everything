# OS

### Chapter 1 — Introduction (low yield, know the definitions)

Almost none of this is interview material, but a few one-liners come up in objective rounds:

**What an OS is** — three framings from the video: intermediary between user and hardware, resource manager/allocator, and platform for running programs. Goals: primary = convenience, secondary = efficiency, reliability, maintainability.

**The classification chain** — this is the one thing worth actually understanding, because they build on each other:

- **Multiprogramming** — several jobs in main memory; when one waits for I/O, the CPU switches to another. Goal: CPU never idle.
- **Multitasking / time sharing** — multiprogramming + time slicing. Switches are so fast each user thinks the machine is theirs. Gives the *illusion* of parallelism via context switching.
- **Multiprocessing** — 2+ physical CPUs sharing bus and memory. This is *true* parallelism, not an illusion.

That last distinction is the asked one: **multiprogramming = one CPU, interleaved; multiprocessing = multiple CPUs, simultaneous.**

**Also worth a line each:** spooling (data held in a buffer so a slow device like a printer doesn't stall the system), hard vs soft real-time (hard = missing a deadline is failure — airbags, pacemakers; soft = degradation only — video playback), symmetric vs asymmetric multiprocessing (all processors equal vs each with a fixed role).

Batch systems, punch cards, distributed OS — recognition only.

### Chapter 2 — OS Structure (three things matter)

**System calls** — the interface through which a user program requests a service only the OS can perform. Six categories: process control, file management, device management, information maintenance, communications, protection. Examples worth naming: `fork`, `exec`, `open`, `read`, `write`, `wait`.

**User mode vs kernel mode** — a **mode bit** in hardware: kernel = 0, user = 1. Your code runs in user mode; a system call triggers a transition to kernel mode, the OS does the privileged work, then switches back. This gets asked, and the answer that scores is naming the mode bit and the transition.

**Monolithic vs microkernel** — monolithic: all OS services in kernel space, fast (no mode-switching between services) but a bug anywhere can crash the system. Microkernel: only the essentials in the kernel, everything else pushed to user space — more stable and extensible, slower because services communicate via message passing. MINIX 3 as the example, ~12,000 lines.

Layered structure, CLI vs GUI, shell names — skip.

### Chapter 3 — Process Basics (this is the payload)

**Program vs process** — a program is a *passive* entity: a file of instructions sitting on disk. A process is an *active* entity: a program loaded into main memory **with a PCB created for it**, holding CPU time, registers, memory. Two copies of the same browser = two separate processes, same text section, different data/heap/stack.

**The four sections of a process** — Text (the code), Data (global variables), Heap (dynamically allocated at runtime), Stack (function parameters, return addresses, local variables). Directly asked, and you should be able to draw it.

**PCB contents** — the seven the video lists: process state, program counter, CPU registers, CPU-scheduling info (priority, queue pointers), memory-management info (base/limit registers, page tables), accounting info, I/O status info. If you remember nothing else: **state, PC, registers** — those are what get saved on a context switch.

**Five process states** — New → Ready → Running → Waiting/Blocked → Terminated. Two transitions people fumble: Running → Ready is *preemption* (time slice expired), Running → Waiting is the process *voluntarily* blocking on I/O. And Waiting always goes back to **Ready**, never straight to Running.

**Three schedulers** — this comparison is a standard question:

|  | Long-term (LTS) | Short-term (STS) | Medium-term (MTS) |
| --- | --- | --- | --- |
| Moves | Job pool → Ready queue | Ready → CPU | Swaps processes out of memory |
| Frequency | Rare | Very frequent | In between |
| Controls | Degree of multiprogramming | CPU allocation | Adjusts degree of multiprogramming |

**Dispatcher vs scheduler** — the sneaky one. The *scheduler* decides which process runs; the *dispatcher* actually hands over the CPU. Its three jobs: switch context, switch to user mode, jump to the right instruction. The time it takes is **dispatch latency**, and it's pure overhead.

**CPU-bound vs I/O-bound** — CPU-bound spends more time computing, I/O-bound more time waiting on I/O. Why it matters: the LTS needs a **good mix**. All I/O-bound and the ready queue sits empty; all CPU-bound and the devices sit idle.

**Context switch** — save the current process's state into its PCB, load the next process's state from its PCB. The line to say: **it's pure overhead — the system does no useful work while switching.** Follow-up they like: thread context switches are cheaper than process ones, because threads share an address space so the memory maps don't need swapping.

### Process vs thread — the most-asked OS question, full stop

A thread is a lightweight unit of execution *inside* a process. One process can hold many threads.

**What threads share:** code (text) section, data section (globals), heap, and open files/OS resources.

**What each thread owns privately:** its own **stack**, **registers**, **program counter**, and thread ID.

That split *is* the answer to most follow-ups. Say it as: "threads share the address space but keep their own execution context."

|  | Process | Thread |
| --- | --- | --- |
| Memory | Separate address space | Shared within the process |
| Creation cost | Heavy | Light |
| Context switch | Expensive — memory maps swap | Cheap — address space unchanged |
| Communication | Needs IPC (kernel-mediated) | Direct, through shared memory |
| Isolation | One crash doesn't kill others | One bad thread can take down the process |
| Synchronization | Rarely needed | Essential — shared data means race conditions |

**Why threads at all** — four benefits worth naming: responsiveness (UI stays alive while work happens), resource sharing, economy (cheaper than processes), scalability (real parallelism across cores).

**User-level vs kernel-level threads** — user-level are managed by a library, invisible to the kernel: fast to create and switch, but **one blocking call blocks every thread in the process**, and they can't use multiple cores. Kernel-level are scheduled by the OS: slower, but one blocking doesn't stall the rest and true parallelism is possible.

**Multithreading models** — many-to-one, one-to-one, many-to-many. Know the names; modern systems use one-to-one.

If you did the Node notes last night, this connects directly: Node's `worker_threads` are kernel threads for CPU work, and "one thread blocks everyone" is the user-level thread problem in a different costume.

### fork()

Creates a new process by **duplicating the calling process**. The child gets a copy of the parent's address space — not shared, copied (in practice copy-on-write, so pages are shared until one side writes).

**Return values — memorize these three:**

- `0` → you're in the **child**
- `> 0` → you're in the **parent**, and the value is the **child's PID**
- `< 0` → fork failed

c

```c
pid_t pid = fork();
if (pid == 0)      printf("child\n");
else if (pid > 0)  printf("parent of %d\n", pid);
```

Both branches run — in different processes. That's the part that trips people.

**The counting question** — *n* consecutive `fork()` calls produce **2ⁿ total processes**, so 2ⁿ − 1 children. Three forks → 8 processes. Objective rounds love this.

**fork vs exec** — `fork` duplicates; `exec` **replaces** the current process image with a new program and does *not* return on success. The standard shell pattern is fork-then-exec: fork a child, have the child exec the new program, parent waits.

### Zombie and orphan processes

**Zombie** — the child has finished, but the parent hasn't called `wait()` yet. The process is dead, yet its PCB entry lingers so the parent can still read the exit status. It holds no memory or CPU — just a process-table slot. Many zombies exhaust the process table, which is the actual harm. Fixed by the parent calling `wait()`.

**Orphan** — the reverse: the **parent** died first, leaving the child running. The child is adopted by **init (PID 1)**, which reaps it properly when it terminates.

One-line memory hook: **zombie = dead child, living parent. Orphan = living child, dead parent.**

### IPC — Inter-Process Communication

Processes have separate address spaces, so they can't just share variables. Two fundamental models:

**Shared memory** — the OS sets up a region both processes map into. After setup, they read and write directly with **no kernel involvement**, which makes it fast. The cost: *you* are responsible for synchronization — this is precisely where race conditions come from, and it's the bridge into Chapter 5.

**Message passing** — processes exchange messages through the kernel via `send()` and `receive()`. Slower (every message is a system call), but no synchronization burden, and it works across machines. Variants worth naming: blocking vs non-blocking (synchronous vs asynchronous), and direct vs indirect (naming a process vs using a mailbox).

**Mechanisms you should be able to list:** pipes (unidirectional, related processes only), named pipes/FIFOs (unrelated processes), message queues, shared memory, **sockets** (across a network), and signals.

The clean spoken contrast: **"shared memory is faster but I have to synchronize it myself; message passing is slower but the kernel handles the coordination."**

### Chapter 4 — CPU Scheduling (the numerical chapter)

**Why scheduling exists** — multiprogramming keeps several processes in memory; when one waits on I/O, the CPU must pick another. The short-term scheduler makes that pick.

**Preemptive vs non-preemptive** — the first thing they ask. Non-preemptive: once a process gets the CPU, it keeps it until it terminates or blocks on I/O. Preemptive: the OS can snatch the CPU away (time slice expires, or a higher-priority process arrives). Scheduling decisions happen at four moments — running→waiting, running→terminated (these two allow non-preemptive), running→ready and waiting→ready (these two are where preemption happens).

**The criteria** — maximize CPU utilization and throughput; minimize turnaround, waiting, and response time. Know which direction each goes.

**The three formulas** — everything on paper reduces to these:

- **TAT = CT − AT** (turnaround: submission to completion)
- **WT = TAT − BT** (waiting: time in ready queue)
- **RT = time of first CPU allocation − AT** (response; equals WT for non-preemptive algorithms)

**Template example** — the shape every numerical takes. Three processes, FCFS, all arriving at 0, bursts 24, 3, 3:

```
Gantt:  | P1        | P2 | P3 |
        0          24   27   30

P1: TAT=24, WT=0    P2: TAT=27, WT=24    P3: TAT=30, WT=27
Avg WT = (0+24+27)/3 = 17
```

Reorder to P2, P3, P1 and average WT drops to 3 — that single comparison *is* the argument for SJF, and it's also the **convoy effect** in action: short jobs crawling behind one long one.

**The algorithms** — for each, know three things: preemptive or not, its win, its fatal flaw.

| Algorithm | Preemptive? | Win | Fatal flaw |
| --- | --- | --- | --- |
| FCFS | No | Simple, fair in arrival order | Convoy effect |
| SJF | No | Minimal avg waiting time | Burst time unknowable; starves long jobs |
| SRTF | Yes (SJF + preemption) | **Provably optimal** avg WT | Same starvation, more context switches |
| Priority | Either | Important work first | **Starvation** → fixed by **ageing** |
| Round Robin | Always | Best response time; built for time-sharing | Quantum tradeoff |
| MLQ | — | Different policies per class | Process stuck in its queue forever → starvation |
| MLFQ | — | Processes **move between queues** | Complex to tune |

The follow-ups that score:

- **Why can't we just use SJF?** Burst time isn't known in advance — it's *predicted* via exponential averaging: τₙ₊₁ = α·tₙ + (1−α)·τₙ. Name the formula, don't derive it.
- **Ageing** — gradually raise the priority of anything waiting too long. One-word answer to "how do you fix starvation."
- **The Round Robin quantum** — too small: context-switch overhead eats the CPU. Too large: RR degenerates into FCFS. The quantum should be large relative to context-switch time.
- **MLQ vs MLFQ** — MLQ assigns a process to one queue permanently; MLFQ demotes CPU hogs and promotes waiters, which makes it ageing-by-design. When asked "what do real systems use," MLFQ-style is the safe answer.

### Chapter 5 — Synchronization (the setup for tomorrow's payoff)

**Race condition** — several processes access shared data concurrently and the result depends on execution order. The canonical example, and you should be able to walk it: two processes both run `counter++`, which is secretly three instructions — load, increment, store. Interleave them badly:

```
P1: load counter (5)
P2: load counter (5)
P1: increment → 6, store
P2: increment → 6, store      ← one update lost; should be 7
```

If asked to *define* a race condition, give this example unprompted. It's the difference between a memorized line and understanding.

**The critical section problem** — the part of the code touching shared data. Structure: **entry section → critical section → exit section → remainder section**. The problem: design the entry/exit protocol.

**The three requirements** — asked by name, answer by name:

1. **Mutual exclusion** — at most one process inside the CS at a time
2. **Progress** — if the CS is free, someone who wants in gets in; processes outside can't block the decision
3. **Bounded waiting** — a limit on how many times others can enter before a waiting process does (no starvation)

**Why the naive attempts fail** — this progression is the whole chapter, and interviewers walk you through it:

- **Simple lock variable** (`while(lock); lock=1;`) — fails mutual exclusion, because the check and the set aren't atomic. Both processes can read `lock == 0` before either writes. The race condition has just moved onto the lock itself. This is the most instructive failure in the chapter.
- **Strict alternation** (a `turn` variable) — mutual exclusion holds, but **progress fails**: if it's your turn and you don't want in, I'm blocked even though the CS is empty. Forced turn-taking.
- **Peterson's solution** — the fix for two processes: `flag[i]` ("I want in") plus `turn` ("but you first"). The intuition to say aloud: *I raise my flag, then politely give away the turn — I only wait if you want in AND it's your turn, so we can't both wait, and we can't both enter.* Satisfies all three requirements. Limitations: two processes only, busy waiting, and modern CPUs reorder memory operations so it's a teaching tool, not production code.
- **TSL / test-and-set** — the hardware answer: one **atomic** instruction that reads the lock and sets it in a single step, closing the gap the lock variable left. Gives mutual exclusion; doesn't by itself guarantee bounded waiting.

**Busy waiting / spinlock** — the thread sits in a loop burning CPU while it waits. Every solution in this chapter has this flaw, and *that* is the motivation for semaphores — a waiting process should sleep, not spin.

### Chapter 6 — Semaphores

**What a semaphore is** — an integer with exactly two atomic operations. That's the entire definition:

```
wait(S):    S-- ; if S < 0 → block, join S's queue     // also called P, down
signal(S):  S++ ; if anyone blocked → wake one          // also called V, up
```

The intuition that carried you today: **a counter of permits**. wait takes a permit or sleeps; signal returns one and wakes a sleeper. Know these two cold — every numerical and every classical problem is just these two lines applied.

**Why semaphores exist at all** — the punchline of Chapter 5: every homemade solution (lock variable, Peterson's, TSL) suffered **busy waiting** — spinning in a loop burning CPU. The semaphore's blocking queue replaces spinning with sleeping. If asked "why semaphores over Peterson's," that's the answer: no busy wait, and it generalizes past two processes.

**Binary vs counting** — binary: value only 0/1, one permit, used as a lock. Counting: any non-negative start value, N permits, controls access to N instances of a resource (connection pools, N buffer slots).

**Mutex vs semaphore** — your planner's most-asked question, and you now own the sharp version:

|  | Mutex | Semaphore |
| --- | --- | --- |
| Nature | Lock | Signaling counter |
| Ownership | **Locker must unlock — enforced** | Any thread may signal — no owner |
| Job | Protect a critical section | Locking *or* cross-thread ordering |
| Analogy | Bathroom key | Doorbell / permit stack |

The distinction you personally dug out: a binary semaphore around a CS is a lock **by convention** — the code promises the same thread signals back. A mutex makes it a **contract** the primitive enforces. And the litmus test for reading any code: same thread waits-and-signals → lock role; **crossed pair** (one thread waits, a different one signals) → signaling role, and no mutex could do that job.

**Semaphore arithmetic** — the objective-round staple. Final value = initial − (#waits) + (#signals). Example: start 10, six waits, four signals → 8. In the textbook implementation the value goes negative, and **|negative value| = number of blocked processes** (S = −3 → three sleeping).

#### The three classical problems — three-line cards + what breaks

The retention bar we set: players, what each semaphore counts, the one thing that breaks. Reconstruct-on-whiteboard beats recital.

**Producer–Consumer (bounded buffer)**

- Players: producer fills an N-slot shelf, consumer empties it.
- Counters: `empty = N` (free slots — producer waits on it), `full = 0` (filled slots — consumer waits on it), `mutex = 1` (the shelf key). Note `empty` and `full` are *crossed pairs* — consumer signals empty, producer signals full. That's signaling, not locking, in the wild.
- **What breaks:** swap the order — wait(mutex) *before* wait(empty) — and a producer facing a full shelf sleeps **holding the key**; the consumer needs that key to make space. Both waiting on each other: deadlock. Order is resource-then-key, always.

**Readers–Writers**

- Players: many readers may share; a writer needs the room alone.
- Machinery: `read_count` (plain int), `mutex` guarding read_count only, `wrt` — the room lock. **First reader in locks wrt on behalf of all readers; last reader out releases it.** Writers just wait(wrt)/signal(wrt).
- Your trace from today, keep it: writer already inside → reader 1 blocks on wrt *while holding mutex* → every later reader queues on mutex. Looks like the deadlock pattern, isn't one — the waker (writer) never needs mutex. **Chain, not loop.** Deadlock needs the cycle.
- **What breaks:** fairness. read_count only hits zero when readers stop coming — a steady reader stream **starves the writer**. This version is "readers-preference"; the fix (a turnstile semaphore queueing everyone) exists but the name is enough.

**Dining Philosophers**

- Players: 5 philosophers, 5 forks (each fork a binary semaphore); eating needs both neighbors' forks.
- **What breaks:** all five grab their left fork simultaneously → everyone holds one, waits on the right, nobody ever releases. A perfect **circular wait** — this is the trailer for Chapter 7.
- Fixes, name any one: allow at most 4 at the table; odd philosophers pick left-first, even pick right-first (breaks the circle's symmetry); pick up both forks atomically.

### Chapter 7 — Deadlock (up to Banker's)

**Definition** — a set of processes where each holds a resource and waits for a resource held by another member of the same set. Nobody can proceed, ever. Distinguish from starvation on demand: **deadlock = nobody moves; starvation = the system moves, one process never wins** (the low-priority job under priority scheduling, the writer under readers-preference).

**The four Coffman conditions** — asked *as a list, by name*, and all four must hold simultaneously:

1. **Mutual exclusion** — the resource can't be shared
2. **Hold and wait** — holding one while requesting another
3. **No preemption** — resources can't be snatched back
4. **Circular wait** — P1→P2→…→Pn→P1

Dining philosophers scores all four, which is why it's the canonical example.

**Resource Allocation Graph** — request edge P→R, assignment edge R→P. The reading rule: **single-instance resources: cycle ⇔ deadlock, guaranteed. Multi-instance: cycle means deadlock is *possible*, not certain.** That asymmetry is a favorite MCQ.

**The four handling strategies** — prevention, avoidance, detection & recovery, and ignoring it:

**Prevention** = design so one Coffman condition can never hold. Per-condition, with the cost:

| Negate | How | Why it hurts |
| --- | --- | --- |
| Mutual exclusion | Make resources sharable | Usually impossible — printers aren't sharable |
| Hold & wait | Request *everything* upfront | Terrible utilization, possible starvation |
| No preemption | Snatch resources from waiters | Only works for save/restore-able resources |
| **Circular wait** | **Global resource ordering — request in increasing number only** | **The practical one; this is the real answer** |

**Avoidance** — don't forbid conditions; check each request against future safety. Key vocabulary: a **safe state** is one from which *some* execution order lets every process finish. **Unsafe ≠ deadlocked — unsafe means deadlock is now possible and the OS has lost the ability to guarantee it won't happen.** The algorithm that does this checking is Banker's — **tomorrow's first item, watched fresh right before you solve it on paper.**

**Detection & recovery** — let deadlock happen, find cycles periodically, then recover by killing processes (all, or one-by-one cheapest-first) or preempting resources.

**Ostrich algorithm** — ignore the problem because it's rare and the cures cost more than the disease. Punchline worth one mark in any interview: **this is what Windows and Linux actually do** for general processes.

### Chapter 7, finished — Banker's Algorithm

**What it is** — the deadlock-*avoidance* algorithm: before granting any request, the OS simulates the future and only says yes if the system stays in a **safe state**. Named for a banker who never lends cash unless he can still satisfy some order of all customers' maximum demands.

**The four structures** — everything is vectors and matrices, per resource type:

- **Available** — free instances of each resource right now
- **Max** — each process's declared maximum demand
- **Allocation** — what each currently holds
- **Need = Max − Allocation** — what each might still ask for. Compute this first in every problem; half the arithmetic errors happen here.

**Safety check, the loop you'll run on paper at 1:00** — find any process whose **Need ≤ Available** (every component). Pretend it runs to completion and releases everything: **Available += its Allocation**. Cross it off. Repeat. All crossed off → **safe**, and the crossing order is the **safe sequence**. Stuck with processes remaining → unsafe.

**Reference template** (my numbers — your gap problems follow the same shape, usually with 5 processes):

Total = (10, 5, 7) of resources A, B, C.

|  | Allocation | Max | Need |
| --- | --- | --- | --- |
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |

Allocated totals (5, 1, 2), so **Available = (5, 4, 5)**.

P1's need (1,2,2) fits → runs, releases → Available (7,4,5). P0's (7,4,3) fits → (7,5,5). P2 fits → done. **Safe, sequence ⟨P1, P0, P2⟩.** Note P0 did *not* fit at the start — order matters, and finding *any* one that works is enough.

**Request-granting version** — when Pᵢ asks for Request: three gates, in order. (1) Request ≤ Need (else it lied about Max — error). (2) Request ≤ Available (else it waits). (3) *Pretend* to grant — Available −= R, Allocation += R, Need −= R — and rerun the safety check. Safe → grant for real. Unsafe → roll back, Pᵢ waits. The exam trick: an "unsafe" verdict doesn't mean deadlock happened — it means the banker refuses to gamble.

**The interview follow-up** — "why doesn't anyone actually use Banker's?" Because it demands every process declare its maximum need *in advance*, and assumes a fixed set of processes and resources. Real workloads can't promise either. Say that and you've out-answered most candidates.

*Ch 8 in one line: everything's in your Day-1 fork/threads notes; the only video-specific add is fork-tree counting — n forks → 2ⁿ processes — trace one tree once and move on.*

### Chapter 9 — Memory Management

**Logical vs physical address** — logical (virtual): what the CPU generates, what your program sees. Physical: the actual DRAM location. The **MMU** translates between them at runtime. One-liner: *the program lives in logical space; the hardware lives in physical space; the MMU is the interpreter.*

**Base + limit** — oldest protection scheme: every access is checked against limit, then base is added. Exceed the limit → trap. Keep this; it returns as segmentation's mechanism.

**Contiguous allocation and the three fits** — early scheme: each process gets one continuous block, leaving variable-sized holes. Placing a new process:

- **First fit** — first hole big enough. Fast, and in practice about as good as it gets.
- **Best fit** — smallest sufficient hole. Sounds smart; manufactures unusably tiny slivers.
- **Worst fit** — largest hole. Almost always the wrong answer, which is exactly why MCQs include it.

**Internal vs external fragmentation** — asked *constantly*, so get the pairing straight:

- **Internal** — waste *inside* an allocated block: you were given more than you use. Paging's disease (the half-empty last page).
- **External** — total free memory is sufficient but scattered in non-contiguous holes, so a request fails anyway. Contiguous allocation's and segmentation's disease.
- **Compaction** cures external by shuffling processes together — expensive, needs relocatable code. Paging cures it structurally, which is the entire reason paging exists. That sentence is the bridge to everything below.

**Paging — the core mechanism.** Chop logical memory into fixed **pages**, physical memory into same-sized **frames**; any page can sit in any frame. A per-process **page table** maps page number → frame number. Address translation:

Logical address splits into **(page number p | offset d)** — offset bits = log₂(page size). Physical = frame(p) × page-size + d.

**The bit-math staple** (this exact pattern appears in objective rounds): 32-bit logical address, 4 KB pages → 12 offset bits, leaving 20 for p → **2²⁰ ≈ 1M page-table entries** per process. At 4 bytes each, that's a 4 MB table — *per process*. Hold that outrage; it motivates the next two ideas.

**TLB — and the EMAT formula.** The page table lives in memory, so naive paging doubles every memory access (one for the table, one for the data). The **TLB** is a small associative cache of recent translations inside the MMU. With hit ratio h, TLB time t, memory access m:

**EMAT = h·(t + m) + (1 − h)·(t + 2m)**

Worked: h = 0.8, t = 20 ns, m = 100 ns → 0.8(120) + 0.2(220) = 96 + 44 = **140 ns**. This is *the* memory-chapter numerical — expect it with different numbers, sometimes asking for h given a target EMAT (same equation, solve backwards). Context switch footnote: the TLB gets flushed (or is tagged with process IDs), which is part of why context switches cost.

**Multilevel paging** — the 4 MB-table fix: page the page table itself. An outer table points to inner tables, and unused address ranges simply have no inner table allocated. Cost: each extra level adds one more memory access on a TLB miss — space bought with time, and the TLB is what makes the bill affordable.

**Segmentation** — from earlier today, now in its proper seat: divide by *meaning* (code, data, stack — programmer-visible), addresses become (segment, offset), table stores **base + limit**, offset ≥ limit → trap — and that trap is the **segmentation fault**. The contrast row you starred stays the exam answer: paging = fixed size, invisible, internal fragmentation; segmentation = variable size, visible, **external** fragmentation — reinventing the disease paging cured, which is why real systems page.

**Inverted page table** — the one-liner: one system-wide table indexed by *frame*, each entry naming which (process, page) occupies it. Saves enormous space, makes lookup a search — patched by hashing and absorbed by the TLB. Recognition depth only.

## Chapter 10 — Virtual Memory, in the order he teaches it

### 1. The idea of virtual memory

The problem it solves: until now, a process had to be fully loaded in RAM to run. But RAM is small and programs are big.

The trick: **keep only the pages the process is actually using in RAM.** Everything else waits on disk, in an area called **swap space**. The process has no idea — it sees one big continuous memory, and the OS quietly shuffles pages in and out behind its back.

What this buys you, and both get asked:

- A program can be **bigger than physical RAM** and still run.
- Each process takes less RAM, so **more processes fit at once** — more multiprogramming, better CPU utilization.

### 2. Demand paging

The loading strategy that makes virtual memory work: **don't load a page until the process actually asks for it.** Lazy loading. If the process never touches a page, it never gets loaded.

How the OS knows a page isn't there: every page table entry carries a **valid–invalid bit**. Valid = the page is sitting in some frame in RAM, go ahead. Invalid = the page is not in RAM (it's on disk, or it doesn't belong to this process at all).

If a process starts with *zero* pages loaded, that's called **pure demand paging** — the very first instruction causes a fault.

### 3. Page fault — what happens when the page isn't there

The CPU asks for a page, the MMU sees the invalid bit, and now the hardware alone can't proceed. This is a **page fault**, and the OS handles it in a fixed sequence. Learn it as a story, because it gets asked as "explain the steps":

1. CPU touches the page → MMU finds the invalid bit
2. **Trap to the OS** — hardware hands over control
3. OS checks: is this a real page on disk, or an illegal address? Illegal → kill the process (this is your segfault cousin)
4. Find a free frame. **No free frame? Run a page replacement algorithm to evict someone** — that's where section 5 comes from
5. Read the page from disk into the frame *(the slow part — milliseconds, while RAM is nanoseconds; ~100,000× slower)*
6. Fix the page table: new frame number, bit set to valid
7. **Restart the instruction that faulted** — it runs again from scratch, it doesn't resume halfway

Step 7 is the one people forget, and step 5 is the reason the next formula exists.

### 4. Performance of demand paging — the EAT formula

If faults are that slow, how slow does memory *feel on average*? With fault rate **p**:

**EAT = (1 − p) × memory access time + p × page fault service time**

Plug in his usual style of numbers: access = 200 ns, fault service = 8 ms = 8,000,000 ns, p = 0.001 (one fault per thousand accesses):

EAT = 0.999 × 200 + 0.001 × 8,000,000 = 199.8 + 8000 ≈ **8200 ns**

Read that result out loud: one fault in a thousand made memory **forty times slower**. That's the entire point — page faults are so expensive that even a tiny rate dominates everything. Exam variant: "what p keeps slowdown under 10%?" — same equation, solve for p.

### 5. Page replacement — the setup

Step 4 above said "evict someone." Evict *who* is the question the rest of the chapter answers.

The exam format: you get a **reference string** (just the sequence of page numbers the process asks for, in order) and a fixed number of **frames** (slots in RAM). You simulate the algorithm and **count the page faults**. Fewer faults = better algorithm.

One thing to get straight before any table — the thing that confused you on the slide: **the boxes are physical frames, not a queue.** A page enters a frame and *stays in that exact frame* until it's evicted. The newcomer takes over whichever frame just got freed. Pages never slide down to make room — moving a page between frames would mean copying 4 KB of memory for zero benefit. The "who came first" information lives in the OS's bookkeeping, not in the picture.

All three algorithms below run on the same string — the one from his slide:

**7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1** — with **3 frames**.

### 6. FIFO — first in, first out

Rule: evict the page that has been in memory the **longest**. Doesn't matter how much it's being used — oldest arrival goes.

Walk the start slowly, exactly as on the slide. 7, 0, 1 arrive and fill the three frames — three faults, unavoidable (every algorithm eats these "cold start" faults). Now **2** arrives and everything is full. Who's oldest? 7. So 2 replaces 7 **in frame 1**, and the frames read 2 / 0 / 1 — top to bottom, exactly the column you asked about. Not 0 / 1 / 2, because nobody moves.

Full trace (F = fault, · = hit):

```
ref   7   0   1   2   0   3   0   4   2   3   0   3   2   1   2   0   1   7   0   1
f1    7   7   7   2   2   2   2   4   4   4   0   0   0   0   0   0   0   7   7   7
f2    -   0   0   0   0   3   3   3   2   2   2   2   2   1   1   1   1   1   0   0
f3    -   -   1   1   1   1   0   0   0   3   3   3   3   3   2   2   2   2   2   1
      F   F   F   F   ·   F   F   F   F   F   F   ·   ·   F   F   ·   ·   F   F   F
```

**15 faults.** Simple to build, and genuinely bad — a page can be evicted *while being heavily used*, just for being old. And FIFO has one more embarrassment waiting in section 9.

### 7. Optimal (OPT) — the impossible benchmark

Rule: evict the page that **won't be needed for the longest time in the future**. Look ahead in the reference string, find whose next use is farthest away, evict that one.

```
ref   7   0   1   2   0   3   0   4   2   3   0   3   2   1   2   0   1   7   0   1
f1    7   7   7   2   2   2   2   2   2   2   2   2   2   2   2   2   2   7   7   7
f2    -   0   0   0   0   0   0   4   4   4   0   0   0   0   0   0   0   0   0   0
f3    -   -   1   1   1   3   3   3   3   3   3   3   3   1   1   1   1   1   1   1
      F   F   F   F   ·   F   ·   F   ·   ·   F   ·   ·   F   ·   ·   ·   F   ·   ·
```

**9 faults** — and this is *provably* the minimum possible on this string. The catch, and it's the exam question: **you can't implement it**, because it requires knowing the future. So what is it for? It's the **benchmark** — you measure real algorithms by how close they get to Optimal. That is its entire job.

### 8. LRU — least recently used

Since we can't see the future, use the past as a prediction: a page that hasn't been touched in a long time probably won't be touched soon. Rule: evict the page whose **last use is furthest in the past**.

```
ref   7   0   1   2   0   3   0   4   2   3   0   3   2   1   2   0   1   7   0   1
f1    7   7   7   2   2   2   2   4   4   4   0   0   0   1   1   1   1   1   1   1
f2    -   0   0   0   0   0   0   0   0   3   3   3   3   3   3   0   0   0   0   0
f3    -   -   1   1   1   3   3   3   2   2   2   2   2   2   2   2   2   7   7   7
      F   F   F   F   ·   F   ·   F   F   F   F   ·   ·   F   ·   F   ·   F   ·   ·
```

**12 faults.** Sits between FIFO's 15 and Optimal's 9 — which is the whole story: LRU is the practical approximation of Optimal.

How would you actually build LRU? Two textbook ways: **counters** (stamp every page on every use, evict the smallest stamp) or a **stack** (touch a page → move it to the top; evict from the bottom). Both are expensive — real hardware won't timestamp every access. So real systems cheat with a **reference bit** and the **second chance (clock) algorithm**: run FIFO, but if the oldest page's reference bit is 1, forgive it once — clear the bit, move on, check the next. One sentence is all you need on it.

Scoreboard for the string: **FIFO 15 · LRU 12 · Optimal 9.**

### 9. Belady's anomaly — the reason FIFO is famous

Common sense says: more frames → fewer faults. **With FIFO, that can be false.** More frames can produce *more* faults. Nobody believes it until they see it, so watch it happen on the classic string **1 2 3 4 1 2 5 1 2 3 4 5**:

**With 3 frames:** 1, 2, 3 fault in. 4 evicts 1. Then 1 evicts 2, 2 evicts 3, 5 evicts 4 — faults piling up. Now 1 and 2 are **hits** (they're resident). 3 evicts 1, 4 evicts 2, and 5 is a hit. **Total: 9 faults.**

**With 4 frames:** 1, 2, 3, 4 fault in. 1 and 2 hit — looking good. But then 5 evicts 1, 1 evicts 2, 2 evicts 3, 3 evicts 4, 4 evicts 5, 5 evicts 1 — a chain where *every single remaining reference faults*. **Total: 10 faults.**

More frames, more faults. **Why does this hit FIFO and not LRU/Optimal?** LRU and Optimal are **stack algorithms**: whatever pages they'd keep in 3 frames is always a *subset* of what they'd keep in 4. An extra frame can only add, never disturb — so it can never hurt. FIFO doesn't have that property; adding a frame reshuffles its entire eviction sequence. The interview-grade answer is "FIFO is not a stack algorithm," not "FIFO is weird."

### 10. Thrashing

The chapter's closing disaster. **Thrashing = the system spends more time swapping pages than executing code.**

How you get there — and it's a feedback loop, which is what makes it exam-worthy: a process has too few frames → it faults constantly → the CPU sits idle waiting for the disk → the OS looks at the idle CPU and concludes "I'm underloaded, let me admit **more** processes" → now everyone has even fewer frames → more faulting → CPU drops further. Round and round until CPU utilization is near zero and the disk light never turns off. The OS's own load-balancing instinct is what drives it off the cliff.

The root cause, in one word: **locality**. At any moment a process actively uses only a small set of pages (a loop, its data). If its frames can't hold that set, it faults on every step of the loop, forever.

Two fixes:

- **Working set model** — watch which pages a process touched in the last Δ references; that set is its working set. Give every process enough frames for its working set, and if all the working sets together exceed total frames, **suspend a process entirely** — better one process paused than everyone thrashing.
- **Page fault frequency (PFF)** — skip the theory, measure directly: if a process faults too often, give it frames; if it barely faults, take some back.

Related vocabulary he closes with: **equal vs proportional allocation** (split frames evenly, or by process size) and **local vs global replacement** (may a faulting process evict only its *own* pages, or anyone's?). Global gives better overall throughput; local protects processes from a greedy neighbour — which is itself a mini thrashing-containment strategy.

## Self-test — say these aloud tonight

The two things virtual memory buys · valid–invalid bit · the seven fault steps, especially "restart the instruction" · EAT with numbers, and why 0.1% hurts 40× · frames are fixed slots — why the slide showed 2/0/1 and not 0/1/2 · FIFO, LRU, Optimal rules plus their scores (15 / 12 / 9) · why Optimal exists if it can't be built · Belady's with the 9-vs-10 example · why stack algorithms are immune · second chance in one line · the thrashing feedback loop · working set vs PFF · local vs global.

# OS — Chapters 11 & 12: Disk & File Systems

*The two chapters you skipped in the video, deliberately, on the grounds that they're light. That was the right call — but "light" isn't "zero," and this is the part that's actually asked. Roughly 30 minutes of reading, and it closes the subject.*

**How to use this:** same as every other file — aloud-first, star the fumbles. This is your Tuesday-night patch material, and it pairs with the GFG Last Minute Notes skim. Everything here is either a standard interview question or a numerical you could be handed in an objective round.

---

## Part 1 · How a disk actually works

Before any algorithm makes sense, the physical picture, because every formula below is just a consequence of it.

A hard disk is a stack of spinning **platters**. Each platter surface is divided into concentric rings called **tracks**, and each track into **sectors** (traditionally 512 bytes, now often 4 KB). The same track number across all platters forms a **cylinder** — a vertical column through the stack. One **read/write head** floats over each surface, and all the heads move together on a single arm.

To read a sector, three things have to happen, and their costs differ by orders of magnitude:

| Component | What it is | Rough cost |
| --- | --- | --- |
| **Seek time** | Moving the arm to the right track | **~5–10 ms — dominant** |
| **Rotational latency** | Waiting for the sector to spin under the head | ~2–4 ms (half a rotation on average) |
| **Transfer time** | Actually streaming the bytes | microseconds |

**Everything in this chapter follows from one fact: seek time dominates.** It's mechanical — an arm physically moving across a platter — while the other two are fast or unavoidable. So the entire job of a disk scheduling algorithm is to **minimise total arm movement**. That's it. If you understand that sentence, every algorithm below is obvious.

Two derived facts worth having ready:

**Average rotational latency = half of one full rotation.** At 7200 RPM, one rotation is 60/7200 s ≈ 8.33 ms, so average latency ≈ **4.17 ms**. That calculation appears in objective rounds — memorise the method, not the number.

**SSDs change everything.** No arm, no platters, no seek time — access is uniform regardless of location. Which means **disk scheduling algorithms are largely irrelevant to SSDs**; they exist to optimise a mechanical constraint that no longer exists. Saying this earns a point, because it shows you understand *why* the algorithms exist rather than just their names.

---

## Part 2 · Disk scheduling algorithms

The standard setup: a queue of pending track requests, and the head currently at some track. Compute **total head movement** — the sum of distances travelled.

Worked on one example throughout, so the algorithms are directly comparable:

> **Queue:** 98, 183, 37, 122, 14, 124, 65, 67
**Head starts at 53.** Disk has tracks 0–199.
> 

### FCFS — first come, first served

Serve requests in arrival order. Fair, simple, and often terrible — the arm can swing wildly back and forth.

```
53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67
45 + 85 + 146 + 85 + 108 + 110 + 59 + 2      = 640 tracks
```

**Use it as the baseline every other algorithm beats.**

### SSTF — shortest seek time first

Always serve the **closest pending request** next. Greedy.

```
53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183
12 + 2 + 30 + 23 + 84 + 24 + 2 + 59           = 236 tracks
```

Far better than FCFS — and it has a familiar flaw. **SSTF can starve** requests at the edges of the disk: if new requests keep arriving near the head, a request out at track 190 may wait forever. This is SJF's starvation problem in a different costume, and interviewers like the parallel.

### SCAN — the elevator algorithm

The head sweeps in **one direction**, serving everything on the way, then reverses at the end of the disk and sweeps back. Exactly like a lift: it doesn't jump to whoever pressed the button first, it finishes going up, then comes down.

Moving toward 0 first:

```
53 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183
16 + 23 + 14 + 65 + 2 + 31 + 24 + 2 + 59      = 236 tracks
```

**No starvation** — the sweep guarantees every request is reached within one pass. Note the head travels all the way to track 0 even with nothing there; that's the definition.

### C-SCAN — circular SCAN

Same as SCAN, but on reaching the end it **jumps straight back to the start without serving anything on the return**, and sweeps in the same direction again.

Why bother, given the wasted return trip? **Uniform waiting time.** Under plain SCAN, a track in the middle gets visited twice per cycle (once going up, once coming down) while a track at the edge is visited once — so waiting time depends on where you sit. C-SCAN treats the disk as a **circular list**, so every track is visited exactly once per cycle and waits are even.

That's the single most-asked question about these two: **SCAN sweeps both ways and favours the middle; C-SCAN sweeps one way only and treats all tracks fairly.**

### LOOK and C-LOOK

The practical refinements: identical to SCAN and C-SCAN, except the head **reverses at the last actual request** rather than travelling all the way to the physical end of the disk.

LOOK on the same queue (toward 0 first) stops at 14, not 0 — saving 28 tracks of pointless movement. **In practice, real systems use LOOK/C-LOOK, not SCAN/C-SCAN** — going to a track nobody asked for is pure waste.

### Summary

| Algorithm | Rule | Starvation? | Note |
| --- | --- | --- | --- |
| FCFS | Arrival order | No | Baseline; poor |
| SSTF | Nearest request | **Yes** | Greedy, good average |
| SCAN | Sweep to end, reverse | No | Favours middle tracks |
| C-SCAN | Sweep one way, jump back | No | **Uniform waiting** |
| LOOK | Sweep to last request | No | SCAN without the waste |
| C-LOOK | One-way to last request | No | The practical choice |

---

## Part 3 · File systems

### What a file is, to the OS

A **file** is a named collection of related information on secondary storage — the OS's abstraction over raw blocks, so you never think about sectors. Attributes stored per file: name, identifier, type, location, size, protection bits, timestamps.

A **directory** is itself a file whose contents are a table mapping names to file locations.

### The inode — the concept to know cold

In Unix-family filesystems, every file has an **inode** (index node): a structure holding **everything about the file except its name** — permissions, owner, size, timestamps, link count, and pointers to the data blocks.

**The name lives in the directory entry, not the inode.** That separation is the single most important idea in this section, and it explains a surprising amount:

- A **directory** is just a table of `(name → inode number)` pairs
- **Hard links** work because several directory entries can point to one inode — they're peer names for the same file, and the data survives until the inode's **link count** hits zero
- **A soft link** is a separate file whose *contents* are a path string, so deleting the target leaves it dangling
- **Renaming a file is cheap** — you edit a directory entry; the inode and the data never move

**Inode structure** — the classic multi-level design. An inode holds a fixed number of pointers: about 12 **direct** pointers to data blocks, then one **single indirect** (points to a block of pointers), one **double indirect** (a block of pointers to blocks of pointers), and one **triple indirect**. Small files are reached in one hop; huge files are still reachable, at the cost of extra lookups.

That's the elegance to note: **small files stay fast, large files stay possible.** The same structural idea as multilevel page tables — pay indirection only when you need it.

### File allocation methods

How a file's blocks are laid out on disk. Three schemes, and the tradeoff table is the answer:

**Contiguous** — the file occupies consecutive blocks. Excellent sequential *and* direct access (block *k* is just `start + k`), minimal seeking. But it suffers **external fragmentation**, and you must know the file's final size in advance — growing a file may require relocating it entirely.

**Linked** — each block contains a pointer to the next. No external fragmentation, files grow freely. But **direct access is impossible** — reaching block 50 means walking 50 pointers — and one corrupt pointer orphans the rest of the file. (FAT is a variant that keeps all the pointers in one table, restoring direct access.)

**Indexed** — one **index block** holds all the file's block pointers. Direct access restored, no external fragmentation. Costs a whole block of overhead per file, painful for tiny files. **The inode is essentially this scheme, extended with indirection for large files.**

|  | Sequential access | Direct access | Fragmentation | Growth |
| --- | --- | --- | --- | --- |
| Contiguous | Excellent | Excellent | **External** | Hard |
| Linked | Good | **Impossible** | None | Easy |
| Indexed | Good | Good | None (internal only) | Easy |

### Free space management

The OS must track which blocks are free. Two methods worth naming: a **bit vector** (one bit per block, 0 = free — simple, and fast to find runs of free blocks using hardware instructions, but the vector must fit in memory) and a **linked list** of free blocks (no memory overhead, but finding contiguous space is slow).

### Directory structure

Single-level (one namespace, no two files may share a name), two-level (per user), and **tree-structured** — what every real system uses, giving paths, subdirectories, and a current working directory. Acyclic-graph structures allow shared subdirectories via links.

### Absolute vs relative path

Absolute starts from the root (`/home/shadab/notes.md`); relative starts from the current working directory (`notes.md`, `../docs/file.txt`). Trivial, occasionally asked.

---

## Self-test — say these aloud

1. The three components of disk access, and which one dominates
2. Why average rotational latency is half a rotation — compute it for 7200 RPM
3. FCFS vs SSTF, and SSTF's flaw
4. SCAN as the elevator analogy
5. **SCAN vs C-SCAN — why the wasted return trip is worth it**
6. LOOK vs SCAN, and why real systems prefer LOOK
7. Work the example queue (98, 183, 37, 122, 14, 124, 65, 67 from head 53) under SSTF and SCAN, and get 236 both times
8. Why disk scheduling barely matters on an SSD
9. What an inode holds — and the one thing it doesn't
10. Why that omission explains hard links, soft links, and cheap renaming
11. Direct / single / double / triple indirect — and the design principle behind it
12. Contiguous vs linked vs indexed, with each one's fatal flaw
13. Bit vector vs linked list for free space

---

*With this, the OS syllabus is closed — all twelve chapters, either watched or patched. What remains is drilling, not learning.*

## Quick Revision — Polling, Thread Pools & File Allocation

### Polling vs Interrupts

**Polling** means the CPU repeatedly checks a device to see whether it needs attention. This wastes CPU cycles when nothing has happened because the CPU keeps asking.

**Interrupts** work the opposite way: the CPU continues doing other work, and the device sends an interrupt signal when an event needs attention. The CPU then temporarily handles that event and resumes its previous work.

**Memory line:** Polling = CPU keeps checking. Interrupt = device tells the CPU.

### Thread Pool

A **thread pool** is a pre-created group of reusable worker threads. Tasks are placed into a queue, and whichever worker is free picks up the next task.

Instead of repeatedly doing `create thread → execute task → destroy thread`, the same threads are reused. This reduces thread creation/destruction overhead and also limits concurrency so the system does not create an uncontrolled number of threads.

A thread pool can also keep a main/request-handling thread responsive by moving work to workers, but that is a use case rather than the definition.

**Memory line:** Thread pool = reusable workers + task queue.

### File Allocation Methods

**Contiguous allocation:** all blocks of a file are stored next to each other on disk. Access is fast, including random access, but finding a large enough continuous free region can become difficult and fragmentation can occur.

**Linked allocation:** each file block contains a pointer to the next block. Blocks can be scattered anywhere on disk, so placement is flexible, but random access is poor because the system may need to follow the chain block by block.

**Indexed allocation:** a separate index block stores pointers to all of the file's data blocks. The blocks may be scattered, but the index allows the system to locate a particular block directly, giving much better random access than linked allocation.

**Memory line:** Contiguous = adjacent blocks. Linked = chain of blocks. Indexed = index points to blocks.