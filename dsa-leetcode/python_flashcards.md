# My Python Flashcards

Personal recall cards — things I keep forgetting and want to lock in. Curated by me, one card at a time.

Format: each card is a question I should be able to answer cold, followed by the answer. Organized loosely by category; newest cards added at the bottom of their category unless noted otherwise.

---

## Table of Contents

- [Syntax & Language Gotchas](#syntax--language-gotchas)
- [Algorithm Patterns](#algorithm-patterns)
- [Bugs I've Actually Hit](#bugs-ive-actually-hit)
- [Concurrency & Internals](#concurrency--internals)
- [Interview Talking Points](#interview-talking-points)

---

## Syntax & Language Gotchas

**Q: How do I square every element of a list using a lambda?**
A:
```python
nums = [1, 2, 3, 4]
squared = list(map(lambda x: x**2, nums))   # [1, 4, 9, 16]

# More Pythonic and usually preferred — a list comprehension does the same
# thing without the extra map()/lambda/list() wrapping:
squared = [x**2 for x in nums]              # [1, 4, 9, 16]
```
`map()` returns a lazy iterator (like a generator), so wrap it in `list(...)` to see/use the values right away. `**` is the power operator (`x**2` = x squared). (see main cheatsheet Section 7 — Comprehensions & Generators)

**Q: How do I sort a list in place vs. get a sorted copy?**
A:
```python
a = [3, 1, 2]
a.sort()            # IN PLACE — mutates a, returns None
print(a)             # [1, 2, 3]

c = [3, 1, 2]
d = sorted(c)        # COPY — returns a NEW sorted list, c is untouched
print(c, d)           # [3, 1, 2] [1, 2, 3]
```
Memory trick: `.sort()` is a **method on the list** (`a.sort()`) — methods that mutate generally return `None`, so `a = a.sort()` is a classic bug that wipes out your list. `sorted(...)` is a **built-in function** that works on any iterable (not just lists) and always hands back a new list, leaving the original alone. Both take `key=` and `reverse=` the same way. (see main cheatsheet Section 3 — Lists, Section 9 — Sorting Tricks)

---

## Algorithm Patterns

**Q: In backtracking, why isn't marking a cell `visited = True` enough — why do I also need to unmark it (`visited = False`) after trying all the neighbors?**
A: Because Python passes the `visited` list into every recursive call as a reference to the *same* object, not a copy ("pass by object reference" / "call by sharing" — distinct from true pass-by-reference, since reassigning the parameter inside the function wouldn't affect the caller, but in-place mutation like `visited[i][j] = True` is visible everywhere that holds that reference). There's only ever one shared grid for a given search, so if a failed branch doesn't undo its own marks before returning, those marks stay poisoned forever — even for completely unrelated sibling branches that come later and have nothing to do with the failed attempt.

Concrete proof (from LeetCode 79, Word Search — review log entry #25): on
```
board = [['A','B','B'],
         ['A','B','A'],
         ['A','B','A']]
word  = "BBABA"
```
cell `(0,2)` gets marked, explored, hits a dead end, and gets unmarked — all inside a branch starting `(0,1)→(1,1)→...` that ultimately fails. The search then backs out to `(0,1)` and tries its *other* neighbor, which is also `(0,2)`. That reuse is only legal because the first attempt cleaned up after itself. Delete the unmark line and replay it: `visited[0][2]` is still `True` from the failed attempt, the second (successful) path through it never gets explored, and the function wrongly reports the word as not found — a real false negative, not a hypothetical one. The fix is the standard backtracking rhythm: mark right before recursing, unmark right after, on every return path (success or failure) — so a cell's "in use" status is scoped to exactly the one branch currently exploring through it, nothing more.
(see main cheatsheet, review log entry #25 — Word Search)

---

## Bugs I've Actually Hit

*(no cards yet)*

---

## Concurrency & Internals

*(no cards yet)*

---

## Interview Talking Points

*(no cards yet)*

---

<!-- Card template:
**Q: <question>**
A: <answer — keep it tight, this is a recall aid not an essay>
(see cheatsheet Section X / review log entry #N, if relevant)
-->
