# DSA mistake log

BEFORE CODING:
[ ] What is every input?
[ ] What is its maximum size?
[ ] Can any arithmetic overflow int?
[ ] What are the valid ranges/indices?
[ ] Have I listed ALL possible moves?
[ ] What invariant am I relying on?

| Bug/Mistake | Prevention |
| --- | --- |
| int overflow on accumulator | get an idea on how big can the accumulator get. use long to be safe. |
| used wrong variable’s length | write the name and size of all variables beforehand. |
| used wrong array length | write the name and size of all variables beforehand. |
| loop started at 0 before 1. | state valid index/range before coding. |
| missed the keep but dont score move in dp | enumerate all moves, even do nothing ones. |
| unnecessary case split on signs | test the invariant dont branch it. |
| wrong data type used “string” instead of “int” | check data types before using |
| Used a working/auxiliary array without initializing it from the original required state. | Copy or populate the auxiliary structure before modifying it. |
| Used `continue` inside a nested loop assuming it would skip the outer loop. | Remember that `continue` affects the nearest enclosing loop; use a flag, labeled loop, or move the `continue` to the correct scope. |
| Continued processing after detecting an impossible/invalid case. | Exit the current test case immediately using `continue`, `break`, or an early return as appropriate. |
| Redeclared a loop variable that was already in scope. | Use different loop-variable names for nested loops. |
| Kept unused variables or arrays in the solution. | Remove unused code before submission to reduce confusion and mistakes. |
| Used magic numbers such as `97` for character indexing. | Use readable literals such as `ch - 'a'`. |
| Did not perform a final compile/dry-run check after completing the algorithm. | Spend the last 2–3 minutes checking variable declarations, loop scopes, control flow, edge cases, and one sample execution. |
| Greedy counterexample construction → I couldn't break my own greedy despite trying. | When two choices score equally now, ask what STATE differs afterward and which future input benefits from that difference.
CF 2232C1: packing A vs opening a table -> # non-empty tables differs -> try E next -> AAE breaks greedy. |
| DP transition contains min/max over previous `k` states | monotonic deque |
| Assumed node indices follow tree order. | Node IDs are arbitrary unless explicitly guaranteed. Build adjacency + BFS/DFS from root.**Trigger →** `parent[]` does not imply traversal order. |
| Used `O(nk)` by scanning previous `k` DP states. | Maintain the window min/max with a monotonic deque → `O(n)` |
| Focused only on removing from left/right and made the solution complicated. | Ask what remains after the operations. Prefix + suffix removal leaves a contiguous middle. |
| Remove from both ends | think about what remains in the middle |
| Took time to connect subarray divisibility with prefix remainders | Equal prefix remainders mean the difference between them is divisible by `k`. |
| Tried comparing each left index with possible right indices → `O(n²)`. | Remove useless left candidates; only prefix minimums can give the best width. |
| Used `Math.abs()` to handle negative modulo while moving backward in a circular array, which changed the intended index direction. | For circular decrement, use `(index - 1 + n) % n`. Never use `Math.abs()` as a replacement for modular wrapping. |
| Tried to decide and delete low-degree vertices in the same traversal, which broke simultaneous-round behavior. | For simultaneous deletions, first freeze the current deletion set, then apply all deletions. Optimal version: process those rounds with a queue. |
| Started coding a sequential state-changing problem before the transformation was fully understood. | Before coding, trace symbolic state transitions explicitly (for example `[a,b] → ...`) through at least one full round. Derive the invariant/cycle first, then implement. |
| Detected periodicity but generalized it before handling the leftover operations correctly. | Split the process into `fullRounds = k / n` and `leftover = k % n`. First derive the exact state after full rounds, then apply only the leftover prefix. |
| Spent ~80 minutes on one OA problem because it felt almost solved, leaving the final question untouched. | OA rule: if ~25–30 minutes pass without a stable algorithm/invariant, park the problem and move on. Revisit only after every question has been attempted. |
| Used `int` for a problem allowing `n < 2^60`. | Read numeric bounds before coding. Anything approaching `2^31` needs `long`; for powers of two in Java prefer exact bit operations such as `1L << p` / `Long.highestOneBit(n)` over `Math.pow()`. |

## Unsolved

- CF 2232C1 — Greedy without proof → locally valid A placement reduced future capacity. Test greedy with a counterexample before coding. [UNSOLVED]
- 2406 - intervals grouping leetcode
- LC 2115 — Find All Possible Recipes from Given Supplies — Topological sort / dependency unlocking practice. [UNSOLVED]
- Amazon OA — Warehouse Robots with Threshold — Could not independently derive the forced/flexible-state counting step after fixing `R`; revisit combination counting. [UNSOLVED]
- Amazon OA — Find Total Efficiency — Reached interval-frequency aggregation and cumulative-smaller-count idea, but could not independently derive the full constraints-safe implementation/data structure. Main gap: turning the correct counting idea into efficient ordered-value queries. \[UNSOLVED\]
- Salesforce AMTS OA — Key Salesforce Teams — Not attempted because Q2 consumed the remaining OA time. No algorithmic diagnosis yet. \[UNSOLVED\]

## Salesforce AMTS — 30 Aug 2026 checkpoint

### Timed-paper diagnosis

- **Q1 — System Energy Reduction:** representation / formulation failure. Initial subtract-only framing was corrected after noticing that addition can help, but no stable algorithm was reached during the timer.
- **Q2 — Updating Salesforce Logs:** sequential-state representation failure first, then implementation/debugging consumed most of the paper.
- **Q3 — Key Salesforce Teams:** not attempted because time was exhausted on Q2. **Do not classify this as a tree-algorithm weakness from the timed attempt.**

### Repair evidence after the paper

For Q2, the mirrored-pair state cycle was later derived correctly:

`[a,b] → [a⊕b,a] → [b,a⊕b] → [a,b]`, period 3.

Correct decomposition:

- `fullRounds = iterations / n`
- `leftover = iterations % n`
- build the state after `fullRounds % 3`
- then simulate only the leftover operations in-place
- odd center element becomes 0 once touched and stays 0

### Active repair queue

1. **HIGH:** OA time control → hard switch around 25–30 minutes without a stable approach.
2. **MAINTAIN / verify later:** sequential state-changing problems → repair passed closed-book on 31 Aug.
3. **MAINTAIN / verify later:** power-of-two transformation reasoning → repair passed on 31 Aug; remember numeric bounds / `long`.

## Repair verification — 31 Aug 2026

### Q2 — Updating Salesforce Logs: **REPAIR → PASS**

Re-derived closed-book:

- mirrored-pair cycle: `[a,b] → [a⊕b,a] → [b,a⊕b] → [a,b]`;
- period = 3 full rounds;
- `fullRounds = iterations / n`;
- `leftover = iterations % n`;
- construct the state after the final complete round, then simulate only the leftover operations in order and in-place;
- for odd `n`, when `cur == opp`, `x ^ x = 0`, and the center stays 0 afterward.

**Status:** sequential-state representation is repaired enough for now. Do not relearn it; verify through future unseen problems.

### Q1 — System Energy Reduction: **REPAIR → PASS**

Independently derived the greedy transformation:

- at each step choose the nearest power of two, either below or above the current value;
- replace `n` with the smaller distance to that power of two;
- count that operation and repeat until `n == 0`.

Example intuition: `15 → 1 → 0` by moving first to 16, showing why an upward power-of-two move can be optimal.

Implementation was structurally correct, with one real bug:

- used `int` / `Math.pow()` despite the bound `n < 2^60`.

**Repair:** use `long` and exact bit operations, e.g. `Long.highestOneBit(n)` or `1L << p`.

**Status:** algorithmic formulation repaired; numeric-bound discipline added to the mistake log.

## Representation Drill — 1 Sep 2026

### Q2 — Efficient Workers: **UNRESOLVED**

- Correctly recognized that the problem depends on how efficiencies are paired after excluding one worker.
- No stable constraints-safe representation was reached independently.
- The attempted “equal count on positive/negative side” framing was not valid in general.
- **Status:** keep unresolved; revisit fresh rather than force the pattern while fatigued.

### Q3 — Maximizing Salesforce Opportunities: **REPRESENTATION PASS**

Local row representation was found correctly:

- if exactly `x` values are removed from the two ends of one row, the values left behind must form one contiguous subarray of length `m - x`;
- therefore the best gain for one row can be represented as `rowSum - minimum subarray sum of length (m - x)`.

The missing 2D bridge was then derived:

- define `dp[i][j]` = maximum score using the first `i` rows with exactly `j` removals;
- for the next row, try taking `x` removals from that row;
- transition: `dp[i][j] = max(dp[i-1][j-x] + bestRemoved[i][x])`;
- combining complexity is `O(n*k*m)`; preprocessing all per-row choices can be done in about `O(n*m^2)`.

**Status:** global representation reached after a short prompt. Main active DSA weakness remains converting a valid local observation into the full global state/transition under pressure.

### Current DSA diagnosis

- Standard patterns remain broadly solid.
- Unfamiliar-problem representation/reframing is still the main active focus.
- Do not overlearn Q3 now; verify this skill later with unseen problems.