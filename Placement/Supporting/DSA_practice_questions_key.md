# Answer Key

Same numbering as `practice_questions.md`. Check one entry only after attempting it.

## 1. Course Schedule III

Primary pattern: Greedy by deadline + max-heap regret replacement

Also touches: Sorting, Heap

Trigger:
Maximise the number of items finishable before individual deadlines.

Why:
Sort by deadline, take everything, and when infeasible drop the longest course taken so far.

Possible alternatives:
DP over (index, elapsed time).

---

## 2. Shortest Subarray with Sum at Least K

Primary pattern: Monotonic deque over prefix sums

Also touches: Prefix sum

Trigger:
Shortest subarray with sum >= K, but values may be negative so the window is not monotone.

Why:
Keep prefix sums in an increasing deque; pop from the front when P[i]-P[front] >= k, pop from the back to keep it increasing.

Possible alternatives:
Naive sliding window (wrong with negatives); balanced BST over prefix sums.

---

## 3. Combination Sum IV

Primary pattern: Order-sensitive unbounded counting DP (permutations)

Also touches: Knapsack loop order

Trigger:
Counts SEQUENCES, not multisets, so the same numbers in another order count twice.

Why:
dp[t] = sum over nums of dp[t-num], with the target loop outside and the item loop inside.

Possible alternatives:
Recursion with memoisation on the remaining target.

---

## 4. Magnetic Force Between Two Balls

Primary pattern: Binary search on the answer + greedy feasibility

Also touches: Sorting

Trigger:
Maximise the minimum gap - a maximin objective with a monotone feasibility predicate.

Why:
Sort, binary search the gap d, and greedily place balls at least d apart to test whether m fit.

Possible alternatives:
Parametric search; DP over placements (too slow).

---

## 5. Russian Doll Envelopes

Primary pattern: Longest increasing subsequence after a sorting trick

Also touches: Patience sorting, Sorting

Trigger:
Two-dimensional nesting with strict inequality in both dimensions.

Why:
Sort by width ascending and height DESCENDING on ties, then run the O(n log n) LIS on heights.

Possible alternatives:
O(n^2) DP over pairs; segment tree over coordinates.

---

## 6. Minimum Interval to Include Each Query

Primary pattern: Offline queries + sort + min-heap of active intervals

Also touches: Sweep line

Trigger:
Per-query output asking for the SMALLEST interval covering each query point.

Why:
Sort queries and intervals by coordinate; push intervals whose start has passed, lazily pop those that ended, and read the heap's smallest length.

Possible alternatives:
Segment tree over compressed coordinates; merge-sort-tree.

---

## 7. Min Cost to Connect All Points

Primary pattern: Minimum spanning tree on a complete graph (Prim)

Also touches: Kruskal, Union-find

Trigger:
Connect all nodes at minimum total cost with a dense implicit edge set.

Why:
Prim's algorithm on the implicit complete graph in O(n^2), or Kruskal over all pairs.

Possible alternatives:
Kruskal with sorted edges and union-find.

---

## 8. 132 Pattern

Primary pattern: Monotonic decreasing stack scanned right to left

Also touches: Prefix minimum

Trigger:
Find i<j<k with a[i] < a[k] < a[j] - an ordering constraint across three separated indices.

Why:
Sweep from the right maintaining a decreasing stack; popped values become the best possible 'a[k]' candidate.

Possible alternatives:
Prefix minimum for a[i] plus an ordered set for a[k]; O(n^2) over j.

---

## 9. Car Fleet

Primary pattern: Sort by position, monotonic stack of arrival times

Also touches: Greedy

Trigger:
Cars merge and never split, so a slower car ahead absorbs everything catching up to it.

Why:
Sort by descending position; a fleet forms whenever the current arrival time exceeds the stack top's.

Possible alternatives:
Scan from the front tracking a running maximum arrival time.

---

## 10. Maximum Number of K-Divisible Components

Primary pattern: Post-order DFS on a tree, cutting edges when a subtree sum is divisible

Also touches: Greedy, Tree DP

Trigger:
Split a tree into the maximum number of parts whose value sums are each divisible by k.

Why:
Post-order accumulate subtree sums mod k; whenever a subtree's residue is zero, cut that edge and count a component.

Possible alternatives:
Iterative peeling of leaves with accumulated residues.

---

## 11. Possible Bipartition

Primary pattern: Two-colouring by BFS/DFS

Also touches: Union-find

Trigger:
Mutual-dislike constraints are exactly 'these two must be on different sides'.

Why:
BFS colouring each component; a conflict on an existing colour proves impossibility.

Possible alternatives:
Union-find with parity, or a union-find of size 2n.

---

## 12. Repeated DNA Sequences

Primary pattern: Rolling hash / fixed-window signature over a small alphabet

Also touches: Hash set, Bit encoding

Trigger:
Find repeated fixed-length substrings over a 4-letter alphabet in a long string.

Why:
Encode each window as a 2-bits-per-character integer and roll it forward, storing seen codes in a set.

Possible alternatives:
Store the raw substrings in a hash set (simpler, more memory).

---

## 13. Shifting Letters II

Primary pattern: Difference array, prefix sum once at the end

Also touches: Modular arithmetic

Trigger:
Many range updates, and the answer is only read after all of them are applied.

Why:
Add +/-1 at the range boundaries, prefix-sum once, then apply the shift modulo 26.

Possible alternatives:
Segment tree with lazy propagation (correct but far heavier).

---

## 14. Single Number III

Primary pattern: XOR everything + isolate a differing bit

Also touches: Bit masking

Trigger:
Everything appears twice except TWO numbers, so a single XOR leaves their combination.

Why:
XOR all to get a^b, take the lowest set bit, and partition the array on it to XOR each group separately.

Possible alternatives:
Hash-map counting (O(n) extra space).

---

## 15. Remove All Adjacent Duplicates in String II

Primary pattern: Stack of (character, run length)

Also touches: Counting

Trigger:
Removing a run of k identical characters closes the gap and can trigger further removals.

Why:
Push (char, count) pairs; increment on a match and pop when the count reaches k.

Possible alternatives:
Repeated scanning passes until the string stabilises (O(n^2)).

---

## 16. Smallest Range Covering Elements from K Lists

Primary pattern: Min-heap of list heads + running maximum

Also touches: K-way merge, Sliding window

Trigger:
The range must contain at least one element from each of k sorted lists.

Why:
Keep one pointer per list in a min-heap alongside the current maximum; advance the minimum and track the best span.

Possible alternatives:
Merge everything into one labelled sorted array and run a variable window over labels.

---

## 17. Replace Non-Coprime Numbers in Array

Primary pattern: Stack with cascading merges + GCD/LCM

Also touches: Number theory

Trigger:
Adjacent elements merge when they share a factor, and the merged value can trigger further merges leftward.

Why:
Push onto a stack; while gcd(top, cur) > 1, pop and replace cur with lcm(top, cur).

Possible alternatives:
Repeated left-to-right passes until stable (too slow).

---

## 18. Single-Threaded CPU

Primary pattern: Event-ordered simulation + min-heap by priority

Also touches: Sorting

Trigger:
Repeatedly take the best available item, where availability changes with time.

Why:
Sort by arrival, push everything available into a heap keyed by (duration, index), and jump time forward when idle.

Possible alternatives:
Two-pointer over sorted arrivals with the same heap.

---

## 19. Find K Pairs with Smallest Sums

Primary pattern: Min-heap over a frontier of candidate pairs

Also touches: K-way merge

Trigger:
Smallest k sums from an implicit sorted matrix of pairs that must not be materialised.

Why:
Seed the heap with the first row's pairs and, on each pop, push only the next candidate in that row.

Possible alternatives:
Binary search on the sum value with a counting predicate.

---

## 20. Minimum Initial Energy to Finish Tasks

Primary pattern: Exchange argument - sort by (minimum - actual) descending

Also touches: Greedy, Binary search

Trigger:
Each task has a consumption and a higher threshold, and only the ORDER is free.

Why:
Sort by (minimum - actual) descending and accumulate; swapping two adjacent tasks proves the rule.

Possible alternatives:
Binary search the starting energy with a greedy feasibility check.

---

## 21. Distribute Coins in Binary Tree

Primary pattern: Post-order DFS returning a signed surplus

Also touches: Greedy

Trigger:
Coins move along edges, and each edge's traffic is decided entirely by its subtree.

Why:
Return (subtree coins - subtree nodes) upward and add |flow| on every edge to the answer.

Possible alternatives:
Same recursion phrased as a flow-conservation argument.

---

## 22. Maximum Width Ramp

Primary pattern: Monotonic stack of candidate left endpoints, scanned from the right

Also touches: Sorting by value

Trigger:
Maximise j-i subject to a[i] <= a[j] - a distance objective under an inequality.

Why:
Push a strictly decreasing stack of candidate i's, then sweep j from the right popping while a[j] >= stack top.

Possible alternatives:
Sort indices by value and track the minimum index seen; binary search over a decreasing prefix-min array.

---

## 23. Minimum Cost For Tickets

Primary pattern: Forward DP where every prefix is an answer

Also touches: Interval jumps, Binary search

Trigger:
Choices at a day affect a forward span of days, and the cost must be known at each prefix.

Why:
dp[d] = min over pass types of dp[max(0, d-span)] + cost, iterating days forward.

Possible alternatives:
DP over travel-day indices with binary search for the next uncovered day.

---

## 24. Minimum Number of Days to Make m Bouquets

Primary pattern: Binary search on the answer + linear greedy check

Also touches: Greedy

Trigger:
'Minimum day such that it becomes possible' - feasibility is monotone in the day.

Why:
Binary search the day, then scan counting maximal runs of bloomed flowers to test whether m bouquets fit.

Possible alternatives:
Sort days and add flowers incrementally with union-find over adjacent runs.

---

## 25. Find the Longest Substring Containing Vowels in Even Counts

Primary pattern: Prefix XOR bitmask + first-occurrence hash map

Also touches: Parity state

Trigger:
Only the PARITY of five counts matters, so the whole state fits in 5 bits.

Why:
Track a 5-bit parity mask; equal masks at i and j mean every vowel appears an even number of times in between.

Possible alternatives:
Prefix counts per vowel with pairwise comparison (O(n^2)).

---

## 26. Jump Game VI

Primary pattern: DP + monotonic deque for a windowed max

Also touches: Sliding window maximum

Trigger:
dp[i] may only jump from the previous k indices, so each transition needs a windowed maximum.

Why:
dp[i] = a[i] + max(dp[i-k..i-1]); a decreasing deque yields that maximum in O(1) amortised.

Possible alternatives:
Max-heap with lazy deletion; segment tree over dp values.

---

## 27. Maximum Number of Removable Characters

Primary pattern: Binary search on the answer + subsequence check

Also touches: Two pointers

Trigger:
'Largest k such that it still works' with a cheap O(n) validity test.

Why:
Feasibility is monotone in k; mark the first k removals and verify p is still a subsequence of s.

Possible alternatives:
Incremental union-find over restorations processed backwards.

---

## 28. Cheapest Flights Within K Stops

Primary pattern: Bellman-Ford with a bounded number of relaxation rounds

Also touches: Dijkstra with state, BFS layers

Trigger:
Cheapest path subject to a hard cap on the number of edges used.

Why:
Relax all edges k+1 times over a snapshot of the previous round's distances.

Possible alternatives:
Dijkstra over (node, stops) states; layered BFS with a cost array.

---

## 29. Maximum Profit in Job Scheduling

Primary pattern: Weighted interval scheduling DP + binary search

Also touches: Sorting, Greedy (fails)

Trigger:
Non-overlapping intervals with WEIGHTS, so the count-based greedy no longer applies.

Why:
Sort by end time; dp[i] = max(skip, profit + dp[last job ending <= start]) with the predecessor found by binary search.

Possible alternatives:
DP over a heap of running best profits; segment tree over coordinates.

---

## 30. Video Stitching

Primary pattern: Greedy interval covering of [0, T]

Also touches: Jump-game DP, Sorting

Trigger:
Cover an entire time range using the fewest given intervals.

Why:
Track the farthest reach for each left endpoint and extend coverage greedily, counting each extension.

Possible alternatives:
DP over positions; BFS treating reachable ranges as layers.

---

## 31. Swim in Rising Water

Primary pattern: Minimax path - Dijkstra, or binary search + BFS, or Kruskal union-find

Also touches: Union-find, Binary search

Trigger:
Minimise the maximum cell value along a path in a grid.

Why:
Dijkstra where a path's cost is the running maximum; equivalently, add cells in increasing value until start and end connect.

Possible alternatives:
Binary search the threshold with a reachability BFS.

---

## 32. Shortest Bridge

Primary pattern: Flood fill to label one component, then multi-source BFS

Also touches: DFS, Grid

Trigger:
Two components, and you need the shortest gap between them.

Why:
DFS/BFS to collect every cell of the first island, push them all as BFS sources, and expand until the second island is hit.

Possible alternatives:
BFS from every cell of one island independently.

---

## 33. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit

Primary pattern: Sliding window + monotonic max deque + monotonic min deque

Also touches: Multiset / two heaps

Trigger:
Longest window whose validity depends on both the window max and the window min.

Why:
Maintain a decreasing deque for max and an increasing deque for min; shrink while max-min > limit.

Possible alternatives:
Ordered multiset (TreeMap) of the window; two heaps with lazy deletion.

---

## 34. Exclusive Time of Functions

Primary pattern: Stack of active frames with a running timestamp

Also touches: Simulation

Trigger:
Nested calls where a parent's own time excludes the time spent inside its children.

Why:
Push on start, and on end pop while crediting elapsed time to the top frame and charging it against its parent.

Possible alternatives:
Recursive parsing of the log into a call tree.

---

## 35. Serialize and Deserialize BST

Primary pattern: Pre-order serialisation exploiting the BST bound invariant

Also touches: Parsing, Recursion

Trigger:
Round-tripping a tree, with the BST property available to remove null markers.

Why:
Emit pre-order values; rebuild recursively using (min, max) bounds to decide where each value belongs.

Possible alternatives:
Pre-order with explicit null markers; store both pre-order and in-order.

---

## 36. Two City Scheduling

Primary pattern: Exchange argument - sort by the pairwise difference

Also touches: Greedy, DP

Trigger:
Exactly half must go each way, and swapping two people exposes the ordering rule.

Why:
Sort by costA - costB and send the first half to A.

Possible alternatives:
Min-cost flow / assignment DP over (index, count sent to A).

---

## 37. Special Array With X Elements Greater Than or Equal X

Primary pattern: Counting / binary search on a self-referential predicate

Also touches: Sorting

Trigger:
x must equal the number of elements >= x - the answer is defined by a count of itself.

Why:
Sort (or bucket-count) and test each candidate x from 1..n; the predicate is monotone.

Possible alternatives:
Counting-sort with a suffix sum over value buckets.

---

## 38. Longest Palindromic Subsequence

Primary pattern: 2D DP over intervals, filled by increasing length

Also touches: LCS with the reverse

Trigger:
One string with palindromic substructure, so the state is a substring and the base case is the diagonal.

Why:
dp[i][j] = dp[i+1][j-1] + 2 on a character match, else max(dp[i+1][j], dp[i][j-1]).

Possible alternatives:
LCS of s with its reverse.

---

## 39. Simplified Fractions

Primary pattern: Coprimality via GCD over an enumerated pair space

Also touches: Number theory, Brute force with a divisibility filter

Trigger:
Every fraction must be in lowest terms, i.e. numerator and denominator coprime, with n small enough to enumerate all pairs.

Why:
Enumerate denominators and numerators and keep the pairs with gcd == 1; the count of kept pairs per denominator is exactly phi(denominator).

Possible alternatives:
Sieve-style totient precomputation when only the COUNT is needed rather than the list.

---

## 40. Count Triplets That Can Form Two Arrays of Equal XOR

Primary pattern: Prefix XOR + the observation that the split point is free

Also touches: Prefix sum analogue

Trigger:
Two adjacent segments must have equal XOR, and the split between them can sit anywhere.

Why:
Equal XOR means prefix[i] == prefix[k+1]; every j in between works, so count pairs and weight by distance.

Possible alternatives:
O(n^2) over (i, k) with a running XOR.

---

## 41. Bitwise AND of Numbers Range

Primary pattern: Common binary prefix of the range endpoints

Also touches: Bit manipulation

Trigger:
The range can span up to 2^31 values, so the numbers cannot be iterated at all.

Why:
The AND over a range equals the shared high-order prefix of left and right; shift both right until they meet.

Possible alternatives:
Brian Kernighan: repeatedly clear the lowest set bit of right while it exceeds left.

---

## 42. Smallest String With Swaps

Primary pattern: Union-find components + sort within each component

Also touches: DFS, Sorting

Trigger:
Swaps are given as pairs but can be composed arbitrarily, so each component is freely permutable.

Why:
Union the swap pairs, then sort each component's characters into its sorted index positions.

Possible alternatives:
DFS to collect components; equivalent connected-component labelling.

---

## 43. Total Hamming Distance

Primary pattern: Per-bit column counting

Also touches: Combinatorics

Trigger:
Sums a pairwise statistic over all pairs, but bits are independent of one another.

Why:
For each of the 32 bit positions, the contribution is (count of ones) * (count of zeros).

Possible alternatives:
O(n^2) pairwise XOR with a popcount.

---

## 44. Sum of Subarray Minimums

Primary pattern: Monotonic stack for contribution counting

Also touches: Previous/next smaller element

Trigger:
Sums a statistic over ALL subarrays, so each element's total contribution must be counted directly.

Why:
For each element find the previous-smaller and next-smaller spans; it is the minimum of left*right subarrays.

Possible alternatives:
O(n^2) enumeration; divide and conquer on the minimum.

---

## 45. Most Stones Removed with Same Row or Column

Primary pattern: Union-find over shared rows and columns

Also touches: DFS components

Trigger:
Removability cascades, and the answer is n minus the number of connected components.

Why:
Union each stone with its row key and column key; count components over the used keys.

Possible alternatives:
DFS/BFS on a graph built from shared coordinates.

---

## 46. Queue Reconstruction by Height

Primary pattern: Greedy insertion after sorting by height

Also touches: Sorting

Trigger:
Each person's constraint counts only people at least as tall, so taller people are placed first.

Why:
Sort by height descending and count ascending, then insert each person at index k.

Possible alternatives:
Binary indexed tree over free slots for an O(n log n) placement.

---

## 47. Step-By-Step Directions From a Binary Tree Node to Another

Primary pattern: Lowest common ancestor + root-to-node paths

Also touches: DFS

Trigger:
A path between two nodes in a rooted tree always goes up to their LCA and back down.

Why:
Find the root paths for both nodes, strip the common prefix, replace the first leg with 'U's.

Possible alternatives:
Locate the LCA first, then search downwards for each target.

---

## 48. Open the Lock

Primary pattern: BFS over an implicit state graph

Also touches: Hash set, Bidirectional BFS

Trigger:
Minimum number of moves with uniform cost over states, not over an explicit graph.

Why:
Treat each 4-digit combination as a node with 8 neighbours; BFS from '0000' avoiding deadends.

Possible alternatives:
Bidirectional BFS; A* with a digit-distance heuristic.

---

## 49. Minimum Height Trees

Primary pattern: Topological peeling of leaves inward

Also touches: BFS, Tree centroids

Trigger:
Asks for the best ROOTS of a tree, which are its one or two centres.

Why:
Repeatedly strip all current leaves; the last one or two nodes remaining are the answers.

Possible alternatives:
Two BFS passes to find the diameter, then take its midpoint(s).

---

## 50. Minimum Cost to Hire K Workers

Primary pattern: Sort by ratio + max-heap of quality

Also touches: Greedy, Heap

Trigger:
Pay is proportional within a group, so the group's cost is driven by its worst ratio.

Why:
Sort by wage/quality; sweep, keeping the k smallest qualities in a max-heap, and score at each ratio.

Possible alternatives:
Enumerate every worker as the ratio setter with a selection over the rest.

---

## 51. Path Sum III

Primary pattern: Prefix sum + hash map, applied on a root-to-node path

Also touches: DFS backtracking

Trigger:
Counts downward paths summing to a target on a tree, with negative values allowed.

Why:
Carry a running root-to-node prefix sum in a map; on entering a node add curr-target, and decrement the map on exit.

Possible alternatives:
O(n^2) DFS from every node.

---

## 52. Number of Ways to Reorder Array to Get Same BST

Primary pattern: Divide and conquer + binomial coefficients modulo a prime

Also touches: Combinatorics, Recursion

Trigger:
Counts interleavings of two independent sequences that preserve a fixed structure.

Why:
Recurse on the left and right subtrees and multiply by C(l+r, l), using precomputed factorials and modular inverses.

Possible alternatives:
Memoised recursion over sorted index sets with Pascal's triangle.

---

## 53. Uncrossed Lines

Primary pattern: Longest common subsequence in disguise

Also touches: 2D string DP

Trigger:
Connections may not cross, which forces the chosen pairs into increasing index order in both arrays.

Why:
Non-crossing + increasing in both sequences is precisely LCS; dp[i][j] over the two arrays.

Possible alternatives:
LIS on matched index pairs.

---

## 54. Amount of Time for Binary Tree to Be Infected

Primary pattern: Convert the tree to an undirected graph, then BFS from a source

Also touches: Multi-source BFS, DFS

Trigger:
Spreading reaches parents as well as children, so the tree must be traversed as a graph.

Why:
Build a parent map (or adjacency list) and BFS from the start node; the answer is the last layer index.

Possible alternatives:
Single DFS returning depth and a burn time simultaneously.

---

## 55. Minimum Cost Tree From Leaf Values

Primary pattern: Monotonic increasing stack (greedy removal of local minima)

Also touches: Interval DP

Trigger:
Build a tree over an array whose cost involves the maximum of each side - looks like classic interval DP.

Why:
Repeatedly remove the smallest element, paying it times its smaller neighbour; a monotonic stack does this in O(n).

Possible alternatives:
O(n^3) interval DP dp[i][j] with a max table - correct and much easier to see.

---

## 56. Min Cost to Connect Ropes

Primary pattern: Min-heap, repeatedly combine the two smallest

Also touches: Greedy, Huffman-style merging

Trigger:
'Minimum total cost to combine all items', where every merge cost equals the combined size.

Why:
Always merge the two smallest current items; a min-heap supplies them in O(log n).

Possible alternatives:
Sorting once is NOT sufficient, because merged values re-enter the pool.

---

## 57. Partition to K Equal Sum Subsets

Primary pattern: Backtracking with pruning, or DP over a subset bitmask

Also touches: Bitmask DP, Subset sum

Trigger:
Partition into k equal-sum groups with n <= 16 - the constraint size openly permits exponential search.

Why:
Sort descending, fill one bucket at a time with pruning; or dp over masks tracking the running remainder.

Possible alternatives:
Memoised search over (mask, current bucket sum).

---

## 58. Minimum Operations to Reduce X to Zero

Primary pattern: Sliding window on the complement

Also touches: Prefix sum

Trigger:
Removals are only from the two ends, which is not a contiguous window - but what stays behind is.

Why:
Minimise the removed prefix+suffix by maximising the middle window summing to total-x.

Possible alternatives:
Two prefix-sum hash maps from each end; DP looks plausible but is unnecessary.

---

## 59. 4Sum II

Primary pattern: Meet in the middle with a hash map of pair sums

Also touches: Hash map counting

Trigger:
Four independent arrays, so an O(n^4) enumeration splits cleanly into two halves.

Why:
Count all sums of the first pair in a map, then look up the negation of each second-pair sum.

Possible alternatives:
Sort two pair-sum arrays and two-pointer across them.

---

## 60. Count Nice Pairs in an Array

Primary pattern: Group by a rearranged key + hash map counting

Also touches: Combinatorics

Trigger:
The pair condition a[i]+rev(a[j]) == a[j]+rev(a[i]) is symmetric, so isolate each index.

Why:
Rearrange to a[i]-rev(a[i]) == a[j]-rev(a[j]); count equal keys and add c*(c-1)/2.

Possible alternatives:
Brute force over all pairs.

---

## 61. Make Sum Divisible by P

Primary pattern: Prefix sum modulo + hash map

Also touches: Sliding window (fails)

Trigger:
Remove the SHORTEST subarray so the remainder becomes divisible by p - a residue condition, not a sum condition.

Why:
Need prefix[j]-prefix[i] = target (mod p); store the last index of each residue and look up (cur-target) mod p.

Possible alternatives:
Brute force over all subarrays; two pointers, which fails because values do not bound the residue.

---

## 62. Maximum XOR of Two Numbers in an Array

Primary pattern: Greedy bit-by-bit construction with a hash set (or a binary trie)

Also touches: Trie

Trigger:
Maximise a pairwise XOR without enumerating pairs.

Why:
Build the answer from the top bit down, testing each candidate prefix against the set of masked prefixes.

Possible alternatives:
Insert all numbers into a binary trie and walk the opposite branch greedily.

---

## 63. Frequency of the Most Frequent Element

Primary pattern: Sort + sliding window on a cost budget

Also touches: Prefix sum, Binary search on answer

Trigger:
Budget k of increment operations, order is free, and the target must be an existing value.

Why:
Sort, then slide a window where cost = a[r]*len - windowSum must stay <= k.

Possible alternatives:
Binary search on the answer length with a prefix-sum feasibility check.

---

## 64. Divide Intervals Into Minimum Number of Groups

Primary pattern: Sweep on start/end events, or a min-heap of end times

Also touches: Greedy

Trigger:
Minimum number of resources so that no two overlapping intervals share one - the classic platform count.

Why:
The answer is the maximum number of intervals overlapping at any instant; sweep +1/-1, or greedily reuse the earliest-freeing group via a heap.

Possible alternatives:
Sort by start and reuse a min-heap of end times.

---

## 65. Stone Game VII

Primary pattern: Interval DP for a two-player zero-sum game

Also touches: Prefix sum, Minimax

Trigger:
Both players play optimally from either end, so the state is the surviving interval.

Why:
dp[i][j] = max over the two removals of (remaining sum - dp of the smaller interval).

Possible alternatives:
Memoised minimax recursion on (i, j).

---

## 66. Map of Highest Peak

Primary pattern: Multi-source BFS on a grid

Also touches: Distance transform

Trigger:
Heights differ by at most one between neighbours, and water cells are fixed at zero.

Why:
Push every water cell as a BFS source at once; the BFS layer number is the height.

Possible alternatives:
Two-pass dynamic-programming distance transform.

---

## 67. Minimum Cost to Merge Stones

Primary pattern: Interval DP with a K-merge residue state

Also touches: Heap greedy (fails), Prefix sum

Trigger:
Merges combine K consecutive piles, so only certain interval lengths can collapse to one pile.

Why:
dp[i][j][m] over intervals and remaining piles, splitting at steps of K-1; feasible only if (n-1) % (K-1) == 0.

Possible alternatives:
Greedy min-heap merging (correct only for K=2); memoised recursion on (i, j, m).

---

## 68. Shortest Unsorted Continuous Subarray

Primary pattern: Prefix maximum and suffix minimum scans

Also touches: Sorting comparison, Monotonic stack

Trigger:
Find the shortest window whose sorting fixes the whole array.

Why:
The right edge is the last index where prefixMax > a[i]; the left edge is the first where suffixMin < a[i].

Possible alternatives:
Sort a copy and compare boundaries; monotonic stack from both sides.

---

## 69. Form Largest Integer With Digits That Add up to Target

Primary pattern: Unbounded knapsack + lexicographic reconstruction

Also touches: Greedy comparison

Trigger:
Digits are reusable with fixed costs and an exact budget, and the answer is the largest NUMBER.

Why:
dp[t] = the maximum number of digits for cost t, then rebuild greedily from digit 9 downwards.

Possible alternatives:
Direct string-DP comparison with a custom bigger-number comparator.

---

## 70. Replace the Substring for Balanced String

Primary pattern: Variable sliding window with an outside-the-window condition

Also touches: Frequency counting

Trigger:
You may rewrite one substring arbitrarily; feasibility depends on the characters NOT in the window.

Why:
Shrink the smallest window such that every count outside it is <= n/4.

Possible alternatives:
Binary search on window length plus a check; greedy fails.

---

## 71. Minimize the Maximum of Two Arrays

Primary pattern: Binary search on the answer + inclusion-exclusion with LCM

Also touches: Number theory

Trigger:
The answer space reaches 10^9 while the check is pure counting - the array can never be built.

Why:
Count how many values <= x are usable by each array using multiples of the divisors and their LCM, then binary search.

Possible alternatives:
Greedy construction; direct simulation (impossible at this size).

---

## 72. Advantage Shuffle

Primary pattern: Greedy matching after sorting both sides

Also touches: Two pointers, Multiset

Trigger:
Maximise the count of positions where you beat the opponent; your order is completely free.

Why:
Sort both; for each opponent value spend the smallest card that beats it, otherwise dump your weakest.

Possible alternatives:
Multiset with upper_bound lookups per opponent value.

---

## 73. Super Pow

Primary pattern: Fast modular exponentiation with an array exponent

Also touches: Modular arithmetic

Trigger:
The exponent is given as a digit array, so it cannot be reconstructed as an integer.

Why:
Apply a^(10d + r) = (a^d)^10 * a^r digit by digit, with binary exponentiation modulo 1337.

Possible alternatives:
Euler's theorem to reduce the exponent modulo phi(1337).

---

## 74. Maximum Number of Events That Can Be Attended

Primary pattern: Sweep by day + min-heap of end days

Also touches: Greedy, Sorting

Trigger:
One event per day, each with a window - the classic 'sort by end' greedy no longer fits directly.

Why:
Advance day by day, push events that have started, discard expired ones, and attend the earliest-ending.

Possible alternatives:
Union-find over the next free day.

---

## 75. Find Eventual Safe States

Primary pattern: Cycle detection via DFS colouring, or reverse-graph topological sort

Also touches: Memoisation

Trigger:
A node is safe iff no path from it reaches a cycle.

Why:
Three-colour DFS marking nodes in the recursion stack, or Kahn's algorithm on the reversed graph.

Possible alternatives:
Iterative topological sort on reversed edges by out-degree.

---

## 76. Count Number of Nice Subarrays

Primary pattern: Prefix count + hash map (or atMost window)

Also touches: Sliding window

Trigger:
Counts subarrays containing exactly k odd numbers - parity turns into a prefix count.

Why:
Treat odd as 1, then it is 'subarrays summing to k' via a prefix-count map.

Possible alternatives:
atMost(k) - atMost(k-1) with a window; gap counting between odd positions.

---

## 77. Path With Minimum Effort

Primary pattern: Binary search on the answer + BFS, or Dijkstra on a minimax path

Also touches: Union-find, Dijkstra

Trigger:
Minimise the maximum edge weight along a path - a minimax route rather than a sum.

Why:
Either binary search the threshold and BFS through allowed edges, or run Dijkstra where the cost is a running max.

Possible alternatives:
Kruskal-style union-find, adding edges by weight until start and end connect.

---

## 78. Count Subarrays Where Max Element Appears at Least K Times

Primary pattern: Sliding window counting subarrays

Also touches: Frequency counting

Trigger:
'At least K occurrences' plus 'count all subarrays' - a monotone property once the max is fixed.

Why:
Slide right, shrink while the count of the global max is >= k, add left as the number of valid starts.

Possible alternatives:
Prefix positions of the max element plus index arithmetic.

---

## 79. Score of Parentheses

Primary pattern: Stack of partial scores

Also touches: Depth counting

Trigger:
Nested structure where an inner result is combined into its enclosing frame.

Why:
Push 0 on '(', and on ')' fold the top as max(2*top, 1) into the frame below.

Possible alternatives:
Count only the depth of each '()' pair and sum 2^depth.

---

## 80. Maximum Length of Subarray With Positive Product

Primary pattern: Linear DP carrying two running lengths

Also touches: Sliding window (misleading), Zero partitioning

Trigger:
Sign, not magnitude, decides validity, and a single negative flips the whole product.

Why:
Carry the longest positive-product and longest negative-product suffix lengths, resetting at zeros.

Possible alternatives:
Split at zeros and use the first/last negative index in each segment.

---

## 81. Find the City With the Smallest Number of Neighbors at a Threshold Distance

Primary pattern: Floyd-Warshall all-pairs shortest paths

Also touches: Dijkstra per node

Trigger:
n <= 100 with all-pairs reachability under a distance threshold - the size explicitly permits O(n^3).

Why:
Run Floyd-Warshall, then count reachable cities per node under the threshold.

Possible alternatives:
Dijkstra from every node.

---

## 82. Number of Ways to Arrive at Destination

Primary pattern: Dijkstra carrying a path count alongside the distance

Also touches: Counting DP

Trigger:
Not just the shortest time, but HOW MANY shortest routes achieve it, modulo 1e9+7.

Why:
During relaxation, reset the count on a strict improvement and add on an equal-distance tie.

Possible alternatives:
Dijkstra first, then a DAG DP over the shortest-path graph.

---

## 83. Subarrays with K Different Integers

Primary pattern: Sliding window (exactly-K via atMost)

Also touches: Hash map counting

Trigger:
Counts contiguous subarrays with EXACTLY K distinct values; a single variable window cannot hold an exact-count invariant.

Why:
exactly(K) = atMost(K) - atMost(K-1); each atMost is a standard shrinking window.

Possible alternatives:
Two-pointer with two synchronized left bounds; brute force O(n^2) with counts.

---

## 84. Recover Binary Search Tree

Primary pattern: In-order traversal invariant + swapped-pair detection

Also touches: Morris traversal

Trigger:
Exactly two nodes are swapped, and in-order must be strictly increasing.

Why:
Track the previous node during in-order; the first and last violations identify the swapped pair.

Possible alternatives:
Morris traversal for O(1) space; collect the in-order list and diff it against its sorted copy.

---

## 85. Maximum Population Year

Primary pattern: Difference array over a small coordinate domain

Also touches: Counting

Trigger:
Intervals of years, and you need the point with maximum coverage - the domain is tiny.

Why:
Increment at birth, decrement at death, prefix-sum the timeline, take the argmax.

Possible alternatives:
Sort start/end events and sweep with a counter.

---

## 86. Binary Tree Cameras

Primary pattern: Post-order DFS with a three-state return + greedy placement

Also touches: Tree DP

Trigger:
Minimise placed nodes under a covering constraint that reaches one level in each direction.

Why:
Return NEEDS_COVER / HAS_CAMERA / COVERED from each child and place a camera only when a child needs one.

Possible alternatives:
dp[node][3] tree DP over the same three states.

---

## 87. Maximum Alternating Subsequence Sum

Primary pattern: State-machine DP with two carried states

Also touches: Greedy

Trigger:
Each element is taken at an even or odd position, or skipped - a tiny state set carried forward.

Why:
Track best sums ending on an even index and on an odd index; each element updates both.

Possible alternatives:
Greedy sum of positive consecutive differences.

---

## 88. Minimum Deletions to Make Character Frequencies Unique

Primary pattern: Frequency counting + greedy decrement into free slots

Also touches: Sorting, Hash set

Trigger:
All frequencies must become distinct, and only deletions are allowed.

Why:
Sort frequencies descending and lower each one until it hits an unused value, accumulating the drops.

Possible alternatives:
Track used counts in a hash set while scanning frequencies.

---

## 89. Best Sightseeing Pair

Primary pattern: Rearrange a pair score, carry the best left contribution

Also touches: Prefix maximum

Trigger:
Score is values[i]+values[j]+i-j with i<j - the index distance is baked into the score.

Why:
Split into (values[i]+i) and (values[j]-j); scan right, keeping the best left term seen so far.

Possible alternatives:
O(n^2) pair enumeration; a monotonic-stack framing.

---

## 90. Grouping

Primary pattern: Bitmask DP over subsets with submask enumeration

Also touches: Set partition DP, Precomputed subset scores

Trigger:
Partition N <= 16 items into groups with a pairwise score - N is tiny, which permits exponential state.

Why:
Precompute the score of every subset, then dp[mask] = max over submasks of dp[mask ^ sub] + score[sub], enumerated as sub = (sub-1) & mask.

Possible alternatives:
Greedy grouping (wrong); brute force over all set partitions (Bell numbers, too slow).

---

## 91. Dungeon Game

Primary pattern: Grid DP computed BACKWARDS from the destination

Also touches: Binary search on answer

Trigger:
The requirement at a cell depends on future cells, so a forward prefix cannot decide it.

Why:
dp[i][j] = max(1, min(dp[i+1][j], dp[i][j+1]) - grid[i][j]), filled from the bottom-right.

Possible alternatives:
Binary search the starting health with a forward feasibility DP.

---

## 92. Last Stone Weight II

Primary pattern: Subset sum / 0-1 knapsack in disguise

Also touches: Partition DP

Trigger:
Every stone ends up with a + or - sign, so the answer is the smallest achievable |difference|.

Why:
Minimise |total - 2*S| over reachable subset sums S; a boolean knapsack over total/2.

Possible alternatives:
Greedy heap simulation (wrong); meet in the middle.

---
