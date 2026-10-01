# DSA Pattern Recognition — LSEG Sprint

<aside>
🎯

**Purpose:** this is not a catalogue to memorize. It is a recognition sheet for converting a new problem into a likely technique quickly. Your main weakness is not reasoning; it is the time taken before the right structure becomes obvious. Use these notes to compress that recognition time.

</aside>

---

# 1. How to use the common-pattern block

Do **not** spend the block studying one pattern deeply after another. With only a few days left, that would consume hours and still leave you weak on unseen questions. You already have substantial solving experience, so the useful goal is **retrieval and recognition**, not first-time learning.

Use a 90-minute common-pattern block like this:

1. **0–10 min — complexity + sorting recall.** Recite the complexity limits from constraints and the main sorting properties. Do not code unless one of the standard sorts feels genuinely forgotten.
2. **10–35 min — trigger drill.** Take 10–15 problem statements or titles. For each, spend at most 90 seconds saying: *constraints → brute force → bottleneck → likely pattern → target complexity*. Do not solve the problem.
3. **35–60 min — implementation recall.** From memory, write only the small reusable algorithms that must come automatically: binary search, BFS/DFS, DSU, heap/PQ usage, linked-list reversal, topological sort, and one DP shape. Stop once the structure is correct.
4. **60–80 min — weak-pattern repair.** Open this page only for patterns you failed to recognize quickly. Spend at most 5–7 minutes on each. The goal is to remember the trigger and invariant, not to read ten examples.
5. **80–90 min — produce the output.** Mark each high-yield pattern **Green / Yellow / Red**. Carry only the Yellow/Red list into the next practice block.

<aside>
✅

**The output of the block should be:** (1) a trigger you can say for each major pattern, (2) the expected complexity, (3) the invariant or key state, and (4) a maximum of 3–4 weak patterns that need practice. If you finish with pages of rewritten notes, the block was used badly.

</aside>

The local Pattern Index uses the same logic: reverse-read the problem, restate it, use constraints to limit the possible complexities, establish brute force, locate the bottleneck, then identify the trigger before coding. That is the process to rehearse.

# 2. The 60-second decision process for any unseen problem

Before searching your memory for a named pattern, answer these questions in order.

**1. What does the constraint permit?** If `n` is around `10^5`, an `O(n^2)` idea is almost certainly not the intended final solution. If `n <= 20`, exponential subset/backtracking approaches become realistic. Complexity removes many false choices immediately.

**2. Is the order fixed?** If the problem is about a contiguous subarray or substring, order is fixed and sliding window, prefix sum, stack or DP become plausible. If the elements may be reordered freely, sorting or greedy reasoning becomes much more likely.

**3. What is repeated in brute force?** Repeated range sums suggest prefix sums. Repeated membership checks suggest hashing. Repeated search over a monotonic answer suggests binary search. Repeated overlapping subproblems suggest DP.

**4. What information from the past is actually needed?** If only the most recent unresolved element matters, think stack. If only the best candidate matters, think heap or running optimum. If a whole window matters, think two pointers/sliding window. If connectivity evolves, think DSU.

**5. What would make the solution invalid?** Sliding window often fails when negative numbers destroy monotonicity. Dijkstra fails with negative edges. Greedy needs an exchange or monotonic argument. Recognizing when a pattern does **not** apply is as important as recognizing when it does.

# 3. Sorting algorithms — what you actually need to remember

You should be able to explain the standard sorts without spending much practice time on them.

| Algorithm | Time | Space / property | Recognition point |
| --- | --- | --- | --- |
| Bubble sort | `O(n²)`, best `O(n)` with early-stop | In-place, stable | Repeated adjacent swaps; mostly theoretical/interview basics. |
| Selection sort | `O(n²)` | In-place, usually unstable, few swaps | Repeatedly select the minimum for the next position. |
| Insertion sort | `O(n²)`, best `O(n)` | In-place, stable, adaptive | Good for small or nearly sorted data. |
| Merge sort | `O(n log n)` | `O(n)` extra, stable | Divide, sort halves, merge. Predictable worst-case. |
| Quick sort | Average `O(n log n)`, worst `O(n²)` | Typically in-place, unstable | Partition around pivot; excellent average practical performance. |

For interview recall, know **why merge sort is stable/predictable** and **why quicksort can degrade** with bad pivots. Do not spend thirty minutes reproducing bubble sort code.

# 4. Hashing / frequency / “have I seen this?”

### When to suspect it

The problem repeatedly asks whether a value has appeared, how often something occurs, whether a complement exists, or groups elements by a computed signature.

Typical phrases are **duplicate**, **frequency**, **pair summing to K**, **anagram**, **seen before**, **count occurrences**, or **group by property**.

### Core invariant

A hash structure stores information that brute force would repeatedly search for. While scanning the current element, the map/set represents the useful history seen so far.

### Watch-outs

Do not use a set when multiplicity matters; use a frequency map. For pair-sum style problems, be clear whether you check the complement before inserting the current value. If values lie in a tiny known alphabet, an array can be simpler and faster than a hash map.

### Recognition test

If your brute force contains an inner loop whose only purpose is **“find whether X exists”**, hashing should immediately enter your candidate list.

# 5. Two pointers

### When to suspect it

Two pointers are strongest when the data is sorted or when two boundaries move monotonically rather than restarting. Common triggers are **sorted pair/triple**, **remove duplicates in place**, **partition**, **palindrome**, or processing from both ends.

### Core invariant

Each pointer movement permanently eliminates part of the search space. You should be able to explain why moving a pointer cannot discard a valid answer.

### Watch-outs

Do not force two pointers onto unsorted data unless the logic still has a monotonic reason. If you sort first, check whether reordering destroys information such as original indices.

### Recognition test

Ask: *If the sum/value is too small, can I safely move only one side in one known direction?* If yes, the problem may have a two-pointer structure.

# 6. Sliding window

### When to suspect it

The problem asks about a **contiguous** subarray/substring and you can update the condition when the left or right boundary moves. Common wording: longest/shortest segment, at most K, no more than K distinct, fixed-size window, maximum in every window.

### Core invariant

The current window is maintained incrementally. With a variable window, expanding right introduces new information; while invalid, advance left until validity is restored.

### Watch-outs

The classic sum-based window depends on monotonic behaviour. With negative values, shrinking the window does not necessarily reduce a sum in a predictable way, so prefix sums + hashing may be required instead. Distinguish **fixed window** from **variable window**.

### Recognition test

If brute force repeatedly recomputes a property for every contiguous range, ask whether that property can be **added on the right and removed on the left**.

# 7. Prefix sums / difference arrays

### When to suspect it

You need many range sums/counts, subarray sums, or repeated range updates.

### Core idea

A prefix sum converts a range query into subtraction: information over `[l..r]` becomes `prefix[r+1] - prefix[l]`. For subarray-sum counting with negatives, combine prefix sums with a hash map of previously seen prefix values.

A **difference array** is the opposite direction: many range updates are recorded only at boundaries, then one final prefix pass reconstructs the values.

### Watch-outs

Be precise with indexing and whether the prefix array has length `n` or `n+1`. Use `long` when cumulative sums can exceed `int`.

### Recognition test

If brute force repeatedly recomputes a range aggregate that can be expressed as **total to R minus total before L**, think prefix sum.

# 8. Stack and monotonic stack

## Ordinary stack

Use a stack when processing an item may resolve or cancel the **most recent unresolved item**. Brackets, expression parsing, nested structures, undo semantics and adjacent-cancellation problems have this shape.

The mental sentence is: **“What I just processed can be undone/resolved by what comes next.”**

## Monotonic stack

### When to suspect it

The question asks for **next/previous greater or smaller**, stock span, nearest boundary, histogram rectangles, or another situation where dominated candidates can be discarded permanently.

### Core invariant

The stack is maintained in increasing or decreasing order. Elements are popped when the current value proves they can no longer be useful. Each element is pushed and popped at most once, which is why many apparently nested solutions remain `O(n)`.

### Watch-outs

Decide whether you need values or indices. Indices are usually necessary when distance/width matters. Decide carefully whether equality should pop (`>=`) or remain (`>`); duplicates often determine correctness.

### Recognition test

If for every element you want the **nearest earlier/later element satisfying an inequality**, a monotonic stack should be one of your first candidates.

<aside>
⚠️

Do not judge your stack knowledge by specialized tricks such as 132 Pattern. Standard next-greater/next-smaller recognition is the foundation; some problems use a much less obvious stack invariant.

</aside>

# 9. Binary search and binary search on answer

## Ordinary binary search

Use it when a sorted/monotonic structure lets you discard half the remaining search space. Be comfortable with exact search and boundary forms such as first true / first occurrence.

## Binary search on answer

### When to suspect it

The wording asks for **minimum X such that...**, **maximum X that remains feasible**, **minimize the maximum**, or **maximize the minimum**. The answer itself may not be an array element.

### Core invariant

There is a monotonic feasibility predicate. For example, if capacity `X` works, every larger capacity also works. Binary search locates the boundary between false and true.

### Watch-outs

Before coding, explicitly say what `check(mid)` means and prove its monotonicity. Most bugs are boundary updates (`lo = mid + 1`, `hi = mid - 1`) and choosing whether to return `lo`, `hi`, or a stored answer.

### Recognition test

Ask: *Can I cheaply answer “is X feasible?”, and does feasibility change only once as X increases?* If yes, binary search on answer is likely.

# 10. Heap / priority queue

### When to suspect it

You repeatedly need the current smallest/largest candidate, **top K**, kth largest/smallest, merge K sorted structures, scheduling by priority, or minimum-cost repeated combination.

### Core invariant

The heap stores only candidates that still matter, and the root is the one you need next. For top K, you often keep only K elements so the heap never grows to `n`.

### Watch-outs

In Java, `PriorityQueue` is a min-heap by default. Be deliberate about comparator overflow: prefer `Integer.compare(a, b)` over `a - b` when values can be large. Know whether stale heap entries need to be discarded in problems where state changes over time.

### Recognition test

If your brute force repeatedly scans all remaining candidates just to find the current minimum/maximum, replace that repeated scan with a heap.

# 11. Linked list patterns

### Fast/slow pointers

Use when the problem asks about a cycle, midpoint, or a relationship between positions that can be exposed by pointers moving at different speeds.

### Reversal

Be able to write iterative reversal automatically: maintain `prev`, save `next`, reverse `curr.next`, then advance. Many harder linked-list questions are just reversal plus careful boundaries.

### Dummy node

Use a dummy head when insertions/deletions at the real head would otherwise require special cases.

### Watch-outs

Most linked-list mistakes are not conceptual; they are pointer-loss bugs. Before changing a `next` pointer, save whatever part of the list you still need.

### Recognition test

If the task is about **relative positions inside a singly linked structure**, first consider fast/slow, reversal, or a dummy node before inventing a complex data structure.

# 12. Intervals and sweep line

### When to suspect it

The input consists of intervals, meeting times, ranges, starts/ends, overlapping coverage or simultaneous events.

### Core patterns

- **Merge overlapping intervals:** sort by start, extend the current interval while overlap continues.
- **Maximum simultaneous intervals / rooms:** treat starts and ends as ordered events or use a heap of active end times.
- **Choose maximum non-overlapping intervals:** sort by end and greedily take the earliest finishing compatible interval.

### Watch-outs

Determine whether touching intervals count as overlapping. `[1,2]` and `[2,3]` may or may not overlap depending on the problem. Tie-breaking between start and end events matters.

### Recognition test

If each item has a start and end and the question concerns interaction between ranges, sorting the intervals should be almost automatic.

# 13. Greedy

### When to suspect it

The problem asks for a maximum/minimum and a local choice appears to permanently simplify the remaining problem: earliest finish, farthest extension, cheapest available choice, best ratio, etc.

### Core requirement

A greedy idea is not justified because it “looks best.” You need a reason that an optimal solution can be transformed to use the greedy choice without becoming worse — an exchange argument — or another monotonic proof.

### Watch-outs

Try to break your greedy with a small counterexample before committing. If future choices depend strongly on the exact earlier combination, DP may be needed instead.

### Recognition test

If sorting by some property makes the next choice locally obvious and the choice never needs to be revisited, greedy is plausible.

# 14. BFS / DFS / grids

## BFS

Use for **shortest path in an unweighted graph**, layer-by-layer processing, minimum number of edges/steps, and multi-source spread. For multi-source BFS, enqueue all starting sources before beginning the traversal.

## DFS

Use for connectivity, components, recursive exploration, subtree/post-order logic, and many cycle-detection tasks.

## Grid problems

A grid is a graph whose cells are nodes. “Islands”, flood fill, maze traversal and infection/spread problems usually reduce immediately to BFS/DFS plus direction arrays.

### Watch-outs

Mark nodes visited at the correct time. For BFS, usually mark when enqueuing, not when dequeuing, otherwise duplicates can flood the queue. In recursive DFS, consider recursion-depth limits on very large graphs.

### Recognition test

If entities are connected and the question asks **reachability, components, levels, shortest unweighted path, or propagation**, think graph traversal before anything more complicated.

# 15. Topological sort

### When to suspect it

The problem contains **prerequisites, dependencies, ordering constraints, courses/jobs that must precede others**, or asks whether all tasks can be completed.

### Core idea

A directed acyclic graph has a valid topological ordering. Kahn's algorithm tracks indegrees and repeatedly processes nodes whose indegree becomes zero. DFS can also produce a topological order using postorder while detecting cycles.

### Watch-outs

If Kahn's algorithm processes fewer than `n` nodes, a directed cycle exists. Be sure edge direction matches the meaning of “A is prerequisite for B”.

### Recognition test

If the question is essentially **“find an order that respects before/after relationships”**, topological sorting should be immediate.

# 16. Union-Find / DSU

### When to suspect it

Connectivity changes through repeated merges, you need to know whether two nodes already belong to the same component, or edges are processed while avoiding cycles. Kruskal's MST is the classic case.

### Core invariant

Every component has a representative parent. `find` returns the representative; `union` merges two representatives. Path compression and union-by-rank/size make operations effectively constant for interview-scale inputs.

### Watch-outs

Only union **roots**, not arbitrary nodes. In edge-processing problems, check whether the roots are already equal before merging.

### Recognition test

If you repeatedly ask **“are these two items already connected?”** while adding connections, DSU is a strong candidate.

# 17. Shortest paths and MST

### Dijkstra

Use for shortest paths with **non-negative weighted edges**. A min-priority queue always expands the best known distance candidate.

### Bellman–Ford

Use when negative edges may exist and single-source shortest paths are required; it can also detect reachable negative cycles.

### Floyd–Warshall

Use for all-pairs shortest paths when `n` is small enough for `O(n³)`.

### MST — Kruskal / Prim

Use when the requirement is to **connect all nodes with minimum total edge cost**, not to minimize the path from one source to each destination. Kruskal sorts edges and uses DSU; Prim grows one connected tree using a priority queue.

### Watch-outs

Do not confuse shortest-path trees with minimum spanning trees. They optimize different objectives.

# 18. Dynamic programming

DP is the pattern most likely to waste time if you search for a memorized recurrence. Instead, derive it from state.

### When to suspect it

The brute force explores many combinations and reaches the **same subproblem more than once**. Typical wording includes choose/skip, number of ways, minimum cost, maximum score, partitions, two strings, paths in a grid, or decisions whose future depends on a small state.

### The four questions

1. **What parameters fully describe the remaining problem?** Those become the DP state.
2. **What choices can I make from this state?** Those become transitions.
3. **What is the base case?** Usually the point where no decision remains.
4. **In what order can states be computed?** That determines tabulation direction.

### High-yield shapes

- **Choose/skip:** `take` vs `notTake`; knapsack/subset-like problems.
- **Linear state DP:** answer at `i` depends on a small number of earlier states; house-robber/non-adjacent style problems.
- **Grid DP:** `dp[r][c]` from allowed predecessor cells.
- **Two strings:** `dp[i][j]` for prefixes/substrings; LCS/edit-distance family.
- **State-machine DP:** holding/not holding, cooldown, transaction count, etc.

### Watch-outs

Do not call something DP merely because it has an array named `dp`. State meaning must be precise. For 1D knapsack, loop direction matters: backwards prevents reusing an item in 0/1 knapsack; forwards permits reuse in unbounded knapsack.

### Recognition test

Write the brute recursive function signature. If the same parameter combination can be reached through different paths, memoization is likely to help.

# 19. Backtracking

### When to suspect it

The problem asks to generate combinations/permutations/subsets, construct all valid configurations, or search a small exponential state space under constraints.

### Core structure

**Choose → recurse → undo.** The current partial solution is the state; pruning stops branches that can no longer become valid.

### Watch-outs

Check constraints before choosing backtracking. At `n = 10^5`, it is almost certainly impossible. Be careful with duplicate values; sorting plus “skip equal siblings” is a common duplicate-control technique.

### Recognition test

If the output itself may contain exponentially many valid configurations, an exponential traversal may be unavoidable and therefore acceptable.

# 20. Bit manipulation

### When to suspect it

The problem emphasizes binary representation, toggling flags, subsets for very small `n`, or the special property that values occur in pairs except for one/two exceptions.

### High-yield operations

- Check bit `i`: `(x & (1 << i)) != 0`
- Set bit `i`: `x | (1 << i)`
- Toggle bit `i`: `x ^ (1 << i)`
- Clear lowest set bit: `x & (x - 1)`
- Power of two for positive `x`: `(x & (x - 1)) == 0`
- XOR cancellation: `a ^ a = 0`, `a ^ 0 = a`

### Watch-outs

Java's signed integers and shift operators can matter. Use `1L << i` if the bit index may exceed the safe range for an `int`.

### Recognition test

If pairing/cancellation or subset-state representation is central, consider XOR/bitmasking before heavier structures.

# 21. Pattern combinations — where interviews get harder

Medium problems often combine two patterns rather than introducing a completely new one. Common combinations include:

- **Sort + two pointers** — sorting creates monotonic movement.
- **Prefix sum + hash map** — subarray conditions with negatives.
- **Sliding window + frequency map** — substring constraints.
- **Sorting + heap** — process events/queries while maintaining active candidates.
- **Graph traversal + state** — BFS where node alone is insufficient, such as `(node, stops)`.
- **Binary search + greedy/check function** — search the answer space while a linear feasibility test validates each candidate.
- **Kruskal = sorting + DSU.**
- **DP + binary search** — optimized LIS-style methods.

If no single pattern seems to fit, ask whether one technique **creates the structure needed by another**.

# 22. LSEG sprint priority

For this short sprint, do not give every section equal time. Use this order:

**Tier A — must be fast:** arrays/strings + hashing, two pointers/sliding window, stack, linked list, binary search, basic DP, BFS/DFS.

**Tier B — must be comfortable:** heap/PQ, topological sort, DSU/MST, intervals/greedy, bit manipulation.

**Tier C — recall, do not grind:** Bellman–Ford/Floyd–Warshall, advanced tree techniques, unusual monotonic-stack tricks, specialized DP families.

The objective is breadth of recognition plus clean reasoning, not proving that you can reproduce every advanced template from memory.

# 23. Green / Yellow / Red scoring

At the end of a pattern drill, classify each topic:

**Green:** within ~30 seconds you can name the likely technique, explain why it fits, state the target complexity, and outline the invariant. You do not need another dedicated revision block.

**Yellow:** you recognize it after some thought but hesitate on the invariant, edge cases, or implementation. Give it one short re-exposure or one representative problem.

**Red:** you fail to recognize the trigger or cannot reconstruct the basic implementation. This gets priority in the next weak-pattern repair block.

Do not turn a Green topic back into an hour of study just because there are harder variants. That is exactly how the preparation becomes unbounded.

# 24. How this connects to the timed LSEG sets

The common-pattern block and the timed coding block have different jobs.

**Pattern block:** recognition speed. It is allowed to look at this page after you commit to your initial guess. You should not spend 30 minutes solving one question.

**Timed LSEG block:** performance under uncertainty. No pattern sheet, no hints, no editorial. Read the problem, reason normally, choose an approach, code it, test it and move on. This is where you measure whether recognition transfers to an unseen problem.

After the timed set, record only:

- **Did I identify a viable approach within 10 minutes?**
- **Was the approach correct for the constraints?**
- **Did implementation bugs consume the time?**
- **Was there a better known pattern I failed to see?**
- **Could I explain brute force → bottleneck → improvement aloud?**

This distinction matters for you in particular: an unconventional correct solution is not a failure. The problem is only when deriving it consumes so much time that you cannot complete the round.

# 25. Final pre-interview recognition checklist

Before coding any DSA problem, force yourself through this compact sequence:

**Constraints → output shape → fixed/free order → brute force → bottleneck → trigger → target complexity → invariant → code.**

Then perform the pre-code safety check:

- What is each input and its size?
- Can any sum/product overflow `int`?
- What does an empty/single-element input do?
- What happens with duplicates/equality?
- Can I hand-trace the first example before running?

If you can make this sequence automatic, the notes have done their job.