# DSA Patterns — LSEG Final Revision

# 0. Frequently Asked Sorting Algorithms

<aside>
📌

Know the **idea, complexity, stability/in-place property, and Java code** for these. For interviews, Merge Sort, Quick Sort, Heap Sort, Insertion Sort, Selection Sort, and Bubble Sort are the main ones to recall.

</aside>

| Algorithm | Best | Average | Worst | Extra Space | Stable? |
| --- | --- | --- | --- | --- | --- |
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) avg recursion | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

## Bubble Sort

**Idea:** repeatedly swap adjacent out-of-order elements; largest remaining element bubbles to the end.

```java
static void bubbleSort(int[] a) {
    int n = a.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (a[j] > a[j + 1]) {
                int t = a[j];
                a[j] = a[j + 1];
                a[j + 1] = t;
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

## Insertion Sort

**Idea:** maintain a sorted prefix and insert the current value into its correct position.

```java
static void insertionSort(int[] a) {
    for (int i = 1; i < a.length; i++) {
        int key = a[i];
        int j = i - 1;

        while (j >= 0 && a[j] > key) {
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = key;
    }
}
```

**Remember:** very good for **small or nearly sorted arrays**.

## Selection Sort

**Idea:** find the minimum element in the unsorted suffix and place it at the current position.

```java
static void selectionSort(int[] a) {
    int n = a.length;
    for (int i = 0; i < n - 1; i++) {
        int min = i;
        for (int j = i + 1; j < n; j++) {
            if (a[j] < a[min]) min = j;
        }

        int t = a[i];
        a[i] = a[min];
        a[min] = t;
    }
}
```

## Merge Sort

**Idea:** divide array into halves, recursively sort both halves, then merge two sorted arrays.

```java
static void mergeSort(int[] a, int l, int r) {
    if (l >= r) return;

    int mid = l + (r - l) / 2;
    mergeSort(a, l, mid);
    mergeSort(a, mid + 1, r);
    merge(a, l, mid, r);
}

static void merge(int[] a, int l, int mid, int r) {
    int[] temp = new int[r - l + 1];
    int i = l, j = mid + 1, k = 0;

    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) temp[k++] = a[i++];
        else temp[k++] = a[j++];
    }

    while (i <= mid) temp[k++] = a[i++];
    while (j <= r) temp[k++] = a[j++];

    for (int x = 0; x < temp.length; x++) {
        a[l + x] = temp[x];
    }
}
```

**Remember:** guaranteed `O(n log n)`, stable, but needs `O(n)` auxiliary space for arrays.

## Quick Sort

**Idea:** choose a pivot, partition smaller elements to one side and larger elements to the other, then recurse.

```java
static void quickSort(int[] a, int l, int r) {
    if (l >= r) return;

    int p = partition(a, l, r);
    quickSort(a, l, p - 1);
    quickSort(a, p + 1, r);
}

static int partition(int[] a, int l, int r) {
    int pivot = a[r];
    int i = l;

    for (int j = l; j < r; j++) {
        if (a[j] <= pivot) {
            int t = a[i];
            a[i] = a[j];
            a[j] = t;
            i++;
        }
    }

    int t = a[i];
    a[i] = a[r];
    a[r] = t;
    return i;
}
```

**Remember:** average `O(n log n)`, worst `O(n²)` with bad pivots. Randomized pivot reduces the chance of worst-case behavior.

## Heap Sort

**Idea:** build a max heap, repeatedly swap the maximum with the end, shrink heap size, and heapify again.

```java
static void heapSort(int[] a) {
    int n = a.length;

    // Bottom-up heap construction: O(n)
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(a, n, i);
    }

    for (int end = n - 1; end > 0; end--) {
        int t = a[0];
        a[0] = a[end];
        a[end] = t;

        heapify(a, end, 0);
    }
}

static void heapify(int[] a, int size, int i) {
    while (true) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;

        if (left < size && a[left] > a[largest]) largest = left;
        if (right < size && a[right] > a[largest]) largest = right;

        if (largest == i) break;

        int t = a[i];
        a[i] = a[largest];
        a[largest] = t;
        i = largest;
    }
}
```

<aside>
🧠

**Heap-build trap:** bottom-up heap construction using sift-down from `n/2 - 1` is **O(n)**, not O(n log n). Repeatedly inserting all elements into a heap is O(n log n).

</aside>

## Java Sorting Syntax

```java
Arrays.sort(nums);                        // primitive ascending
Arrays.sort(arr, (a, b) -> a[0] - b[0]); // objects / int[][] comparator

// Safer comparator when values may overflow subtraction:
Arrays.sort(arr, (a, b) -> Integer.compare(a[0], b[0]));

// Sort by first field, then second:
Arrays.sort(arr, (a, b) -> {
    if (a[0] != b[0]) return Integer.compare(a[0], b[0]);
    return Integer.compare(a[1], b[1]);
});
```

<aside>
⚡

**Interview recall:** Merge Sort = divide + merge. Quick Sort = partition around pivot. Heap Sort = build max heap + repeatedly extract max. Insertion = sorted prefix. Selection = repeatedly select minimum. Bubble = adjacent swaps.

</aside>

---

<aside>
⚡

**Use this as a trigger sheet, not a syllabus.** For an unseen problem: read constraints → state brute force → identify bottleneck → look for the structural trigger below → use the first correct complexity that fits. Do not keep optimizing after you have a viable solution.

</aside>

# 1. HashMap / HashSet

## Trigger

- Need **existence / frequency / complement / last-seen / first-seen** in O(1) average time.
- Unsorted array and you are repeatedly asking “have I seen X?”

### Two Sum

```java
Map<Integer, Integer> map = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    int need = target - nums[i];
    if (map.containsKey(need)) {
        return new int[]{map.get(need), i};
    }
    map.put(nums[i], i);
}
```

### Frequency Map

```java
Map<Integer, Integer> freq = new HashMap<>();
for (int x : nums) {
    freq.put(x, freq.getOrDefault(x, 0) + 1);
}
```

---

# 2. Prefix Sum + HashMap

## Trigger

- **Contiguous subarray** with target sum / count / remainder condition.
- Negatives exist, so ordinary sliding window is unsafe.

### Longest Subarray Sum = K

```java
Map<Long, Integer> first = new HashMap<>();
first.put(0L, -1);
long sum = 0;
int ans = 0;

for (int i = 0; i < nums.length; i++) {
    sum += nums[i];

    if (first.containsKey(sum - k)) {
        ans = Math.max(ans, i - first.get(sum - k));
    }

    first.putIfAbsent(sum, i); // earliest index for longest length
}
```

### Count Subarrays Sum = K

```java
Map<Long, Integer> freq = new HashMap<>();
freq.put(0L, 1);
long sum = 0;
long count = 0;

for (int x : nums) {
    sum += x;
    count += freq.getOrDefault(sum - k, 0);
    freq.put(sum, freq.getOrDefault(sum, 0) + 1);
}
```

### Prefix Modulo

Use when the condition is divisibility.

```java
Map<Integer, Integer> last = new HashMap<>();
last.put(0, -1);
int pref = 0;

for (int i = 0; i < nums.length; i++) {
    pref = (pref + nums[i]) % p;
    int need = (pref - targetRemainder + p) % p;

    if (last.containsKey(need)) {
        // candidate subarray
    }

    last.put(pref, i); // placement matters: update every iteration
}
```

**Remember:** earliest index → longest; latest index → shortest.

---

# 3. Sliding Window

## Trigger

- **Contiguous** segment.
- Condition changes monotonically when left/right move.
- Especially useful when values are non-negative or condition is frequency-based.

### Variable Window

```java
int left = 0;
for (int right = 0; right < nums.length; right++) {
    // add nums[right]

    while (/* window invalid */) {
        // remove nums[left]
        left++;
    }

    // window [left..right] is valid
}
```

### Minimum Length Sum >= Target, Positive Numbers

```java
int left = 0;
long sum = 0;
int ans = Integer.MAX_VALUE;

for (int right = 0; right < nums.length; right++) {
    sum += nums[right];

    while (sum >= target) {
        ans = Math.min(ans, right - left + 1);
        sum -= nums[left++];
    }
}
```

### Character-Frequency Window

```java
int[] freq = new int[128];
int left = 0;

for (int right = 0; right < s.length(); right++) {
    freq[s.charAt(right)]++;

    while (/* invalid */) {
        freq[s.charAt(left)]--;
        left++;
    }
}
```

---

# 4. Two Pointers

## Trigger

- Sorted array.
- Pair/triplet relation.
- Opposite ends or read/write compaction.

### Pair Sum in Sorted Array

```java
int l = 0, r = nums.length - 1;
while (l < r) {
    long sum = (long) nums[l] + nums[r];
    if (sum == target) break;
    if (sum < target) l++;
    else r--;
}
```

### Remove Duplicates / Compact In Place

```java
int write = 0;
for (int read = 0; read < nums.length; read++) {
    if (/* keep nums[read] */) {
        nums[write++] = nums[read];
    }
}
```

---

# 5. Binary Search

## Trigger

- Sorted data.
- Need first/last valid location.
- Or a **monotonic yes/no answer space**.

### Exact Search

```java
int l = 0, r = nums.length - 1;
while (l <= r) {
    int mid = l + (r - l) / 2;
    if (nums[mid] == target) return mid;
    if (nums[mid] < target) l = mid + 1;
    else r = mid - 1;
}
return -1;
```

### Lower Bound: First Index >= Target

```java
int l = 0, r = nums.length - 1;
while (l <= r) {
    int mid = l + (r - l) / 2;
    if (nums[mid] >= target) r = mid - 1;
    else l = mid + 1;
}
return l;
```

### Binary Search on Answer

Trigger: “minimum X such that possible”, “maximum X such that feasible”.

```java
long l = low, r = high;
while (l <= r) {
    long mid = l + (r - l) / 2;

    if (can(mid)) {
        // for minimum feasible
        r = mid - 1;
    } else {
        l = mid + 1;
    }
}
return l;
```

The key requirement is **monotonicity**: once `can(x)` becomes true, it stays true in one direction.

---

# 6. Monotonic Stack

## Trigger

- Next greater/smaller.
- Previous greater/smaller.
- Histogram / contribution / “first element to left/right that breaks condition”.

### Next Greater Element to Right

```java
int n = nums.length;
int[] ans = new int[n];
Arrays.fill(ans, -1);
Deque<Integer> st = new ArrayDeque<>();

for (int i = 0; i < n; i++) {
    while (!st.isEmpty() && nums[i] > nums[st.peek()]) {
        ans[st.pop()] = nums[i];
    }
    st.push(i);
}
```

### Previous Smaller Index

```java
Deque<Integer> st = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!st.isEmpty() && nums[st.peek()] >= nums[i]) st.pop();
    int prevSmaller = st.isEmpty() ? -1 : st.peek();
    st.push(i);
}
```

---

# 7. Monotonic Deque

## Trigger

- Sliding window needs current **minimum or maximum** efficiently.
- Longest valid window with `max - min <= limit`.

```java
Deque<Integer> maxD = new ArrayDeque<>();
Deque<Integer> minD = new ArrayDeque<>();
int left = 0;
int ans = 0;

for (int right = 0; right < nums.length; right++) {
    while (!maxD.isEmpty() && nums[maxD.peekLast()] <= nums[right]) maxD.pollLast();
    while (!minD.isEmpty() && nums[minD.peekLast()] >= nums[right]) minD.pollLast();

    maxD.offerLast(right);
    minD.offerLast(right);

    while ((long) nums[maxD.peekFirst()] - nums[minD.peekFirst()] > limit) {
        if (maxD.peekFirst() == left) maxD.pollFirst();
        if (minD.peekFirst() == left) minD.pollFirst();
        left++;
    }

    ans = Math.max(ans, right - left + 1);
}
```

---

# 8. Heap / PriorityQueue

## Trigger

- Repeatedly need current smallest/largest among changing candidates.
- Kth element.
- Scheduling / simulation.
- Merge K sorted streams.

### Java Syntax

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

### Custom Comparator

```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> {
    if (a[1] != b[1]) return Integer.compare(a[1], b[1]);
    return Integer.compare(a[0], b[0]);
});
```

### Keep K Largest → Min Heap of Size K

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
for (int x : nums) {
    pq.offer(x);
    if (pq.size() > k) pq.poll();
}
// pq.peek() = kth largest
```

### Scheduling / Simulation Skeleton

```java
Arrays.sort(tasks, Comparator.comparingInt(a -> a[0])); // arrival
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> {
    if (a[1] != b[1]) return Integer.compare(a[1], b[1]);
    return Integer.compare(a[2], b[2]);
});

long time = 0;
int i = 0;
while (i < tasks.length || !pq.isEmpty()) {
    if (pq.isEmpty() && time < tasks[i][0]) time = tasks[i][0];

    while (i < tasks.length && tasks[i][0] <= time) {
        pq.offer(tasks[i++]);
    }

    int[] cur = pq.poll();
    time += cur[1];
}
```

---

# 9. Intervals

## A. Maximum Simultaneous Overlap / Minimum Rooms or Groups

### Trigger

“How many are active at the same time?”

Use sweep line.

```java
List<int[]> events = new ArrayList<>();
for (int[] in : intervals) {
    events.add(new int[]{in[0], +1});
    events.add(new int[]{in[1], -1});
}

// Tie rule depends on whether endpoints overlap.
events.sort((a, b) -> {
    if (a[0] != b[0]) return Integer.compare(a[0], b[0]);
    return Integer.compare(b[1], a[1]); // starts before ends for inclusive intervals
});

int active = 0, ans = 0;
for (int[] e : events) {
    active += e[1];
    ans = Math.max(ans, active);
}
```

## B. Maximum Number of Non-Overlapping Intervals

### Trigger

“Choose as many intervals as possible.”

Greedy: earliest finish time.

```java
Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));
int count = 0;
int lastEnd = Integer.MIN_VALUE;

for (int[] in : intervals) {
    if (in[0] >= lastEnd) {
        count++;
        lastEnd = in[1];
    }
}
```

## C. Merge Intervals

```java
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
List<int[]> res = new ArrayList<>();

for (int[] cur : intervals) {
    if (res.isEmpty() || res.get(res.size() - 1)[1] < cur[0]) {
        res.add(cur.clone());
    } else {
        res.get(res.size() - 1)[1] = Math.max(res.get(res.size() - 1)[1], cur[1]);
    }
}
```

<aside>
⚠️

Do not confuse **maximum overlap** with **maximum events you can schedule**. Sweep line answers concurrency; greedy/heap scheduling answers assignment/selection.

</aside>

---

# 10. Greedy

## Trigger

- Need local choices that preserve future flexibility.
- Often sort by one decisive quantity: end time, cost, deadline, value.

Before committing to greedy, ask:

1. What is the local choice?
2. Why can choosing it never make the future worse?
3. Can I produce a counterexample?

### Lexicographically Smallest After One Deletion

```java
int remove = s.length() - 1;
for (int i = 0; i + 1 < s.length(); i++) {
    if (s.charAt(i) > s.charAt(i + 1)) {
        remove = i;
        break;
    }
}

String ans = s.substring(0, remove) + s.substring(remove + 1);
```

---

# 11. 1-D Dynamic Programming

## Trigger

- Answer at index depends on a small number of earlier states.
- “Pick/skip”, non-adjacent selection, minimum cost, number of ways.

### Pick / Skip

```java
long[] dp = new long[n + 1];
dp[0] = 0;
dp[1] = Math.max(0, nums[0]);

for (int i = 2; i <= n; i++) {
    dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i - 1]);
}
```

### Space Optimized

```java
long prev2 = 0;
long prev1 = Math.max(0, nums[0]);

for (int i = 1; i < nums.length; i++) {
    long cur = Math.max(prev1, prev2 + nums[i]);
    prev2 = prev1;
    prev1 = cur;
}
```

### Exact K Picks with No Adjacent Elements

State idea: `dp[i][k]` = best using first `i` items with exactly `k` chosen.

```java
long NEG = Long.MIN_VALUE / 4;
long[][] dp = new long[n + 1][K + 1];
for (long[] row : dp) Arrays.fill(row, NEG);
dp[0][0] = 0;

for (int i = 1; i <= n; i++) {
    for (int k = 0; k <= K; k++) {
        dp[i][k] = Math.max(dp[i][k], dp[i - 1][k]); // skip

        if (k > 0) {
            if (i == 1) {
                if (k == 1) dp[i][k] = Math.max(dp[i][k], nums[0]);
            } else if (dp[i - 2][k - 1] != NEG) {
                dp[i][k] = Math.max(dp[i][k], dp[i - 2][k - 1] + nums[i - 1]);
            }
        }
    }
}
```

---

# 12. Knapsack-Style DP

## Trigger

- Each item has a choice: take/not take.
- Target sum / capacity / partition.

### 0/1 Boolean Subset Sum

```java
boolean[] dp = new boolean[target + 1];
dp[0] = true;

for (int x : nums) {
    for (int s = target; s >= x; s--) {
        dp[s] |= dp[s - x];
    }
}
```

**Direction matters:**

- 0/1 item → iterate capacity backward.
- unlimited reuse → iterate forward.

---

# 13. 2-D DP: Strings / Sequences

## Trigger

- Two strings / two indices.
- Match vs skip decisions.

### Longest Common Subsequence

```java
int n = a.length(), m = b.length();
int[][] dp = new int[n + 1][m + 1];

for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= m; j++) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) {
            dp[i][j] = 1 + dp[i - 1][j - 1];
        } else {
            dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
}
```

---

# 14. BFS

## Trigger

- Unweighted shortest path.
- Minimum number of moves/steps.
- Level-by-level exploration.

```java
Queue<Integer> q = new ArrayDeque<>();
boolean[] vis = new boolean[n];
q.offer(src);
vis[src] = true;

while (!q.isEmpty()) {
    int u = q.poll();

    for (int v : graph[u]) {
        if (!vis[v]) {
            vis[v] = true;
            q.offer(v);
        }
    }
}
```

### Level BFS

```java
int dist = 0;
while (!q.isEmpty()) {
    int size = q.size();
    while (size-- > 0) {
        int u = q.poll();
        // process
    }
    dist++;
}
```

---

# 15. DFS

## Trigger

- Connected components.
- Reachability.
- Trees / graph traversal.
- Backtracking-style structural exploration.

```java
void dfs(int u, List<Integer>[] graph, boolean[] vis) {
    vis[u] = true;
    for (int v : graph[u]) {
        if (!vis[v]) dfs(v, graph, vis);
    }
}
```

### Grid DFS

```java
void dfs(int r, int c, int[][] grid, boolean[][] vis) {
    if (r < 0 || c < 0 || r >= grid.length || c >= grid[0].length) return;
    if (vis[r][c] || grid[r][c] == 0) return;

    vis[r][c] = true;
    dfs(r + 1, c, grid, vis);
    dfs(r - 1, c, grid, vis);
    dfs(r, c + 1, grid, vis);
    dfs(r, c - 1, grid, vis);
}
```

---

# 16. Cycle Detection / Topological Sort

## Directed Cycle: DFS States

Use `0 = unvisited`, `1 = currently in recursion path`, `2 = fully processed`.

```java
boolean dfs(int u, List<Integer>[] g, int[] state) {
    state[u] = 1;

    for (int v : g[u]) {
        if (state[v] == 1) return true;
        if (state[v] == 0 && dfs(v, g, state)) return true;
    }

    state[u] = 2;
    return false;
}
```

## Kahn's Topological Sort

### Trigger

- Prerequisites / dependencies.
- DAG ordering.

```java
int[] indeg = new int[n];
for (int u = 0; u < n; u++) {
    for (int v : g[u]) indeg[v]++;
}

Queue<Integer> q = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    if (indeg[i] == 0) q.offer(i);
}

List<Integer> order = new ArrayList<>();
while (!q.isEmpty()) {
    int u = q.poll();
    order.add(u);

    for (int v : g[u]) {
        if (--indeg[v] == 0) q.offer(v);
    }
}

boolean hasCycle = order.size() != n;
```

---

# 17. DSU / Union-Find

## Trigger

- Dynamic connectivity.
- “Are these nodes already connected?”
- Kruskal MST.

```java
class DSU {
    int[] parent, rank;

    DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;

        if (rank[ra] < rank[rb]) {
            parent[ra] = rb;
        } else if (rank[ra] > rank[rb]) {
            parent[rb] = ra;
        } else {
            parent[rb] = ra;
            rank[ra]++;
        }
        return true;
    }
}
```

---

# 18. Dijkstra

## Trigger

- Weighted graph.
- All edge weights non-negative.
- Shortest path.

```java
long[] dist = new long[n];
Arrays.fill(dist, Long.MAX_VALUE);
dist[src] = 0;

PriorityQueue<long[]> pq = new PriorityQueue<>(Comparator.comparingLong(a -> a[1]));
pq.offer(new long[]{src, 0});

while (!pq.isEmpty()) {
    long[] cur = pq.poll();
    int u = (int) cur[0];
    long d = cur[1];

    if (d != dist[u]) continue;

    for (int[] e : graph[u]) {
        int v = e[0], w = e[1];
        if (d + w < dist[v]) {
            dist[v] = d + w;
            pq.offer(new long[]{v, dist[v]});
        }
    }
}
```

**Do not use Dijkstra with negative-weight edges.**

---

# 19. Bellman-Ford

## Trigger

- Shortest paths with possible negative edges.
- Limited number of edges/stops can often use repeated relaxation.

```java
long INF = Long.MAX_VALUE / 4;
long[] dist = new long[n];
Arrays.fill(dist, INF);
dist[src] = 0;

for (int i = 0; i < n - 1; i++) {
    long[] next = dist.clone();

    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] != INF) {
            next[v] = Math.min(next[v], dist[u] + w);
        }
    }

    dist = next;
}
```

Using `clone()` per round is useful when the problem limits the number of edges/stops and you must not chain multiple new relaxations in the same round.

---

# 20. Minimum Spanning Tree

## Kruskal

### Trigger

- Connect all nodes with minimum total edge cost.

```java
Arrays.sort(edges, Comparator.comparingInt(a -> a[2]));
DSU dsu = new DSU(n);
long cost = 0;
int used = 0;

for (int[] e : edges) {
    if (dsu.union(e[0], e[1])) {
        cost += e[2];
        used++;
        if (used == n - 1) break;
    }
}
```

---

# 21. Linked List Essentials

## Reverse

```java
ListNode prev = null;
ListNode cur = head;

while (cur != null) {
    ListNode next = cur.next;
    cur.next = prev;
    prev = cur;
    cur = next;
}
return prev;
```

## Fast / Slow

### Trigger

- Cycle detection.
- Middle node.
- Nth-from-end variants.

```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
```

### Cycle

```java
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true;
}
return false;
```

---

# 22. Tree DFS

## Trigger

- Height/depth.
- Path sums.
- Subtree information.
- Diameter.

```java
int dfs(TreeNode node) {
    if (node == null) return 0;

    int left = dfs(node.left);
    int right = dfs(node.right);

    // combine left/right information here
    return 1 + Math.max(left, right);
}
```

### BST Inorder = Sorted

```java
void inorder(TreeNode node, List<Integer> out) {
    if (node == null) return;
    inorder(node.left, out);
    out.add(node.val);
    inorder(node.right, out);
}
```

---

# 23. Backtracking

## Trigger

- Generate all combinations/permutations/subsets.
- Need to make a choice, recurse, undo choice.

```java
void backtrack(int start, List<Integer> cur) {
    // optionally record cur

    for (int i = start; i < nums.length; i++) {
        cur.add(nums[i]);
        backtrack(i + 1, cur);
        cur.remove(cur.size() - 1);
    }
}
```

### Avoid Duplicate Combinations After Sorting

```java
if (i > start && nums[i] == nums[i - 1]) continue;
```

---

# 24. Bit Manipulation Survival Kit

```java
// check kth bit
boolean set = (x & (1 << k)) != 0;

// set kth bit
x = x | (1 << k);

// clear kth bit
x = x & ~(1 << k);

// toggle kth bit
x = x ^ (1 << k);

// odd/even
boolean odd = (x & 1) == 1;

// remove lowest set bit
x = x & (x - 1);

// power of two
boolean powerOfTwo = x > 0 && (x & (x - 1)) == 0;
```

### Count Set Bits

```java
int count = 0;
while (x != 0) {
    x &= (x - 1);
    count++;
}
```

### XOR Identities

```
x ^ x = 0
x ^ 0 = x
```

Unique element when every other element appears twice:

```java
int ans = 0;
for (int x : nums) ans ^= x;
```

---

# 25. Sorting Patterns to Recall

## Comparator Syntax

```java
Arrays.sort(arr, (a, b) -> Integer.compare(a[0], b[0]));
```

For primitive `int[]`, use normal `Arrays.sort(nums)`.

### Common sorting decisions

- Sort by **start** → merge / sweep / process arrivals.
- Sort by **end** → interval scheduling / maximize non-overlap.
- Sort + two pointers → pair/triplet relation.
- Sort + heap → scheduling where candidates become available over time.

---

# 26. Complexity Quick Map

| Constraint size | Usually safe target |
| --- | --- |
| n ≤ 20 | O(2^n), backtracking / bitmask may be possible |
| n ≈ 100 | O(n³) sometimes acceptable |
| n ≈ 1,000 | O(n²) often acceptable |
| n ≈ 100,000 | O(n log n) or O(n) |
| n ≈ 1,000,000 | Usually O(n) |

Always use the actual time limit and constants as context, but this is a useful first filter.

---

# 27. OA Decision Process

<aside>
🎯

**Tomorrow's objective is not to prove you can solve every unseen problem. It is to maximize solved count.**

</aside>

1. **Read literally.** Identify exactly what can/cannot change.
2. **State brute force.** Even if bad, make the problem concrete.
3. **Find the bottleneck.** Repeated search? repeated sum? repeated minimum? huge answer space?
4. **Match structure, not wording.** Contiguous → window/prefix. Repeated min/max → heap/deque. Connectivity → DFS/BFS/DSU. Monotonic feasibility → binary search.
5. **Commit to the first correct complexity that fits.** Do not hunt for a prettier solution.
6. **If logic is sound but code fails, debug before redesigning.** Trace a tiny case; inspect initialization, update order, and whether state updates accidentally sit inside an `if`.
7. **If there is no viable direction after ~10–15 minutes, move.** Return after securing easier questions.

---

# Final 60-Second Trigger Sheet

- **Contiguous + target sum, negatives** → prefix sum + hashmap.
- **Contiguous + positive values / frequency constraint** → sliding window.
- **Sorted pair/triplet** → two pointers.
- **First/last position** → binary search / lower bound.
- **Minimum/maximum feasible value** → binary search on answer.
- **Next/previous greater/smaller** → monotonic stack.
- **Window max/min** → monotonic deque.
- **Repeated smallest/largest / scheduling candidates** → heap.
- **Maximum simultaneous intervals** → sweep line.
- **Maximum non-overlapping intervals** → sort by end greedy.
- **Pick/skip / non-adjacent** → 1-D DP.
- **Target subset/capacity** → knapsack DP.
- **Two strings / two indices** → 2-D DP.
- **Unweighted shortest path** → BFS.
- **Components/reachability** → DFS/BFS.
- **Dependencies** → topo sort.
- **Dynamic connectivity / MST** → DSU.
- **Weighted shortest path, non-negative** → Dijkstra.
- **Negative edges / bounded relaxation** → Bellman-Ford style.
- **Generate choices** → backtracking.
- **Linked-list cycle/middle** → fast/slow.
- **Bit uniqueness/parity/subset masks** → XOR / bit operations.

<aside>
✅

For tomorrow, knowing **when** to use these is more important than memorizing every line. The code templates are here only so implementation recall does not become the bottleneck.

</aside>