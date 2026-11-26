# Python cheat sheet for big tech OAs

Nov 25, 2026 · @Lucas

## Syntax basics

```python
x, y = 3, 4                    # multiple assignment
x += 1                         # no ++ or --
if x is None: ...              # use `is` for None, not ==

if x > 0 and y > 0: ...
elif x < 0 or not y: ...
else: ...

label = 'pos' if x > 0 else 'neg'   # ternary
0 < x < 10                          # chained comparison
```

Loops:

```python
for i in range(n): ...               # 0 .. n-1
for i in range(a, b): ...            # a .. b-1
for i in range(n - 1, -1, -1): ...   # n-1 down to 0
for i, v in enumerate(arr): ...      # index and value
for p, q in zip(xs, ys): ...         # stops at the shorter one
for v in reversed(arr): ...
while cond: ...                      # break and continue work as usual
```

Falsy values are `0`, `''`, `[]`, `()`, `{}`, `set()` and `None`. Use `if not arr:` to test for empty. `map`, `filter`, `zip`, `reversed` and `enumerate` return iterators, so wrap them in `list()` if you need to index or print them.

## Numbers and math

Python ints have arbitrary precision, so there is no overflow.

```python
7 // 2            # 3      floor division
-7 // 2           # -4     floors toward negative infinity
-7 % 3            # 2      result takes the sign of the divisor
divmod(7, 2)      # (3, 1) quotient and remainder
7 / 2             # 3.5    / always returns a float
int(-3.7)         # -3     int() truncates toward zero
2 ** 10           # 1024
-(-a // b)        # ceiling division with ints

MOD = 10**9 + 7   # write it with ints, not 1e9 + 7 (that is a float)
INF = float('inf')
```

```python
import math
math.gcd(12, 18)              # 6
math.isqrt(17)                # 4      exact integer square root
math.sqrt(16)                 # 4.0
math.ceil(2.1); math.floor(2.9)   # 3, 2
math.comb(5, 2)               # 10     n choose k
math.isclose(0.1 + 0.2, 0.3)  # True; never compare floats with ==
```

Bit operations:

```python
a & b, a | b, a ^ b, a << k, a >> k
x & 1                 # 1 if x is odd
x & (x - 1)           # clears the lowest set bit; 0 means power of two (for x > 0)
(x >> k) & 1          # read bit k
x | (1 << k)          # set bit k
x & ~(1 << k)         # clear bit k
x.bit_count()         # number of 1 bits, Python 3.10+
bin(5)                # '0b101'
```

## Strings

Strings are immutable. To change one, convert to a list, edit, and join. Build long strings by appending to a list and calling `''.join(parts)` once, not with repeated `+=`.

```python
s = 'hello'
s[0], s[-1]          # 'h', 'o'
s[1:4]               # 'ell'    slice is [start, stop), stop excluded
s[::-1]              # 'olleh'  reverse

chars = list(s)
chars[0] = 'j'
''.join(chars)       # 'jello'
```

```python
s.split()                   # split on whitespace, no empty strings
'a,b,c'.split(',')          # ['a', 'b', 'c']
','.join(['a', 'b'])         # 'a,b'   items must all be str
s.strip()
s.lower(); s.upper()
s.startswith('he'); s.endswith('lo')
s.find('l')                 # 2, or -1 if missing
s.count('l')                # 2
s.replace('l', 'L')         # returns a new string
'ab' in 'cabd'              # True, substring test
s.isdigit(); s.isalpha(); s.isalnum()
```

Characters, conversion and formatting:

```python
ord('a')                # 97
chr(97)                 # 'a'
ord(c) - ord('a')       # 0..25 index for a lowercase letter
chr(ord('a') + i)       # letter from index
str(123); int('123'); list('abc')
''.join(sorted(s))      # sorted letters, a common anagram key
f'{x} {r:.2f} {z:05d}'  # 2 decimals; zero-pad to width 5
```

## Lists and tuples

A list is a dynamic array: O(1) indexing and append, O(n) insert and delete near the front.

```python
a = [0] * n                # n zeros
a.append(x)                # O(1) amortized
a.extend([4, 5])
a.insert(i, x)             # O(n)
a.pop()                    # remove and return the last item, O(1)
a.pop(i)                   # O(n)
a.remove(x)                # first match, ValueError if missing
del a[i]
a.index(x)                 # ValueError if missing
a.reverse()                # in place, returns None
a.sort()                   # in place, returns None
x in a                     # O(n)
a[-1]                      # last item

a[1:3]                     # items 1 and 2; slices copy and never raise IndexError
a[:k]; a[k:]; a[::-1]
a[:]                       # shallow copy (also a.copy())
```

2D grids:

```python
grid = [[0] * cols for _ in range(rows)]   # correct
bad  = [[0] * cols] * rows                 # WRONG: every row is the same list object

rows, cols = len(grid), len(grid[0])
dirs = [(0, 1), (1, 0), (0, -1), (-1, 0)]  # right, down, left, up
for dr, dc in dirs:
    nr, nc = r + dr, c + dc
    if 0 <= nr < rows and 0 <= nc < cols:
        ...

list(zip(*grid))                           # transpose (rows become tuples)
```

Tuples are immutable and hashable, so they work as dict keys and set members. Lists do not: convert with `tuple(lst)`. Tuples compare element by element, which is why `(dist, node)` works as a heap item or sort key.

```python
a, b = b, a                    # swap
x, y, z = [1, 2, 3]            # unpack any iterable
first, *rest = [1, 2, 3, 4]    # first = 1, rest = [2, 3, 4]
seen.add((r, c))               # visited cell
```

## Dicts and sets

Both are hash tables: O(1) average lookup, insert and delete. Keys and set items must be hashable (int, str, tuple), not list, set or dict. Dicts keep insertion order.

```python
d = {}                        # empty dict; an empty set is set(), not {}
d = {'a': 1, 'b': 2}
d['c'] = 3                    # insert or overwrite
d['x']                        # KeyError if missing
d.get('x', 0)                 # 0 if missing
d.setdefault('x', [])         # returns d['x'], inserting [] first if absent
d.pop('a', None)              # remove; no error if missing
del d['b']
'a' in d                      # key test
for k, v in d.items(): ...    # also d.keys() and d.values()
{k: v for k, v in pairs}      # dict comprehension
```

Counting, grouping, and best key by value:

```python
cnt = {}
for x in arr:
    cnt[x] = cnt.get(x, 0) + 1

groups = {}
for x in arr:
    groups.setdefault(key(x), []).append(x)

max(d, key=d.get)                          # key with the largest value
sorted(d.items(), key=lambda kv: kv[1])    # pairs sorted by value
```

Adding or removing keys while looping over a dict raises `RuntimeError`. Loop over `list(d)` if you need to change it.

Sets:

```python
s = set()                  # empty set
s = set(arr)               # dedupe
s.add(x)
s.discard(x)               # no error if missing (s.remove raises KeyError)
x in s
a | b; a & b; a - b        # union, intersection, difference
len(set(arr)) != len(arr)  # True if arr has duplicates
```

Sets are unordered, so you cannot index them. For repeated lookups, convert a list to a set once, because `x in list` scans the whole list each time.

## The collections module

```python
from collections import deque, Counter, defaultdict
```

Deque: O(1) at both ends. Use it for queues (BFS). `list.pop(0)` is O(n), so never use a list as a queue.

```python
dq = deque([1, 2, 3])
dq.append(x)                 # right end
dq.appendleft(x)             # left end
dq.pop()                     # from right
dq.popleft()                 # from left
dq[0]; dq[-1]                # peek at the ends

q = deque([start])           # BFS skeleton
while q:
    node = q.popleft()
    ...
```

Counter is a dict of counts. Missing keys read as 0 and are not inserted.

```python
c = Counter(arr)             # or Counter('aabbbc')
c['a']                       # count of 'a', 0 if missing
c['a'] += 1
c.most_common(2)             # [('b', 3), ('a', 2)]
Counter(s1) == Counter(s2)   # anagram test
```

Defaultdict calls a factory to create a missing value on first access:

```python
dd = defaultdict(int)        # missing -> 0
dd[x] += 1
graph = defaultdict(list)    # missing -> []
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)
```

Reading `dd[k]` inserts `k` if it is missing. Test with `k in dd` when you do not want that.

## heapq

`heapq` is a min-heap stored in a plain list. There is no max-heap, so negate values to get one.

```python
import heapq
h = []
heapq.heappush(h, 5)         # O(log n)
heapq.heappop(h)             # removes and returns the smallest, O(log n)
h[0]                         # peek at the smallest, O(1)
heapq.heapify(arr)           # turn a list into a heap in place, O(n)
heapq.nlargest(3, arr)       # O(n log k)
heapq.nsmallest(3, arr)
```

Max-heap by negating:

```python
heapq.heappush(h, -x)
largest = -heapq.heappop(h)
```

Tuples order by the first item, then the next on ties. Use them for priorities:

```python
heapq.heappush(h, (dist, node))
heapq.heappush(h, (-count, word))     # highest count first, then alphabetical word
d, node = heapq.heappop(h)
```

If priorities can tie and the second items cannot be compared (dicts, custom objects), add a unique counter in between, such as `(prio, next(tie), obj)` with `tie = itertools.count()`.

## Sorting and bisect

Sorting is O(n log n) and stable: items with equal keys keep their original order.

```python
a.sort()                                     # in place, returns None
b = sorted(a)                                # new list; accepts any iterable
sorted(a, reverse=True)
sorted(words, key=len)
sorted(words, key=lambda w: (len(w), w))     # length, then alphabetical
sorted(pairs, key=lambda p: (-p[0], p[1]))   # first desc, second asc
sorted(range(n), key=lambda i: a[i])         # indexes ordered by value
intervals.sort()                             # lists and tuples sort by first item, then next
```

Negating a key only works for numbers. To sort strings descending, use `reverse=True`.

bisect works on an already sorted list in O(log n):

```python
from bisect import bisect_left, bisect_right
a = [1, 2, 2, 4]
bisect_left(a, 2)       # 1   first index with a[i] >= x
bisect_right(a, 2)      # 3   first index with a[i] > x
bisect_left(a, 3)       # 3   where 3 would be inserted
bisect_right(a, x) - 1  # index of the last item <= x, or -1 if none
```

Python has no built-in sorted set or tree map.

## Comprehensions, itertools and recursion

```python
[x * x for x in arr]
[x for x in arr if x % 2 == 0]                 # filter
[x if x > 0 else 0 for x in arr]               # if/else goes before the for
[(i, j) for i in range(3) for j in range(3)]   # nested: outer loop first
{x: i for i, x in enumerate(arr)}              # dict comprehension
{x % 3 for x in arr}                           # set comprehension
```

Aggregates over generators need no brackets, and `any` and `all` stop early:

```python
sum(x * x for x in arr)
sum(1 for x in arr if x > 0)         # count matches
any(x < 0 for x in arr)
all(x > 0 for x in arr)
max(arr, key=len)
max(arr, default=0)                  # avoids ValueError on an empty input
```

itertools:

```python
from itertools import accumulate, combinations, permutations, product

list(accumulate([1, 2, 3]))               # [1, 3, 6]   prefix sums
prefix = [0] + list(accumulate(arr))      # sum(arr[i:j]) == prefix[j] - prefix[i]
list(combinations([1, 2, 3], 2))          # [(1, 2), (1, 3), (2, 3)]
list(permutations([1, 2, 3], 2))          # 6 ordered pairs
list(product([0, 1], repeat=3))           # all 8 binary triples
```

The number of results grows fast (n! for full permutations), so keep n small.

Memoization and DP tables:

```python
from functools import cache

@cache                         # Python 3.9+; use lru_cache(maxsize=None) on older versions
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

dp = [0] * (n + 1)
dp = [[0] * (m + 1) for _ in range(n + 1)]
```

Cached arguments must be hashable, so pass ints, strings or tuples, not lists.

The default recursion limit is about 1000 frames. Raise it with `sys.setrecursionlimit(10**6)`, but very deep recursion (around 10^5 or more) can still crash, so use an explicit stack (`stack = [root]`, then `stack.pop()` in a loop) when depth could be that large.

## Functions and classes

```python
def f(a, b=0, *args, **kwargs): ...   # defaults, extra positionals, extra keywords
key = lambda x: x[1]

def add(x, acc=None):                 # never write acc=[] as a default
    if acc is None:
        acc = []
    acc.append(x)
    return acc
```

A nested function can read outer variables. To assign to one, declare it `nonlocal`, otherwise Python treats it as a new local and raises `UnboundLocalError`. Calling `.append` or `d[k] = v` on an outer object needs no declaration.

```python
def outer():
    best = 0
    def dfs(node):
        nonlocal best
        best = max(best, node.val)
    dfs(root)
    return best
```

Node classes that OA prompts usually give you, but which you may need to write for local tests:

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

LeetCode-style signatures carry type hints such as `nums: List[int]`. Python ignores them at runtime, and inside a `class Solution` you call other methods through `self`.

## Input and output

Many OAs are function-based: you return a value and print nothing. Stdin-based platforms (HackerRank, some custom judges) need the code below.

```python
n = int(input())                        # input() strips the newline
a = list(map(int, input().split()))     # '1 2 3' -> [1, 2, 3]
s = input().strip()

import sys
data = sys.stdin.read().split()         # all tokens at once
for line in sys.stdin:                  # loop until EOF
    ...
```

```python
print(x)
print(*arr)                             # space-separated values
print(a, b, sep=', ', end='')           # defaults are sep=' ' and end='\n'
print('\n'.join(map(str, arr)))          # one per line; faster than many print calls
```

## Time complexity of built-ins

Costs are for CPython. n is the container size and k is the size of the slice or result.

| Structure | Operation | Time |
| --- | --- | --- |
| list | `a[i]`, `append`, `pop()`, `len(a)` | O(1) |
| list | `pop(0)`, `insert`, `remove`, `x in a`, `index` | O(n) |
| list | `a[i:j]`, `a.copy()`, `a + b` | O(k) |
| list | `sort`, `sorted` | O(n log n) |
| deque | `append`, `appendleft`, `pop`, `popleft` | O(1) |
| dict, set | get, set, delete, `in` | O(1) average |
| dict, set | iterate, `copy()` | O(n) |
| heapq | `heappush`, `heappop` | O(log n) |
| heapq | `heapify` | O(n) |
| bisect | `bisect_left`, `bisect_right` | O(log n) |
| str | `s[i]`, `len(s)` | O(1) |
| str | slice, `s + t`, `split`, `replace`, `t in s` | O(n) |

A rough guide for CPython, which runs about 10^7 simple operations per second: n up to 10 allows O(n!), up to 20 allows O(2^n), up to 500 allows O(n^3), up to 5,000 allows O(n^2), up to 10^5 allows O(n log n), and 10^6 or more needs O(n). Treat these as estimates and check the problem's stated limits.

## Common gotchas

```python
# Assignment aliases, it does not copy
b = a                       # b and a are the same list
b = a[:]                    # shallow copy: inner lists are still shared
b = copy.deepcopy(a)        # full copy

# sort() returns None
a = a.sort()                # a is now None. Use a.sort() or a = sorted(a)

# Removing while iterating skips items
for x in a:
    if bad(x):
        a.remove(x)         # wrong
a = [x for x in a if not bad(x)]   # right

# Slices with k = 0
a[-k:]                      # whole list when k == 0, not empty
a[len(a) - k:]              # last k items, safe for k == 0

# Empty inputs
max([])                     # ValueError. Use max(a, default=0)

# Mutable default argument is shared between calls
def f(x, acc=[]): ...       # use acc=None and create the list inside
```

Test small inputs (empty, one item, duplicates, negatives) before submitting.
