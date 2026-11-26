# Algorithms and patterns cheat sheet for big tech OAs

## Spotting the pattern

Match a clue in the problem statement to a technique, then use the template in the section below. When nothing matches, write the brute force first and ask what work it repeats.

| Clue in the problem | Technique |
| --- | --- |
| Sorted input, "find", or O(log n) required | Binary search |
| "Minimum X such that it works" or "maximize the minimum" | Binary search on the answer |
| Pair or triple that hits a target in sorted data; palindrome; in-place dedupe | Two pointers |
| Longest or shortest contiguous subarray or substring with a property | Sliding window |
| Subarray sum equals k, especially with negative numbers | Prefix sums plus hash map |
| Counting, duplicates, anagrams, "seen before", pair lookup | Hash map or set |
| Fewest steps or moves in an unweighted graph or grid | BFS |
| Shortest path with non-negative weights | Dijkstra with a heap |
| Count islands or components, explore all paths, tree depth, cycle check | DFS |
| Generate all subsets, permutations or combinations (n up to about 20) | Backtracking |
| Count the ways, min or max cost, "can we reach", choices that overlap | Dynamic programming |
| k largest or smallest, kth element, running median, merge k sorted lists | Heap |
| Prerequisites, dependencies, build or course order | Topological sort |
| Groups that merge over time, "are these connected" | Union-find |
| Next greater or smaller element, histogram, stock span | Monotonic stack |
| Overlapping intervals, meeting rooms, schedules | Sort by start, then sweep |
| Prefix lookup, autocomplete, word dictionary | Trie |
| Cycle or middle of a linked list | Fast and slow pointers |
| A local choice looks safe and you cannot find a counterexample | Greedy (if you can, use DP) |

The input size says which complexity is allowed:

| Constraint | Target |
| --- | --- |
| n up to about 20 | O(2^n) or O(n!) for tiny n: backtracking, bitmask |
| n up to about 500 | O(n^3) |
| n up to about 5,000 | O(n^2): 2D DP, nested loops |
| n up to about 10^5 | O(n log n): sorting, heaps, binary search |
| n 10^6 or more | O(n) or O(log n): hash maps, two pointers, prefix sums |

Two more habits help. Check whether a cheap fact about the data is given (sorted, non-negative, values bounded by a small number), since each one unlocks a faster technique. And look at the output: "return the count or the best value" suggests DP or greedy, while "return all of them" suggests backtracking.

## Binary search

Use it when the answer sits at a boundary between a region where a condition is false and a region where it is true. O(log n).

Exact match in a sorted list:

```python
def search(a, target):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target:
            return mid
        if a[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

The more useful template finds the first index or value where a condition turns true. It covers lower bound, insertion point and search on the answer:

```python
def first_true(lo, hi, ok):      # ok(x) is False ... False True ... True
    while lo < hi:               # smallest x in [lo, hi] where ok(x) is True
        mid = (lo + hi) // 2
        if ok(mid):
            hi = mid             # mid might be the answer, keep it
        else:
            lo = mid + 1
    return lo                    # if ok may never be True, pass hi = n and treat n as 'none'
```

The mirror image finds the last true, for a condition that is True ... True False ... False. It uses the upper middle so the loop cannot stall:

```python
def last_true(lo, hi, ok):
    while lo < hi:
        mid = (lo + hi + 1) // 2
        if ok(mid):
            lo = mid
        else:
            hi = mid - 1
    return lo
```

Search on the answer: binary search over possible answers, with a function that checks one candidate. Example: the smallest eating speed k that finishes all piles within h hours.

```python
def min_speed(piles, h):
    def ok(k):
        return sum((p + k - 1) // k for p in piles) <= h   # ceiling division
    return first_true(1, max(piles), ok)
```

Spot it by the clues:

- The input is sorted, or the problem gives an O(log n) bound.
- The problem asks for the minimum or maximum value of something, and checking whether one candidate works is easy, usually one O(n) pass.
- If x works then x + 1 also works (or the reverse). Without that monotonic property, binary search does not apply.
- Phrases such as "minimize the largest", "maximize the smallest", "at most k days" and "the smallest capacity".

Pitfalls: pick the template that matches the loop condition (`lo < hi` with `hi = mid`, or `lo <= hi` with `hi = mid - 1`) and do not mix them. Use the upper middle whenever you set `lo = mid`. For floating-point answers, loop a fixed 60 to 100 times instead of testing equality. Python's `bisect_left` and `bisect_right` cover plain sorted lists.

## Two pointers and sliding window

Both turn an O(n^2) scan over pairs or subarrays into one O(n) pass by moving indexes instead of restarting.

Opposite ends, for sorted input or palindromes:

```python
def two_sum_sorted(a, target):
    l, r = 0, len(a) - 1
    while l < r:
        s = a[l] + a[r]
        if s == target:
            return [l, r]
        if s < target:
            l += 1               # need a bigger sum
        else:
            r -= 1               # need a smaller sum
    return []
```

Same direction, a read pointer and a write pointer (in-place dedupe of a sorted list):

```python
def dedupe(a):
    w = 0
    for x in a:
        if w == 0 or x != a[w - 1]:
            a[w] = x
            w += 1
    return w                     # new length
```

Variable window, longest valid. Grow `right` every step, shrink `left` while the window is invalid:

```python
def longest_unique(s):           # longest substring without repeated characters
    last = {}                    # char -> most recent index
    left = best = 0
    for right, c in enumerate(s):
        if c in last and last[c] >= left:
            left = last[c] + 1
        last[c] = right
        best = max(best, right - left + 1)
    return best
```

Variable window, shortest valid. Record the answer inside the shrink loop:

```python
def min_len(a, target):          # shortest subarray with sum >= target, all values positive
    left = total = 0
    best = float('inf')
    for right, x in enumerate(a):
        total += x
        while total >= target:
            best = min(best, right - left + 1)
            total -= a[left]
            left += 1
    return 0 if best == float('inf') else best
```

Fixed window of size k:

```python
def max_sum_k(a, k):
    cur = sum(a[:k])
    best = cur
    for i in range(k, len(a)):
        cur += a[i] - a[i - k]   # add the new item, drop the old one
        best = max(best, cur)
    return best
```

Spot it by the clues:

- Two pointers: the input is sorted (or order does not matter, so you can sort it), you want a pair or triple, or the problem is a palindrome check, a merge of two sorted lists, or a partition. For 3Sum, sort, fix one item, then run two pointers on the rest.
- Sliding window: the problem says contiguous subarray or substring, with "longest", "shortest", "at most k distinct" or "no repeats".
- The window must get worse as it grows (or better as it shrinks), which holds for positive numbers and character counts. If the array has negative numbers, use prefix sums instead.
- For a window with counts (anagram, at most k distinct), keep a `Counter` or dict and update it as items enter and leave.

## Prefix sums and hash maps

A prefix sum answers any range-sum query in O(1) after one O(n) pass. A hash map remembers what you have already seen so you can look up the partner of the current item instead of scanning for it.

```python
prefix = [0]
for x in a:
    prefix.append(prefix[-1] + x)
# sum of a[i:j] == prefix[j] - prefix[i]
```

Count subarrays with sum k. This works with negative numbers, which is where a sliding window fails. The idea: a subarray ending here sums to k when some earlier prefix equals `total - k`.

```python
def subarray_sum(a, k):
    seen = {0: 1}                # prefix value -> how many times seen
    total = count = 0
    for x in a:
        total += x
        count += seen.get(total - k, 0)
        seen[total] = seen.get(total, 0) + 1
    return count
```

Two sum on unsorted input, one pass:

```python
def two_sum(a, target):
    idx = {}                     # value -> index
    for i, x in enumerate(a):
        if target - x in idx:
            return [idx[target - x], i]
        idx[x] = i
    return []
```

Grouping by a canonical key, for example anagrams:

```python
from collections import defaultdict
groups = defaultdict(list)
for w in words:
    groups[''.join(sorted(w))].append(w)
```

Spot it by the clues:

- "Subarray sum equals k" or "divisible by k" (key the map on `total % k`).
- Many range-sum queries on a fixed array.
- "Find two numbers", "count pairs", "does a duplicate exist", "first repeated character".
- "Longest subarray with equal 0s and 1s": turn each 0 into -1, then store the first index where each prefix value appears and look for the same value again.
- Any time your first idea is a nested loop that searches for a matching item, ask whether a dict or set can answer the search in O(1).

## BFS

Breadth-first search visits nodes in order of distance from the start, so the first time it reaches a node is by a shortest path (when every edge costs 1). It runs in O(V + E) with a queue.

Graph given as an adjacency list:

```python
from collections import deque

def bfs(graph, src):
    dist = {src: 0}
    q = deque([src])
    while q:
        u = q.popleft()
        for v in graph[u]:
            if v not in dist:           # dist doubles as the visited set
                dist[v] = dist[u] + 1
                q.append(v)
    return dist
```

Shortest path on a grid (0 is open, 1 is a wall):

```python
def shortest_path(grid, start, goal):   # start and goal are (r, c)
    rows, cols = len(grid), len(grid[0])
    q = deque([(start[0], start[1], 0)])
    seen = {start}
    while q:
        r, c, d = q.popleft()
        if (r, c) == goal:
            return d
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 0 and (nr, nc) not in seen:
                seen.add((nr, nc))      # mark when you push, not when you pop
                q.append((nr, nc, d + 1))
    return -1
```

Level by level, when the answer depends on which layer a node is in (tree level order, "minutes until everything spreads"):

```python
def level_order(root):
    out = []
    q = deque([root] if root else [])
    while q:
        level = []
        for _ in range(len(q)):         # exactly the nodes of the current layer
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        out.append(level)
    return out
```

For several starting points at once (rotting oranges, distance to the nearest 0), put all sources in the queue at distance 0 before the loop starts. This is called multi-source BFS and it still runs in one pass.

Spot it by the clues:

- "Minimum number of steps, moves, turns, flips or transformations".
- "Shortest path" where every move costs the same, on a grid, a maze or a graph.
- "Nearest", "closest", or something that spreads outward each minute.
- Level or layer language: "level order", "how many rounds".
- The nodes are not given but implied: word ladder (words are nodes, one-letter changes are edges), puzzles and lock combinations (a board or a code is the node). Put everything that defines a state in the visited key, for example `(r, c, keys_held)`.

If edges have different non-negative weights, use Dijkstra instead (see Heaps).

## DFS and trees

Depth-first search goes as deep as it can before backing up. Use it to explore everything reachable, count regions, or compute a property of a tree from its children. O(V + E).

Iterative version for a graph, with an explicit stack (safe from recursion limits):

```python
def reachable(graph, src):
    seen = {src}
    stack = [src]
    while stack:
        u = stack.pop()
        for v in graph[u]:
            if v not in seen:
                seen.add(v)
                stack.append(v)
    return seen
```

Grid flood fill, here counting islands of `'1'` cells. Overwriting a cell marks it visited, so no separate set is needed:

```python
def num_islands(grid):
    rows, cols = len(grid), len(grid[0])

    def dfs(r, c):
        if not (0 <= r < rows and 0 <= c < cols) or grid[r][c] != '1':
            return
        grid[r][c] = '0'
        dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1)

    count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                dfs(r, c)
                count += 1
    return count
```

Trees: write the recursion as "what do I return for this node, given what my children return". The base case is the empty node.

```python
def max_depth(node):
    if not node:
        return 0
    return 1 + max(max_depth(node.left), max_depth(node.right))

def diameter(root):                      # longest path, counted in edges
    best = 0
    def height(node):
        nonlocal best
        if not node:
            return 0
        l, r = height(node.left), height(node.right)
        best = max(best, l + r)          # path that bends at this node
        return 1 + max(l, r)             # what the parent can extend
    height(root)
    return best
```

Cycle detection in a directed graph uses three states: unseen, on the current path, and finished. Meeting a node that is on the current path means a cycle.

```python
def has_cycle(graph, n):
    state = [0] * n                      # 0 unseen, 1 on path, 2 done

    def dfs(u):
        state[u] = 1
        for v in graph[u]:
            if state[v] == 1:
                return True
            if state[v] == 0 and dfs(v):
                return True
        state[u] = 2
        return False

    return any(state[u] == 0 and dfs(u) for u in range(n))
```

In-order traversal of a binary search tree visits values in sorted order. Pre-order handles a node before its children (copying or serializing a tree), and post-order handles it after (deleting, or computing from children).

Spot it by the clues:

- "Number of islands", "connected components", "regions", "fill", "is there a path from A to B".
- A tree question about depth, balance, diameter, path sums, ancestors, or validating a BST. Solve it with a recursive function that returns a value.
- "Detect a cycle" or "can all courses be finished".
- You need every path or every arrangement (that is backtracking, covered next).

BFS or DFS: if the problem needs the shortest number of steps, use BFS. If it only needs reachability, grouping or tree structure, DFS is shorter to write. Recursive DFS deeper than about 1000 levels raises a recursion error, so raise the limit or use the stack version.

## Backtracking

Backtracking builds an answer one choice at a time and undoes the choice after exploring it. Every version follows the same three steps: choose, explore, unchoose.

Subsets. Every partial path is itself a valid subset, so record at the top of each call:

```python
def subsets(nums):
    out, path = [], []

    def go(start):
        out.append(path[:])              # save a copy, not the list you keep mutating
        for i in range(start, len(nums)):
            path.append(nums[i])         # choose
            go(i + 1)                    # explore
            path.pop()                   # unchoose

    go(0)
    return out
```

Permutations. Order matters, so any unused item can go next:

```python
def permutations(nums):
    out, path = [], []
    used = [False] * len(nums)

    def go():
        if len(path) == len(nums):
            out.append(path[:])
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            used[i] = True
            path.append(nums[i])
            go()
            path.pop()
            used[i] = False

    go()
    return out
```

Combination sum with pruning. Candidates can be reused, and sorting lets you stop a loop early:

```python
def combination_sum(cands, target):
    cands.sort()
    out, path = [], []

    def go(start, remaining):
        if remaining == 0:
            out.append(path[:])
            return
        for i in range(start, len(cands)):
            if cands[i] > remaining:
                break                    # sorted, so every later candidate is too big as well
            path.append(cands[i])
            go(i, remaining - cands[i])  # i, not i + 1, because reuse is allowed
            path.pop()

    go(0, target)
    return out
```

When the input has duplicates and you must not repeat results, sort first and skip a value that equals the previous one at the same depth: `if i > start and nums[i] == nums[i - 1]: continue`.

Spot it by the clues:

- "Return all", "list every", "generate all" subsets, permutations, combinations, partitions or valid arrangements.
- Small limits: n up to about 15 to 20. The cost is 2^n for subsets and n! for permutations, which only works because n is small.
- Constraint puzzles: N-Queens, Sudoku, word search on a grid, generate valid parentheses.
- If the problem only asks how many ways or the best value and n is large, the same recursion probably repeats work. Add memoization and it becomes DP.

Three habits that prevent bugs: save `path[:]`, always undo what you did, and prune as early as a branch is known to fail.

## Dynamic programming

DP applies when a problem breaks into smaller versions of itself and the same smaller versions come up again and again. Store each answer once instead of recomputing it.

Five questions define any DP:

1. What is the state (the few numbers that describe a subproblem)?
2. What does `dp[state]` mean in words?
3. How is it built from smaller states (the transition)?
4. What are the base cases?
5. In what order do you fill it, and where is the final answer?

Top-down is the brute-force recursion plus a cache. It is the easiest way to start:

```python
from functools import cache

def climb(n):                      # ways to climb n stairs taking 1 or 2 steps
    @cache
    def go(i):
        if i <= 1:
            return 1
        return go(i - 1) + go(i - 2)
    return go(n)
```

Bottom-up fills a table from the base cases, which avoids recursion depth issues:

```python
def coin_change(coins, amount):    # fewest coins to make amount, -1 if impossible
    INF = float('inf')
    dp = [0] + [INF] * amount      # dp[a] = fewest coins that sum to a
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return dp[amount] if dp[amount] != INF else -1

def rob(nums):                     # best sum with no two adjacent items
    take = skip = 0
    for x in nums:
        take, skip = skip + x, max(take, skip)
    return max(take, skip)
```

Two sequences, a 2D table:

```python
def lcs(a, b):                     # length of the longest common subsequence
    n, m = len(a), len(b)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[n][m]
```

0/1 knapsack (each item used once). The loop runs backwards so an item cannot be counted twice:

```python
def knapsack(weights, values, cap):
    dp = [0] * (cap + 1)
    for w, v in zip(weights, values):
        for c in range(cap, w - 1, -1):
            dp[c] = max(dp[c], dp[c - w] + v)
    return dp[cap]
```

Longest increasing subsequence in O(n log n):

```python
from bisect import bisect_left

def lis(a):
    tails = []                     # tails[k] = smallest tail of an increasing run of length k + 1
    for x in a:
        i = bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)
```

The common families and their states:

| Family | State | Typical problems |
| --- | --- | --- |
| Linear | `dp[i]` from `dp[i - 1]`, `dp[i - 2]` | Climbing stairs, house robber, decode ways |
| Unbounded knapsack | `dp[amount]` | Coin change, number of ways to make a sum |
| 0/1 knapsack | `dp[capacity]`, loop backwards | Subset sum, partition equal subset |
| Two sequences | `dp[i][j]` over prefixes | Longest common subsequence, edit distance |
| Grid | `dp[r][c]` from the cell above and to the left | Unique paths, minimum path sum |
| Subsequence | `dp[i]` for a run ending at i | Longest increasing subsequence |
| Substring or interval | `dp[i][j]` for the range i to j | Palindromic substrings, burst balloons |

Spot it by the clues:

- "Count the number of ways", "minimum cost", "maximum profit", "is it possible to make exactly", "longest subsequence".
- At each step you make a choice (take or skip, go left or down, use this coin or not).
- Your recursive brute force calls the same arguments more than once.
- The limits are about 1,000 to 5,000 for a 2D table or up to about 10^6 for a 1D table.

How to start: write the plain recursion, find the arguments that change, and add `@cache`. If it passes, you are done. If the table is large, convert it to bottom-up and keep only the previous row when `dp[i]` depends only on `dp[i - 1]`.

DP or greedy: if you can find a small counterexample where the locally best choice fails, it is DP. For coin change with coins 1, 3, 4 and amount 6, greedy takes 4 + 1 + 1 (3 coins) but the best is 3 + 3 (2 coins).

## Heaps, top-k and Dijkstra

A heap gives you the smallest (or largest) item so far in O(log n) per insert or removal. Reach for it when you repeatedly need "the best remaining item" while items keep arriving.

Kth largest. Keep a min-heap of size k, and the root is the answer:

```python
import heapq

def kth_largest(a, k):
    h = []
    for x in a:
        heapq.heappush(h, x)
        if len(h) > k:
            heapq.heappop(h)         # drop the smallest of the k + 1
    return h[0]
```

For the top k most frequent items, `Counter(a).most_common(k)` does the work. For the k smallest, use `heapq.nsmallest(k, a)`.

Merge k sorted lists. The heap holds one candidate from each list:

```python
def merge_k(lists):
    h = [(l[0], i, 0) for i, l in enumerate(lists) if l]   # (value, list index, position)
    heapq.heapify(h)
    out = []
    while h:
        val, i, j = heapq.heappop(h)
        out.append(val)
        if j + 1 < len(lists[i]):
            heapq.heappush(h, (lists[i][j + 1], i, j + 1))
    return out
```

Running median with two heaps. `lo` holds the smaller half as a max-heap (negated values) and `hi` holds the larger half as a min-heap. Keep `lo` the same size as `hi` or one bigger:

```python
class MedianFinder:
    def __init__(self):
        self.lo, self.hi = [], []

    def add(self, x):
        heapq.heappush(self.lo, -x)
        heapq.heappush(self.hi, -heapq.heappop(self.lo))   # move the largest of lo to hi
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def median(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

Dijkstra finds shortest paths when edges have non-negative weights. It is BFS with a heap in place of the queue:

```python
def dijkstra(graph, src):             # graph[u] = [(v, weight), ...]
    INF = float('inf')
    dist = {src: 0}
    h = [(0, src)]
    while h:
        d, u = heapq.heappop(h)
        if d > dist.get(u, INF):
            continue                  # stale entry, a shorter path was already found
        for v, w in graph[u]:
            nd = d + w
            if nd < dist.get(v, INF):
                dist[v] = nd
                heapq.heappush(h, (nd, v))
    return dist
```

Spot it by the clues:

- "k largest", "k smallest", "kth", "top k frequent", "k closest points".
- "Median of a stream", or items arriving while you must answer queries.
- "Merge k sorted lists or streams".
- "Always process the cheapest, earliest or largest remaining": task scheduling, meeting rooms (a heap of end times), connecting ropes at minimum cost.
- A shortest path with weights on edges (Dijkstra). With negative weights it fails, so use Bellman-Ford.

When you need the kth item once, on a list that is already in memory, `sorted(a)[-k]` is simpler and fast enough. The heap earns its place when k is small compared to n or the data is a stream.

## Topological sort and union-find

Topological sort orders the nodes of a directed graph so every edge points forward. Kahn's algorithm repeatedly takes nodes with no remaining prerequisites:

```python
from collections import deque

def topo_order(n, edges):             # edge (a, b) means a must come before b
    graph = [[] for _ in range(n)]
    indeg = [0] * n
    for a, b in edges:
        graph[a].append(b)
        indeg[b] += 1
    q = deque(i for i in range(n) if indeg[i] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in graph[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return order if len(order) == n else []   # fewer than n nodes means there is a cycle
```

Watch the direction. A course list like `[course, prereq]` becomes the edge `(prereq, course)`.

Union-find (disjoint set union) tracks which items belong to the same group. Merging two groups and asking whether two items are in the same group are both close to O(1):

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.groups = n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]   # path halving keeps trees flat
            x = self.parent[x]
        return x

    def union(self, a, b):             # returns False if a and b were already together
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        self.groups -= 1
        return True
```

Spot topological sort by the clues:

- "Prerequisites", "dependencies", "must happen before", "build order", "course schedule".
- "Can all tasks be finished?" means "is there a cycle?", and the answer is `len(order) == n`.
- Ordering letters from a list of sorted words (alien dictionary).

Spot union-find by the clues:

- "Number of connected components" or "provinces" when edges are given up front or arrive one by one.
- "Are x and y connected?" asked many times.
- "Redundant connection": the first edge whose `union` returns False closes a cycle.
- Merging accounts, friend groups or equivalent items.
- Minimum spanning tree (Kruskal): sort edges by weight and add each one whose `union` succeeds.

DFS can also count components. Union-find is the better fit when the groups change over time or you ask about connectivity repeatedly.

## Monotonic stack

A monotonic stack keeps its items in sorted order. Each new item pops everything that breaks the order, and those popped items have just found their answer. Every item is pushed and popped once, so the pass is O(n).

Next greater element, as an index-based stack. The stack holds items still waiting, from largest at the bottom to smallest at the top:

```python
def next_greater(a):
    res = [-1] * len(a)
    stack = []                        # indexes whose answer is not known yet
    for i, x in enumerate(a):
        while stack and a[stack[-1]] < x:
            res[stack.pop()] = x      # x is the next greater value for that index
        stack.append(i)
    return res
```

For "how many days until a warmer day", store `i - j` in place of `x`. For next smaller, flip the comparison.

Largest rectangle in a histogram. When a bar is popped, the stack tells you how far left the rectangle can reach and the current index tells you how far right:

```python
def largest_rectangle(heights):
    stack = []                        # indexes of bars with increasing heights
    best = 0
    for i, h in enumerate(heights + [0]):      # the 0 at the end flushes the stack
        while stack and heights[stack[-1]] >= h:
            height = heights[stack.pop()]
            left = stack[-1] + 1 if stack else 0
            best = max(best, height * (i - left))
        stack.append(i)
    return best
```

A plain stack (not monotonic) handles matching and nesting:

```python
def valid_brackets(s):
    pairs = {')': '(', ']': '[', '}': '{'}
    stack = []
    for c in s:
        if c in pairs:
            if not stack or stack.pop() != pairs[c]:
                return False
        else:
            stack.append(c)
    return not stack
```

Spot it by the clues:

- "Next greater element", "next smaller element", "days until a warmer temperature", "stock span".
- "Largest rectangle" in a histogram or a binary matrix.
- "Remove k digits to make the smallest number".
- For each item you want the nearest item to the left or right that is larger or smaller.
- Brackets, nested structures, or "undo the most recent unmatched thing" (plain stack).

## Intervals and greedy

Most interval problems start with a sort. Sort by start to merge or sweep, and sort by end to pick as many non-overlapping intervals as possible.

Merge overlapping intervals (each interval is a list `[start, end]`):

```python
def merge(intervals):
    intervals.sort()
    out = []
    for s, e in intervals:
        if out and s <= out[-1][1]:
            out[-1][1] = max(out[-1][1], e)    # overlaps the last one, extend it
        else:
            out.append([s, e])
    return out
```

Minimum meeting rooms. The heap holds the end time of each room in use. If the earliest-ending room is free by the next start, reuse it:

```python
import heapq

def min_rooms(intervals):
    intervals.sort()
    ends = []
    for s, e in intervals:
        if ends and ends[0] <= s:
            heapq.heapreplace(ends, e)         # reuse a room
        else:
            heapq.heappush(ends, e)            # open a new room
    return len(ends)
```

Maximum number of non-overlapping intervals. Always keep the one that ends first. The minimum number to remove is `len(intervals)` minus this count:

```python
def max_non_overlapping(intervals):
    intervals.sort(key=lambda x: x[1])
    count, last_end = 0, float('-inf')
    for s, e in intervals:
        if s >= last_end:
            count += 1
            last_end = e
    return count
```

A greedy example with no sort, tracking the farthest index reachable (jump game):

```python
def can_jump(nums):
    far = 0
    for i, x in enumerate(nums):
        if i > far:
            return False
        far = max(far, i + x)
    return True
```

Spot intervals by the clues: "overlapping", "merge", "insert an interval", "meeting rooms", "minimum number of arrows or platforms", anything with start and end times.

Spot greedy by the clues:

- "Minimum number of" or "maximum number of" with a natural rule such as "take the earliest finishing", "take the largest first" or "extend as far as possible".
- The input is often sorted first, or processed with a heap.
- You try a few small examples and cannot break the rule.

To test a greedy idea, search for a small counterexample (3 to 5 items). If one exists, switch to DP.

## Linked list tricks

A node has a `val` and a `next`, as in `ListNode(val=0, next=None)`. Three tricks cover most problems.

Dummy head. A placeholder node before the real head removes special cases when the head might change (merging, removing nodes):

```python
def merge_two(a, b):                  # merge two sorted lists
    dummy = tail = ListNode()
    while a and b:
        if a.val <= b.val:
            tail.next, a = a, a.next
        else:
            tail.next, b = b, b.next
        tail = tail.next
    tail.next = a or b
    return dummy.next
```

Fast and slow pointers. Slow moves one step and fast moves two. When fast reaches the end, slow is at the middle. If the list has a cycle, they meet:

```python
def middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow

def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False
```

Reverse in place:

```python
def reverse(head):
    prev = None
    while head:
        head.next, prev, head = prev, head, head.next
    return prev
```

For "remove the nth node from the end", start `fast` n steps ahead of `slow` (with a dummy head), then move both until `fast` reaches the end. For "is it a palindrome", find the middle, reverse the second half, and compare.

Spot it by the clues: a list of `ListNode`, "in place", "O(1) extra space", "cycle", "middle", "nth from the end", "merge", "reverse". Draw three nodes and the arrows on paper before you write the pointer updates, and update them in an order that never loses a reference.

## OA tips

A routine for any problem:

1. Work one small example by hand and write down the input and expected output.
2. Read the constraints and decide the target complexity (see the second table in Spotting the pattern).
3. Write the brute force in your head, or in code if it is short. It often passes some tests, and many OAs give partial credit.
4. Find the repeated work in the brute force. A repeated lookup suggests a hash map, a repeated range sum suggests prefix sums, and repeated subproblems suggest DP.
5. Match the clues to a technique with the first table, then adapt the template.
6. Code it, run the examples, then run the edge cases.

Edge cases to try every time:

- Empty input, a single item, two items.
- All items equal, duplicates, already sorted, sorted in reverse.
- Negative numbers, zero, and the largest value the constraints allow.
- A graph with disconnected parts, self loops, or a cycle.
- A grid with one row, one column, or no valid path.

When the answer is wrong:

- Compare against a slow brute force on random small inputs. A 10-line brute force finds more bugs than rereading your code.
- Check off-by-one errors first: `range` end, slice end, `lo <= hi` versus `lo < hi`.
- Check that you did not change the input or a shared list (aliasing, a missing `path[:]` copy).
- In BFS and DFS, check that visited is marked at the right moment and that the start node is marked.

When the answer is right but too slow:

- Look for `x in list`, `list.pop(0)`, `list.insert(0, x)`, or slicing inside a loop. Each one hides an O(n) cost.
- Replace repeated string concatenation with a list and `''.join`.
- Add `@cache` to a recursion that repeats arguments.
- Read many lines with `sys.stdin` and print once with `'\n'.join(...)`.

Split the timer across problems when you start, and move on if one problem has eaten much more than its share. A working brute force on every problem usually beats one perfect solution and two blanks.
