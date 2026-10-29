# Python Cheatsheet — Competitive Programming & Interviews

A single-file reference for everything you need in coding rounds: core syntax, built-in data structures, stdlib power tools, and ready-to-use algorithm templates.

For system-design/senior-round topics — GIL, threading, multiprocessing, asyncio, decorators, generators, context managers, and the object model — see the companion file `python_advanced_cheatsheet.md`.

---

## Table of Contents

1. [Basics & Gotchas](#1-basics--gotchas)
2. [Strings](#2-strings)
3. [Lists](#3-lists)
4. [Tuples](#4-tuples)
5. [Dictionaries](#5-dictionaries)
6. [Sets](#6-sets)
7. [Comprehensions & Generators](#7-comprehensions--generators)
8. [Loops, Conditionals & Functions](#8-loops-conditionals--functions)
9. [Sorting Tricks](#9-sorting-tricks)
10. [collections Module](#10-collections-module)
11. [heapq — Priority Queues](#11-heapq--priority-queues)
12. [bisect — Binary Search on Sorted Lists](#12-bisect--binary-search-on-sorted-lists)
13. [itertools](#13-itertools)
14. [math, functools & More](#14-math-functools--more)
15. [Fast I/O & CP Setup](#15-fast-io--cp-setup)
16. [Two Pointers](#16-two-pointers)
17. [Sliding Window](#17-sliding-window)
18. [Binary Search Templates](#18-binary-search-templates)
19. [Prefix Sums](#19-prefix-sums)
20. [Linked Lists](#20-linked-lists)
21. [Stacks & Monotonic Stack](#21-stacks--monotonic-stack)
22. [Trees (Binary Trees & BST)](#22-trees-binary-trees--bst)
23. [Graphs — BFS / DFS / Topological Sort](#23-graphs--bfs--dfs--topological-sort)
24. [Shortest Paths (Dijkstra)](#24-shortest-paths-dijkstra)
25. [Union-Find (Disjoint Set)](#25-union-find-disjoint-set)
26. [Backtracking](#26-backtracking)
27. [Dynamic Programming](#27-dynamic-programming)
28. [Trie](#28-trie)
29. [Intervals](#29-intervals)
30. [Matrix / Grid Tricks](#30-matrix--grid-tricks)
31. [Bit Manipulation](#31-bit-manipulation)
32. [Number Theory](#32-number-theory)
33. [OOP & Interview Extras](#33-oop--interview-extras)
34. [Big-O of Python Operations](#34-big-o-of-python-operations)

---

## 1. Basics & Gotchas

```python
# Integer division vs true division
7 // 2      # 3   (floor division — rounds DOWN, so -7 // 2 == -4)
7 / 2       # 3.5 (always float)
-7 % 3      # 2   (result has sign of divisor — unlike C++/Java!)
divmod(7, 2)            # (3, 1) — quotient and remainder at once

# Truncate toward zero (C++-style division)
int(-7 / 2)             # -3

# Python ints are arbitrary precision — no overflow, ever
2 ** 100                # works fine

# Useful "infinity"
INF = float('inf')
-float('inf') < -10**18 # True

# Swapping — no temp variable
a, b = 1, 2
a, b = b, a

# Chained comparison
x = 5
if 0 <= x < 10: pass    # same as 0 <= x and x < 10

# Multiple assignment & unpacking
x, y, z = 1, 2, 3
first, *rest = [1, 2, 3, 4]      # first=1, rest=[2,3,4]
*init, last = [1, 2, 3, 4]       # init=[1,2,3], last=4

# Ternary
ans = "yes" if x > 0 else "no"

# Walrus operator (3.8+) — assign inside an expression
nums = [1, 2, 3]
if (n := len(nums)) > 2:
    print(n)

# Falsy values: 0, 0.0, '', [], {}, set(), None, False
if not []: pass          # True — empty list is falsy

# GOTCHA: mutable default arguments are shared across calls
def bad(lst=[]): lst.append(1); return lst       # bug!
def good(lst=None):
    if lst is None: lst = []
    return lst

# GOTCHA: is vs ==
# `==` compares values, `is` compares identity. Use `is` only for None.
x = None
if x is None: pass

# Rounding: Python uses banker's rounding
round(2.5)   # 2 (not 3!)
round(3.5)   # 4
import math
math.floor(2.7), math.ceil(2.1)   # 2, 3
# Ceil division without math:
def ceil_div(a, b): return -(-a // b)   # or (a + b - 1) // b for positive
```

## 2. Strings

Strings are **immutable** — build with lists + `join`, never `+=` in a loop.

```python
s = "hello world"

# Indexing / slicing (works on lists too)
s[0]        # 'h'
s[-1]       # 'd'
s[0:5]      # 'hello'   (end exclusive)
s[::-1]     # reverse:  'dlrow olleh'
s[::2]      # every 2nd char: 'hlowrd'

# Common methods
s.upper(); s.lower(); s.title(); s.capitalize()
s.strip(); s.lstrip(); s.rstrip()      # remove whitespace (or given chars)
s.split()                # ['hello', 'world'] — splits on any whitespace
"a,b,c".split(",")       # ['a', 'b', 'c']
"-".join(["a", "b"])     # 'a-b'
s.replace("world", "py") # 'hello py'
s.find("lo")             # 3 (index, or -1 if absent)
s.index("lo")            # 3 (raises ValueError if absent)
s.count("l")             # 3
s.startswith("he"); s.endswith("ld")   # True, True

# Character tests
"abc".isalpha(); "123".isdigit(); "a1".isalnum()
"abc".islower(); "ABC".isupper(); " ".isspace()

# char <-> int
ord('a')     # 97
chr(97)      # 'a'
ord('c') - ord('a')      # 2 — index into a 26-size array

# f-strings
name, score = "Alice", 95.5
f"{name}: {score:.1f}"   # 'Alice: 95.5'
f"{42:b}"                # '101010' (binary); :o octal, :x hex
f"{7:03d}"               # '007' (zero-pad)
f"{score=}"              # debug: "score=95.5"

# Build strings efficiently — O(n) not O(n^2)
parts = []
for c in "abc":
    parts.append(c.upper())
result = "".join(parts)

# Sorted chars as canonical form (anagram key)
key = "".join(sorted("listen"))     # 'eilnst'
# Or without sorting: tuple of 26 counts (O(n) per word)
```

## 3. Lists

```python
a = [1, 2, 3]

# Add / remove
a.append(4)          # O(1) — add to end
a.pop()              # O(1) — remove & return last
a.pop(0)             # O(n) — avoid! use collections.deque
a.insert(1, 99)      # O(n)
a.remove(99)         # O(n) — removes first occurrence by VALUE
del a[0]             # O(n) — remove by index
a.extend([5, 6])     # append all;  a += [5, 6] same thing

# Search / info
a = [1, 2, 3, 2]
len(a); sum(a); min(a); max(a)
a.index(2)           # first index of value (ValueError if absent)
a.count(2)
3 in a               # O(n) membership — use a set for repeated checks

# Copy (GOTCHA: b = a aliases, doesn't copy)
b = a[:]             # shallow copy;  also list(a) or a.copy()
import copy
deep = copy.deepcopy([[1, 2], [3]])   # nested structures

# Reverse / sort
a.reverse()          # in place
b = a[::-1]          # reversed copy
a.sort()             # in place, O(n log n), stable
b = sorted(a)        # sorted copy

# Initialize
zeros = [0] * 10
grid  = [[0] * cols for _ in range(rows)]   # CORRECT 2D init
# GOTCHA: [[0]*cols]*rows makes rows that ALIAS each other!

# enumerate & zip
for i, v in enumerate(a): pass          # index, value
for i, v in enumerate(a, start=1): pass
for x, y in zip([1,2], ['a','b']): pass # pairs; stops at shorter
list(zip(*[[1,2],[3,4]]))               # transpose: [(1,3),(2,4)]

# Unpack in calls
print(*a)            # print(1, 2, 3)

# any / all
any(x > 2 for x in a)    # True if any match
all(x > 0 for x in a)    # True if all match
```

## 4. Tuples

Immutable, hashable (usable as dict keys / set members), lighter than lists.

```python
t = (1, 2, 3)
single = (1,)            # note the comma!
x, y = (10, 20)          # unpacking

# As dict keys — perfect for coordinates and memo states
visited = {(0, 0), (1, 2)}
memo = {(i, j): 42 for i in range(2) for j in range(2)}

# Compare lexicographically — great for sorting composite keys
(1, 5) < (2, 0)          # True
(1, 5) < (1, 6)          # True

# namedtuple: lightweight record
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2); p.x     # 1
```

## 5. Dictionaries

Average O(1) insert / lookup / delete. Preserves insertion order (3.7+).

```python
d = {"a": 1, "b": 2}

# Access
d["a"]                   # 1 (KeyError if missing)
d.get("z")               # None (no error)
d.get("z", 0)            # 0 (default)
d.setdefault("z", []).append(1)   # insert default if missing, then use

# Modify
d["c"] = 3
d.pop("c")               # remove & return (KeyError if missing)
d.pop("zz", None)        # safe pop
del d["b"]
d.update({"x": 9, "y": 8})

# Iterate
for k in d: pass                 # keys
for k, v in d.items(): pass      # key-value pairs
for v in d.values(): pass

# Membership is on KEYS
"a" in d                 # True

# Useful patterns
counts = {}
for ch in "hello":
    counts[ch] = counts.get(ch, 0) + 1     # frequency count by hand

inverted = {v: k for k, v in {"a": 1, "b": 2}.items()}   # invert (values must be unique & hashable)

# Max key by value
best = max(counts, key=counts.get)          # most frequent char

# Merge (3.9+)
merged = {"a": 1} | {"b": 2}

# Sort dict by value
sorted(counts.items(), key=lambda kv: kv[1], reverse=True)

# dict.fromkeys — also an ordered de-dupe trick
list(dict.fromkeys([3, 1, 3, 2]))          # [3, 1, 2]
```

## 6. Sets

Average O(1) add / remove / membership. Elements must be hashable.

```python
s = {1, 2, 3}
empty = set()            # NOT {} — that's an empty dict

s.add(4)
s.discard(9)             # no error if absent
s.remove(2)              # KeyError if absent
s.pop()                  # remove arbitrary element
2 in s                   # O(1) membership — the main reason to use sets

# Set algebra
a, b = {1, 2, 3}, {2, 3, 4}
a | b        # union         {1, 2, 3, 4}
a & b        # intersection  {2, 3}
a - b        # difference    {1}
a ^ b        # symmetric diff {1, 4}
a <= b       # subset?  a >= b superset?

# De-dupe (loses order — use dict.fromkeys to keep order)
unique = set([1, 1, 2, 3])

# frozenset — immutable, hashable set (usable as dict key)
fs = frozenset([1, 2])
```

## 7. Comprehensions & Generators

```python
squares = [x * x for x in range(10)]
evens   = [x for x in range(10) if x % 2 == 0]
pairs   = [(i, j) for i in range(3) for j in range(3) if i != j]
matrix  = [[0] * 3 for _ in range(3)]
flat    = [x for row in matrix for x in row]        # flatten 2D

d = {c: ord(c) for c in "abc"}       # dict comprehension
s = {x % 3 for x in range(10)}       # set comprehension

# Conditional expression inside comprehension
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]

# Generator expression — lazy, O(1) memory
total = sum(x * x for x in range(10**6))
first_match = next((x for x in range(100) if x % 7 == 3), -1)

# Generator function
def fib():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b
```

## 8. Loops, Conditionals & Functions

```python
# range(start, stop, step) — stop is exclusive
for i in range(5): pass          # 0..4
for i in range(2, 10, 3): pass   # 2, 5, 8
for i in range(9, -1, -1): pass  # 9..0 (countdown)
for i in reversed(range(10)): pass

# while with else (else runs if loop wasn't broken)
for x in [1, 2, 3]:
    if x == 99: break
else:
    pass       # executed — no break occurred

# match statement (3.10+)
def http(status):
    match status:
        case 200: return "OK"
        case 404: return "Not Found"
        case _:   return "Unknown"

# Functions
def f(a, b=2, *args, **kwargs):
    return a + b

# Lambda
add = lambda x, y: x + y
key = lambda p: (p[0], -p[1])

# map / filter (comprehensions usually cleaner)
list(map(int, "1 2 3".split()))          # [1, 2, 3] — classic CP input parse
list(filter(None, ["a", "", "b"]))       # ['a', 'b'] — drop falsy

# Scope: closures need nonlocal to rebind outer variables
def counter():
    n = 0
    def inc():
        nonlocal n
        n += 1
        return n
    return inc
```

## 9. Sorting Tricks

`sort()`/`sorted()` are **stable** (equal elements keep relative order) — chain sorts or use tuple keys.

```python
a = [3, 1, 2]
a.sort()                          # in place
b = sorted(a, reverse=True)       # copy, descending

# Sort by key
words = ["banana", "pie", "kiwi"]
sorted(words, key=len)                     # by length
sorted(words, key=lambda w: (len(w), w))   # length, then alphabetical

# Descending on one field, ascending another (numeric): negate
pts = [(1, 5), (1, 3), (2, 1)]
sorted(pts, key=lambda p: (p[0], -p[1]))   # x asc, y desc

# Descending string field? Use two stable sorts (last key sorted first... 
# actually: sort by secondary key first, then primary)
data = [("b", 2), ("a", 1), ("b", 1)]
data.sort(key=lambda t: t[1])              # secondary
data.sort(key=lambda t: t[0], reverse=True)  # primary

# Sort dict items / by value
freq = {"a": 3, "b": 1}
sorted(freq.items(), key=lambda kv: -kv[1])

# operator.itemgetter / attrgetter — slightly faster than lambda
from operator import itemgetter
sorted(pts, key=itemgetter(0, 1))

# Custom comparator (rare — e.g., largest number from digits)
from functools import cmp_to_key
def cmp(x, y):        # return negative if x before y
    return (x > y) - (x < y)
sorted([3, 1, 2], key=cmp_to_key(cmp))

# min/max with key
max(words, key=len)               # 'banana'

# k largest / smallest
import heapq
heapq.nlargest(2, [5, 1, 8, 3])   # [8, 5]
heapq.nsmallest(2, [5, 1, 8, 3])  # [1, 3]
```

## 10. collections Module

```python
from collections import Counter, defaultdict, deque, OrderedDict

# Counter — frequency map with extras
c = Counter("mississippi")
c['s']                    # 4
c.most_common(2)          # [('i', 4), ('s', 4)] or [('s',4),('i',4)] — ties arbitrary
c.total()                 # 11 (3.10+)
Counter("aab") - Counter("ab")     # Counter({'a': 1}) — drops non-positive
Counter("aab") & Counter("abb")    # intersection: min counts
Counter([1,2,2]) == Counter([2,1,2])   # True — anagram check

# defaultdict — auto-initializing dict
g = defaultdict(list)              # adjacency list
g[0].append(1)                     # no KeyError, [] created on demand
cnt = defaultdict(int)
cnt['x'] += 1
nested = defaultdict(dict)

# deque — O(1) append/pop at BOTH ends (list.pop(0) is O(n))
dq = deque([1, 2, 3])
dq.append(4); dq.appendleft(0)
dq.pop(); dq.popleft()
dq.rotate(1)                       # rotate right
window = deque(maxlen=3)           # auto-drops from left when full

# BFS queue idiom
queue = deque([start_node]) if (start_node := 0) is not None else deque()
```

## 11. heapq — Priority Queues

Min-heap only. For max-heap, **negate values**.

```python
import heapq

h = [5, 1, 8]
heapq.heapify(h)             # O(n), in place — h[0] is now the min
heapq.heappush(h, 2)         # O(log n)
smallest = heapq.heappop(h)  # O(log n) — returns 1
h[0]                         # peek min, O(1)

# Max-heap: push negatives
mx = []
for v in [5, 1, 8]:
    heapq.heappush(mx, -v)
largest = -heapq.heappop(mx)     # 8

# Heap of tuples — sorts by first element, then second...
tasks = []
heapq.heappush(tasks, (2, "low"))
heapq.heappush(tasks, (1, "high"))
heapq.heappop(tasks)             # (1, 'high')

# Tie-breaking with unhashable/incomparable payloads: add a counter
from itertools import count
tie = count()
heapq.heappush(tasks, (1, next(tie), {"payload": True}))

# pushpop / replace (single log-n op instead of two)
heapq.heappushpop(h, 3)      # push 3 then pop min
heapq.heapreplace(h, 3)      # pop min then push 3

# Top-k pattern: keep a min-heap of size k
def top_k(nums, k):
    h = []
    for x in nums:
        heapq.heappush(h, x)
        if len(h) > k:
            heapq.heappop(h)
    return h                 # k largest elements
```

## 12. bisect — Binary Search on Sorted Lists

```python
import bisect

a = [1, 3, 3, 5, 7]

bisect.bisect_left(a, 3)     # 1 — first index where 3 could go (leftmost)
bisect.bisect_right(a, 3)    # 3 — past the last 3   (alias: bisect.bisect)
bisect.insort(a, 4)          # insert keeping sorted (O(n) due to shift)

# Count occurrences of x in sorted list
def count(a, x):
    return bisect.bisect_right(a, x) - bisect.bisect_left(a, x)

# Does x exist?
def contains(a, x):
    i = bisect.bisect_left(a, x)
    return i < len(a) and a[i] == x

# Predecessor / successor
def le(a, x):    # rightmost value <= x
    i = bisect.bisect_right(a, x)
    return a[i-1] if i else None
def gt(a, x):    # leftmost value > x
    i = bisect.bisect_right(a, x)
    return a[i] if i < len(a) else None

# key= parameter (3.10+)
pairs = [(1, 'a'), (3, 'b'), (5, 'c')]
bisect.bisect_left(pairs, 3, key=lambda p: p[0])   # 1
```

## 13. itertools

```python
from itertools import (permutations, combinations, combinations_with_replacement,
                       product, accumulate, chain, groupby, pairwise,
                       islice, cycle, repeat, starmap)

list(permutations([1, 2, 3], 2))    # all orderings: (1,2),(1,3),(2,1)...
list(combinations([1, 2, 3], 2))    # unordered picks: (1,2),(1,3),(2,3)
list(combinations_with_replacement("ab", 2))   # ('a','a'),('a','b'),('b','b')
list(product([0, 1], repeat=3))     # all binary triples — nested loops in one
list(product("ab", "xy"))           # ('a','x'),('a','y'),('b','x'),('b','y')

list(accumulate([1, 2, 3, 4]))              # prefix sums: [1, 3, 6, 10]
import operator
list(accumulate([1, 2, 3], operator.mul))   # running product: [1, 2, 6]
list(accumulate([3, 1, 4], max))            # running max: [3, 3, 4]

list(chain([1, 2], [3, 4]))          # [1, 2, 3, 4]
list(chain.from_iterable([[1], [2, 3]]))    # flatten one level

# groupby — groups CONSECUTIVE equal keys (sort first for full grouping)
s = "aaabbc"
[(k, len(list(g))) for k, g in groupby(s)]   # [('a',3),('b',2),('c',1)]

list(pairwise([1, 2, 3, 4]))         # (1,2),(2,3),(3,4)  (3.10+)
list(islice(cycle([1, 2]), 5))       # [1, 2, 1, 2, 1]
```

## 14. math, functools & More

```python
import math

math.gcd(12, 18)             # 6
math.lcm(4, 6)               # 12 (3.9+)
math.gcd(12, 18, 24)         # multiple args OK (3.9+)
math.isqrt(17)               # 4 — exact integer sqrt (no float error!)
math.comb(5, 2)              # 10 — n choose k
math.perm(5, 2)              # 20
math.factorial(5)            # 120
math.log2(8); math.log10(100); math.log(math.e)
math.inf; math.pi; math.e
math.hypot(3, 4)             # 5.0 — Euclidean distance
math.dist((0,0), (3,4))      # 5.0

from functools import lru_cache, cache, reduce, cmp_to_key, partial

@cache                       # unlimited memo (3.9+); use lru_cache(None) before
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
fib.cache_clear()            # reset between test cases

reduce(lambda x, y: x * y, [1, 2, 3, 4])       # 24

import random
random.randint(1, 10)        # inclusive both ends
random.choice([1, 2, 3])
random.shuffle([1, 2, 3])
random.sample(range(100), 5) # 5 distinct

# String constants
import string
string.ascii_lowercase       # 'abcdefghijklmnopqrstuvwxyz'
string.digits                # '0123456789'
```

## 15. Fast I/O & CP Setup

```python
import sys
input = sys.stdin.readline          # much faster than built-in input()

# Standard parsing patterns
n = int(input())
a, b = map(int, input().split())
arr = list(map(int, input().split()))
s = input().strip()                  # readline keeps '\n' — strip it!

# Read everything at once (fastest)
data = sys.stdin.buffer.read().split()
idx = 0
def rd():
    global idx
    idx += 1
    return int(data[idx - 1])

# Output: join once instead of many prints
out = []
for i in range(3):
    out.append(str(i))
sys.stdout.write("\n".join(out) + "\n")
print(*arr)                          # space-separated
print(*arr, sep="\n")                # newline-separated

# Recursion limit (Python default is ~1000 — raise it for deep DFS)
sys.setrecursionlimit(300000)
# Note: on tight limits, prefer iterative DFS — Python recursion is slow.

# Multiple test cases skeleton
def solve():
    n = int(input())
    arr = list(map(int, input().split()))
    print(sum(arr))

t = int(input())
for _ in range(t):
    solve()
```

## 16. Two Pointers

```python
# Pair with target sum in SORTED array
def two_sum_sorted(a, target):
    i, j = 0, len(a) - 1
    while i < j:
        s = a[i] + a[j]
        if s == target: return (i, j)
        if s < target: i += 1
        else: j -= 1
    return None

# Remove duplicates in place from sorted array (slow/fast pointers)
def dedupe(a):
    if not a: return 0
    w = 1
    for r in range(1, len(a)):
        if a[r] != a[w - 1]:
            a[w] = a[r]
            w += 1
    return w          # new length

# Valid palindrome
def is_pal(s):
    i, j = 0, len(s) - 1
    while i < j:
        if s[i] != s[j]: return False
        i += 1; j -= 1
    return True
# (Quick version: s == s[::-1])

# Merge two sorted arrays
def merge(a, b):
    i = j = 0; out = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]: out.append(a[i]); i += 1
        else:            out.append(b[j]); j += 1
    out.extend(a[i:]); out.extend(b[j:])
    return out
```

## 17. Sliding Window

```python
# Fixed-size window: max sum of k consecutive elements
def max_sum_k(a, k):
    window = sum(a[:k])
    best = window
    for r in range(k, len(a)):
        window += a[r] - a[r - k]     # slide: add new, drop old
        best = max(best, window)
    return best

# Variable window: longest substring without repeating characters
def longest_unique(s):
    seen = {}          # char -> last index
    left = best = 0
    for r, c in enumerate(s):
        if c in seen and seen[c] >= left:
            left = seen[c] + 1
        seen[c] = r
        best = max(best, r - left + 1)
    return best

# Variable window (shrink-while-invalid): smallest subarray with sum >= target
def min_subarray_len(target, a):
    left = total = 0
    best = float('inf')
    for r, x in enumerate(a):
        total += x
        while total >= target:
            best = min(best, r - left + 1)
            total -= a[left]
            left += 1
    return best if best != float('inf') else 0

# Monotonic deque: max of every window of size k — O(n)
from collections import deque
def window_max(a, k):
    dq, out = deque(), []          # dq holds indices, values decreasing
    for i, x in enumerate(a):
        while dq and a[dq[-1]] <= x: dq.pop()
        dq.append(i)
        if dq[0] <= i - k: dq.popleft()
        if i >= k - 1: out.append(a[dq[0]])
    return out
```

## 18. Binary Search Templates

```python
# Classic: find x in sorted array
def bsearch(a, x):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == x: return mid
        if a[mid] < x: lo = mid + 1
        else: hi = mid - 1
    return -1

# THE template to memorize: first index where predicate is True.
# Works for "find boundary" problems: [F, F, F, T, T, T] -> first T
def first_true(lo, hi, pred):
    while lo < hi:
        mid = (lo + hi) // 2
        if pred(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo            # == hi; may be original hi if no True exists

# Binary search on the ANSWER: e.g., min capacity to ship in D days
def ship_within_days(weights, days):
    def can(cap):
        d, cur = 1, 0
        for w in weights:
            if cur + w > cap:
                d += 1; cur = 0
            cur += w
        return d <= days
    return first_true(max(weights), sum(weights), can)

# Rotated sorted array search
def search_rotated(a, target):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target: return mid
        if a[lo] <= a[mid]:              # left half sorted
            if a[lo] <= target < a[mid]: hi = mid - 1
            else: lo = mid + 1
        else:                            # right half sorted
            if a[mid] < target <= a[hi]: lo = mid + 1
            else: hi = mid - 1
    return -1
```

## 19. Prefix Sums

```python
# 1D: sum of a[l..r] in O(1) after O(n) prep
a = [3, 1, 4, 1, 5]
pre = [0]
for x in a:
    pre.append(pre[-1] + x)
# sum of a[l..r] inclusive:
l, r = 1, 3
range_sum = pre[r + 1] - pre[l]        # 1+4+1 = 6

# One-liner
from itertools import accumulate
pre = [0] + list(accumulate(a))

# Count subarrays with sum == k (prefix sum + hashmap)
from collections import defaultdict
def subarray_sum(a, k):
    seen = defaultdict(int)
    seen[0] = 1
    total = ans = 0
    for x in a:
        total += x
        ans += seen[total - k]
        seen[total] += 1
    return ans

# 2D prefix sums
def build_2d(g):
    R, C = len(g), len(g[0])
    P = [[0] * (C + 1) for _ in range(R + 1)]
    for i in range(R):
        for j in range(C):
            P[i+1][j+1] = g[i][j] + P[i][j+1] + P[i+1][j] - P[i][j]
    return P
# sum of rectangle (r1,c1)..(r2,c2) inclusive:
# P[r2+1][c2+1] - P[r1][c2+1] - P[r2+1][c1] + P[r1][c1]

# Difference array: many range updates, one final read
def range_updates(n, updates):       # updates: (l, r, val)
    diff = [0] * (n + 1)
    for l, r, v in updates:
        diff[l] += v
        diff[r + 1] -= v
    out, cur = [], 0
    for i in range(n):
        cur += diff[i]
        out.append(cur)
    return out
```

## 20. Linked Lists

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Reverse — the classic
def reverse_list(head):
    prev = None
    while head:
        head.next, prev, head = prev, head, head.next
    return prev

# Find middle (slow/fast)
def middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow

# Detect cycle (Floyd)
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow, fast = slow.next, fast.next.next
        if slow is fast: return True
    return False

# Merge two sorted lists — dummy node idiom
def merge_lists(l1, l2):
    dummy = tail = ListNode()
    while l1 and l2:
        if l1.val <= l2.val: tail.next, l1 = l1, l1.next
        else:                tail.next, l2 = l2, l2.next
        tail = tail.next
    tail.next = l1 or l2
    return dummy.next

# Remove nth from end — two pointers with gap
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)
    lead = lag = dummy
    for _ in range(n): lead = lead.next
    while lead.next:
        lead, lag = lead.next, lag.next
    lag.next = lag.next.next
    return dummy.next
```

## 21. Stacks & Monotonic Stack

```python
# Plain stack = list
stack = []
stack.append(1)      # push
stack.pop()          # pop
# stack[-1] peeks (check non-empty first)

# Valid parentheses
def valid_parens(s):
    pairs = {')': '(', ']': '[', '}': '{'}
    st = []
    for c in s:
        if c in pairs:
            if not st or st.pop() != pairs[c]: return False
        else:
            st.append(c)
    return not st

# Monotonic stack: next greater element to the right (-1 if none)
def next_greater(a):
    res = [-1] * len(a)
    st = []                      # indices, values decreasing
    for i, x in enumerate(a):
        while st and a[st[-1]] < x:
            res[st.pop()] = x
        st.append(i)
    return res

# Daily temperatures (days until warmer)
def daily_temperatures(t):
    res = [0] * len(t)
    st = []
    for i, x in enumerate(t):
        while st and t[st[-1]] < x:
            j = st.pop()
            res[j] = i - j
        st.append(i)
    return res

# Largest rectangle in histogram — O(n)
def largest_rectangle(heights):
    st, best = [], 0             # indices, heights increasing
    for i, h in enumerate(heights + [0]):    # sentinel flushes stack
        while st and heights[st[-1]] >= h:
            H = heights[st.pop()]
            W = i if not st else i - st[-1] - 1
            best = max(best, H * W)
        st.append(i)
    return best

# Min-stack: push (value, current_min) pairs
class MinStack:
    def __init__(self): self.st = []
    def push(self, x):
        m = min(x, self.st[-1][1]) if self.st else x
        self.st.append((x, m))
    def pop(self): return self.st.pop()[0]
    def top(self): return self.st[-1][0]
    def get_min(self): return self.st[-1][1]
```

## 22. Trees (Binary Trees & BST)

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Traversals (recursive)
def inorder(node, out):
    if node:
        inorder(node.left, out)
        out.append(node.val)
        inorder(node.right, out)
# preorder: visit, left, right.  postorder: left, right, visit.
# Inorder of a BST is sorted — key fact.

# Inorder ITERATIVE (stack)
def inorder_iter(root):
    out, st, cur = [], [], root
    while cur or st:
        while cur:
            st.append(cur)
            cur = cur.left
        cur = st.pop()
        out.append(cur.val)
        cur = cur.right
    return out

# Level-order (BFS) with levels
from collections import deque
def level_order(root):
    if not root: return []
    out, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):          # process one level at a time
            node = q.popleft()
            level.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        out.append(level)
    return out

# Max depth
def depth(node):
    if not node: return 0
    return 1 + max(depth(node.left), depth(node.right))

# Validate BST (bounds method)
def is_bst(node, lo=float('-inf'), hi=float('inf')):
    if not node: return True
    if not (lo < node.val < hi): return False
    return is_bst(node.left, lo, node.val) and is_bst(node.right, node.val, hi)

# Lowest common ancestor (binary tree)
def lca(root, p, q):
    if not root or root is p or root is q: return root
    l = lca(root.left, p, q)
    r = lca(root.right, p, q)
    if l and r: return root
    return l or r

# Diameter (longest path) — return height, track best via nonlocal
def diameter(root):
    best = 0
    def height(node):
        nonlocal best
        if not node: return 0
        lh, rh = height(node.left), height(node.right)
        best = max(best, lh + rh)
        return 1 + max(lh, rh)
    height(root)
    return best
```

## 23. Graphs — BFS / DFS / Topological Sort

```python
from collections import deque, defaultdict

# Build adjacency list from edge list
def build(n, edges, directed=False):
    g = defaultdict(list)
    for u, v in edges:
        g[u].append(v)
        if not directed:
            g[v].append(u)
    return g

# BFS — shortest path in UNWEIGHTED graph
def bfs(g, start):
    dist = {start: 0}
    q = deque([start])
    while q:
        u = q.popleft()
        for v in g[u]:
            if v not in dist:
                dist[v] = dist[u] + 1
                q.append(v)
    return dist

# DFS — recursive
def dfs(g, u, visited):
    visited.add(u)
    for v in g[u]:
        if v not in visited:
            dfs(g, v, visited)

# DFS — iterative (safe from recursion limits)
def dfs_iter(g, start):
    visited, st = set(), [start]
    while st:
        u = st.pop()
        if u in visited: continue
        visited.add(u)
        for v in g[u]:
            if v not in visited:
                st.append(v)
    return visited

# Count connected components
def components(n, g):
    seen, cnt = set(), 0
    for u in range(n):
        if u not in seen:
            cnt += 1
            # BFS flood fill
            q = deque([u]); seen.add(u)
            while q:
                x = q.popleft()
                for v in g[x]:
                    if v not in seen:
                        seen.add(v); q.append(v)
    return cnt

# Grid BFS (islands-style) — grid IS the graph
def num_islands(grid):
    R, C = len(grid), len(grid[0])
    seen = set()
    def flood(r, c):
        q = deque([(r, c)]); seen.add((r, c))
        while q:
            x, y = q.popleft()
            for dx, dy in ((1,0), (-1,0), (0,1), (0,-1)):
                nx, ny = x + dx, y + dy
                if 0 <= nx < R and 0 <= ny < C and (nx, ny) not in seen \
                        and grid[nx][ny] == '1':
                    seen.add((nx, ny)); q.append((nx, ny))
    cnt = 0
    for r in range(R):
        for c in range(C):
            if grid[r][c] == '1' and (r, c) not in seen:
                cnt += 1
                flood(r, c)
    return cnt

# Topological sort — Kahn's algorithm (BFS). Detects cycles too.
def topo_sort(n, edges):        # nodes 0..n-1, edges (u -> v)
    g = defaultdict(list)
    indeg = [0] * n
    for u, v in edges:
        g[u].append(v)
        indeg[v] += 1
    q = deque(u for u in range(n) if indeg[u] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in g[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return order if len(order) == n else None    # None => cycle
```

## 24. Shortest Paths (Dijkstra)

```python
import heapq
from collections import defaultdict

# Dijkstra — non-negative weights, O(E log V)
def dijkstra(g, start):          # g[u] = list of (v, weight)
    dist = {start: 0}
    pq = [(0, start)]
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist.get(u, float('inf')):    # stale entry — skip
            continue
        for v, w in g[u]:
            nd = d + w
            if nd < dist.get(v, float('inf')):
                dist[v] = nd
                heapq.heappush(pq, (nd, v))
    return dist

# 0-1 BFS — weights only 0 or 1, O(V + E), deque instead of heap
from collections import deque
def bfs01(g, start, n):          # g[u] = list of (v, w) with w in {0, 1}
    INF = float('inf')
    dist = [INF] * n
    dist[start] = 0
    dq = deque([start])
    while dq:
        u = dq.popleft()
        for v, w in g[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                if w == 0: dq.appendleft(v)
                else:      dq.append(v)
    return dist

# Bellman-Ford — handles negative weights, O(V*E)
def bellman_ford(n, edges, start):    # edges: (u, v, w)
    INF = float('inf')
    dist = [INF] * n
    dist[start] = 0
    for _ in range(n - 1):
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    # one more pass: any improvement => negative cycle
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            return None
    return dist

# Floyd-Warshall — all pairs, O(V^3), tiny graphs
def floyd_warshall(n, edges):
    INF = float('inf')
    d = [[INF] * n for _ in range(n)]
    for i in range(n): d[i][i] = 0
    for u, v, w in edges:
        d[u][v] = min(d[u][v], w)
    for k in range(n):
        for i in range(n):
            for j in range(n):
                if d[i][k] + d[k][j] < d[i][j]:
                    d[i][j] = d[i][k] + d[k][j]
    return d
```

## 25. Union-Find (Disjoint Set)

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.count = n                    # number of components

    def find(self, x):                    # path compression
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x

    def union(self, a, b):                # union by size
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False                  # already connected (cycle!)
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        self.count -= 1
        return True

    def connected(self, a, b):
        return self.find(a) == self.find(b)

# Kruskal's MST — sort edges, union greedily
def kruskal(n, edges):                    # edges: (w, u, v)
    dsu = DSU(n)
    total = 0
    for w, u, v in sorted(edges):
        if dsu.union(u, v):
            total += w
    return total if dsu.count == 1 else None
```

## 26. Backtracking

```python
# Subsets (power set)
def subsets(nums):
    out = []
    def bt(i, path):
        if i == len(nums):
            out.append(path[:])           # COPY the path!
            return
        bt(i + 1, path)                   # exclude nums[i]
        path.append(nums[i])              # include nums[i]
        bt(i + 1, path)
        path.pop()                        # undo — the essence of backtracking
    bt(0, [])
    return out

# Permutations
def permute(nums):
    out = []
    def bt(path, used):
        if len(path) == len(nums):
            out.append(path[:])
            return
        for i, x in enumerate(nums):
            if used[i]: continue
            used[i] = True; path.append(x)
            bt(path, used)
            path.pop(); used[i] = False
    bt([], [False] * len(nums))
    return out

# Combinations: k numbers from 1..n
def combine(n, k):
    out = []
    def bt(start, path):
        if len(path) == k:
            out.append(path[:])
            return
        for x in range(start, n + 1):
            path.append(x)
            bt(x + 1, path)
            path.pop()
    bt(1, [])
    return out

# Combination sum (reuse allowed) — prune with sorting
def combination_sum(candidates, target):
    candidates.sort()
    out = []
    def bt(start, remain, path):
        if remain == 0:
            out.append(path[:])
            return
        for i in range(start, len(candidates)):
            c = candidates[i]
            if c > remain: break          # prune
            path.append(c)
            bt(i, remain - c, path)       # i (not i+1): reuse allowed
            path.pop()
    bt(0, target, [])
    return out

# N-Queens counting — sets for columns and diagonals
def n_queens(n):
    cols, d1, d2 = set(), set(), set()
    def bt(row):
        if row == n: return 1
        total = 0
        for c in range(n):
            if c in cols or row - c in d1 or row + c in d2: continue
            cols.add(c); d1.add(row - c); d2.add(row + c)
            total += bt(row + 1)
            cols.remove(c); d1.discard(row - c); d2.discard(row + c)
        return total
    return bt(0)
```

## 27. Dynamic Programming

```python
from functools import cache

# Top-down (memoization) — write brute-force recursion, slap @cache on it
@cache
def climb(n):                     # ways to climb n stairs (1 or 2 steps)
    if n <= 1: return 1
    return climb(n - 1) + climb(n - 2)

# Bottom-up 1D: house robber
def rob(nums):
    take = skip = 0
    for x in nums:
        take, skip = skip + x, max(take, skip)
    return max(take, skip)

# Coin change — min coins to make amount
def coin_change(coins, amount):
    INF = float('inf')
    dp = [0] + [INF] * amount
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != INF else -1

# 0/1 Knapsack — 1D array, iterate capacity DOWNWARD
def knapsack(weights, values, cap):
    dp = [0] * (cap + 1)
    for w, v in zip(weights, values):
        for c in range(cap, w - 1, -1):        # reverse => each item once
            dp[c] = max(dp[c], dp[c - w] + v)
    return dp[cap]
# (Unbounded knapsack: iterate capacity UPWARD instead.)

# Longest Increasing Subsequence — O(n log n) with patience sorting
import bisect
def lis(nums):
    tails = []                    # tails[i] = smallest tail of LIS of len i+1
    for x in nums:
        i = bisect.bisect_left(tails, x)
        if i == len(tails): tails.append(x)
        else: tails[i] = x
    return len(tails)

# Longest Common Subsequence — 2D table
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]

# Edit distance
def edit_distance(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1): dp[i][0] = i
    for j in range(n + 1): dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i-1] == b[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j],      # delete
                                   dp[i][j-1],      # insert
                                   dp[i-1][j-1])    # replace
    return dp[m][n]

# Kadane's — max subarray sum
def max_subarray(a):
    best = cur = a[0]
    for x in a[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best

# Grid DP — unique paths with obstacles
def unique_paths(grid):
    R, C = len(grid), len(grid[0])
    dp = [0] * C
    dp[0] = 1 - grid[0][0]
    for r in range(R):
        for c in range(C):
            if grid[r][c] == 1: dp[c] = 0        # obstacle
            elif c > 0: dp[c] += dp[c - 1]
    return dp[-1]
```

## 28. Trie

```python
class Trie:
    def __init__(self):
        self.root = {}

    def insert(self, word):
        node = self.root
        for c in word:
            node = node.setdefault(c, {})
        node['$'] = True                  # end-of-word marker

    def search(self, word):
        node = self._walk(word)
        return node is not None and '$' in node

    def starts_with(self, prefix):
        return self._walk(prefix) is not None

    def _walk(self, s):
        node = self.root
        for c in s:
            if c not in node: return None
            node = node[c]
        return node
```

## 29. Intervals

```python
# Merge overlapping intervals — sort by start
def merge_intervals(intervals):
    intervals.sort()
    out = []
    for s, e in intervals:
        if out and s <= out[-1][1]:
            out[-1][1] = max(out[-1][1], e)     # extend (use lists inside)
        else:
            out.append([s, e])
    return out

# Max non-overlapping intervals — sort by END, greedy
def max_non_overlapping(intervals):
    intervals.sort(key=lambda iv: iv[1])
    cnt, end = 0, float('-inf')
    for s, e in intervals:
        if s >= end:
            cnt += 1
            end = e
    return cnt

# Meeting rooms II (min rooms) — sweep line / heap
import heapq
def min_rooms(intervals):
    intervals.sort()
    heap = []                     # end times of active meetings
    for s, e in intervals:
        if heap and heap[0] <= s:
            heapq.heappop(heap)   # reuse a room
        heapq.heappush(heap, e)
    return len(heap)

# Sweep line with events
def min_rooms_sweep(intervals):
    events = []
    for s, e in intervals:
        events.append((s, 1))
        events.append((e, -1))
    events.sort()                 # end (-1) before start (+1) at same time
    cur = best = 0
    for _, delta in events:
        cur += delta
        best = max(best, cur)
    return best
```

## 30. Matrix / Grid Tricks

```python
g = [[1, 2, 3],
     [4, 5, 6]]
R, C = len(g), len(g[0])

# 4-directional / 8-directional moves
DIR4 = [(1, 0), (-1, 0), (0, 1), (0, -1)]
DIR8 = [(dr, dc) for dr in (-1, 0, 1) for dc in (-1, 0, 1) if (dr, dc) != (0, 0)]

# Bounds check idiom
def in_bounds(r, c): return 0 <= r < R and 0 <= c < C

# Transpose
t = [list(row) for row in zip(*g)]       # [[1,4],[2,5],[3,6]]

# Rotate 90° clockwise = transpose + reverse each row
rot = [list(row)[::-1] for row in zip(*g)]
# Rotate 90° counterclockwise:
ccw = [list(row) for row in zip(*g)][::-1]

# Iterate diagonals: cells on same diagonal share r - c (or r + c for anti)
from collections import defaultdict
diags = defaultdict(list)
for r in range(R):
    for c in range(C):
        diags[r - c].append(g[r][c])

# Flatten index <-> 2D (useful for DSU on grids)
def to_id(r, c): return r * C + c
def to_rc(i): return divmod(i, C)
```

## 31. Bit Manipulation

```python
x = 13                    # 0b1101

x & 1                     # 1 — is odd?
x >> 1                    # 6 — floor divide by 2
x << 2                    # 52 — multiply by 4
x & (x - 1)               # 12 — clears LOWEST set bit
x & (-x)                  # 1 — isolates lowest set bit
x | (1 << 2)              # set bit 2
x & ~(1 << 2)             # clear bit 2
x ^ (1 << 2)              # toggle bit 2
(x >> 2) & 1              # get bit 2

bin(13)                   # '0b1101'
f"{13:b}"                 # '1101'
int('1101', 2)            # 13
13 .bit_length()          # 4
13 .bit_count()           # 3 — popcount (3.10+); before: bin(x).count('1')

# Power of two check
def is_pow2(n): return n > 0 and n & (n - 1) == 0

# XOR facts: a^a=0, a^0=a, XOR is commutative
# -> find the single non-duplicated number:
from functools import reduce
import operator
def single_number(nums): return reduce(operator.xor, nums)

# Iterate all subsets of a bitmask
def subsets_of(mask):
    sub = mask
    while sub:
        yield sub
        sub = (sub - 1) & mask
    yield 0

# Bitmask over n items (subset DP skeleton)
def iterate_masks(n):
    for mask in range(1 << n):
        for i in range(n):
            if mask >> i & 1:
                pass              # item i is in this subset
```

## 32. Number Theory

```python
import math

# Sieve of Eratosthenes — all primes < n
def sieve(n):
    is_prime = [True] * n
    is_prime[0:2] = [False, False]
    for i in range(2, math.isqrt(n - 1) + 1):
        if is_prime[i]:
            for j in range(i * i, n, i):
                is_prime[j] = False
    return [i for i in range(n) if is_prime[i]]

# Primality test (trial division) — fine up to ~1e12
def is_prime(n):
    if n < 2: return False
    if n % 2 == 0: return n == 2
    for i in range(3, math.isqrt(n) + 1, 2):
        if n % i == 0: return False
    return True

# Prime factorization
def factorize(n):
    factors = {}
    d = 2
    while d * d <= n:
        while n % d == 0:
            factors[d] = factors.get(d, 0) + 1
            n //= d
        d += 1
    if n > 1:
        factors[n] = factors.get(n, 0) + 1
    return factors

# Fast modular exponentiation — built in!
MOD = 10**9 + 7
pow(2, 100, MOD)                  # (2^100) % MOD in O(log n)

# Modular inverse (MOD prime)
def modinv(a, mod=MOD): return pow(a, -1, mod)     # 3.8+

# nCr % p with precomputed factorials
def make_ncr(n_max, mod=MOD):
    fact = [1] * (n_max + 1)
    for i in range(1, n_max + 1):
        fact[i] = fact[i - 1] * i % mod
    inv = [1] * (n_max + 1)
    inv[n_max] = pow(fact[n_max], -1, mod)
    for i in range(n_max, 0, -1):
        inv[i - 1] = inv[i] * i % mod
    def ncr(n, r):
        if r < 0 or r > n: return 0
        return fact[n] * inv[r] % mod * inv[n - r] % mod
    return ncr

# All divisors in O(sqrt(n))
def divisors(n):
    out = []
    for i in range(1, math.isqrt(n) + 1):
        if n % i == 0:
            out.append(i)
            if i != n // i:
                out.append(n // i)
    return sorted(out)
```

## 33. OOP & Interview Extras

```python
# Class essentials
class Animal:
    species_count = 0                    # class variable (shared)

    def __init__(self, name):
        self.name = name                 # instance variable
        Animal.species_count += 1

    def speak(self):                     # instance method
        return f"{self.name} makes a sound"

    def __repr__(self):                  # unambiguous representation
        return f"Animal({self.name!r})"

    def __eq__(self, other):
        return isinstance(other, Animal) and self.name == other.name

    def __hash__(self):                  # needed if __eq__ defined & used in sets
        return hash(self.name)

    def __lt__(self, other):             # enables sorting / heapq
        return self.name < other.name

class Dog(Animal):                       # inheritance
    def speak(self):                     # override
        return f"{self.name} barks"
    def fetch(self):
        return super().speak()           # call parent method

# dataclass — auto __init__, __repr__, __eq__
from dataclasses import dataclass, field
@dataclass(order=True)                   # order=True: comparable/sortable
class Task:
    priority: int
    name: str = "untitled"
    tags: list = field(default_factory=list)   # mutable default done right

# Exceptions
class CustomError(Exception): pass
try:
    raise CustomError("oops")
except CustomError as e:
    msg = str(e)
except (ValueError, KeyError):
    pass
else:
    pass          # runs only if NO exception
finally:
    pass          # always runs

# Context manager
from contextlib import contextmanager
@contextmanager
def timer():
    import time
    t0 = time.perf_counter()
    yield
    elapsed = time.perf_counter() - t0

# Iterator protocol / custom iterable
class Countdown:
    def __init__(self, n): self.n = n
    def __iter__(self): return self
    def __next__(self):
        if self.n <= 0: raise StopIteration
        self.n -= 1
        return self.n + 1

# LRU Cache implementation (classic design question) — OrderedDict
from collections import OrderedDict
class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.d = OrderedDict()
    def get(self, key):
        if key not in self.d: return -1
        self.d.move_to_end(key)
        return self.d[key]
    def put(self, key, value):
        if key in self.d: self.d.move_to_end(key)
        self.d[key] = value
        if len(self.d) > self.cap:
            self.d.popitem(last=False)    # evict least-recent

# Type hints (increasingly expected in interviews)
def k_largest(nums: list[int], k: int) -> list[int]:
    ...

from typing import Optional
def find(x: int) -> Optional[int]:       # or `int | None` (3.10+)
    ...
```

## 34. Big-O of Python Operations

| Structure | Operation | Complexity |
|---|---|---|
| **list** | `append`, `pop()` (end), index `a[i]` | O(1) |
| | `insert`, `pop(0)`, `remove`, `in` | O(n) |
| | slice `a[i:j]` | O(j−i) |
| | `sort` | O(n log n) |
| **dict / set** | insert, lookup, delete, `in` | O(1) avg |
| **deque** | `append/pop` both ends | O(1) |
| | index `dq[i]` | O(n) |
| **heapq** | `heappush`, `heappop` | O(log n) |
| | `heapify` | O(n) |
| | peek `h[0]` | O(1) |
| **str** | concat `s + t` | O(len) — avoid in loops |
| | `join` of n pieces | O(total length) |
| **Counter** | build from iterable | O(n) |
| | `most_common(k)` | O(n log k) |

### Rules of thumb for CP time limits (Python)

- ~10<sup>7</sup> simple operations per second is a safe planning number (CPython).
- n ≤ 10<sup>6</sup> → need O(n) or O(n log n).
- n ≤ 10<sup>5</sup> → O(n log n) comfortable, O(n²) too slow.
- n ≤ 5000 → O(n²) OK.
- n ≤ 500 → O(n³) OK.
- n ≤ 20 → O(2ⁿ) bitmask/backtracking OK.
- Prefer built-ins (`sum`, `max`, `sorted`, `join`, slicing) — they run in C and are much faster than manual loops.

---

## 35. My Solution Review Log

Optimization reviews of my own solutions — original vs. optimized, with the takeaway pattern to remember. Newest entries at the bottom.

<!-- Entry template:
### #N — Problem Name (source/link)
**Constraints:** n ≤ ...
**My solution:** O(...) time, O(...) space
```python
(original code)
```
**Issue:** what made it slow/suboptimal
**Optimized:** O(...) time, O(...) space
```python
(optimized code)
```
**Takeaway:** the reusable pattern (see Section X)
-->

### Day 1 Recommendations (after problems #1–#5)

Based on the first 5 reviewed solutions — Add Two Numbers (#1), Longest Substring Without Repeating Characters (#2), Palindrome Number (#3), Longest Common Prefix (#4), Roman to Integer (#5). Small sample, but clear patterns already. Will revisit and refine after more entries.

**What's already solid:** every solution here was logically correct — each was checked against a brute-force or reference implementation on thousands of random/edge cases with 0 mismatches. Three of five (#1, #3, #5) were already asymptotically optimal on arrival. The gaps below are about *recognizing* the optimal shape faster and about code hygiene, not about getting wrong answers.

**1. Biggest lever — don't restart a scan, extend it (entries #2, #4).** Both O(n²)-class solutions had the same root cause: at each step, the loop re-verified something from scratch (`charSet.clear()` in #2, re-running `startswith` on the full growing prefix in #4) instead of reusing what the previous step already established. This was the single most expensive pattern found — it turned O(n) problems into O(n²) and O(n·L) into O(n·L²). Before coding a string/array scan, ask: *if I already have the answer through position i, what's the minimal update to get position i+1?* That question is the trigger for sliding window (Section 17), two pointers (Section 16), and incremental scans like `zip(*matrix)` (Section 30). Concretely: if you're about to write `.clear()` on a collection inside a loop, or restart an inner loop from a fixed offset on every outer iteration, stop and check whether that's throwing away reusable state.

**2. Look for the one general rule before hardcoding special cases (entries #5, #1).** The Roman numeral solution enumerated all six subtractive pairs (IV, IX, XL, XC, CD, CM) by name, when they're one rule: *value < next value → subtract*. Add Two Numbers similarly opened with three explicit None-guard branches that the main loop already handled on its own. When a brute-force-correct solution has several near-identical branches, ask whether one condition unifies them — it's shorter and removes a whole class of "forgot one of the special cases" bugs.

**3. Use standard structural idioms for building output (entry #1).** For linked-list construction, `dummy = tail = ListNode()` (return `dummy.next`) removes every "is this the first node?" branch. For loops with trailing state to flush (a carry, a pending group), try folding it into the loop condition (`while l1 or l2 or carry`) instead of duplicating the loop body afterward.

**4. Reach for dict lookups and enumerate/zip over manual if/elif chains and index bookkeeping (entries #1, #5).** A 7-branch if/elif mapping characters to values (#5) is more typo-prone than one dict; `while i < n: ... i = i+1` is more error-prone than `for i, c in enumerate(s)`. This isn't just style — fewer places to introduce an off-by-one or a copy-paste value error.

**5. Pre-submit hygiene pass (entries #1, #3, #5).** Strip debug `print()` calls before submitting — measured a real ~3x slowdown from one leftover print, and on some judges stray output can fail the submission outright. Don't shadow builtins (`sum`, `list`, `id`, `type`, `str`, `max`, `min`) even when it "still works" — entry #5 named an accumulator `sum`. Delete dead code — redundant early-return guards the main loop already covers, or a variable assignment immediately overwritten (`isPrefixExist = True` before a loop that always sets it).

**6. Test helper functions independently of their call site (entry #3).** `reverse()` in Palindrome Number had a genuine infinite loop on negative input — Python's floor-division `%`/`divmod` behaves differently from C on negatives (Section 1) — but it was never caught because the only caller filtered negatives first. A helper that's only "correct" because of what its caller happens to pass in is a latent bug. Test helpers against their full plausible input range directly, not just through the current call site.

**Before-submit checklist (running):**
- Any inner loop or check that restarts from scratch when it could extend the previous step's state instead?
- Any hardcoded list of special cases that's actually one general condition?
- Building a list/linked list? Use a dummy/sentinel node.
- Any leftover `print()`/debug code?
- Any builtin names shadowed (`sum`, `list`, `str`, `id`, `type`, `input`, `max`, `min`)?
- Would each helper function still behave correctly if called directly, without the current caller's guards?

---

### #1 — Add Two Numbers (LeetCode 2)

**Constraints:** lists have 1–100 nodes, digits stored in reverse order.
**My solution:** O(max(m,n)) time, O(max(m,n)) space — already asymptotically optimal. The issue is code complexity, not speed.

```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        carryOver = 0
        result = None
        head = None
        if l1 is None and l2 is None:
            return None
        if l1 is None and l2 is not None:
            return l2
        if l1 is not None and l2 is None:
            return l1
        while l1 is not None or l2 is not None:
            val = 0
            if l1 is not None:
                val = l1.val
            if l2 is not None:
                val = val + l2.val
            val = val + carryOver
            carryOver = val // 10
            val = val % 10
            node = ListNode(val, None)
            if result is None:
                result = node
                head = result
            else:
                result.next = node
                result = result.next
            if l1 is not None:
                l1 = l1.next
            if l2 is not None:
                l2 = l2.next
        if carryOver != 0:
            node = ListNode(carryOver, None)
            if result is None:
                result = node
                head = result
            else:
                result.next = node
                result = result.next
        return head
```

**Issues:**

1. No dummy node → the `if result is None: ... else: ...` first-node special case is duplicated twice (once in the loop, once for the final carry). A dummy head deletes all of it.
2. Leftover carry handled in a separate block after the loop → duplicated node-creation logic. Folding `carry` into the loop condition (`while l1 or l2 or carry`) makes the loop handle it for free.
3. The three early-return None checks are dead weight — the main loop already handles one list being shorter/empty. (The `return l1`/`return l2` shortcuts also silently assume no work is needed, which is only true because carry starts at 0 — fragile.)
4. Minor style: `if x is not None` → `if x` for nodes; `carry, digit = divmod(val, 10)` replaces the `//` + `%` pair.

**Optimized:** same O(max(m,n)) time/space, ~45 lines → 10 lines

```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        dummy = tail = ListNode()
        carry = 0
        while l1 or l2 or carry:
            if l1: carry += l1.val; l1 = l1.next
            if l2: carry += l2.val; l2 = l2.next
            carry, digit = divmod(carry, 10)
            tail.next = ListNode(digit)
            tail = tail.next
        return dummy.next
```

**Takeaway:** When building a linked list, start with `dummy = tail = ListNode()` and return `dummy.next` — it eliminates every "is this the first node?" branch (see Section 20). And when a loop has leftover state to flush afterward (carry, pending group, buffer), try folding it into the loop condition instead of duplicating the body after the loop.

### #2 — Longest Substring Without Repeating Characters (LeetCode 3)

**Constraints:** 0 ≤ len(s) ≤ 5·10⁴, chars are letters/digits/symbols/spaces.
**My solution:** O(n·k) time (k = window length; **O(n²) worst case** when all chars distinct), O(k) space. Correct, but restarts the scan from every index.

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        charSet = set()
        maxLen = 0
        for i in range(len(s)):
            charSet.add(s[i])
            for j in range(i+1, len(s)):
                if s[j] in charSet:
                    maxLen = max(maxLen, j-i)
                    charSet.clear()
                    break
                else:
                    charSet.add(s[j])
            maxLen = max(maxLen, len(charSet))
            charSet.clear()
        return maxLen
```

**Issues:**

1. **Thrown-away work.** For each start `i`, the inner loop re-scans characters the previous iteration already examined, then `charSet.clear()` discards everything learned. The window from `i` and the window from `i+1` overlap almost entirely — that overlap is exactly what a sliding window keeps.
2. **The key insight being missed:** when a duplicate of `s[j]` appears, the next candidate start isn't `i+1` — it's *one past the previous occurrence of `s[j]`*. Every start between those two positions is doomed to hit the same duplicate. A last-seen-index map lets `left` jump straight there.
3. Benchmarks: with a 52-char alphabet at n = 5·10⁴ it survives (~58 ms) because the window can't exceed 53 chars, but with all-distinct input (allowed by constraints) it's ~28 **seconds** vs ~6 ms for the sliding window — a 5000x gap and a guaranteed TLE.

**Optimized:** O(n) time — each index visited once; O(min(n, alphabet)) space

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last = {}                    # char -> index of its last occurrence
        left = best = 0              # window is s[left..r]
        for r, c in enumerate(s):
            if c in last and last[c] >= left:
                left = last[c] + 1   # jump past the previous occurrence
            last[c] = r
            best = max(best, r - left + 1)
        return best
```

**Takeaway:** "Longest/shortest substring (or subarray) satisfying a condition" is the sliding-window signature (Section 17). If your solution restarts a scan from every index i, ask: what does the window starting at i share with the window at i+1? Keeping that shared state instead of clearing it is what turns O(n²) into O(n). The `last[c] >= left` guard matters — an old occurrence *behind* the window (e.g. `"abba"`, `"tmmzuxt"`) must not drag `left` backward.

### #3 — Palindrome Number (LeetCode 9)

**Constraints:** -2³¹ ≤ x ≤ 2³¹ − 1.
**My solution:** O(d) time (d = digit count) — already optimal asymptotically, but with three concrete issues.

```python
class Solution:
    def reverse(self, x: int) -> int:
        num = 0
        while x:
            q, digit = divmod(x, 10)
            num = num*10 + digit
            x = q
        print(num)
        return num

    def isPalindrome(self, x: int) -> bool:
        if x < 0:
            return False
        return self.reverse(x) == x
```

**Issues:**

1. **Leftover debug `print()`** — measured ~3x slowdown, and on real CP judges (with actual terminal/pipe I/O, not a buffer) far worse; stray prints can also cause "Wrong Answer" verdicts on judges that compare full output. Always strip debug prints before submitting.
2. **Latent infinite loop:** `reverse()` never terminates for negative x — `divmod(-1, 10)` is `(-1, 9)` in Python (floor division!), so x never reaches 0. It's unreachable here only because of the `x < 0` guard, but reused as-is (e.g. for LeetCode 7 "Reverse Integer") it hangs. Helpers should be safe standalone: handle the sign explicitly.
3. **Reverses the whole number** — fine in Python (big ints), but in C++/Java the reversed value can overflow a 32-bit int, and interviewers specifically probe this. The half-reversal avoids ever building a number bigger than the input.

**Optimized (interview answer):** reverse only half the digits — stop when the reversed half catches up

```python
class Solution:
    def isPalindrome(self, x: int) -> bool:
        if x < 0 or (x % 10 == 0 and x != 0):   # trailing zero can't mirror
            return False
        half = 0
        while x > half:
            half = half * 10 + x % 10
            x //= 10
        return x == half or x == half // 10     # even / odd digit count
```

**Pythonic one-liner (fastest in CPython — string ops run in C):**

```python
class Solution:
    def isPalindrome(self, x: int) -> bool:
        s = str(x)
        return x >= 0 and s == s[::-1]
```

Measured on 10k random 32-bit ints: original ~12 ms, half-reversal ~4 ms, string ~2 ms.

**Takeaway:** Three habits from one problem: strip debug prints before submitting; make helpers safe standalone (Python's floor-division `%`/`divmod` on negatives differs from C — see Section 1); and know the half-reversal trick — it's the expected answer to "now do it without converting to a string / without overflow."

### #4 — Longest Common Prefix (LeetCode 14)

**Constraints:** 1 ≤ strs.length ≤ 200, 0 ≤ strs[i].length ≤ 200.
**My solution:** O(n·L²) worst case (n = number of strings, L = length of the true common prefix) — redoes work at every length instead of building on the previous check.

```python
class Solution:
    def longestCommonPrefix(self, strs: List[str]) -> str:
        maxPrefixLen = len(strs[0])
        for item in strs:
            maxPrefixLen = min(maxPrefixLen, len(item))

        prefixLen = 1
        longestCommonPrefix = ""
        while prefixLen <= maxPrefixLen:
            prefix = strs[0][0:prefixLen]
            i = 1
            isPrefixExist = True
            while i < len(strs):
                if strs[i].startswith(prefix):
                    isPrefixExist = True
                else:
                    isPrefixExist = False
                    break
                i = i + 1
            prefixLen = prefixLen + 1
            if isPrefixExist:
                longestCommonPrefix = prefix
            else:
                return longestCommonPrefix
        return longestCommonPrefix
```

**Issue:** For each candidate `prefixLen` from 1 up to L, it re-checks `strs[i].startswith(prefix)` against the *entire* prefix from scratch for every string. Since consecutive prefixes overlap almost completely (prefix of length k is a prefix of length k+1's prefix), each length re-verifies characters the previous length already confirmed matched. Summed over lengths 1..L this is O(n·L²) character comparisons, not O(n·L). (`isPrefixExist = True` before the inner loop is also dead — it's always overwritten on the first iteration or the loop never runs — harmless but noise.)

To isolate the real asymptotic gap from constant-factor noise (Python's `str.startswith` is C-level memcmp, so wall-clock alone doesn't show it until L is huge), I counted actual character comparisons directly:

| L | original comparisons | optimized comparisons | ratio |
|---|---|---|---|
| 100 | 242,650 | 4,852 | 50x |
| 200 | 975,300 | 9,752 | 100x |
| 400 | 3,910,600 | 19,552 | 200x |
| 800 | 15,699,600 | 39,200 | 400x |
| 1600 | 62,682,400 | 78,352 | 800x |

The ratio doubling every time L doubles is the signature of O(L²) vs O(L) — textbook confirmation, not just a benchmark artifact.

**Optimized:** O(n·L) time — vertical scan, one pass, stops at the first mismatching column

```python
class Solution:
    def longestCommonPrefix(self, strs: List[str]) -> str:
        for i, chars in enumerate(zip(*strs)):   # chars = column i across all strings
            first = chars[0]
            for c in chars[1:]:
                if c != first:
                    return strs[0][:i]
        return min(strs, key=len)                # all strings agree up to the shortest one
```

**Takeaway:** When a solution re-derives length-k's answer from scratch instead of extending length-(k-1)'s, look for the incremental version — here, scanning column-by-column with `zip(*strs)` naturally reuses every character exactly once and stops at the first disagreement, with no re-scanning. `zip(*matrix)` for "transpose and scan by column" is worth remembering generally (Section 30).

### #5 — Roman to Integer (LeetCode 13)

**Constraints:** 1 ≤ s.length ≤ 15, valid Roman numeral, 1 ≤ value ≤ 3999.
**My solution:** O(n) time, O(1) space — already asymptotically optimal, single pass. This review is about code quality, not speed.

```python
class Solution:
    def romanToInt(self, s: str) -> int:
        n = len(s)
        sum = 0
        i = 0
        while i < n:
            if s[i] == "I":
                if i+1<n and s[i+1] == "V":
                    sum = sum + 4
                    i = i+1
                elif i+1<n and s[i+1] == "X":
                    sum = sum + 9
                    i = i+1
                else:
                    sum = sum + 1
            elif s[i] == "V":
                sum = sum + 5
            elif s[i] == "X":
                if i+1<n and s[i+1] == "L":
                    sum = sum + 40
                    i = i+1
                elif i+1<n and s[i+1] == "C":
                    sum = sum + 90
                    i = i+1
                else:
                    sum = sum + 10
            elif s[i] == "L":
                sum = sum + 50
            elif s[i] == "C":
                if i+1<n and s[i+1] == "D":
                    sum = sum + 400
                    i = i+1
                elif i+1<n and s[i+1] == "M":
                    sum = sum + 900
                    i = i+1
                else:
                    sum = sum + 100
            elif s[i] == "D":
                sum = sum + 500
            elif s[i] == "M":
                sum = sum + 1000
            i = i+1
        return sum
```

**Issues:**

1. **Hardcodes all 6 subtractive pairs by name** (IV, IX, XL, XC, CD, CM) with duplicated `if i+1<n and s[i+1]==...` branches for I, X, and C. The actual rule generalizes: *a numeral is subtracted exactly when it's smaller than the numeral immediately after it* — no need to enumerate which pairs are "special," and nothing to get wrong if you mistype one of the six checks.
2. **Shadows the builtin `sum`** by naming the accumulator `sum`. Harmless here since `sum()` isn't called elsewhere in the function, but it's a habit worth breaking — it silently shadows the builtin for the rest of the function's scope and can bite you later if you add a `sum(...)` call.
3. Char-to-value mapping is spread across 7 branches of an if/elif chain instead of a single dict lookup — more surface area for a typo (e.g. an accidental `50` vs `500`).

**Optimized:** same O(n) time, O(1) space — same algorithmic complexity, far less code

```python
class Solution:
    def romanToInt(self, s: str) -> int:
        values = {'I':1,'V':5,'X':10,'L':50,'C':100,'D':500,'M':1000}
        total = 0
        n = len(s)
        for i, c in enumerate(s):
            v = values[c]
            if i+1 < n and v < values[s[i+1]]:
                total -= v
            else:
                total += v
        return total
```

Verified against every valid Roman numeral from 1 to 3999 (generated with a known-correct `int_to_roman`) — both versions match exactly, 0 mismatches. Also ~1.5x faster on 100k calls (137 ms vs 91 ms) purely from replacing the branch chain with dict lookups — a nice side effect, not the main point.

**Takeaway:** When a problem has a short list of "special cases" you're tempted to hardcode (subtractive pairs, edge characters, exceptions), check whether they're really instances of one general rule (here: *value < next value → subtract*). The general rule is shorter, harder to typo, and doesn't need updating if the special-case list changes. Also: never shadow a builtin name (`sum`, `list`, `id`, `type`, `input`, `str`, `max`, `min`, ...) even when it "still works" — pick a domain-specific name (`total`, `nums`, `s`) instead.

### #6 — Valid Parentheses (LeetCode 20)

**Constraints:** 1 ≤ s.length ≤ 10⁴, s consists only of `()[]{}`.
**My solution:** O(n) time, O(n) space — already asymptotically optimal (worst case, e.g. all opening brackets, genuinely needs O(n) stack space). Same shape as entry #5: correct, but hardcodes what a single rule already covers.

```python
def isValidParanthesis(self, s: str) -> bool:
    stack = []
    n = len(s)
    for i in range(n):
        if s[i] == "(" or s[i] == "[" or s[i] == "{":
            stack.append(s[i])
        else:
            if len(stack)==0:
                return False
            lastChar = stack[-1]
            if s[i] == ")" and (lastChar == "[" or lastChar == "{"):
                return False
            elif s[i] == "]" and (lastChar == "(" or lastChar == "{"):
                return False
            elif s[i] == "}" and (lastChar == "(" or lastChar == "["):
                return False
            else:
                stack.pop()
    return len(stack) == 0
```

**Issue:** For each closing bracket, it enumerates the two *wrong* opening brackets it must NOT match, across three near-identical `if`/`elif` branches (6 hardcoded pairings total: `)` vs `[`/`{`, `]` vs `(`/`{`, `}` vs `(`/`[`). The actual rule is one positive check: *pop the stack and compare it to the one correct opener for this closer* — a dict lookup replaces all six comparisons and removes the "did I list the wrong pair by mistake" risk entirely (same lesson as entry #5's Roman numeral pairs).

**Optimized:** same O(n) time / O(n) space — same complexity class, the win is fewer branches and ~3x fewer comparisons per closing bracket

```python
class Solution:
    def isValid(self, s: str) -> bool:
        pairs = {')': '(', ']': '[', '}': '{'}
        stack = []
        for c in s:
            if c in "([{":
                stack.append(c)
            else:
                if not stack or stack.pop() != pairs[c]:
                    return False
        return not stack
```

Verified against a brute-force reference (repeatedly stripping `()`, `[]`, `{}` until nothing changes) on 20,000+ random strings plus known tricky cases (`([)]`, `{[(])}`, unmatched leftovers like `(((`) — 0 mismatches. On a 10,000-char string, the dict version ran ~3.3x faster (0.35 ms vs 1.15 ms per call) purely from replacing the 3-way branch-with-two-comparisons-each with one dict lookup and one comparison.

**Takeaway:** This is the same pattern as entry #5: when you're checking "does X correctly pair with Y," look for the positive rule (*the one correct match*) instead of enumerating the incorrect matches to exclude. A `char -> its valid partner` dict is the standard, expected way to write this problem — worth having memorized, since Valid Parentheses / bracket-matching is one of the most common warm-up stack questions.

### #7 — Merge Two Sorted Lists (LeetCode 21)

**Constraints:** 0 ≤ list length ≤ 50 each. **My solution:** O(m+n) time, O(1) extra space — already asymptotically optimal (nodes are relinked, not copied). Issue is structural, not speed — though it does cost real constant factor.

```python
class Solution:
    def mergeTwoLists(self, list1, list2):
        dummy = ListNode(0)
        head = dummy
        while list1 or list2:
            if list1 and list2:
                if list1.val <= list2.val:
                    dummy.next = list1
                    dummy = list1
                    list1 = list1.next
                else:
                    dummy.next = list2
                    dummy = list2
                    list2 = list2.next
            elif list1:
                dummy.next = list1
                dummy = list1
                list1 = list1.next
            elif list2:
                dummy.next = list2
                dummy = list2
                list2 = list2.next
        return head.next
```

**Issues:**

1. **The variable names are inverted from the usual convention** — `head` holds the fixed sentinel node, while `dummy` is reassigned every iteration to be the moving tail pointer. The idiomatic pattern (used elsewhere in this cheatsheet, Section 20) is `dummy = tail = ListNode()`: `dummy` stays fixed as the anchor you return `.next` from, `tail` moves. Here it's backwards — harmless today since the sentinel is only read once at the end, but a real risk if the function grows: any code added later that assumes "dummy is the fixed anchor" (a very natural assumption, given the name) would silently operate on the wrong node.
2. **Three-way branching duplicates the same "attach + advance" logic once per case** (`list1 and list2`, `elif list1`, `elif list2`), and once one list is exhausted, it re-enters the loop and re-checks all three conditions for *every remaining node*, one at a time. Since a fully-consumed list only ever needs its remaining nodes attached once — the rest of the chain is already correctly ordered — this is wasted branching, not wasted asymptotic work (it's not overall a Big-O problem, just avoidable per-node overhead the size of `min(len(list1), len(list2))` extra branch evaluations).

**Optimized:** same O(m+n) time / O(1) space — one merge loop for the overlap, one wholesale attach for whatever's left

```python
class Solution:
    def mergeTwoLists(self, list1, list2):
        dummy = tail = ListNode()
        while list1 and list2:
            if list1.val <= list2.val:
                tail.next, list1 = list1, list1.next
            else:
                tail.next, list2 = list2, list2.next
            tail = tail.next
        tail.next = list1 or list2      # attach whichever list still has nodes, all at once —
        return dummy.next                # no need to walk it node-by-node, it's already sorted
```

Verified against a sorted-merge reference on 5,000+ random-length lists (including both empty, one empty, all-equal values, negatives) — 0 mismatches. Benchmarked with a deliberately lopsided case (10,000-node list merged against a 100-node list, so most of the run is "remainder" attachment): optimized was ~1.3x faster (377 ms vs 494 ms over 300 runs) purely from replacing the node-by-node remainder walk with one direct pointer assignment.

**Takeaway:** Two reusable habits. First, keep dummy-node naming consistent with intent — the sentinel that anchors the result should be the variable that *doesn't move* (Section 20's `dummy = tail = ListNode()` convention); swapping the names invites bugs in future edits even when today's code is correct. Second, when a loop is merging two sequences and one runs out, you don't need to keep looping node-by-node — once only one input remains, it's already in its final relative order, so splice it in with a single `tail.next = remaining` instead of iterating it.

### #8 — Remove Duplicates from Sorted Array (LeetCode 26)

**Constraints:** 1 ≤ nums.length ≤ 3·10⁴, nums sorted non-decreasing (so guaranteed non-empty on the judge).
**My solution:** O(n) time, O(1) space — right complexity class, but with a real, measurable constant-factor cost: roughly 2x the loop iterations actually needed. Also a latent edge-case bug.

```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        i = 0
        j = 0
        k = 1
        n = len(nums)
        while i<n and j<n:
            if nums[i]==nums[j]:
                    j = j+1
            else:
                nums[k] = nums[j]
                k = k+1
                i = j
        return k
```

**Issues:**

1. **A third pointer (`i`) tracks information `k` already has.** `i` exists to remember "the position of the last value we compared against," but after every write, `nums[k-1]` *is* that value — it's the most recently written unique element. Maintaining `i` separately, then doing `i = j` on every mismatch, means the very next loop iteration is a guaranteed-true, do-nothing self-comparison (`nums[i] == nums[j]` when `i == j`) that exists purely to advance `j` by one — a wasted trip through the loop body.
2. **Measured cost of that extra pointer:** I instrumented both versions to count loop iterations. The original consistently takes ~2.00x the iterations of the direct-comparison version (19,999 vs 9,999 at n=10,000) — not a Big-O change, but a real, reproducible constant-factor cost, confirmed by timing too (1135 ms vs 557 ms over 200 runs on a 50,000-element all-distinct array).
3. **Latent bug on empty input:** `k` is initialized to `1` and the `while` loop never runs when `n == 0`, so `removeDuplicates([])` returns `1`, not `0`. LeetCode's constraints guarantee `nums.length >= 1`, so this never surfaces on the judge — but it's the same category of issue as entry #3's `reverse()`: a function that's only correct because of what its caller happens to guarantee, not because it handles its stated input range.

**Optimized:** same O(n) time / O(1) space — the standard slow/fast two-pointer form, comparing directly against the last written value

```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        if not nums:
            return 0
        k = 1
        for j in range(1, len(nums)):
            if nums[j] != nums[k - 1]:
                nums[k] = nums[j]
                k += 1
        return k
```

This is exactly the `dedupe` template already in Section 16 (Two Pointers) of this cheatsheet — worth noting since it means this exact pattern was already on file before hitting it live. Verified against `len(set(nums))` on 5,000+ random sorted arrays (including all-equal and all-distinct) — every non-empty case matched; the only mismatches were the empty-array case, which confirms issue #3 above and is now fixed with an explicit guard.

**Takeaway:** When a pointer's only job is "remember the value at another pointer's last stopping point," check whether a value you're already writing (here, `nums[k-1]`) makes that pointer redundant — collapsing three pointers to two here also happened to delete the wasted self-comparison iteration. And once again (see entry #3): test the empty/boundary input directly, even when the problem's stated constraints rule it out — reused helpers don't come with the original problem's constraints attached.

### #9 — Remove Element (LeetCode 27)

**Constraints:** 0 ≤ nums.length ≤ 100, order of the result doesn't matter.
**My solution:** O(n) time, O(1) space — correct, and already the right complexity class. I checked it carefully for a correctness bug (a tuple-swap two-pointer variant is less common than plain overwrite, and worth being suspicious of) — it's genuinely correct, just doing more work than the operation needs.

```python
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        n = len(nums)
        i = 0
        j = 0
        while j<n:
            if nums[j] != val:
                nums[i], nums[j] = nums[j], nums[i]
                i = i+1
                j = j+1
            else:
                j = j+1
        return i
```

**Why it's correct (worth spelling out, since it's not the "obvious" version):** whenever `i` lags behind `j`, position `i` holds only ever a discarded `val` (or a val relayed forward by an earlier swap) — never an unexamined kept element — because `j` has already visited every index below itself. So swapping `nums[i]` and `nums[j]` is safe; it just also does more than necessary.

**Issue:** the full tuple swap `nums[i], nums[j] = nums[j], nums[i]` writes to **both** `nums[i]` and `nums[j]`, but nothing ever reads `nums[j]`'s new value again — once `j` moves past it, that slot is never revisited, and the problem doesn't require anything specific to be there (order doesn't matter, and only the first `i` elements are read by the caller). Relaying the junk value forward via swap is wasted work. It's most visible when `i == j` (no `val` seen yet): the swap becomes a complete no-op that still does 2 reads + 2 writes + a tuple allocation, for every single kept element before the first removal.

**Optimized:** same O(n) time / O(1) space — direct overwrite instead of swap

```python
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        i = 0
        for j, x in enumerate(nums):
            if x != val:
                nums[i] = x
                i += 1
        return i
```

Verified against a multiset comparison (order doesn't matter, so compared `Counter` of the first `i` elements, not exact sequences) on 5,000+ random arrays — 0 mismatches. Benchmarked on two shapes to see the constant-factor difference cleanly: swap-based 176 ms vs direct-overwrite 132 ms when few elements are removed, and 86 ms vs 76 ms when most are removed — the direct-overwrite version wins in both cases, as expected from doing strictly less work per element.

**Bonus — the interview follow-up ("optimize when elements to remove are rare"):** both versions above still touch every element once. When `val` is uncommon, a *different* algorithm does less total work by swapping a found `val` with an unexamined element from the **end** of the array and shrinking the array, instead of shifting everything after it:

```python
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        i, end = 0, len(nums)
        while i < end:
            if nums[i] == val:
                end -= 1
                nums[i] = nums[end]     # pull in an unexamined element from the back
            else:
                i += 1
        return i
```

This isn't better in general — it does *more* work than direct-overwrite when `val` is common (117 ms vs 76 ms in the benchmark, since every match forces reading from a shrinking, possibly also-matching tail) — but it's meaningfully faster when removals are rare (89 ms vs 132 ms), because it only touches elements near a match instead of shifting the entire remaining array forward by one slot each time. Good to have in your pocket as the answer to "can you do better if very few elements match?"

**Takeaway:** Even a correct, already-optimal-complexity two-pointer solution can carry avoidable per-element cost — check whether every write in the loop body is actually needed, or if one side of it (here, writing back into the slot you're reading from) is dead weight. And know that "two-pointer array filtering" has more than one shape: shift-in-place (this entry, and entries #2/#8) versus swap-with-the-end (this bonus) trade off differently depending on how common the thing you're filtering is — worth naming explicitly if an interviewer asks for the follow-up optimization.

### #10 — Find the Index of the First Occurrence in a String (LeetCode 28)

**Constraints:** 1 ≤ haystack.length, needle.length ≤ 10⁴.
**My solution:** O((n−m)·m) worst case (n = len(haystack), m = len(needle)) — same shape as entries #2 and #4: re-checks a full needle-length slice at every start position instead of reusing partial-match information from the previous attempt.

```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        n_needle = len(needle)
        n_haystack = len(haystack)
        n = n_haystack - n_needle
        i = 0
        while i<=n:
            if needle == haystack[i:i+n_needle]:
                return i
            else:
                i = i+1
        return -1
```

Correctness is solid, including the edge case your comments flagged: when `needle` is longer than `haystack`, `n` goes negative and `while i<=n` (with `i=0`) is immediately false, correctly returning -1 without needing an explicit guard — good instinct to check that case even though no code changed for it.

**Issue:** at each start position `i`, `haystack[i:i+n_needle] == needle` re-examines the needle length from scratch, even when the previous attempt already matched a partial prefix before failing. On adversarial input (e.g. `haystack = "a"*n`, `needle = "a"*(n//2) + "b"` — matches almost the whole needle every single time, then fails at the last character) this is genuinely O(n·m), not just a pessimistic bound.

**A wall-clock subtlety worth knowing (same lesson as entry #4):** at moderate sizes, this naive version can look *faster* than the fix below, because `haystack[i:i+n_needle] == needle` runs as a single fast C-level slice-and-compare, while a hand-written Python loop pays per-character interpreter overhead. The asymptotic gap only dominates once inputs get large enough — I pushed `n` up to find the actual crossover:

| n | naive | KMP | ratio |
|---|---|---|---|
| 20,000 | 3.7 ms | 5.3 ms | naive *faster* |
| 40,000 | 16.9 ms | 11.7 ms | 1.4x |
| 80,000 | 79.4 ms | 19.9 ms | 4.0x |
| 160,000 | 297.4 ms | 40.5 ms | 7.4x |

n doubling from 20,000 → 160,000 (8x) makes naive's time grow ~80x (≈ 8² — quadratic) while KMP's grows ~7.7x (≈ linear) — the signature that confirms O(n·m) vs O(n+m), not just a benchmark artifact. Below the crossover, "faster in my test" would have been the wrong conclusion to draw.

**Optimized:** KMP (Knuth-Morris-Pratt), O(n+m) worst case — the standard answer to "can you do better than brute force?" for substring search

```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        m = len(needle)
        if m == 0:
            return 0

        # Build the "longest proper prefix that's also a suffix" table for needle.
        # lps[k] = length of the longest prefix of needle[:k+1] that's also a suffix of it.
        lps = [0] * m
        length = 0
        i = 1
        while i < m:
            if needle[i] == needle[length]:
                length += 1
                lps[i] = length
                i += 1
            elif length:
                length = lps[length - 1]     # fall back using previously-computed matches
            else:
                lps[i] = 0
                i += 1

        # Scan haystack once. On a mismatch, use lps to skip re-comparing the needle
        # prefix we already know matches — never rewind the haystack pointer `i`.
        i = j = 0
        n = len(haystack)
        while i < n:
            if haystack[i] == needle[j]:
                i += 1; j += 1
                if j == m:
                    return i - j
            elif j:
                j = lps[j - 1]
            else:
                i += 1
        return -1
```

Verified against Python's own `str.find()` as an oracle on 20,000+ random strings (small alphabets to force lots of partial matches) plus known tricky cases (`"aaaaaaaaab"`/`"aaab"`, `"mississippi"`/`"issip"`) — 0 mismatches.

**Takeaway:** Third time this exact shape has shown up (see entries #2 and #4): "does re-attempting from the next start position throw away information the previous attempt already established?" is the question that turns O(n·m) into O(n+m) for substring/window problems. KMP's `lps` table is the general tool for that specific case — precompute, for every prefix of the pattern, how far you could fall back to without losing a possible match, so a mismatch never means "start over." And separately: don't trust a single wall-clock comparison at one input size to settle an asymptotic question — check whether the gap *grows* with n, not just whether it exists.

### #11 — Longest Palindromic Substring (LeetCode 5)

**Constraints:** 1 ≤ s.length ≤ 1000.
**My solution:** O(n³) worst case — generates all O(n²) substrings, and checking each with `s_sub == s_sub[::-1]` costs up to O(n) itself, so the two multiply. Your own comments correctly named this as "Brute Force Approach: O(n²)" for the substring generation, but the palindrome check inside adds a third factor that the comment didn't account for.

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        n = len(s)
        maxPalLen = 0
        posx = -1
        posy = -1
        for i in range(n):
            for j in range(i,n):
                s_sub = s[i:j+1]
                if s_sub == s_sub[::-1] and len(s_sub)>maxPalLen:
                    maxPalLen = max(maxPalLen, len(s_sub))
                    posx = i
                    posy = j+1
        return s[posx:posy] if posx!=-1 else ""
```

**Issue:** at `n = 1000` (the actual LeetCode constraint ceiling, not a hypothetical), this is a genuine TLE risk, not just theoretically slow. I benchmarked the true worst case — a string of one repeated character, where every substring *is* a palindrome, so the `s_sub == s_sub[::-1]` check never short-circuits early and always does the full O(length) comparison:

| n | brute force | expand-around-center | ratio |
|---|---|---|---|
| 200 | 3.8 ms | 1.24 ms | 3x |
| 500 | 34.1 ms | 8.82 ms | 4x |
| 1000 | 203.5 ms | 35.9 ms | 6x |

The ratio climbing as n grows (not staying flat) confirms this is a real complexity gap, not just a constant-factor difference — and 200 ms for a single worst-case call at the stated constraint ceiling is the kind of number that turns into a judge timeout once a test suite runs many cases.

**Optimized:** expand around center, O(n²) time, O(1) space — the standard interview-expected answer

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        if not s:
            return ""
        start, end = 0, 0

        def expand(l, r):
            while l >= 0 and r < len(s) and s[l] == s[r]:
                l -= 1
                r += 1
            return l + 1, r - 1        # last valid (l, r) before the mismatch

        for i in range(len(s)):
            l1, r1 = expand(i, i)          # odd-length palindrome centered at i
            if r1 - l1 > end - start:
                start, end = l1, r1
            l2, r2 = expand(i, i + 1)      # even-length palindrome centered between i, i+1
            if r2 - l2 > end - start:
                start, end = l2, r2
        return s[start:end + 1]
```

The key idea: instead of generating every substring and checking each independently (throwing away all prior work, same root cause as entries #2/#4/#10), treat every position — and every gap between two positions — as the *center* of a candidate palindrome and grow outward only as far as it stays valid. There are exactly `2n - 1` centers, and each expansion does no more than O(n) work, giving O(n²) total instead of O(n²) substrings × O(n) each.

Verified against an independently-built O(n²) DP table (`dp[i][j] = s[i]==s[j] and dp[i+1][j-1]`, a genuinely different algorithm from both solutions above, to avoid the two sharing a blind spot) on 3,000+ random and edge-case strings, checking length, that each returned string is actually a palindrome, and that it's actually a substring of `s` — since LeetCode accepts *any* maximum-length answer when ties exist, not necessarily an identical string. 0 mismatches.

**Bonus — the O(n) answer if pushed further:** Manacher's algorithm finds the longest palindromic substring in linear time by reusing palindrome-length information from previously-computed centers instead of expanding each one from scratch (via mirroring around a rightmost-reaching center). It's rarely required in an interview for this specific problem, but worth being able to *name* as "the O(n) approach exists, here's roughly how it works" if asked to push past O(n²).

**Takeaway:** A comment that labels a nested double-loop "O(n²)" is only accounting for the substring generation — always check whether the work done *inside* the loop (here, `s_sub == s_sub[::-1]`, itself O(length)) adds another factor before trusting your own complexity comment. And expand-around-center is this cheatsheet's fourth encounter with the same underlying idea as entries #2/#4/#10: don't re-derive from scratch what a smarter traversal order lets you build incrementally — here, by growing outward from a shared center instead of re-testing every independent substring.

### #12 — Container With Most Water (LeetCode 11)

**Constraints:** 2 ≤ height.length ≤ 10⁵.
**My solution:** O(n) time, O(1) space — this is the correct, fully optimal two-pointer solution on the first attempt. No fix needed; logging it as a clean baseline entry, since the log should also capture what's already right, not only what's wrong.

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        i, j = 0, len(height)-1
        maxArea = 0
        while i<j:
            area = min(height[i], height[j])*(j-i)
            maxArea = max(area, maxArea)
            if height[i]<height[j]:
                i = i+1
            else: 
                j = j-1
        return maxArea
```

**What I checked, since "already optimal" is a claim worth verifying rather than assuming:** ran it against an independently-written O(n²) brute force (`max over all i<j of min(height[i],height[j])*(j-i)`) on 5,000+ random arrays, plus adversarial shapes chosen to stress the two-pointer logic specifically — strictly increasing, strictly decreasing, all-equal (plateau, forces the tie-break branch on every step), zigzag, and boundary heights of 0. 0 mismatches, including the classic `[1,8,6,2,5,4,8,3,7] → 49` example. Scales linearly as claimed: ~17 ms for a single call at the actual constraint ceiling (n=100,000), where the brute force wouldn't finish in reasonable time (extrapolating from n=2000 already taking ~930 ms, growing quadratically).

**Why the greedy pointer-move is actually correct (worth being able to say out loud, since interviewers often ask "why does that work?" as a follow-up, not just "does it work"):** at each step, the current area is bounded above by `min(height[i], height[j]) * (j - i)`. If you move the pointer at the *taller* bar inward instead of the shorter one, the width shrinks but the height is still capped by the same shorter bar (or something shorter) — so every pairing you'd check with the taller bar held fixed is provably no better than the one you already have. Moving the *shorter* bar is the only move that has a chance of finding something taller, and therefore possibly better, at a smaller width. This eliminates a whole swath of the O(n²) pairs without ever looking at them individually — which is what gets this down from O(n²) to O(n): each pointer only ever moves inward, so the total number of steps across the whole run is bounded by n, not n².

**Nothing to optimize algorithmically, so a few small polish notes instead:** `maxArea = max(area, maxArea)` reads slightly more naturally as `maxArea = max(maxArea, area)` (accumulator first) — purely cosmetic, not a bug. And the up-front comments walking through the use cases (single-max-area ties, monotonic arrays, size-2 arrays) before writing any code is a good habit worth keeping — it's the same kind of edge-case reasoning that would have caught the empty-array bug in entry #8 if it had been applied there too.

**Takeaway:** Not every review finds something to fix — the log should include confirmations, not just corrections, so "I already know this pattern" is something you can trust rather than re-derive under pressure. The proof sketch above (why discarding the taller pointer's pairings is always safe) is the reusable piece: it's the same "eliminate a dominated region instead of checking it" reasoning that shows up in other two-pointer problems (Section 16), not just this one.

### #13 — 3Sum (LeetCode 15)

**Constraints:** 3 ≤ nums.length ≤ 3000. **This one has a genuine correctness bug, not just a performance issue** — worth reading carefully, since it's the kind of bug that can pass casual eyeballing and even pass by luck on a small hand-traced example.

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        n = len(nums)
        numMap = {} # value, list
        for i in range(n):
            if i in numMap:
                pos = numMap[i]
            else: 
                pos = []
            pos.append(i)
            numMap[nums[i]] = pos

        result = []
        for i in range(n):
            for j in range(i+1, n):
                temp = -1*(nums[i] + nums[j])
                if temp in numMap:
                    pos = numMap[temp]
                    for k in pos:
                        if k not in (i, j):
                            tup = [nums[i], nums[j], nums[k]]
                            result.append(tup)
        return result
```

**Root cause — a value/index mix-up that then triggers a second, worse bug:** `numMap` is meant to map *value → list of indices holding that value*, so the lookup to check "have I seen this value's index list before" should be `if nums[i] in numMap`. It's instead `if i in numMap` — checking whether the **loop index** happens to collide with a **key already in the map** (which are values from `nums`, not indices). These are unrelated quantities that just happen to share a type, so Python never raises an error; it silently does the wrong thing.

When that coincidental collision happens, `pos = numMap[i]` retrieves an *existing* list — but it's the wrong one, and worse, it's a **mutable reference**, not a copy. Appending to `pos` and then doing `numMap[nums[i]] = pos` means two different dict keys can end up pointing to the *literal same list object*. This is the exact aliasing gotcha already in Section 3 of this cheatsheet ("`b = a` aliases, doesn't copy") and Section 1's mutable-default-argument gotcha — same root mechanism, just triggered through a dict instead of a default parameter.

**Concrete minimal repro** — `nums = [1, 2, -2, -1]` has **no** valid zero-sum triplet (I checked all 4 combinations by hand: none sum to 0), so the correct answer is `[]`. The buggy code returns `[[1, -2, 2]]` — a triplet that sums to **1, not 0**:

```
numMap ends up: {1: [0, 1, 2], 2: [0, 1, 2], -2: [0, 1, 2], -1: [3]}
```

Three different keys (`1`, `2`, `-2`) all point to the *same* `[0, 1, 2]` list — because at `i=1`, `nums[1]=2` happened to equal a key already in the map (`1`, inserted at `i=0`), triggering the retrieve-and-mutate-in-place bug; then at `i=2`, `nums[2]=-2` triggered it again against the now-corrupted list. The lookup for `k` then returns indices that don't actually hold the target value, producing a triplet that doesn't sum to zero at all.

**How often this actually bites, measured against 3,000+ random arrays (values in a small range, so index/value collisions are common — which is realistic, not a contrived edge case):**
- 1,142 of 3,005 cases (38%) produced at least one **incorrect** triplet — one that doesn't actually sum to zero.
- 1,943 of 3,005 cases (65%) produced **duplicate/permutation entries** of the same triplet in the output — e.g. `[1,-1,0]` and `[-1,0,1]` and `[0,1,-1]` all listed separately. LeetCode 15 explicitly requires *"the solution set must not contain duplicate triplets"* — this is a second, independent bug even setting the map corruption aside: nothing here deduplicates by value.
- 0 of 3,005 cases had a valid triplet go completely *missing* — the corruption tends to manifest as wrong/duplicate answers rather than silently dropping correct ones, but that's incidental to this bug, not something to rely on.

**Optimized (and corrected):** sort + two-pointer, O(n²) time (O(n log n) sort + O(n²) scan), O(1) extra space — the standard approach for "N numbers summing to target, no duplicate results," and it sidesteps the whole class of bug above because dedup falls out of the sorted order instead of needing separate bookkeeping

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        n = len(nums)
        res = []
        for i in range(n - 2):
            if i > 0 and nums[i] == nums[i - 1]:
                continue                   # skip duplicate 'i' — avoids duplicate triplets
            if nums[i] > 0:
                break                       # smallest remaining value is positive -> no triplet can sum to 0
            l, r = i + 1, n - 1
            while l < r:
                s = nums[i] + nums[l] + nums[r]
                if s < 0:
                    l += 1
                elif s > 0:
                    r -= 1
                else:
                    res.append([nums[i], nums[l], nums[r]])
                    l += 1; r -= 1
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1              # skip duplicate 'l'
                    while l < r and nums[r] == nums[r + 1]:
                        r -= 1              # skip duplicate 'r'
        return res
```

Verified against an independent brute-force reference (`itertools.combinations` over all triples, checked for sum-zero, deduplicated as sorted tuples) on 3,886 random and edge cases (all-zeros, empty, length-3, many duplicate values) — 0 mismatches, and 0 cases with duplicate entries in the raw output (versus 65% for the original).

**Takeaway:** Two habits worth locking in from this one. First, when building a `value -> list of indices` map, the membership check must test the *same thing* the keys are — mixing up "is this index present" with "is this value present" is a bug Python will never flag for you, since both sides are just integers; a quick sanity print of the map's shape on a tiny example would have caught this immediately. Second, retrieving a mutable container from a dict/list and continuing to mutate it (`pos = numMap[i]; pos.append(...)`) always carries aliasing risk — if that same object could plausibly get stored under more than one key, copy it (`pos = list(numMap[i])`) or, better, use `numMap.setdefault(nums[i], []).append(i)` so there's only ever one dict access and no ambiguity about which key owns which list. And for "find all fixed-size combinations summing to a target, deduplicated" problems generally: sort + multi-pointer (this entry) usually beats a hashmap approach precisely because dedup is nearly free once the array is sorted, rather than needing to be bolted on with a separate `set` of canonicalized results afterward.

### #14 — Intersection of Two Arrays (LeetCode 349)

**Constraints:** 1 ≤ nums1.length, nums2.length ≤ 1000. **My solution:** O(n1+n2) time, O(n1+n2) space — already the right complexity (can't beat looking at every element at least once). One small redundant check, and a shorter idiomatic version worth knowing.

```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        n1 = set(nums1)
        res = set()
        for num in nums2:
            if num in n1 and num not in res:
                res.add(num)
        return list(res)
```

**Issue:** `num not in res` is checked before every `res.add(num)`, but `set.add()` is already a no-op when the value is present — adding a duplicate to a set doesn't error, doesn't grow it, and doesn't need to be guarded against. The check doesn't change the result at all; it just spends an extra O(1) lookup on every iteration of `nums2` for no behavioral difference.

**Optimized:** same O(n1+n2) time/space — drop the redundant guard

```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        n1 = set(nums1)
        res = set()
        for num in nums2:
            if num in n1:
                res.add(num)
        return list(res)
```

**Even more idiomatic — Python's set intersection operator does exactly this problem in one line:**

```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1) & set(nums2))
```

Verified all three against `set(nums1) & set(nums2)` as the reference on 5,000+ random cases (including empty arrays, all-duplicates, and no-overlap cases) — 0 mismatches, as expected since none of the three change what's being computed. Benchmarked on two 50,000-element arrays: dropping the redundant check gave a modest but real ~5% speedup (209 ms vs 222 ms over 50 runs); the one-liner came in about the same as the original, since `&` on two sets does comparable underlying work in C either way.

**Takeaway:** Know what your data structure already guarantees before adding a manual check for it — a set never needs a "have I already added this?" guard, since that's precisely what `.add()` already handles internally. This is a smaller-scale version of the redundant-write lesson in entry #9 (`removeElement`): before adding a condition, ask whether the operation you're guarding is already safe to call unconditionally.

---

### #15 — Two Sum II - Input Array Is Sorted (LeetCode 167)

**Constraints:** array sorted ascending, exactly one solution guaranteed, 1-indexed answer required. **My solution:** two-pointer scan from both ends, O(n) time, O(1) extra space — this is already the optimal approach, nothing to improve.

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        # sorted array ascending
        i, j = 0, len(numbers)-1

        while i<j:
            if numbers[i] + numbers[j] == target:
                return [i+1, j+1]
            elif numbers[i] + numbers[j] > target:
                j = j-1
            else:
                i = i+1
            
        return []
```

**Issue:** none — this is the textbook-correct use of the sorted-array two-pointer pattern. Each iteration either finds the answer or shrinks the search window by one from whichever side is provably wrong (if the current pair sums too high, the largest element can never work with anything to its left either, so drop it; symmetric logic for too low), and the window only shrinks, so the whole scan is at most n steps. It's worth noticing *why* this beats the general Two Sum (LeetCode 1) hashmap approach here specifically: that problem has an unsorted array, so a hashmap (O(n) time, O(n) space) is necessary. Here the sort order is a gift — it lets two pointers do the same O(n) time in O(1) space, with no hashing overhead at all.

**Verified** the solution's returned indices against a brute-force O(n²) double loop across 20,000 randomized cases (sorted arrays with duplicates and negatives, targets both guaranteed-solvable and random) — for every case the returned pair (when non-empty) summed correctly to target with valid 1-indexed positions, and every case where brute force found nothing, this solution also correctly returned `[]`. 0 mismatches. Hand-checked edge cases (2-element array, all-duplicate array, negative numbers, target requiring the two endpoints) all passed.

**Benchmarked** against a hashmap version on a worst-case 200,000-element array (target chosen so no pair sums to it, forcing a full scan): two-pointer averaged ~25 ms/run vs ~45 ms/run for the hashmap approach — almost 2x faster — while `tracemalloc` showed the two-pointer version used about 96 bytes of extra memory versus roughly 20 MB for the hashmap's dictionary. So it's a clean win on both axes here, not a time/space tradeoff.

**Takeaway:** When you're handed a sorted array, always ask whether that ordering can replace a hashmap. A hashmap turns "have I seen the complement?" into an O(1) lookup at the cost of O(n) space; sorted order often lets two pointers answer the same question by using the fact that moving one side changes the sum monotonically, giving you O(n) time in true O(1) space instead. This is the same family of insight as sliding window and monotonic stack — exploiting a structural property of the input (sortedness, monotonicity) to avoid paying for extra memory a general-purpose data structure would need.

---

### #16 — Boats to Save People (LeetCode 881)

**Constraints:** 1 ≤ people.length ≤ 5×10⁴, 1 ≤ people[i] ≤ limit ≤ 3×10⁴. **My solution:** sort + two pointers, O(n log n) time (dominated by the sort), O(1) extra space. Correct, but built from three overlapping `if/elif` branches where one is doing something subtly odd — worth understanding exactly why it still works.

```python
class Solution:
    def numRescueBoats(self, people: List[int], limit: int) -> int:
        people.sort()
        i, j = 0, len(people)-1
        numBoats = 0
        while i<=j:
            if people[j] == limit:
                numBoats += 1
                j = j-1
            elif people[i] + people[j] <= limit:
                numBoats += 1
                i = i+1
                j = j-1
            elif people[i] + people[j] > limit and people[j]<limit:
                numBoats += 1
                j = j-1
        return numBoats
```

**Issue:** the three branches split into "heaviest alone because they exactly fill the boat," "pair heaviest with lightest," and "heaviest alone because pairing overflows" — but the first branch is actually unreachable-as-written in the sense that it's just a special case already covered by the general pattern; it exists only because the code re-checks `people[i] + people[j]` from scratch instead of noticing that once you know whether the lightest remaining person fits with the heaviest, everything else follows. There's also a subtler thing worth flagging: when `i == j` (one person left), `people[i] + people[j]` double-counts that single person's weight against themselves. I checked whether this produces wrong answers, and it doesn't — every branch that can trigger in that state still results in exactly one boat being added and the loop terminating, so the double-counted sum never leaks into the wrong branch being taken. But it's fragile: the code is correct by coincidence of how the branches are structured, not because it was designed to handle `i == j` explicitly, and a small refactor elsewhere could break it.

**Optimized — same O(n log n) time, same O(1) space, but expressed as the actual algorithm ("try to pair the lightest with the heaviest; either way, the heaviest always gets on a boat"):**

```python
class Solution:
    def numRescueBoats(self, people: List[int], limit: int) -> int:
        people.sort()
        i, j = 0, len(people) - 1
        numBoats = 0
        while i <= j:
            if people[i] + people[j] <= limit:
                i += 1
            j -= 1
            numBoats += 1
        return numBoats
```

This reads directly as the greedy proof: the heaviest remaining person always needs a boat (nobody heavier is left to share with more cheaply), so `j` always advances. The only question is whether the *lightest* remaining person can ride along for free, which is exactly the `if` condition. When `i == j`, the same "does person fit with themselves" check still resolves correctly — if `2 * people[i] <= limit` then `i` advances past `j`, otherwise it doesn't, and either way `j` drops below `i` and the loop ends having counted exactly one boat for that last person.

**Verified** both versions against an independently-coded reference (a deque-based greedy pop-from-both-ends implementation, structurally different code path from either solution) across 20,000 randomized cases — array sizes 1–40, limits 1–50, including many all-duplicate and single-element arrays to specifically stress the `i == j` edge case. 0 mismatches for both. Explicit hand-checked edge cases (`[5]`, `[3,3]` with limit 3, 50 copies of `1` with limit 2, etc.) also matched.

**Benchmarked** at the problem's max constraints (n=50,000, limit=30,000): both versions run at ~10 ms/call, confirming the simplification is purely a readability win, not a speed one — Python's sort dominates either way and the branch count inside the loop barely matters next to that. I also checked a real Big-O lever: since weights are bounded by `limit` rather than arbitrary, a counting sort (`O(n + limit)` instead of `O(n log n)`) is possible in principle. Building the sorted array via a counts bucket first and then running the same two-pointer pass verified correct on the same 20,000 cases, and at n=50,000/limit=30,000 it ran in ~6.6 ms vs ~9.9 ms for `.sort()` — a real ~33% win when n and limit are close in magnitude. But I also benchmarked a more realistic shape (n=1,000, limit=30,000, i.e. limit ≫ n): there, counting sort was over 10x *slower* (~0.85 ms vs ~0.07 ms), because it has to allocate and scan a 30,001-length bucket array no matter how few people are actually being sorted. So counting sort is a genuine but conditional win here — worth knowing the trick exists, not worth reaching for by default.

**Takeaway:** two lessons stacked in one problem. First, when you find yourself writing multiple `if/elif` branches that recompute the same comparison to reach subtly different verdicts, ask whether they're really expressing one decision (here: "does the lightest fit with the heaviest?") dressed up as three — collapsing them isn't just prettier, it removes room for an edge case like `i == j` to be correct by accident instead of by design. Second, "the value range is bounded, so I could counting-sort instead of comparison-sort" is a real O(n log n) → O(n + limit) upgrade, but it's only a win when `limit` isn't much larger than `n` — always weigh the bucket array's fixed setup cost against the actual input size before swapping in a bounded-range trick, rather than assuming a better asymptotic bound always wins in practice.

---

### #17 — Combination Sum (LeetCode 39)

**Constraints:** 2 ≤ candidates.length ≤ 30, all candidates distinct, 1 ≤ target ≤ 40, unlimited reuse of each candidate allowed. **This one took three rounds to land** — worth keeping the full journey, since each round fixed one real bug and the sequence itself is the lesson.

**Round 1 — the original submission:**

```python
def recurCombSum(self, candidates: list[int], target: int, res: list[list[int]], recurCombRes: list[int]):
        for candidate in candidates:
            if candidate==target:
                recurCombRes.append(candidate)
                res.append(recurCombRes)
                return res
            elif candidate<target:
                recurCombRes.append(candidate)
                res = self.recurCombSum(candidates, target-candidate, res, recurCombRes)
                recurCombRes.pop()
        
        return res

    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        if len(candidates) == 0:
            return [[]]
        return self.recurCombSum(candidates, target, [], [])
```

Two independent bugs here, both confirmed by running it: `res.append(recurCombRes)` stores a *reference* to the in-progress path, not a snapshot — since `recurCombRes` keeps getting mutated by later backtracking, every entry that ends up in `res` ends up identical (same aliasing bug as entry #13's `numMap`). And `return res` inside the `if candidate==target` branch exits the current frame's loop entirely the moment one completion is found, instead of continuing to look for other candidates. On `[2,3,6,7], target=7` this returned `[[2,2,3,7],[2,2,3,7],[2,2,3,7],[2,2,3,7]]` — wrong values (should be `[[2,2,3],[7]]`) and duplicated four times over from the aliasing.

**Round 2 — fixed the aliasing and the early return, but introduced a different problem:**

```python
def recurCombSum(self, candidates, target, res, recurCombRes):
        for candidate in candidates:
            if candidate==target:
                recurCombRes.append(candidate)
                res.add(tuple(sorted(tuple(recurCombRes.copy()))))
                recurCombRes.pop()
            elif candidate<target:
                recurCombRes.append(candidate)
                res = self.recurCombSum(candidates, target-candidate, res, recurCombRes)
                recurCombRes.pop()
        return res

    def combinationSum(self, candidates, target):
        res = set()
        return list(self.recurCombSum(candidates, target, res, []))
```

Correct this time — verified against an independent start-index oracle on 200 random cases plus the LeetCode examples, 0 mismatches — but because the recursion still restarts from the full `candidates` list on every call, it generates every *ordering* of each valid multiset (`[2,3]` and `[3,2]` both get explored as separate branches) and relies on `set(sorted(tuple(...)))` to collapse the duplicates after the fact. I benchmarked this against a version that never generates the duplicate orderings: at `target=20` it was already ~12x slower (0.62ms vs 0.05ms), and by `target=30` it was over 250x slower (33ms vs 0.13ms) — an exponential gap that gets worse the higher the target goes, since the number of orderings of a multiset grows factorially with its length.

**Round 3 — added a start-index parameter, but passed the wrong variable to the recursive call:**

```python
def recurCombSum(self, selectedIndex, candidates, target, res, recurCombRes):
        n = len(candidates)
        for i in range(selectedIndex, n):
            candidate = candidates[i]
            if candidate==target:
                recurCombRes.append(candidate)
                res.append(recurCombRes.copy())
                selectedIndex = i
                recurCombRes.pop()
            elif candidate<target:
                recurCombRes.append(candidate)
                res = self.recurCombSum(selectedIndex, candidates, target-candidate, res, recurCombRes)
                recurCombRes.pop()
        return res
```

Right idea — restrict the loop to `range(selectedIndex, n)` so a completed choice can't be revisited — but the recursive call passes `selectedIndex` (this frame's *original* starting point) instead of `i` (the position just chosen), so every deeper call is still free to walk backward to wherever this frame started. On `[2,3,6,7], target=7` this produced `(2,2,3)` three times over; on `[6,4,2,1], target=8` it produced 60 entries where only 11 were correct.

**Final, correct and efficient version:**

```python
class Solution:
    def recurCombSum(self, selectedIndex, candidates: list[int], target: int, res, recurCombRes: list[int]):
        n = len(candidates)
        for i in range(selectedIndex, n):
            candidate = candidates[i]
            if candidate==target:
                recurCombRes.append(candidate)
                res.append(recurCombRes.copy())
                recurCombRes.pop()
            elif candidate<target:
                recurCombRes.append(candidate)
                res = self.recurCombSum(i, candidates, target-candidate, res, recurCombRes)
                recurCombRes.pop()
        
        return res

    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        if len(candidates) == 0:
            return [[]]
        res = []
        res =  self.recurCombSum(0, candidates, target, res, [])
        return res
```

This is O(candidates.length) per branch decision with no wasted exploration — every recursive call only ever looks forward from where its caller left off, so no ordering of the same multiset is ever generated twice, and nothing needs to be deduplicated afterward.

**Verified** against an independent start-index backtracking oracle on 3,000 randomized cases (candidate values up to 15, targets up to 20) plus the LeetCode examples — 0 mismatches, and confirmed the raw output itself has no duplicate entries (not just "duplicates that happen to normalize the same," an explicit separate check). **Benchmarked** against a reference implementation of the same algorithm shape at targets 15/25/35: times tracked within ~10% of each other (0.03ms vs 0.04ms, 0.13ms vs 0.17ms, 0.41ms vs 0.44ms) — confirming the exponential blowup from Round 2 is genuinely gone, not just reduced. Also checked `[2,2,3], target=4` (a candidates list with a literal duplicate value, which LeetCode 39's constraints rule out but its sibling problem Combination Sum II, LeetCode 40, explicitly allows): this version still produces `(2,2)` three times over in that case, confirming the fix here is specific to *ordering* duplicates, not *value* duplicates — worth knowing before reusing this code for LeetCode 40 unmodified.

**Takeaway:** two lessons, one from each half of the journey. First (Round 1), the same aliasing trap as entry #13: appending a mutable object you're still going to backtrack-and-mutate always needs a `.copy()` at the point of appending, not just "eventually." Second (Rounds 2 and 3), for "combinations with unlimited reuse" backtracking, the fix for not revisiting earlier choices isn't a post-hoc dedup step over generated results — it's a start-index threaded through the recursion, and the index you pass down must be *where you currently are* (`i`), not *where your own search began* (`selectedIndex`/`start`). Getting that one variable wrong reintroduces the exact duplication problem the index parameter was meant to solve, just in a different shape — which is why re-testing against the same two or three examples after every change (here, `[2,3,6,7], target=7`, `[2,3,5], target=8`) is what actually caught it, rather than eyeballing the diff.

---

### #18 — Subsets (LeetCode 78)

**Constraints:** 1 ≤ nums.length ≤ 10, all elements distinct. **Two-round problem — the first attempt accidentally combined two independent, already-complete backtracking templates into one function, which doesn't just look redundant, it changes the actual growth rate of the algorithm.**

**Round 1 — the original submission:**

```python
def recurSubset(self, nums, index, res, subset):
    n = len(nums)

    if index==n:
        res.append(subset.copy())

    for i in range(index, n):
        subset.append(nums[i])
        self.recurSubset(nums, i+1, res, subset) #selecting
        subset.pop()

        self.recurSubset(nums, i+1, res, subset) #not selecting

    return res

def subsets(self, nums: list[int]) -> list[list[int]]:
    return self.recurSubset(nums, 0, [], [])
```

I ran this before doing anything else: `nums=[1,2]` should return exactly 4 subsets, but returned 6, with `[2]` and `[]` each appearing twice. I measured the exact output size across `n = 1..8`: 2, 6, 18, 54, 162, 486, 1458, 4374 — that's `2 * 3^(n-1)`, tripling at every level instead of doubling. Not "some duplicates mixed into the right answer" — a different, worse exponential growth rate than the correct `O(2^n)`.

**Root cause:** the code mixes two separate, independently-complete templates for generating subsets. One is the `for i in range(index, n)` loop itself (the same shape as `combine()` and `combination_sum` already in Section 26) — in that template, the loop *is* the exclusion mechanism: advancing from trying `i` to trying `i+1` already means everything between them was left out, with nothing extra needed to say so. The other is the explicit second recursive call, `self.recurSubset(nums, i+1, res, subset) #not selecting` — that's the mechanism from the *other* standard template (binary include/exclude, one recursive call per element position, no loop involved at all). Layering the second template's "skip" call on top of the first template's loop means every position's "don't include this" decision gets explored more than once, through two different paths that both eventually converge on the same remaining subsets — hence the multiplicative-per-level blowup rather than a flat duplicate count.

**Round 2 — dropped the loop, kept only the include/exclude recursion:**

```python
class Solution:
    def recurSubset(self, nums, index, res, subset):
        n = len(nums)

        if index==n:
            res.append(subset.copy())
            return res

        subset.append(nums[index])
        self.recurSubset(nums, index+1, res, subset) #selecting
        subset.pop()
        self.recurSubset(nums, index+1, res, subset) #not selecting

        return res

    def subsets(self, nums: list[int]) -> list[list[int]]:
        return self.recurSubset(nums, 0, [], [])
```

This is the standard binary include/exclude template, and it's correct: each call makes exactly one decision (include `nums[index]` or don't) and both branches advance `index` by exactly one, so there's no way for two different paths to reconverge on the same partial state.

**Verified** against Python's own `itertools.combinations` (summed over all sizes `0..n`, as an independent reference) on 500 randomized arrays (sizes 0–8, values including negatives) plus hand-picked edge cases (empty array, single element) — 0 mismatches, and confirmed no duplicate entries in the raw output at any size. **Growth check:** exact `2^n` at every `n` from 1 to 15 (2, 4, 8, ..., 32768), confirming Round 1's `3^n` blowup is fully gone, not just reduced. This is also already the best possible complexity for this problem: any correct solution must output `2^n` subsets, so `O(2^n)` time is a hard lower bound, not just "good enough."

**Takeaway:** when two backtracking templates both look like plausible ways to generate the same kind of output (here: a start-index loop that advances via recursion, versus a binary include/exclude recursion with no loop), pick one and commit to it — don't reach for a piece of each. They're not composable "half solutions" that combine into something more thorough; each one is already a complete algorithm on its own, and stacking them multiplies redundant work instead of adding coverage. The tell that this happened here was the output *count* growing at the wrong rate (`3^n` instead of `2^n`) rather than just "a few extra duplicates" — a reminder that when debugging combinatorial backtracking, checking the exact result size against the known closed-form (here, `2^n`) is a fast, cheap sanity check worth doing before diving into a full oracle comparison.

---

### #19 — Longest Increasing Subsequence (LeetCode 300)

**Constraints:** 1 ≤ nums.length ≤ 2500, -10^4 ≤ nums[i] ≤ 10^4. Solved directly with the O(n log n) patience-sorting approach — no O(n²) attempt first this time, since this problem was introduced specifically as the stepping stone toward the O(n log n) reduction needed for Russian Doll Envelopes (entry pending).

**Submission:**

```python
class Solution:
    def lengthOfLIS(self, nums: list[int]) -> int:
        seq = []
        n = len(nums)

        for i in range(0, n):
            index = bisect.bisect_left(seq, nums[i])

            if len(seq)>0 and index<len(seq): #Seq exists and element is added in the middle - same element or diff element
                seq[index] = nums[i]
            else:
                seq.append(nums[i])
        return len(seq)
```

**How it works:** `seq` is not "the actual longest increasing subsequence" — it's an auxiliary array where `seq[k]` holds the *smallest possible tail value* achievable by an increasing subsequence of length `k+1`, given everything scanned so far. This array is always sorted, which is what makes binary search valid on it. For each new number: `bisect_left(seq, nums[i])` finds the first position whose value is `>= nums[i]`. If found, that position's value gets overwritten with `nums[i]` (a strictly smaller-or-equal tail for that length is always at least as good going forward — it can only make future extensions easier, never harder). If nothing qualifies (i.e. `nums[i]` is bigger than every tail so far), it gets appended, extending the longest length found by one. The final answer is just `len(seq)` — the array's *length* tracks the LIS length; its contents are a valid witness only for tail values, not necessarily an actual subsequence from the array.

`bisect_left` specifically (not `bisect_right`) is the correct choice here because LIS is strictly increasing: `bisect_left` finds the leftmost position with a value `>= nums[i]`, so an equal value gets overwritten rather than the new element being inserted after it — which correctly refuses to extend a subsequence with a duplicate value. (`bisect_right` would find the position after any equal values, which would be wrong for the strict variant of this problem, though it's what you'd want for a non-decreasing/non-strict LIS variant instead.)

The `len(seq)>0 and index<len(seq)` guard is redundant but harmless: when `seq` is empty, `bisect_left` on an empty list always returns `0`, which already equals `len(seq)` (`0`), so the `else` branch (append) fires regardless of whether the `len(seq)>0` check is there.

**Verified** against an independent O(n²) DP reference (`dp[i]` = LIS length ending at index `i`, taking the max over all valid predecessors) on the LeetCode example set plus edge cases (empty array, single element, strictly increasing, strictly decreasing, all-equal) — all matched — and on 5,000 small randomized arrays (n ≤ 30) plus 300 larger randomized arrays (n ≤ 300): 0 mismatches across all 5,300+ trials.

**Benchmarked** to confirm this is a genuine complexity win, not a constant-factor illusion: at n=1000/2000/4000, the O(n²) reference took 0.040s/0.162s/0.649s (each doubling of n roughly quadrupling the time — the O(n²) signature), while this O(n log n) solution took 0.0002s/0.0004s/0.0008s at the same sizes, and stayed under 0.004s even out to n=16,000 (well past where the O(n²) reference becomes impractical to even benchmark). This is a real asymptotic gap, not measurement noise: at n=4000 the O(n log n) version is already over 800x faster.

**Takeaway:** the key mental shift from the O(n²) DP to the O(n log n) patience-sorting approach is *what the auxiliary array indexes by*. The O(n²) version's `dp[i]` is indexed by position in the input and answers "longest subsequence ending here," which forces an O(n) backward scan per element to compute. The O(n log n) version's `seq[k]` is indexed by subsequence *length* and answers "smallest tail achievable at this length," which is a monotonic (sorted) quantity — and sorted data is what unlocks binary search, turning an O(n) scan into an O(log n) one. This "reindex by an invariant that turns out to be monotonic" move is the same core trick that shows up again in the LIS-based reduction for 2D problems like Russian Doll Envelopes.

---

### #20 — Russian Doll Envelopes (LeetCode 354)

**Constraints:** 1 ≤ envelopes.length ≤ 10^5, envelope dims 1 ≤ wᵢ, hᵢ ≤ 10^5. **Two independent issues found in the original submission: a hard crash bug (unrelated to algorithm logic), and a logically-correct-but-asymptotically-insufficient algorithm. Both are resolved in the optimized version below, which reduces the problem to 1D LIS (entry #19).**

**Original submission:**

```python
class Solution:
    def isFit(self, env1: list[int], env2: list2):
        if env1[0]<env2[0] and env1[1]<env2[1]:
            return True
        return False

    def maxEnvelopes(self, envelopes: list[list[int]]) -> int:
        n = len(envelopes)
        envelop = [1]*(n+1)
        envelopes = sorted(envelopes, key=lambda x: (x[0], x[1]))
        for i in range(0, n):
            for j in range(i+1, n):
                if self.isFit(envelopes[i], envelopes[j]):
                    envelop[j+1] = max(envelop[j+1], envelop[i+1] + 1)
        
        return max(envelop)
```

**Issue #1 — immediate crash, unrelated to algorithm logic:** `env2: list2` is a typo (`list2` isn't a real type — presumably meant `list[int]` again, or just `list`). Confirmed by direct execution that this raises `NameError: name 'list2' is not defined` the moment the class body is defined, not when the method is called — a reminder that parameter type annotations are evaluated eagerly at function-definition time in standard Python (unless `from __future__ import annotations` is active), so a typo in an annotation is a hard crash before any of your actual logic ever runs.

**Issue #2 — the underlying algorithm is logically correct but has the wrong asymptotic complexity for the stated constraints.** With the typo fixed (swap `list2` for `list`), this is a standard O(n²) all-pairs DP: sort by `(width, height)` ascending, then for every pair `(i, j)` with `i < j`, check if envelope `j` can nest around envelope `i`, and take the best chain ending at each index. Verified against an independent `itertools.combinations`-based brute-force oracle on 200 randomized trials (n ≤ 7, small value ranges) plus the classic hand-picked cases (`[[5,4],[6,4],[6,7],[2,3]]`→3, `[[1,1],[1,1],[1,1]]`→1, `[[4,5],[4,6],[6,7],[2,3],[1,1]]`→4) — 0 mismatches. Notably, this O(n²) version doesn't even need a special tie-break for same-width envelopes the way the O(n log n) reduction will — `isFit`'s strict double inequality (`env1[0]<env2[0] and env1[1]<env2[1]`) already refuses to let two same-width envelopes nest, since neither can be strictly wider than the other.

**Benchmarked** at n=500/1000/2000/4000: 0.022s / 0.081s / 0.330s / 1.437s — a clean ~4x-per-doubling growth, the O(n²) signature. At the problem's actual constraint ceiling of n up to 1×10^5, that extrapolates to roughly 10^10 operations — a guaranteed TLE on the real judge, regardless of constant-factor tuning. This is the same lesson as Russian Doll's stepping-stone problem (LIS, entry #19): being logically correct on every test case is necessary but not sufficient — an algorithm's growth rate has to actually fit inside the problem's stated constraints, and the only way to know for sure is to benchmark at realistic scale, not just eyeball the nested loops.

**Optimized version — reduction to 1D LIS:**

```python
class Solution:
    def maxEnvelopes(self, envelopes: list[list[int]]) -> int:
        n = len(envelopes)
        seq = []

        envelopes = sorted(envelopes, key = lambda x: (x[0], -x[0]*x[1]))

        for i in range(0, n):
            height = envelopes[i][1]
            index = bisect.bisect_left(seq, height)
            if index<len(seq):
                seq[index] = height
            else:
                seq.append(height)
        return len(seq)
```

**How it works:** this is entry #19's LIS trick applied to one dimension after removing the other. Sorting by `(width ascending, height descending-within-tie)` guarantees that by the time you scan left to right, width is already handled — any strictly-increasing run you find in the *heights alone* automatically has strictly-increasing widths too, except for one danger case: two envelopes with the *same* width. The secondary sort key `-x[0]*x[1]` handles exactly that — within a fixed width `x[0]` (a positive constant across the tie group), `-x[0]*x[1]` is just a monotonically decreasing function of `x[1]`, so it sorts those ties by height descending, same effect as `-x[1]` would (the extra `x[0]` factor is unnecessary but harmless — it doesn't change the relative order within a width group, since it's the same positive constant multiplying every element there). With ties broken this way, two same-width envelopes can never both participate in an increasing run of heights: the taller one is placed *before* the shorter one in traversal order, so the LIS scan would need height to go from small to large across them, which the descending tie order specifically prevents.

**Verified** against three layers of testing, chosen specifically to stress the tie-break: (1) the classic hand-picked LeetCode cases, all matching; (2) a dedicated same-width stress test — 20 envelopes all sharing width 5 with distinct heights — correctly returning 1 (none may nest, since none is strictly wider); (3) 3,000 general randomized trials plus 2,000 randomized trials specifically constructed with heavy width collisions (only 3 distinct width values allowed, forcing large tie groups) against the independent O(n²) reference from this same entry's original (typo-fixed) submission — 0 mismatches across all 5,300 trials, including the adversarial tie-heavy set where a wrong tie-break would have shown up immediately.

**Benchmarked** at the problem's actual constraint scale: n=1,000/5,000/20,000/50,000/100,000 → 0.001s/0.004s/0.018s/0.053s/0.138s. Compare to the O(n²) original, which took 1.447s at just n=4,000 and would be projected at roughly 10^10 operations at n=100,000 (untestable in practice — it would run for hours). The optimized version handles the full n=100,000 constraint ceiling in under 0.14s. This is the same real, measured complexity gap as entry #19, not a constant-factor illusion: growth from n=1,000 to n=100,000 (100x) only costs about 115x runtime, consistent with n log n rather than n².

**Takeaway:** this is the payoff of entry #19's "reindex by a monotonic invariant" trick applied to a genuinely 2D problem: sorting collapses one dimension (width) into "already handled by traversal order," which turns a 2D increasing-chain problem into a plain 1D LIS problem on the other dimension (height) — but only if ties in the collapsed dimension are broken in the direction that prevents equal values from silently chaining together. The general pattern to remember: whenever a sort is being used to enforce "the first dimension is already increasing," any dimension where equal values are possible needs an explicit, deliberately-chosen tie-break — otherwise the reduction silently admits invalid chains that the original problem's strict inequality was supposed to forbid.

### #21 — Divide Intervals Into Minimum Number of Groups (LeetCode 2406)

**Constraints:** 1 ≤ intervals.length ≤ 10^5, 1 ≤ startᵢ ≤ endᵢ ≤ 10^6. Two rounds: an O(n²) linear-scan-over-groups submission (correct, wrong complexity), then an O(n log n) min-heap submission (correct, genuinely faster, with two minor edge/robustness notes worth understanding but neither blocking for this problem).

**Original submission — O(n²) linear scan over open groups:**

```python
class Solution:
    def minGroups(self, intervals: list[list[int]]) -> int:
        intervals = sorted(intervals, key = lambda x: (x[0], x[1]))
        groups = []
        nInterval = len(intervals)
        groupNum = -1
        for i in range(0, nInterval):
            groupReq = True
            intervalStartTime = intervals[i][0]
            for j, group in enumerate(groups):
                if intervalStartTime > group:
                    groupReq = False
                    groupNum = j
                    break

            if groupReq:
                groups.append(intervals[i][1])
            else:
                groups[groupNum] = intervals[i][1]
        return len(groups)
```

**The logic is correct:** sort by start time, then for each interval reuse the first currently-free group (one whose stored end time is strictly before this interval's start — correctly using `>` rather than `>=`, so touching intervals like `[1,5]` and `[5,10]` are treated as overlapping and do *not* share a group) or open a new one if none is free. Verified against an independent oracle (scans every candidate point and counts how many intervals cover it, taking the max) across 3,000 randomized trials — 0 mismatches — plus hand-picked LeetCode examples.

**The complexity problem:** the inner `for j, group in enumerate(groups)` is a full linear scan over every open group, run once per interval, with no early exit guarantee — in the worst case (many mutually-overlapping intervals, so nothing is ever free to reuse) this degrades to a genuine O(n²). Confirmed two ways: (1) an explicit worst-case construction of all-mutually-overlapping intervals, and (2) benchmarking at n=1,000/2,000/4,000/8,000/16,000 → 0.0035s/0.0142s/0.0571s/0.2280s/0.9157s roughly, a clean ~4x-per-doubling growth — the O(n²) signature. At the real constraint ceiling of n=10^5 this extrapolates to a guaranteed TLE.

**Hint given (Socratic, since the core greedy idea was already right):** the sort-by-start-time and "reuse a free group or open a new one" logic didn't need to change — only *how* a free group gets found needed to improve from a linear scan to something sublinear. Nudged toward: since the only thing that matters about an open group is *when it frees up*, and only the *earliest*-freeing one is worth checking each time (if even the earliest-ending group isn't free yet, none of the others are either — they all end at least as late), a **min-heap of end times** gives O(log n) access to exactly that value, via `heapq`.

**Optimized version — min-heap of end times:**

```python
class Solution:
    def minGroups(self, intervals: list[list[int]]) -> int:
        intervals = sorted(intervals, key = lambda x: (x[0], x[1]))
        groups = []
        nInterval = len(intervals)
        groupNum = 1
        for i in range(0, nInterval):
            intervalStartTime = intervals[i][0]
            intervalEndTime = intervals[i][1]
            if len(groups)>0:
                minEndTime, minEndTimeGroup = groups[0]
                if intervalStartTime <= minEndTime: #new group required
                    groupNum +=1
                else:
                    minEndTime, minEndTimeGroup = heapq.heappop(groups)
            heapq.heappush(groups, (intervalEndTime, groupNum))

        return groupNum
```

**How it works:** `groups` is a min-heap keyed by end time, so `groups[0]` is always the earliest-ending (soonest-to-free) open group without needing a scan. If the current interval's start doesn't clear even that earliest end time, no group is free, so a new one is opened (`groupNum` incremented); otherwise that group is popped (freed) and effectively reused, then either way the current interval's end time is pushed back onto the heap under the relevant group id. This turns the O(n) "find a free group" step into O(log n), making the whole sweep O(n log n).

**Verified** against the same reference oracle across 10,000+ combined randomized trials, including a batch specifically constructed with heavy tied end-times (values restricted to a 1–6 range) to stress the heap's tie-breaking behavior — 0 correctness mismatches on any non-empty input. Two things surfaced during testing that are worth recording but don't affect this submission's correctness for LeetCode's actual test space:

- **A real but functionally-inert bug:** in the reuse branch, `heapq.heappop(groups)` correctly retrieves `(minEndTime, minEndTimeGroup)` — the id of the group actually being freed — but the subsequent `heapq.heappush(groups, (intervalEndTime, groupNum))` pushes the outer running counter `groupNum` instead of `minEndTimeGroup`. This mislabels *which* group the new end time belongs to. It happens to be harmless here only because the heap's ordering and every decision in this algorithm depend solely on the end-time (the first tuple element) — the group id is carried along but never read back to make a decision, only returned incidentally as part of the final `groupNum` count. Confirmed via the tied-end-time stress batch (2,000 trials) that this never produces a wrong count. It would matter the moment a follow-up variant asked *which* group each interval was assigned to, not just how many groups exist.
- **A genuine but LeetCode-unreachable edge case:** `groupNum` is initialized to `1` and is never adjusted for the zero-iteration case, so `minGroups([])` returns `1` instead of the true answer `0`. This showed up as every mismatch (out of ~10,600 trials) tracing back to a randomly-generated empty list. LeetCode 2406's stated constraint `1 <= intervals.length` makes this input impossible on the real judge, so it's not a blocking issue for this submission — but worth knowing, since `groupNum = 0` would be the more defensively correct initialization.

**Benchmarked** at realistic scale, n=1,000/5,000/20,000/50,000/100,000 → 0.0011s/0.0030s/0.0143s/0.0398s/0.0899s — and, critically, re-run against the exact all-mutually-overlapping worst case that broke the O(n²) original: n=2,000/8,000/32,000/100,000 → 0.0006s/0.0026s/0.0091s/0.0363s, now fast even in the adversarial case. Growth from n=1,000 to n=100,000 (100x input) costs roughly 80x runtime — consistent with n log n, not the ~10,000x a true O(n²) would show, and a genuine fix for the worst case that previously took nearly a full second at just n=16,000.

**Round 3 — refinement, directly resolving the id-mislabeling note above:**

```python
class Solution:
    def minGroups(self, intervals: list[list[int]]) -> int:
        intervals = sorted(intervals, key = lambda x: (x[0], x[1]))
        groups = []
        nInterval = len(intervals)
        groupNum = 1
        for i in range(0, nInterval):
            intervalStartTime = intervals[i][0]
            intervalEndTime = intervals[i][1]
            if len(groups)>0:
                minEndTime = groups[0]
                if intervalStartTime <= minEndTime: #new group required
                    groupNum +=1
                else:
                    minEndTime = heapq.heappop(groups)
            heapq.heappush(groups, intervalEndTime)

        return groupNum
```

This drops the group-id entirely — `groups` is now a plain heap of end times, not `(end, id)` tuples. Since the previous round's own analysis showed the id was never read back into any decision, carrying it at all was dead weight; removing it doesn't just tidy the code, it makes the id-mislabeling bug structurally impossible rather than merely inert. **Verified** against the same reference oracle across 10,000 combined randomized trials (including a dedicated heavy-tie batch, value range 1–6, and a wider-range/larger-n batch) — 0 non-empty-input mismatches — plus the three hand-picked cases (`[[1,5],[5,10],[10,15]]`→2, `[[1,3],[5,6],[8,10],[11,13]]`→1, `[[1,10],[2,7],[3,4]]`→3). **Benchmarked** identically to round 2: n=1,000→100,000 at 0.0014s→0.1223s (random) and the all-overlapping worst case n=2,000→100,000 at 0.0004s→0.0297s — same genuine O(n log n) profile, confirming the simplification cost nothing. The empty-list edge case (`groupNum=1` vs. true answer `0`) is unchanged and still LeetCode-unreachable per the `1 <= intervals.length` constraint.

**Takeaway:** for a pure *counting* problem ("how many groups/rooms are needed"), it's enough to know *whether* some resource is free — you never need to know or care *which specific one*, since freeing any one of them has the same effect on future availability. That insight is what allows swapping a linear "scan every open group" for a heap that only ever needs to check the single most-promising candidate (the earliest-ending one) in O(log n) — and, taken one step further, it means the resource's *identity* doesn't need to be tracked at all if the problem never asks for it: round 2 carried a group id that was correct-but-unused bookkeeping (a latent trap for any future variant that *does* need to know which group), and round 3 shows that the cleanest fix for "this data is never read back" is often to stop storing it, not just to store it correctly.

---

### #22 — Find the Difference (LeetCode 389)

**Constraints:** 0 ≤ s.length ≤ 1000, `t.length == s.length + 1`, lowercase English letters only, `t` is a shuffle of `s` plus one extra random letter.

**Submission:**

```python
class Solution:
    def findTheDifference(self, s: str, t: str) -> str:
        charDict = {}
        n = len(s) 
        
        for i in range(n):
            if s[i] in charDict:
                charCount = charDict.get(s[i])
                charCount +=1
            else:
                charCount = 1
            charDict[s[i]] = charCount
        
        for i in range(n+1):
            if t[i] in charDict:
                charCount = charDict.get(t[i])
                charCount -=1
                if charCount==0:
                    charDict.pop(t[i])
                else:
                    charDict[t[i]] = charCount
            else:
                return t[i]
        
        for k, v in charDict.items():
            if v>=0:
                return k
```

**Correct on arrival.** Build a frequency count of `s`, then walk `t` decrementing (popping a key once its count hits 0); the moment a character in `t` isn't found in the dict, that's the added letter, returned immediately. Verified against an independent oracle (`chr(sum(ord(c) for c in t) - sum(ord(c) for c in s))` — the classic ASCII-sum-difference trick, deliberately a *different* technique than the submission, not just a restatement of it) across 20,000 randomized trials using a deliberately small 5-letter alphabet to force heavy character collisions, plus hand-picked edge cases (`s=""`, `t="y"` → `"y"`; `s="a"`, `t="aa"` → `"a"`; the LeetCode example) — 0 mismatches.

**Code-quality finding — the final loop is unreachable dead code.** The trailing `for k, v in charDict.items(): if v>=0: return k` can never execute. Reasoning: every character in `t` that's still present in the dict gets decremented, and a count is only ever removed via `pop` at exactly 0 — it's never allowed to go negative, because once it hits 0 and is popped, any *further* occurrence of that same character in `t` will find it missing and fall into the `else: return t[i]` branch instead. Since `t` has exactly one more character than `s` (n+1 decrements against a starting total of n), and every decrement either keeps the dict non-negative or triggers the "missing" branch, the "missing" branch is mathematically guaranteed to fire exactly once somewhere during the scan of `t` — so the function always returns from inside the second loop. Confirmed empirically: instrumented the same logic to record which branch actually returns, and across all 20,000 trials the third loop's return was reached **0 times**. Not a bug (the code never produces a wrong answer or hits the implicit `None` return), but worth knowing: the `if v>=0` guard suggests the author suspected a leftover entry might need handling, when in fact the second loop already covers every case.

**Optimized version — XOR reduction (no hash map at all):**

```python
class Solution:
    def findTheDifference(self, s: str, t: str) -> str:
        result = 0
        for ch in s + t:
            result ^= ord(ch)
        return chr(result)
```

**How it works:** XOR-ing every character code in `s` together with every character code in `t` cancels out any letter that appears the same number of times in both (`x ^ x = 0`), leaving only the one letter that appears an odd number of extra times — exactly the added character. This is the "find the single non-duplicated element" trick (the same one behind LeetCode 136, Single Number) applied across the *combined* multiset of two strings instead of one array.

**Verified** against the submission itself as the oracle across 5,000 randomized trials (full 26-letter alphabet, lengths up to 20) — 0 mismatches — plus the same hand-picked cases.

**Benchmarked** for constant-factor comparison (both are O(n) time — the dict version is already bounded to O(1) extra space too, since the alphabet is fixed at 26 letters, so this isn't an asymptotic win, just a cleaner one): at n=1,000/10,000/100,000/1,000,000, the dict version took 0.16ms/2.02ms/18.0ms/178.7ms versus the XOR version's 0.04ms/0.49ms/4.3ms/41.5ms — a consistent ~4x speedup at every scale (both scale linearly at the same ratio per 10x input, confirming this is a real constant-factor difference from avoiding hash map overhead entirely, not a hidden complexity gap).

**Round 3 — the ASCII-sum-difference approach, submitted independently right after (and, notably, the exact technique used as this entry's own verification oracle):**

```python
class Solution:
    def findTheDifference(self, s: str, t: str) -> str:
        n = len(s)
        charSumS = sum(ord(char) for char in s)
        charSumT = sum(ord(char) for char in t)
        return chr(charSumT - charSumS)
```

**How it works:** instead of canceling matched characters bitwise, this cancels them arithmetically — summing every character code in `s`, summing every character code in `t`, and subtracting. Every letter that appears in both strings the same number of times contributes equally to both sums and cancels out in the subtraction, leaving exactly the ASCII value of the one extra letter. Same underlying idea as the XOR version (reduce a multiset comparison to a single running scalar), different cancellation operator (`+`/`-` instead of `^`).

**Verified** against the dict version (round 1) as the independent oracle across 20,000 randomized trials (full 26-letter alphabet, lengths up to 25) — 0 mismatches — plus the same three hand-picked edge cases.

**Benchmarked** alongside both prior versions at the same scales: n=1,000/10,000/100,000/1,000,000 → dict 0.15ms/1.49ms/15.9ms/179.1ms, XOR 0.04ms/0.43ms/5.4ms/43.0ms, sum-difference 0.05ms/0.52ms/5.8ms/49.3ms. The sum-difference version tracks XOR closely (both ~3.5-4x faster than the dict version at every scale) — very slightly behind XOR in practice, consistent with doing two separate `sum()` passes plus a final subtraction versus XOR's single fused loop, but this is a minor constant-factor nuance, not a different complexity class.

**Takeaway:** not every review finds an asymptotic problem — sometimes multiple versions land on the same Big-O, and the interesting finding is a genuine (measured, not assumed) constant-factor difference plus a cleaner mental model. This entry ended up with three independently-correct O(n) solutions built on two different flavors of the same core idea ("reduce a two-multiset comparison to a single scalar that cancels out everything but the answer"): a hash map (round 1, correct but carries bookkeeping — and, as it turned out, one truly dead branch), and two cancellation-based reductions (XOR and sum-difference, rounds 2 and 3) that are both faster and simpler because they never need to store *which* characters matched, only a running aggregate. Worth reaching for the cancellation pattern by reflex on any "everything appears in matched pairs/sets except one thing" problem.

---

### #23 — Meeting Rooms II (LeetCode 253)

**Constraints:** 0 ≤ intervals.length ≤ 10^4, `0 <= start[i] < end[i] <= 10^6` (note: strictly `start < end` — zero-length meetings are not a valid input). Picked deliberately as the closest sibling to entry #21 (Divide Intervals Into Minimum Number of Groups, LeetCode 2406) — same min-heap-of-end-times technique, same "minimum resources needed" shape, but with one subtle and important difference in intersection semantics.

**Submission:**

```python
"""
Definition of Interval:
class Interval(object):
    def __init__(self, start, end):
        self.start = start
        self.end = end
"""

class Solution:
    def minMeetingRooms(self, intervals: List[Interval]) -> int:
        intervals = sorted(intervals, key=lambda x: (x.start, x.end))
        rooms = []

        for interval in intervals:
            startTime = interval.start
            endTime = interval.end

            if len(rooms)>0:
                roomEndTime = rooms[0]
                if roomEndTime <=startTime: #Room creation is not required
                    heapq.heappop(rooms)
                    heapq.heappush(rooms, endTime)
                else:
                    heapq.heappush(rooms, endTime)
            else:
                heapq.heappush(rooms, endTime)
        return len(rooms)
```

**Correct on arrival — and correctly caught the exact semantic trap flagged going in.** The structure is the entry #21 heap pattern transplanted directly: `rooms` is a min-heap of end times, `rooms[0]` is always the soonest-freeing room, and a free room is reused (popped + the new end time pushed) or a new one is opened (end time just pushed, heap grows) otherwise. The one thing that *must* differ from 2406 is the comparison direction: 2406 requires `start > end` (strict) to treat two intervals as non-overlapping, because touching intervals there count as conflicting; Meeting Rooms II uses the standard half-open-interval convention where a meeting ending at time `5` and another starting at time `5` do *not* conflict — the room is already free. This submission correctly uses `roomEndTime <= startTime` (non-strict), matching that convention.

**Verified** against an independent sweep-line oracle (treat every start as +1, every end as −1, sort events with ends processed before starts at equal timestamps to encode "touching = free," then track the running max) across 10,000 randomized trials — 0 mismatches — plus hand-picked cases: `[[0,30],[5,10],[15,20]]` → 2, `[[7,10],[2,4]]` → 1, `[[1,5],[5,10]]` → 1 (confirms touching is correctly treated as non-conflicting, the opposite of entry #21's requirement), and the empty-list case → 0 (no edge-case bug here, unlike entry #21's round 2, since there's no separate counter to mis-initialize — the heap's own length is the answer).

Note on test methodology: an earlier pass of this same test generator allowed zero-length meetings (`start == end`) and produced ~460 apparent "mismatches" out of 8,000 trials — but those were all artifacts of the generator creating inputs outside the problem's actual constraint (`start < end` is required, so a zero-length meeting is not a real test case), not bugs in the submission. Tightening the generator to only emit valid inputs eliminated every mismatch. A useful reminder for future reviews: an oracle built against the wrong constraint space produces false positives, and a flood of "mismatches" is itself a reason to double check the test's own validity before suspecting the submission.

**Benchmarked** at n=1,000/5,000/20,000/50,000/100,000 → 0.0031s/0.0049s/0.0160s/0.0484s/0.1197s, and against the all-mutually-overlapping worst case at n=2,000/8,000/32,000/100,000 → 0.0005s/0.0025s/0.0068s/0.0365s — the same genuine O(n log n) profile as entry #21's optimized version, as expected given it's structurally the same algorithm.

**Minor code-quality note:** all three branches end in `heapq.heappush(rooms, endTime)` — the only thing that varies is whether a `heapq.heappop(rooms)` happens first. This could collapse to:
```python
if rooms and rooms[0] <= startTime:
    heapq.heappop(rooms)
heapq.heappush(rooms, endTime)
```
removing the duplicated push across three branches (and the now-unnecessary `len(rooms)>0` / empty-heap special case, since `rooms[0]` is simply never evaluated when `rooms` is falsy due to short-circuiting). Purely a style simplification — doesn't change behavior or complexity.

**Takeaway:** this is a clean example of pattern transfer done right — recognizing that two problems share the same underlying resource-counting shape (sort by start, min-heap of end times, reuse-or-open) while correctly spotting the one place they diverge: the direction of the boundary comparison, driven by whether the problem's own definition of "overlap" includes or excludes touching endpoints. The general lesson: when reusing a memorized pattern for a "new" problem, don't just copy the comparison operator — re-derive it from that specific problem's own overlap definition, since `<=` vs `>` flips depending on whether touching counts as a conflict.

---

### #24 — Single Number (LeetCode 136)

**Constraints:** 1 ≤ nums.length ≤ 3×10^4, every element appears exactly twice except one which appears exactly once, required O(n) time and O(1) extra space. Given right after entry #22 specifically because it's the same cancellation trick in its original, single-array form.

**Submission:**

```python
class Solution:
    def singleNumber(self, nums: list[int]) -> int:
        res = 0
        for num in nums:
            res = res^num
        return res
```

**Correct on arrival, and already optimal** — this is the textbook application of `x ^ x = 0` and `x ^ 0 = x`: XOR-ing every element together cancels each paired value (order doesn't matter, XOR is commutative and associative), leaving only the element with no partner. Already meets both stated constraints: O(n) time, O(1) auxiliary space (a single accumulator, regardless of input size) — no hash map or sort needed at all, an even tighter bound than entry #22's dict version required.

**Verified** against an independent `collections.Counter`-based oracle across 20,000 randomized trials (small value range to force collisions, negative numbers and zero included) — 0 mismatches — plus `[1]`→1, `[4,1,2,1,2]`→4, `[-1,-1,-2]`→-2 (negative-number handling), and `[0,1,0]`→1 (confirms XOR-with-zero doesn't special-case incorrectly).

Note on test methodology, same class of issue as entry #23's: the first test run showed 50/20,000 "mismatches," every one traced to the generator occasionally picking the "should appear once" value from the same small range already used for the pairs — accidentally making that value appear *three* times instead of once, which breaks the problem's own stated invariant and made the independent `Counter` oracle (which specifically looks for a count-of-exactly-1 element) report `None` where none should exist. The XOR solution still produced a defensible answer in those cases (it finds whatever has odd parity, not strictly "count exactly 1"), which is actually a slightly different and more general guarantee than the stated problem requires — but the apparent mismatch was still purely a generator bug, not a solution bug. Fixed by excluding the pool's values when picking the singleton, after which the trial count returned 0/20,000.

**Benchmarked** against a dict/Counter-based alternative purely for a constant-factor comparison, since both are O(n) time, but the dict version isn't truly O(1) space: n=1,000/10,000/100,000/1,000,000 → XOR 0.019ms/0.18ms/2.6ms/34.2ms vs. dict 0.058ms/0.80ms/13.2ms/218.9ms — XOR roughly 3-6x faster at every scale (growing somewhat with n, consistent with the dict version paying real hashing/bucket overhead per element that a bare integer XOR never does), on top of the dict version's non-constant memory footprint.

**Takeaway:** a short entry, deliberately — sometimes the point of a repeat problem isn't to find a bug or a complexity gap but to confirm a pattern has actually generalized on the first try, not just been memorized for one specific shape (here: single array, one cancellation operator, no secondary dimension like entry #22's two-string version). Also a second instance of the entry #23 lesson: when an oracle disagrees with a solution en masse, check whether the *oracle's own input generation* actually satisfies the problem's stated constraints before concluding the solution is wrong.

---

### #25 — Word Search (LeetCode 79)

**Constraints:** `1 <= m, n <= 6`, `1 <= word.length <= 15` — deliberately tiny bounds, which turn out to matter (see the benchmark below). Picked specifically to revise the **backtracking** concept. Two rounds: a genuinely broken iterative attempt that conflated BFS/DFS mechanics with shared mutable state, then a correct recursive backtracking solution arrived at after three rounds of Socratic hints.

**Original submission — iterative stack-of-coordinates with a single shared `formedWord`:**

```python
class Solution:
    def checkBound(self, i, j, visited, m, n):
        if i<0 or i>=m or j<0 or j>=n:
            return False
        if visited[i][j]:
            return False
        return True

    def exist(self, board: list[list[str]], word: str) -> bool:
        m = len(board)
        n = len(board[0])

        q = deque()

        for i in range(m):
            for j in range(n):
                visited = [[False] * n for _ in range(m)]
                q.append((i,j))
                formedWord = ""
                
                while q:
                    a,b = q.pop()
                    character = board[a][b]
                    visited[a][b] = True
                    
                    if len(formedWord+character) <len(word) and formedWord+character == word[0:len(formedWord+character)]:
                        formedWord = formedWord+character
                        if self.checkBound(a+1, b, visited, m, n):
                            q.append((a+1, b))
                        if self.checkBound(a-1, b, visited, m, n):
                            q.append((a-1, b))
                        if self.checkBound(a, b+1, visited, m, n):
                            q.append((a, b+1))
                        if self.checkBound(a, b-1, visited, m, n):
                            q.append((a, b-1))
                    elif formedWord+character == word:
                        return True
        
        return False
```

**The bug — a single shared `formedWord` corrupted across branches, plus no unmark-on-backtrack.** First, a traversal-order misconception needed clearing up before the real bug could even be discussed: this uses a `deque` with `.append()` and `.pop()` both acting on the right end, which is LIFO — already DFS order, not BFS, despite being named `q`. Switching to an explicit `stack` would have changed nothing.

The actual defect: `formedWord` is one variable shared by the *entire* `while q` loop, no matter how many divergent paths are sitting in `q` at once, and `visited` is marked when a cell is popped but never unmarked — so a cell that was part of one path's progress stays "used" for the rest of that start-cell's whole search, with no way to free it for a sibling path. Verified against an independent reference (classic recursive DFS with proper mark/unmark) across 3,000 randomized small boards: **105/3,000 mismatches, all false positives** (claimed a word existed when it didn't). Traced the smallest failing case by hand:

```
board = [['A','B'],
         ['A','B']]
word  = "BAABB"   # correct answer: False
```

Starting from `(0,1)`, the path `B→A(0,0)→A(1,0)` correctly reaches `formedWord="BAA"`. At that point, cell `(1,1)` gets pushed onto the stack **twice** — once as a neighbor of `(0,1)` (pushed early, before `(1,1)` was ever visited) and again as a neighbor of `(1,0)` (pushed later, same reason). Popping the first copy extends `formedWord` to `"BAAB"` — a legitimate 4-letter path using 4 distinct cells. Popping the *second*, stale copy of `(1,1)` right after reuses that same physical cell as the 5th letter, completing `"BAABB"` by using one cell twice — which the problem explicitly forbids.

**Hints given (Socratic, three rounds, since this was a genuine structural misunderstanding rather than a one-line slip):**
1. First hint pointed at the trace evidence above and asked what it means for `(1,1)` to be queued twice before either copy is processed, and whether `formedWord` being one shared variable across all pending branches could be the actual issue — deliberately not yet naming recursion.
2. User asked "do I need a stack instead of a queue?" — corrected the misconception that traversal order (BFS vs. DFS) was ever the problem, since the submission was already stack-based/DFS-ordered; the real gap was that neither a manual stack nor a queue fixes it unless each entry carries its own independent path state, and a cell's "in use" status needs to be scoped to one path's lifetime (mark-then-unmark) rather than persisting for the whole search.
3. User asked for the logic directly after two rounds without landing on it — given the recursive template in words: track an index `k` into `word` instead of building a string, fail fast on out-of-bounds/mismatch/already-visited, mark before recursing into all 4 neighbors with `k+1`, combine with `or`, and **unmark immediately after**, regardless of outcome — explaining that the call stack itself replaces the need for an explicit stack of coordinates, and that unmarking on return is what "backtracking" actually refers to.

**Optimized/corrected version — recursive backtracking with proper mark/unmark:**

```python
class Solution:
    def search(self, board, word, i, j, k, m, n, visited):
        #Invalid situations or stopping conditions
        if i<0 or i>=m or j<0 or j>=n or k>=len(word) or visited[i][j]:
            return False
        #Character is not right. 
        if word[k]!=board[i][j]:
            return False
        visited[i][j] = True
        
        if word[0:k+1] == word:
            return True

        res = self.search(board, word, i+1, j, k+1, m, n, visited) or self.search(board, word, i-1, j, k+1, m, n, visited) or self.search(board, word, i, j+1, k+1, m, n, visited) or self.search(board, word, i, j-1, k+1, m, n, visited)
        visited[i][j] = False
        return res

    def exist(self, board: list[list[str]], word: str) -> bool:
        m = len(board)
        n = len(board[0])
        for i in range(m):
            for j in range(n):
                visited = [[False]*n for _ in range(m)]
                res = self.search(board, word, i, j, 0, m, n, visited)
                if res:
                    return res
        return False
```

**How it works, and why it fixes both problems at once:** replacing the shared `formedWord` string with an index `k` means every recursive call carries its *own* copy of "how far I've matched," passed as a plain function argument rather than a mutable shared variable — different branches of the recursion tree can never corrupt each other's progress. And because `visited[i][j] = False` runs unconditionally right after the four recursive branches are tried (success or failure), a cell is only ever "in use" for the duration of the one call-stack branch currently exploring through it; the moment that branch unwinds, the cell is free again for a sibling branch. This is the textbook backtracking shape: mark → recurse → unmark.

**Verified** against the same independent reference oracle: re-ran all 3 of the original submission's concrete failing cases (including the hand-traced `BAABB` example) — all now correctly return `False` — plus 8,000 fresh randomized trials across small boards and alphabets — **0 mismatches**. Classic LeetCode examples also confirmed: `"ABCCED"`→True, `"SEE"`→True, `"ABCB"`→False.

**Benchmarked**, with an interesting twist specific to this problem: at realistic/typical inputs (6×6 to 12×12 boards, word length 10) it's essentially instant — 0.06ms to 0.32ms. But at the problem's actual worst case — a 6×6 board filled entirely with the same letter, and a 15-character word of that same letter plus one mismatching final character (forcing the algorithm to explore every simple path up to length 14 before concluding failure) — it took **1.72 seconds**. This isn't a bug or an inefficiency to fix: Word Search's backtracking solution is inherently worst-case exponential (roughly O(m·n·4^L) for a word of length L), and this is precisely *why* LeetCode constrains `m, n ≤ 6` and `word.length ≤ 15` for this problem — the constraints are small specifically because no sub-exponential general algorithm is known for this problem, and backtracking-with-pruning is the accepted standard solution despite the exponential worst case.

**Minor code-quality notes, neither a bug:**
- `word[0:k+1] == word` recomputes a string slice and comparison every call to check "is this the last character," when `k+1 == len(word)` (or equivalently `k == len(word) - 1`) is a plain integer comparison doing the identical check in O(1) instead of O(k) — negligible at this problem's tiny constraints, but worth knowing as a reflex substitution.
- `exist()` allocates a fresh `visited` matrix for every one of the `m*n` starting cells. Since every mark inside `search` is unconditionally paired with an unmark before that call returns, the matrix is guaranteed to be back to all-`False` by the time each top-level call finishes — meaning a single shared `visited` matrix, reset-free, could be reused across all starting cells instead of reallocated each time. Doesn't change the dominant complexity here, but a nice confirmation that the mark/unmark discipline is airtight.

**Takeaway:** this entry's real lesson isn't about Word Search specifically — it's about what "backtracking" structurally requires that an ad hoc iterative traversal doesn't automatically give you for free: each candidate path needs its *own* copy of its progress (don't share one mutable accumulator across branches), and a resource's "claimed" status must be scoped to exactly the lifetime of the one branch using it (explicit unmark on every return path, success or failure). Recursion gives you the first property for free via the call stack and makes the second property natural to write correctly (mark right before recursing, unmark right after) — which is why "just convert your iterative BFS/DFS into recursion" is often not a stylistic preference but the actual fix when backtracking-style problems go wrong.

---

### #26 — Island Perimeter (LeetCode 463)

**Constraints:** `1 <= grid.length, grid[0].length <= 100`, grid contains exactly one connected island. A sharp lesson in why "correct on small randomized tests" isn't the same as "correct at the problem's actual stated scale."

**Submission:**

```python
class Solution:
    def islandPerimeter(self, grid: list[list[int]]) -> int:
        row = len(grid)
        col = len(grid[0])
        
        visited = [[False]*col for _ in range(row)]
        
        def calc(visited, i, j):
            if i<0 or i>=row or j<0 or j>=col: #boundary
                return 1
            if visited[i][j]==1:
                return 0
            if grid[i][j]==0: #boundary
                return 1
            visited[i][j] = True
            val = calc(visited, i+1, j) + calc(visited, i-1, j) + calc(visited, i, j+1) + calc(visited, i, j-1)
            return val

        for a in range(row):
            for b in range(col):
                if grid[a][b]:            
                    return calc(visited, a, b)
```

**The logic is genuinely correct, and the early `return` is a deliberate, valid shortcut, not a bug.** `calc` is a flood fill: each never-before-seen land cell is marked, then its four neighbor-calls each contribute 1 (boundary or water — an exposed edge), 0 (already-visited land — don't double-count), or recurse further (fresh land — explore and fold in its entire contribution). Because every land cell is visited exactly once and each visit's own edge-count is summed exactly once into the final total, the recursive accumulation is airtight — no double-counting, no missed cells. The outer loop's `return` on the *first* land cell found is safe specifically because the problem guarantees exactly one connected island: a flood fill from any single land cell necessarily reaches every land cell in the grid, so there's never a second island left unvisited. (Worth flagging as a note, not a bug: this early-return would be wrong the moment the problem allowed multiple islands.)

Verified against an independent, structurally different oracle (for every land cell, directly check its 4 neighbors for boundary/water and sum — no flood fill, no recursion, no shared state at all) across 3,000 randomized single-island grids up to 8×8, generated via a random connected-growth process — **0 mismatches** — plus the classic LeetCode example (expects 16) and the single-cell case `[[1]]` (expects 4).

**The real finding — a `RecursionError`, not a logic bug, and it hits within the problem's own stated constraints.** My first round of fuzz testing only used grids up to 8×8 (64 cells) and found nothing wrong. Since the actual constraint allows grids up to `100 × 100` (10,000 cells, all of which can legally be one single connected island), I re-tested at that scale — and a fully-filled `100×100` grid crashes:

```
RecursionError: maximum recursion depth exceeded in comparison
```

Binary-searched the threshold on filled square grids: `30×30` (900 cells) still works, `32×32` (1,024 cells) already crashes — consistent with Python's default recursion limit of 1,000, since this specific traversal order (`down, up, right, left` tried in that order at every cell) happens to build one long unbroken chain of nested calls across nearly the entire grid before any of them return. Notably, a hand-built "snake"-shaped 100×100 island (alternating full rows joined by a single connector column — the shape that *looks* like it should maximize recursion depth) actually completed fine; the plain, unremarkable fully-filled square was the one that broke. This is a reminder that "the worst case" for a recursive algorithm's *stack depth* isn't always the input shape that looks adversarial by eye — it depends on the specific order the code tries its branches in, and has to be found by testing at real scale, not reasoned out from intuition.

**Fix — the simplest resolution here is to realize the flood fill was never actually necessary.** Computing the perimeter doesn't require traversing connectivity at all — it only requires, for every land cell independently, counting how many of its 4 sides face water or the grid boundary:

```python
class Solution:
    def islandPerimeter(self, grid: list[list[int]]) -> int:
        rows, cols = len(grid), len(grid[0])
        total = 0
        for i in range(rows):
            for j in range(cols):
                if grid[i][j] == 1:
                    for di, dj in ((1,0), (-1,0), (0,1), (0,-1)):
                        ni, nj = i+di, j+dj
                        if ni < 0 or ni >= rows or nj < 0 or nj >= cols or grid[ni][nj] == 0:
                            total += 1
        return total
```

No recursion, no call stack to blow, no `visited` matrix, and no reliance on the "exactly one island" guarantee either — this version is correct even for multiple disconnected islands, a strictly more general result than the problem even asks for. This is exactly the reference oracle used to verify the original submission, confirming it independently: 0 mismatches across the same 3,000 trials, the classic example, and — critically — the `100×100` fully-filled grid that crashed the recursive version now returns instantly (perimeter 400, the outer boundary of a solid square) with no stack involved at all.

**Round 2 — an independently-submitted non-recursive rewrite, landing on the same fix:**

```python
class Solution:
    def islandPerimeter(self, grid: list[list[int]]) -> int:
        row = len(grid)
        col = len(grid[0])
        perimeterLength = 0

        def isBounded(i, j):
            if i<0 or i>=row or j<0 or j>=col:
                return 1
            if grid[i][j]==0:
                return 1
            return 0

        def perimeter(i, j):
            length = 0
            if isBounded(i+1, j):
                length += 1
            if isBounded(i-1, j):
                length += 1
            if isBounded(i, j+1):
                length += 1
            if isBounded(i, j-1):
                length += 1
            return length

        for a in range(row):
            for b in range(col):
                if grid[a][b]:            
                    perimeterLength += perimeter(a, b)
        
        return perimeterLength
```

Same idea as the suggested fix, arrived at independently: no `calc`/flood-fill, no `visited`, no recursion at all — `isBounded` just asks "is this neighbor off-grid or water," and `perimeter` sums that check across a cell's 4 sides. **Verified** against the same independent oracle across 3,000 randomized trials — 0 mismatches — plus the classic example and single-cell case. **Confirmed the fix actually resolves the recursion-depth problem**, not just avoids it by accident: re-ran the exact `100×100` fully-filled grid that crashed round 1 with `RecursionError` — this version returns `400` in 0.0027s, no stack depth involved anywhere. Benchmarked at 10×10/50×50/100×100 → 0.00004s/0.00068s/0.0026s, confirming clean O(rows×cols) scaling with no hidden cost from the two-level helper-function structure (`isBounded` nested inside `perimeter`, both called per-cell) — Python's function-call overhead here is negligible next to the trivial O(1) work each call does.

**Takeaway:** the flood-fill instinct here wasn't wrong, exactly — it correctly computes the right answer — but it's solving a harder problem than the one actually being asked (full connectivity traversal) when a flat, position-independent per-cell check suffices. And separately, this entry is a concrete reminder that Python's recursive call stack is a real, finite resource (default depth 1,000) that a correct, finite-time algorithm can still exhaust well within a problem's own stated constraints — "my fuzz tests all passed" only proves correctness at the scale actually tested, which is why pushing a benchmark to the problem's real constraint ceiling (not just "big enough to feel thorough") is part of the review, not an optional extra.

---

### #27 — Climbing Stairs (LeetCode 70)

**Constraints:** `1 <= n <= 45`. A clean, correct-on-arrival entry with one space-complexity note.

**Submission:**

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n==0:
            return 0
        if n==1: 
            return 1
        if n==2:
            return 2

        dp = [0]*(n+1)
        dp[0] = 0
        dp[1] = 1
        dp[2] = 2
        for i in range(3, n+1):
            dp[i] = dp[i-1] + dp[i-2]
        
        return dp[n]
```

**Correct on arrival.** `dp[i]` = number of distinct ways to reach step `i` taking 1 or 2 steps at a time; the recurrence `dp[i] = dp[i-1] + dp[i-2]` (reach step `i` either from a single step at `i-1`, or a double step at `i-2`) is the right one, and matches the textbook Fibonacci-shifted-by-one relationship for this problem. Verified against an independent brute-force oracle (`brute(n) = brute(n-1) + brute(n-2)` with `brute(0) = 1`, i.e. directly counting compositions of `n` using parts `{1, 2}` — same recurrence shape deliberately, but anchored at a different base case to make it a genuine independent check rather than a restatement) across the full actual constraint range `n = 1` to `45` — **0 mismatches**.

**Edge-case note, same class as entries #21 and #23's LeetCode-unreachable gaps:** `climbStairs(0)` returns `0`, but the conventional answer (by the same logic used everywhere else — "how many ways to be standing at step 0" — there's exactly 1, the empty sequence of moves) is `1`. This provably never matters here: LeetCode 70's own stated constraint is `1 <= n <= 45`, so `n = 0` is never a real input, and this is the same category of "defensively-coded but unreachable" branch seen before, not a bug that can actually be triggered.

**Code-quality / space-complexity note:** `dp[0] = 0` is assigned but never actually read — the loop only ever touches `dp[i-1]` and `dp[i-2]` starting from `i = 3`, and `dp[1]`, `dp[2]` are hardcoded directly rather than derived from `dp[0]`. More generally, this problem only ever needs the *previous two* values at any point, so the full `dp` array (O(n) space) is more than necessary — two rolling variables suffice for O(1) space:

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n <= 2:
            return n
        prev2, prev1 = 1, 2
        for i in range(3, n+1):
            prev2, prev1 = prev1, prev2 + prev1
        return prev1
```

**Verified** this rewrite against the original across the full `n = 1..45` range — 0 mismatches. **Benchmarked** at `n = 45` (the actual constraint ceiling): the array version took ~2.10µs/call versus the rolling-variable version's ~0.94µs/call — about 2.2x faster. This is purely a constant-factor win from skipping list allocation and indexing, not an asymptotic one — both versions are O(n) time; the array version is simply O(n) *space* where O(1) suffices.

**Takeaway:** a short, clean entry on a problem small enough (`n ≤ 45`) that the space difference barely matters in absolute terms, but the pattern is worth having as a reflex: any DP recurrence that only ever looks back a fixed, small number of steps (here, exactly 2) can drop the full table for a handful of rolling variables, and it's worth checking whether a "DP array" is actually using more than its last `k` entries before assuming the array is necessary.

---

### #28 — Maximum Depth of Binary Tree (LeetCode 104)

**Constraints:** number of nodes in `[0, 10^4]`. Logic is textbook-correct; the finding is a direct repeat of entry #26's lesson, this time on a tree instead of a grid.

**Submission:**

```python
class Solution:
    def maxDepth(self, root: TreeNode | None) -> int:
        if root is None:
            return 0
        
        return max(self.maxDepth(root.left), self.maxDepth(root.right))+1
```

**The logic is correct.** This is the standard recursive definition: an empty tree has depth 0, otherwise depth is 1 plus the deeper of the two subtrees. Verified against an independent iterative BFS oracle (count levels by draining a queue one full level at a time — a structurally different traversal than the recursive DFS under test) across 2,000 randomized trees up to 50 nodes — **0 mismatches** — plus the empty-tree (`None` → 0) and single-node (→ 1) edge cases.

**The same `RecursionError` finding as entry #26, this time on a tree.** My first fuzz pass only generated trees up to 50 nodes and found nothing wrong. The actual constraint allows up to `10^4` nodes, and a tree is only guaranteed to be *balanced* in the happiest case — nothing stops it from being a completely degenerate, linked-list-shaped tree (every node has exactly one child), which is a perfectly legal input. Built one and found the exact threshold:

```
skewed tree n=997:  OK, depth=997
skewed tree n=998:  RecursionError
skewed tree n=1000: RecursionError
skewed tree n=10000: RecursionError
```

Same root cause as entry #26: Python's default recursion limit is 1,000, and a maximally-skewed tree at the problem's own stated scale sails right past it. This is a crash, not a wrong answer — but on the real judge, a crash on a legal input is exactly as fatal as an incorrect one.

**Fix — iterative traversal (explicit stack, no call-stack dependency):**

```python
class Solution:
    def maxDepth(self, root: TreeNode | None) -> int:
        if root is None:
            return 0
        depth = 0
        stack = [(root, 1)]
        while stack:
            node, d = stack.pop()
            depth = max(depth, d)
            if node.left:
                stack.append((node.left, d+1))
            if node.right:
                stack.append((node.right, d+1))
        return depth
```

Each stack entry carries its own depth alongside the node, so there's no shared mutable accumulator and no reliance on Python's call stack at all — the explicit `stack` list can grow to 10,000 entries with zero issue, since it's just a list, not nested function-call frames. **Verified** against the same BFS oracle across the same 2,000 trials plus edge cases — 0 mismatches. **Confirmed the fix actually resolves the crash**: re-ran the exact `n=998` through `n=10,000` skewed trees that broke the recursive version — all now complete in under 2.1ms, including the full `n=10,000` constraint ceiling.

**Takeaway:** this is the second time in this log that correct-looking recursive tree/grid logic has hit Python's recursion limit at a problem's real constraint ceiling (see entry #26) rather than at any toy-scale test. Worth internalizing as a standing check specifically for **tree and grid problems with no explicit depth bound in the constraints**: always construct the degenerate/skewed shape (a tree that's really a linked list; a grid that's one long snake) at the *actual* maximum N before considering a recursive solution reviewed, since balanced-looking randomized test data will almost never surface this on its own.

---

### #29 — Path Sum II (LeetCode 113)

**Constraints:** number of nodes in `[0, 5000]`, node values and `targetSum` in `[-1000, 1000]`. Third occurrence in a row of the exact same `RecursionError`-at-constraint-ceiling pattern as entries #26 and #28 — this is now a confirmed recurring theme, not a one-off.

**Submission:**

```python
class Solution:
    def pathSum(self, root: TreeNode | None, targetSum: int) -> list[list[int]]:
        pathSumResult = []

        def isLeaf(node):
            if node.left is None and node.right is None:
                return True
            return False

        def pathSumUtil(node, cumSum, result):
            if node is None:
                return None
            
            #Visiting Node
            cumSum += node.val
            result.append(node.val)
            
            if cumSum  == targetSum and isLeaf(node):
                pathSumResult.append(result.copy())

            pathSumUtil(node.left, cumSum, result)
            pathSumUtil(node.right, cumSum, result)

            result.pop()
        
        pathSumUtil(root, 0, [])
        return pathSumResult
```

**The logic is correct, and is a clean, textbook-correct instance of the mark/unmark backtracking rhythm covered in the entry #25 flashcard.** `cumSum` is passed by value, so each recursive call naturally gets its own running total with no shared-state risk at all. `result` *is* shared and mutated in place across the whole traversal — but it follows the discipline exactly: append the current node's value before recursing into children, and `result.pop()` unconditionally after, regardless of whether a match was found. Crucially, `pathSumResult.append(result.copy())` takes a **copy** at the moment of a match — appending `result` directly (without `.copy()`) would be a classic aliasing bug, since every later `.pop()` would retroactively mutate every previously "saved" path through the same shared list object (the same object-sharing mechanic explained in the entry #25 flashcard, applied correctly here instead of incorrectly).

Verified against an independent oracle (an explicit iterative stack that reconstructs a brand-new path list at every node via `path + [node.val]` rather than mutating one shared list — a deliberately different mechanism to avoid validating the submission against its own approach) across 3,000 randomized trees up to 20 nodes — **0 mismatches** — plus the classic LeetCode example (target 22 on the standard 8-node example tree → `[[5,4,11,2],[5,8,4,5]]`), the empty-tree case (→ `[]`), and single-node matching/non-matching cases.

**The now-familiar `RecursionError`, confirmed a third time at this problem's own constraint ceiling.** `[0, 5000]` nodes, and nothing requires balance — a 5,000-node, single-child-per-node skewed tree is legal. Found the same kind of threshold as entries #26 and #28:

```
skewed tree n=997:  OK
skewed tree n=998:  RecursionError
skewed tree n=5000: RecursionError
```

**Fix — iterative, explicit stack, each entry carrying its own path and running sum:**

```python
class Solution:
    def pathSum(self, root: TreeNode | None, targetSum: int) -> list[list[int]]:
        if root is None:
            return []
        results = []
        stack = [(root, [root.val], root.val)]
        while stack:
            node, path, cumSum = stack.pop()
            if node.left is None and node.right is None:
                if cumSum == targetSum:
                    results.append(path)
                continue
            if node.left:
                stack.append((node.left, path + [node.left.val], cumSum + node.left.val))
            if node.right:
                stack.append((node.right, path + [node.right.val], cumSum + node.right.val))
        return results
```

This sidesteps the mark/unmark discipline entirely rather than needing to get it right under a stack-depth constraint: each stack entry builds its *own* new path list (`path + [node.left.val]` allocates a fresh list rather than mutating a shared one), so there's nothing to pop or backtrack — every entry is already a self-contained snapshot. **Verified** against the same oracle across the same 3,000 trials — 0 mismatches. **Confirmed the fix resolves the crash**: the exact `n=998` through `n=5,000` skewed trees that broke the recursive version now complete in under 18ms, including the full constraint ceiling.

**Takeaway:** the third strike makes this official — Python's ~1,000-frame recursion limit is a standing risk for *any* tree problem whose constraints don't explicitly bound depth, independent of whether the recursive logic itself is correct (it was, in both this entry and #28). The practical habit this log now has three data points for: after verifying correctness on randomized/balanced trees, always also construct the single-path skewed tree at the problem's actual maximum node count and run it once before calling a recursive tree solution done.

---

## 36. Revision Queue (Problems Worth Re-Solving Cold)

Problems flagged as worth coming back to later and re-solving from scratch (no peeking at the logged solution first) — either because the core idea was a genuine leap, or because the problem is a good test of whether a technique actually stuck. Newest flags at the bottom. Once re-attempted, note the outcome (solved cleanly / needed a nudge / same mistake again) rather than removing the entry.

| Problem | Why it's flagged | Review log entry | Flagged on | Re-attempt outcome |
|---|---|---|---|---|
| Russian Doll Envelopes (LeetCode 354) | The 2D→1D reduction to LIS via sort-by-width-ascending/height-descending-on-ties is a non-obvious trick — good test of whether the "collapse one dimension via sort + monotonic tie-break" pattern actually generalizes, not just memorized for this one problem. | #20 | 2026-09-25 | *(not yet re-attempted)* |

---

*Quick-reference version 1.0 — Python 3.10+ assumed (a few features noted inline need 3.9/3.10).*
