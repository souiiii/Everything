# DSA Pattern Index

*A lookup table from **problem features** to **techniques**. Not organised by data structure — organised by the thing you notice while reading. The point is that reading a problem should fire a trigger before you start thinking about implementation.*

Problem solving guide:

1. **Reverse-read**
Output → input → constraints → story.
2. **Restate in 2 lines**
Convert the statement into the smallest correct model.
3. **Use `n` / constraints to estimate target complexity**
Decide what is even feasible before thinking too deeply.
4. **Write the brute-force idea**
Establish the obvious correct baseline first.
5. **Find the bottleneck + scan for triggers/patterns**
Ask what makes brute force slow and what known structure could remove that cost.
6. **State the final approach aloud before coding**
Force yourself to confirm the algorithm before implementation.

**How to use this**

- Read a problem → follow the 6-step guide → identify the trigger → use the table to validate the likely technique. Don't reason from scratch.
- Practise the reverse drill: read a problem, say only the trigger and the technique, no code. Ten problems in fifteen minutes.
- Add a line whenever you meet a trigger that isn't here. This file should keep growing after placements.

---

## §0 · Read the constraints first — they eliminate most of the table

Before you think about the problem at all, the size tells you what's permitted:

| N up to | Permitted complexity | Techniques still on the table |
| --- | --- | --- |
| ~20 | O(2ⁿ), O(n!) | Bitmask DP, backtracking, brute force over subsets |
| ~100 | O(n³), O(n⁴) | Floyd–Warshall, interval DP, matrix chain |
| ~2000 | O(n²) | 2D DP, LCS-family, all-pairs comparisons |
| ~10⁵ | O(n log n) | Sorting, binary search, heap, two pointers, sweep line |
| ~10⁶–10⁷ | O(n) | Single pass, hashing, prefix sums, counting |
| ~10⁹+ | O(log n), O(1) | Binary search on answer, maths, closed form — *the input cannot be iterated* |

**A value far larger than the number of operations is a deliberate signal.** Line length 10⁹ with 10⁵ queries means "you can't build the line — track only what's touched." Never ignore that gap.

---

## §1 · Arrays & strings

| Trigger | Technique |
| --- | --- |
| Contiguous subarray, **all values positive** | Two pointers / sliding window |
| Subarray sum with **negatives present** | Prefix sum + hash map (window breaks) |
| Fixed-size window | Sliding window, add right / remove left |
| Longest / shortest window satisfying a condition | Variable sliding window |
| **Range updates**, many of them, query at the end | Difference array, then prefix-sum once |
| Many range-sum queries, no updates | Prefix sum array |
| Count elements covering / covered by ranges | Sort + sweep, or counting array + suffix sum |
| Sorted array, find a pair/triple | Two pointers from both ends |
| Unsorted, find pairs summing to K | Hash set (one pass) |
| "Kth largest / smallest" | Heap of size K, or quickselect |
| Top K frequent | Hash map + heap, or bucket by frequency |
| Find duplicate / missing, values in **1..n** | Index-as-hash (mark by negation or swap) |
| Majority element (> n/2) | Boyer–Moore voting |
| Merge overlapping intervals | Sort by start, sweep |
| Many point queries over intervals + need best/smallest covering interval | Sort queries + intervals → sweep → min-heap of active intervals based on length/suitability |
| Max overlapping intervals at any time | Sort start/end events, sweep with a counter |
| Product / sum "except self" | Prefix and suffix arrays |
| Rotate, reverse in place | Reversal trick (reverse parts, then whole) |
| Next greater / smaller element | **Monotonic stack** |
| Trapping water, largest rectangle in histogram | Monotonic stack, or two pointers with running max |
| Pair score = contribution from left + contribution from right, with distance/index involved | Rearrange the formula → keep the best contribution from one side while scanning |
| Longest / shortest contiguous window where condition depends on min/max | Sliding window + monotonic min deque + monotonic max deque |

---

## §2 · Stacks & queues

| Trigger | Technique |
| --- | --- |
| **Adjacent elements cancel, gap closes, cascades** | Stack |
| Balanced brackets, valid sequences | Stack |
| Undo / most-recent-first semantics | Stack |
| Next greater / next smaller / span | Monotonic stack |
| Sliding window maximum or minimum | Monotonic deque |
| Expression evaluation, postfix, nested decoding | Stack |
| Process in arrival order, BFS layers | Queue |
| Max distance + inequality | monotonic stack may store only useful candidates. discard later endpoints if an earlier one is already better. |

*The stack tell: whatever you just processed can be undone by what comes next.*

---

## §3 · Binary search

| Trigger | Technique |
| --- | --- |
| Sorted array, find a value or boundary | Plain binary search (`lower_bound` / `upper_bound` shapes) |
| **"Minimum X such that a condition holds"** | **Binary search on the answer** |
| "Maximum X such that still feasible" | Binary search on the answer |
| Minimise the maximum / maximise the minimum | Binary search on the answer + greedy feasibility check |
| Rotated sorted array | Binary search with a sorted-half test |
| Answer space is huge but checking is cheap | Binary search on the answer |
| Split array into K parts minimising the largest sum | Binary search on the answer |

*The signature: the answer is monotonic — if X works, everything above X works. Then you're searching the answer space, not the array.*

---

## §4 · Dynamic programming

**First: what changes between subproblems? Those parameters are your state.**

| Trigger | Technique |
| --- | --- |
| **"Choose or skip" each item, running total** | 0/1 knapsack — `dp[i][capacity]`, loop capacity **backwards** in 1D |
| Same item usable unlimited times | Unbounded knapsack — loop capacity **forwards** |
| Count ways vs minimise cost | Same recurrence, `+` instead of `min` |
| Split array into two equal halves | Subset sum with target = total/2 |
| **Answer needed at every prefix** | **Build forward, not backward** — each cell is an answer |
| Two strings compared | 2D DP `dp[i][j]` — LCS, edit distance, matching |
| One string, palindromic substructure | 2D DP filled by increasing length, diagonal base case |
| Best move at each step, states carry forward | Linear DP with a small state set (e.g. holding / not holding) |
| Buy/sell with cooldown or transaction limits | State machine DP |
| Longest increasing subsequence | O(n²) DP, or patience sorting + binary search for O(n log n) |
| Grid, moves restricted to right/down | 2D DP over the grid |
| Partition into groups, small N | Bitmask DP (only when N ≤ ~20) |
| Intervals combined pairwise, N ≤ ~500 | Interval DP `dp[i][j]` |

**Memo → tabulation conversion:** state from the parameters, sizes from their ranges, base cases become initialised rows, **loop direction opposite to the recursion's direction**, recursive calls become lookups, and *the loop must skip cells you initialised*.

---

## §5 · Greedy

| Trigger | Technique |
| --- | --- |
| Maximise count of non-overlapping items | Sort by **end** time, take greedily |
| Minimise rooms / platforms / resources | Sort events, sweep with a counter |
| **Ordering matters, and swapping two adjacent items shows a rule** | Exchange argument — sort by that rule |
| "Minimum starting resource to complete all tasks" | Sort by (requirement − consumption) descending |
| Always take the current best / worst | Heap |
| Cheapest K of N things | Sort, take K — *check whether the structure actually constrains you* |
| Fractional selection allowed | Greedy by ratio |
| Minimum number of intervals needed to cover `[0, T]` | From everything available so far, keep extending coverage as far as possible |

*If greedy feels plausible, try to break it with a counterexample before committing. If you can't, it's probably right — and the exchange argument is the proof.*

---

## §6 · Graphs

| Trigger | Technique |
| --- | --- |
| Shortest path, **unweighted** | BFS |
| Shortest path, weighted, non-negative | Dijkstra |
| Shortest path with negative edges | Bellman–Ford |
| All-pairs shortest paths, N ≤ ~400 | Floyd–Warshall |
| Connectivity, "how many groups" | DFS/BFS flood fill, or **union-find** |
| Dynamic merging of groups | Union-find with path compression |
| Cheapest set of edges connecting everything | MST — Kruskal (sort + union-find) or Prim |
| Ordering with prerequisites | Topological sort (Kahn's BFS or DFS) |
| Cycle detection, directed | DFS with a recursion-stack colour marking |
| Cycle detection, undirected | Union-find, or DFS tracking the parent |
| Grid problems — islands, rot spreading, maze | BFS/DFS treating cells as nodes |
| Multiple simultaneous starting points | **Multi-source BFS** (push all sources first) |
| Bipartite / two-colouring | BFS colouring |
| **Tree, delete edges to make K components** | Any K−1 edges split a tree into K parts → sort weights |

*A "grid" is a graph. A "tree" is a graph with N−1 edges and no cycles — its properties usually simplify the problem drastically.*

---

## §7 · Trees

| Trigger | Technique |
| --- | --- |
| Anything about subtrees | Post-order DFS — children before parent |
| Anything level-by-level | BFS with a queue |
| BST + "sorted" or "kth" | In-order traversal is sorted |
| Lowest common ancestor | Recursive LCA, or binary lifting if repeated |
| Path sums through nodes | DFS returning the best downward path, updating a global |
| Validate / search a BST | Use the ordering invariant, carry min/max bounds |
| Serialise / rebuild a tree | Pre-order with null markers |

---

## §8 · Heaps

| Trigger | Technique |
| --- | --- |
| Kth largest / smallest | Min-heap of size K (or max-heap of size K) |
| Merge K sorted lists | Min-heap of heads |
| Running median | Two heaps, max-heap and min-heap balanced |
| Schedule by priority, repeatedly take the best | Priority queue |
| "Minimum cost to combine all items" | Min-heap, repeatedly combine the two smallest |

---

## §9 · Hashing & counting

| Trigger | Technique |
| --- | --- |
| "Has this been seen before" | Hash set |
| Frequency of anything | Hash map, or an array when the alphabet is small |
| Anagram / permutation matching | Character counts (26-length array) |
| Group things by a computed key | Hash map keyed by that signature |
| Subarray summing to K, negatives allowed | Prefix sum + hash map of counts |
| Sparse data over a huge coordinate range | Hash map instead of an array |

---

## §10 · Bit manipulation

| Trigger | Technique |
| --- | --- |
| Everything appears twice except one | XOR everything |
| Enumerate all subsets, N ≤ ~20 | Bitmask loop `0 .. 2ⁿ−1` |
| Toggle / set / check a specific bit | `x ^ (1<<i)`, `x | (1<<i)`, `x & (1<<i)` |
| Count set bits | `Integer.bitCount`, or Brian Kernighan's `x & (x-1)` |
| Powers of two, alignment checks | `x & (x-1) == 0` |

---

## §11 · Maths

| Trigger | Technique |
| --- | --- |
| Count coprime numbers below N | Euler's totient φ |
| GCD / LCM relationships | Euclid; `lcm = a*b/gcd` |
| Primes up to N | Sieve of Eratosthenes |
| Factorise a single number up to ~10¹² | Trial division to √n |
| Large exponents with a modulus | Fast modular exponentiation |
| Combinatorics with a modulus | Precompute factorials + modular inverse |

---

## §12 · The five features to extract from every problem

Before choosing anything, note these — most problems are decided by them alone:

1. **Constraint size** → what complexity is permitted (§0)
2. **Output shape** — one value, one per query, or an array. *Per-query output almost always means maintain incrementally, never recompute.*
3. **Is order fixed or free?** Fixed → subarray/substring/DP. Free → sorting or greedy is probably in play.
4. **What actually varies between subproblems?** Those are your DP state parameters.
5. **Restate the rule in your own words.** "Delete AB or BB" → "a B eats its left neighbour" → stack. This step alone converts more problems than any technique in this file.

---

## §13 · Pre-code checklist

Sixty seconds, before writing anything:

- Write down **what each input is**, with its size. Two different arrays means two different sizes — never assume they match.
- State the **output shape and type**. `long` or `int`?
- Ask **"how big can this get?"** on every accumulator. Sum of 10⁵ values of 10⁹ overflows `int`.
- Hand-trace **example 1** and predict the output *before* running.

*Every mistake worth naming this week died at one of those four lines.*