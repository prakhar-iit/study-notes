# CS Interview Teasers

Frequently asked teasers from backend, systems and AI-platform interviews. Each one opens with a full problem statement. Stop there, think for about 5 minutes and sketch your answer, then expand the hint or the answer.

- **Practise out loud.** Aim for under 2 minutes per teaser, using five beats: concept, mechanism, number, failure mode, trade-off.
- **Ask before you solve.** Most teasers hinge on a clarifying question (the alphabet, the memory limit, whether the answer may be approximate). Asking it is part of what's being scored.
- **Size it first.** Do the arithmetic out loud before proposing a design. Numbers you'll reuse are in section 6.
- **Keep it growing.** Add new teasers under the matching section, and log each attempt in the practice log at the end.

## Contents

1. [Big data, small memory](#1-big-data-small-memory)
2. [Counting and probability](#2-counting-and-probability)
3. [Classic logic puzzles](#3-classic-logic-puzzles)
4. [Bit and array tricks](#4-bit-and-array-tricks)
5. [Distributed systems and backend](#5-distributed-systems-and-backend)
6. [Estimation](#6-estimation)
7. [Choosing the right data structure](#7-choosing-the-right-data-structure)
8. [Practice log](#practice-log)

---

## 1. Big data, small memory

Almost every one of these is solved by one of three moves: shrink the key space into a bitmap, hash-partition to disk so that equal items land together, or accept an approximate answer from a sketch.

### 1.1 Duplicates among 1 billion 5-character strings

**Problem:** A file holds 1,000,000,000 strings, one per line, each exactly 5 characters long. Your machine has 1 GB of RAM and plenty of disk. Output every string that appears more than once, listing each duplicate once. What would you ask the interviewer before designing anything?

<details><summary>Hint</summary>

Compare the data size with RAM, then count how many *different* 5-character strings can exist.

</details>

<details><summary>Answer</summary>

The data is 5 GB, five times the RAM, so first ask what the alphabet is. Encode each string as a base-|alphabet| integer and index a bitmap.

- **Alphanumeric:** 62⁵ ≈ 916 M possible strings, so one bitmap is 115 MB. Keep two (`seen`, `dup`), about 230 MB in one sequential pass.
- **Lowercase only:** 26⁵ ≈ 11.9 M possible strings. Duplicates are guaranteed (pigeonhole).
- **Arbitrary bytes:** 256⁵ ≈ 1.1 T possible strings, a 137 GB bitmap. Instead hash-partition into 64 files, so equal strings share a file, then count each file in a hash map of about 250 MB.

**Follow-up:** "What if one bucket is huge?" A count map's memory depends on *distinct* keys, so many copies of one string are harmless. Too many distinct keys means re-splitting that bucket with a new hash seed.

</details>

### 1.2 A missing integer among 4 billion

**Problem:** A file contains 4 billion non-negative 32-bit integers (values 0 to 2³² − 1) in random order, and some may repeat. Output any value that does not appear in the file. Solve it with 1 GB of RAM, then with only 10 MB. You may read the file more than once.

<details><summary>Hint</summary>

How many possible values are there, compared with how many are in the file?

</details>

<details><summary>Answer</summary>

**1 GB:** There are 2³² ≈ 4.29 billion possible values and only 4 billion in the file, so one is missing. A bitmap of 2³² bits is 512 MB. Set a bit for each value, then scan for a zero.

**10 MB:** Use two passes.

1. Split the range into 2¹⁶ blocks of 2¹⁶ values each, and count how many file values fall in each block. That's 65,536 counters at 4 bytes each, 256 KB.
2. Any block with fewer than 65,536 values has a gap. Re-read the file, bitmapping only that block (8 KB).

**Follow-up:** "Does it still work with duplicates?" Yes. The block counts sum to 4 billion, which is less than 2³², so some block must have fewer than 2¹⁶ values.

</details>

### 1.3 Top 10 queries in a 100 GB log

**Problem:** A 100 GB log file has one search query per line (each up to about 100 bytes). Your machine has 4 GB of RAM. Return the 10 most frequent queries with their counts. Then: the log is an endless live stream, and you need the current top 10 in near real time.

<details><summary>Hint</summary>

Identical queries must be counted together, but all distinct queries won't fit in RAM at once.

</details>

<details><summary>Answer</summary>

**Exact:** Hash-partition by query into N files, so each query lives in exactly one file. Count each file in a hash map, and keep a **min-heap of size K** across all files. That's O(n log K) time.

**Streaming, approximate:** A Count-Min Sketch plus a size-K heap, or the Space-Saving or Misra–Gries algorithms with a fixed number of counters.

**Follow-up:** "Why a min-heap?" The root is the smallest of your current top K, so each new item is compared against it and evicts it in O(log K). "Top K in the last hour?" Keep per-minute count buckets and sum the last 60.

</details>

### 1.4 Sort 100 GB with 1 GB of RAM

**Problem:** A 100 GB file of fixed-size records (100 bytes each, with an 8-byte key) must be written out sorted by key. You have 1 GB of RAM and enough disk for a few copies of the data. Describe the algorithm and estimate the total disk I/O.

<details><summary>Hint</summary>

You can sort 1 GB at a time. What do you do with 100 sorted pieces?

</details>

<details><summary>Answer</summary>

External merge sort.

1. Read 1 GB at a time, sort it in memory, and write it out as a sorted run. That gives 100 runs.
2. Do one 100-way merge with a min-heap holding the head of each run, giving each run about a 10 MB read buffer.

Total I/O is about 400 GB: read and write twice.

**Follow-up:** "Too many runs for one merge?" Merge in multiple passes. Replacement selection produces runs about twice the memory size on random input, which halves the run count.

</details>

### 1.5 Count unique visitors in a huge stream

**Problem:** Your site emits a stream of page-view events, each carrying a 64-bit visitor ID, billions per day. Report the number of unique visitors per day and per week, using at most a few megabytes of memory. An error of about ±1% is acceptable. Why can't you get the exact number within that memory?

<details><summary>Hint</summary>

In a stream of random hashes, a long run of leading zeros is rare. How rare tells you roughly how many distinct values you've seen.

</details>

<details><summary>Answer</summary>

An exact count needs memory proportional to the number of distinct IDs: 1 billion IDs in a hash set is roughly 16 GB. Use HyperLogLog instead. Hash each item, use the first p bits to pick a register, and record the longest run of leading zeros in the remaining bits. Standard error is about 1.04/√m for m registers. Redis's implementation uses 12 KB for about 0.81% error.

**Follow-up:** HLLs merge by taking the per-register max, so daily sketches combine into weekly uniques. Intersections are inaccurate, because subtracting two estimates compounds their errors.

</details>

### 1.6 Uniform sample from a stream of unknown length

**Problem:** Items arrive one at a time from a stream whose total length n you don't know in advance and can't store. In one pass, using memory for only k items, keep a sample such that when the stream ends, every item has the same probability (k/n) of being in it.

<details><summary>Hint</summary>

Start with k = 1. When the i-th item arrives, with what probability should it replace the item you're holding?

</details>

<details><summary>Answer</summary>

Reservoir sampling. Keep the first k items. For item i (counting from 1, i > k), pick a random j from 1 to i, and if j ≤ k, replace slot j.

**Why (k = 1):** Item i is kept with probability 1/i and then survives each later item j with probability (j − 1)/j. The product telescopes to 1/n.

**Follow-up:** "Weighted sampling?" Give each item a key u^(1/w), with u uniform in (0, 1), and keep the k largest keys. "Across several machines?" The same keyed approach merges: take the top k keys overall.

</details>

### 1.7 Has the crawler seen this URL?

**Problem:** A web crawler will encounter about 1 billion distinct URLs (100 bytes each on average) and has about 2 GB of RAM for its "already fetched?" check. It must never re-fetch a URL it already has, but skipping about 1% of new URLs by mistake is acceptable. Design the check and size it.

<details><summary>Hint</summary>

Storing the URLs themselves needs 100 GB. What if you stored only a few bits per URL and accepted one-sided errors?

</details>

<details><summary>Answer</summary>

A Bloom filter: a bit array of m bits and k hash functions. Set k bits on insert, and check all k on lookup. False positives are possible (a new URL looks seen and is skipped); false negatives are not (a fetched URL is never re-fetched), which matches the requirement.

```math
\frac{m}{n} = \frac{-\ln p}{(\ln 2)^2}, \qquad k = \frac{m}{n}\ln 2
```

A 1% false-positive rate needs about 9.6 bits per item and 7 hashes, so 1 billion URLs need about 1.2 GB. A 0.1% rate needs about 14.4 bits per item.

**Follow-up:** "Deletions?" Use a counting Bloom filter or a cuckoo filter. "Where is it used?" LSM-tree stores such as RocksDB and Cassandra use one per SSTable to skip disk reads for keys that aren't there.

</details>

---

## 2. Counting and probability

These reward a clean setup more than clever math: define the outcomes, count them, and check against a tiny case.

### 2.1 Counting pairs

**Problem:** You have n distinct items. How many different unordered pairs can you choose? Derive the formula rather than recalling it, then generalize to groups of k. Finally, how long would it take to compare every pair among 1 billion records at 10⁹ comparisons per second?

<details><summary>Hint</summary>

Count ordered pairs first, then ask how many times each unordered pair was counted.

</details>

<details><summary>Answer</summary>

There are n choices for the first item and n − 1 for the second, giving n(n − 1) *ordered* pairs. Each unordered pair {A, B} was counted twice, as (A, B) and (B, A), so divide by 2.

```math
\binom{n}{2} = \frac{n(n-1)}{2}, \qquad \binom{n}{k} = \frac{n!}{k!\,(n-k)!}
```

**Second view:** Handshakes. Person i shakes hands with the i − 1 people before them, so the total is 0 + 1 + … + (n − 1).

**Why it matters:** 1 billion records give about 5 × 10¹⁷ pairs, roughly 16 years at 10⁹ per second. That's why the big-data teasers avoid pairwise comparison.

</details>

### 2.2 Birthday paradox and hash collisions

**Problem:** (a) In a room of n people whose birthdays are uniform over 365 days (ignore leap years), what is the smallest n for which the chance that at least two share a birthday exceeds 50%? (b) You give records IDs from a random 32-bit hash. Roughly how many records can you create before a collision is more likely than not? What about a 64-bit hash, and a random UUID v4?

<details><summary>Hint</summary>

It's easier to compute the probability that *nobody* matches. And count pairs, not people.

</details>

<details><summary>Answer</summary>

(a) 23. The chance that nobody matches is the product of (1 − i/365) for i from 0 to n − 1, which drops below 0.5 at n = 23.

(b) Collisions become likely at about √N items, not N; a 50% chance arrives after roughly 1.18 × √N items.

- **32-bit hash:** about 77,000 items.
- **64-bit hash:** about 5 billion items.
- **UUID v4 (122 random bits):** about 2.7 × 10¹⁸ items, which is why random UUIDs are safe as IDs.

</details>

### 2.3 Expected number of colliding pairs

**Problem:** You hash n keys into m buckets; each key lands in a uniformly random bucket, independently of the others. What is the expected number of pairs of keys that land in the same bucket? Evaluate it for n = 10,000 and a 32-bit hash (m = 2³²).

<details><summary>Hint</summary>

Linearity of expectation: add up the collision probability of each pair.

</details>

<details><summary>Answer</summary>

Each of the nC2 pairs collides with probability 1/m, so the expectation is nC2 / m. For 10,000 keys that's about 5 × 10⁷ / 4.3 × 10⁹ ≈ 0.012 expected colliding pairs.

</details>

### 2.4 Monty Hall

**Problem:** A game show has three closed doors: one hides a car, two hide goats. You pick a door. The host, who knows where the car is, always opens one of the other two doors to reveal a goat, then offers you the chance to switch to the remaining closed door. Should you switch? What are your chances of winning if you stay, and if you switch?

<details><summary>Hint</summary>

What's the chance your first pick was wrong? In that case, where is the car after the host opens a door?

</details>

<details><summary>Answer</summary>

Switch. Staying wins 1/3 of the time; switching wins 2/3. Your first pick is wrong 2/3 of the time, and in every one of those cases the host's reveal leaves the car behind the other closed door.

**Follow-up:** With 100 doors and 98 opened, switching wins 99/100. That version makes the intuition obvious.

</details>

### 2.5 A fair coin from a biased coin

**Problem:** You have a coin that lands heads with some unknown probability p (0 < p < 1). Using only this coin, produce a perfectly fair 50/50 outcome. How many flips does your method take on average?

<details><summary>Hint</summary>

Look for two outcomes that must have exactly the same probability, whatever p is.

</details>

<details><summary>Answer</summary>

Von Neumann's trick. Flip twice: HT means heads, TH means tails, and HH or TT means start over. HT and TH both have probability p(1 − p), so they're equally likely.

**Follow-up:** Each round uses 2 flips and succeeds with probability 2p(1 − p), so the expected count is 1 / (p(1 − p)) flips. A very biased coin is slow.

</details>

### 2.6 rand7() from rand5()

**Problem:** You have a function rand5() that returns an integer from 1 to 5, uniformly at random. Using only rand5(), write rand7() that returns an integer from 1 to 7 uniformly. Show that it's uniform, and estimate how many rand5() calls it makes on average.

<details><summary>Hint</summary>

Two calls can produce 25 equally likely outcomes. How many of those can you use?

</details>

<details><summary>Answer</summary>

Rejection sampling. Compute x = 5 × (rand5() − 1) + rand5(), which is uniform on 1 to 25. If x ≤ 21, return ((x − 1) mod 7) + 1; otherwise try again. The 21 accepted values map 3 each onto 1 to 7.

**Follow-up:** "Why not (rand5() + rand5()) mod 7?" A sum of two dice isn't uniform; middle values are more likely. Expected rand5() calls are 2 × 25/21 ≈ 2.38.

</details>

### 2.7 Coupon collector

**Problem:** A load balancer sends each request to one of 100 servers, chosen uniformly at random. On average, how many requests are sent before every server has received at least one? Give the general formula for n servers.

<details><summary>Hint</summary>

Split the process into stages: waiting for the 1st new server, the 2nd new server, and so on. How long does each stage take on average?

</details>

<details><summary>Answer</summary>

With i servers already hit, a new one comes up with probability (n − i)/n, so that stage takes n/(n − i) requests on average. Summing gives n × Hₙ ≈ n ln n + 0.577n. For n = 100 that's about 519.

</details>

### 2.8 Waiting for two heads in a row

**Problem:** You flip a fair coin repeatedly until you see two heads in a row (HH). What is the expected number of flips? Then do the same for heads followed by tails (HT). Why are the two answers different?

<details><summary>Hint</summary>

Write an equation for the expected remaining flips from each state: "just started" and "last flip was heads".

</details>

<details><summary>Answer</summary>

6. Let E be the expected flips from the start and E_H the expected flips after one head.

- E = 1 + ½ × E_H + ½ × E
- E_H = 1 + ½ × 0 + ½ × E

Solving gives E = 6.

**Follow-up:** HT takes only 4. A failed HH attempt (a tail) sends you back to the start, while a failed HT attempt (another head) leaves you one step in.

</details>

### 2.9 Shuffle an array uniformly

**Problem:** Given an array of n elements and a function that returns a uniform random integer in any range you ask for, rearrange the array in place so that each of the n! orderings is equally likely. Then explain what goes wrong with the "naive" version that swaps each position with a random index from 0 to n − 1.

<details><summary>Hint</summary>

Build the result one position at a time: how many choices should the last position have, then the one before it?

</details>

<details><summary>Answer</summary>

Fisher–Yates. For i from n − 1 down to 1, swap a[i] with a[j], where j is random in 0 to i. That's n × (n − 1) × … × 1 = n! equally likely paths, one per ordering.

**Follow-up:** The naive version has nⁿ equally likely paths, which isn't divisible by n! for n ≥ 3 (27 paths onto 6 orderings for n = 3), so some orderings must be more likely.

</details>

---

## 3. Classic logic puzzles

The interviewer is grading how you reason: state the bound ("each weighing has 3 outcomes") before the construction, and test a small case aloud.

### 3.1 Two eggs, 100 floors

**Problem:** You have a 100-story building and two identical eggs. There is a threshold floor F: an egg dropped from floor F or higher breaks, and from any lower floor it survives (F may be anywhere from 1 to 100, or the egg may never break). A surviving egg can be reused; a broken one can't. Find F with the fewest drops in the worst case. What is your strategy, and how many drops does it need?

<details><summary>Hint</summary>

Once the first egg breaks, the second must go floor by floor. So if the first egg breaks on its k-th drop, how far apart could its drops have been?

</details>

<details><summary>Answer</summary>

14 drops. Each later jump of the first egg must be one floor shorter, so the total work stays constant. Drop it at 14, 27, 39, 50, 60, 69, 77, 84, 90, 95, 99, 100. You need n(n + 1)/2 ≥ 100, and 14 × 15/2 = 105.

**Follow-up:** "k eggs?" Use DP on moves and eggs: f(m, k) = f(m − 1, k − 1) + f(m − 1, k) + 1 floors are coverable with m moves and k eggs. Find the smallest m with f(m, k) ≥ floors.

</details>

### 3.2 25 horses, 5 lanes

**Problem:** There are 25 horses and a track that races 5 at a time. You have no stopwatch: each race tells you only the finishing order of the 5 horses in it. Each horse always runs at the same speed. What is the minimum number of races needed to identify the 3 fastest horses, in order?

<details><summary>Hint</summary>

After racing everyone once, race the winners. Then list which horses could still be 2nd or 3rd overall.

</details>

<details><summary>Answer</summary>

7 races.

1. Races 1–5: race each group of 5.
2. Race 6: race the 5 group winners. The winner is fastest overall. Call the groups of the 1st, 2nd and 3rd finishers A, B and C.
3. Race 7: the only remaining candidates for places 2 and 3 are A2, A3, B1, B2 and C1. Race them; the top two finishers are 2nd and 3rd overall.

</details>

### 3.3 1,000 bottles, one poisoned

**Problem:** You have 1,000 wine bottles, and exactly one is poisoned. Even a sip kills, but symptoms appear exactly 24 hours later. The party starts in 24 hours, so there's time for only one round of tasting. Each tester can sip from as many bottles as you like. What is the minimum number of testers needed to identify the poisoned bottle for certain? What if you had 48 hours?

<details><summary>Hint</summary>

Each tester ends up in one of two states, alive or dead. How many bottles can n yes/no answers distinguish?

</details>

<details><summary>Answer</summary>

10, since 2¹⁰ = 1,024. Number the bottles in binary; tester i drinks from every bottle whose bit i is 1. The set of testers who die spells the bottle's number.

**Follow-up:** With 48 hours, each tester has three outcomes (dies on day 1, dies on day 2, survives), so 3⁷ = 2,187 means 7 testers.

</details>

### 3.4 Find the heavier ball

**Problem:** You have 8 identical-looking balls. Seven weigh the same and one is slightly heavier. You have a two-pan balance that only shows which side is heavier, or that they balance. What is the minimum number of weighings that guarantees finding the heavy ball? What's the largest number of balls you could handle with the same number of weighings?

<details><summary>Hint</summary>

Each weighing has three possible results, not two.

</details>

<details><summary>Answer</summary>

2 weighings. Weigh 3 against 3. If one side is heavier, weigh 1 against 1 from that side (balanced means it's the third). If balanced, weigh the remaining 2. Since each weighing has 3 outcomes, k weighings separate up to 3ᵏ candidates: 9 balls in 2 weighings.

</details>

### 3.5 12 coins, one fake

**Problem:** Twelve coins look identical. Exactly one is counterfeit and is either heavier or lighter than the rest; you don't know which. Using a two-pan balance exactly three times, identify the counterfeit coin and say whether it's heavier or lighter. Why can't it be done in two weighings?

<details><summary>Hint</summary>

Count the possible answers (coin × heavier/lighter) and compare with the number of outcomes three weighings can produce.

</details>

<details><summary>Answer</summary>

There are 24 possibilities (12 coins × heavier or lighter), and 3³ = 27 outcomes is just enough; two weighings give only 9. Start by weighing 4 against 4.

- **Balanced:** the fake is among the other 4. Weigh 3 of them against 3 known-good coins.
- **Unbalanced:** the fake is among the 8, and you know which side each could be heavy or light on. Rotate coins between the pans so that every outcome narrows to 3 or fewer cases.

</details>

### 3.6 Two burning ropes

**Problem:** You have two ropes and a lighter. Each rope takes exactly 60 minutes to burn from one end to the other, but it burns unevenly, so half a rope does not take 30 minutes. You can't cut or measure the ropes. Measure exactly 45 minutes.

<details><summary>Hint</summary>

Lighting a rope at both ends halves its burn time, however unevenly it burns.

</details>

<details><summary>Answer</summary>

Light rope A at both ends and rope B at one end. A finishes at 30 minutes; at that moment light B's other end, and it finishes 15 minutes later.

</details>

### 3.7 Bridge and torch

**Problem:** Four people must cross a bridge at night. The bridge holds at most two people at a time, and they have one torch, which must be carried on every crossing (it can't be thrown). Alone, they cross in 1, 2, 5 and 10 minutes; a pair moves at the slower person's pace. What is the shortest total time for all four to get across?

<details><summary>Hint</summary>

The obvious plan (the fastest person escorts everyone) takes 19 minutes. Can the two slowest cross together?

</details>

<details><summary>Answer</summary>

17 minutes.

1. 1 and 2 cross (2).
2. 1 returns (1).
3. 5 and 10 cross (10).
4. 2 returns (2).
5. 1 and 2 cross (2).

The trick is sending the two slowest together, and letting the two fastest shuttle the torch.

</details>

### 3.8 100 doors

**Problem:** 100 doors in a row all start closed. On pass 1 you toggle every door; on pass 2, every 2nd door; on pass k, every k-th door; and so on up to pass 100. (Toggling opens a closed door and closes an open one.) Which doors are open at the end, and why?

<details><summary>Hint</summary>

Door d is toggled once for each of its divisors. Which numbers have an odd number of divisors?

</details>

<details><summary>Answer</summary>

The 10 perfect squares (1, 4, 9 … 100). Divisors normally come in pairs (d and n/d), except for a square root, so only perfect squares have an odd count.

</details>

---

## 4. Bit and array tricks

These usually come with an "O(1) extra space" constraint. The tools are XOR (x ^ x = 0, x ^ 0 = x), arithmetic identities, and treating an array as a linked structure.

### 4.1 The missing number

**Problem:** An array of length n − 1 holds distinct integers from 1 to n in any order, with exactly one number missing. Find the missing number in O(n) time and O(1) extra space. Bonus: avoid integer overflow when n is large.

<details><summary>Hint</summary>

You know what all the numbers together should add up to.

</details>

<details><summary>Answer</summary>

The expected sum is n(n + 1)/2; subtract the actual sum. To avoid overflow, XOR all the indices 1 to n with all the values instead; everything cancels except the missing number.

</details>

### 4.2 The number that appears once

**Problem:** In a non-empty integer array, every element appears exactly twice except one, which appears once. Find that element in O(n) time and O(1) extra space (no hash set, no sorting). Follow-up: what if every other element appears three times?

<details><summary>Hint</summary>

Which operation makes a value cancel itself out?

</details>

<details><summary>Answer</summary>

XOR every element. Pairs cancel, leaving the single number.

**Follow-up:** For the three-times version, count each bit position across all numbers mod 3; the leftover bits form the answer.

</details>

### 4.3 Two numbers that appear once

**Problem:** In an integer array, exactly two elements appear once and every other element appears exactly twice. Find both single elements in O(n) time and O(1) extra space.

<details><summary>Hint</summary>

XOR-ing everything gives a ^ b. Since a ≠ b, at least one bit of it is 1. What does that bit tell you about a and b?

</details>

<details><summary>Answer</summary>

Take the lowest set bit of a ^ b with `x & -x`; a and b differ there. Split all numbers into two groups by that bit and XOR each group separately. Each group contains one of the singles plus pairs that cancel.

</details>

### 4.4 Find the duplicate without changing the array

**Problem:** An array has n + 1 integers, each between 1 and n inclusive, so by pigeonhole some value repeats. Exactly one value is repeated (possibly more than twice). Find it without modifying the array, using O(1) extra space and less than O(n²) time.

<details><summary>Hint</summary>

Treat the array as a function i → a[i] and follow it from index 0. What shape does that path make?

</details>

<details><summary>Answer</summary>

The path is a linked list with a cycle, and the duplicate value (a node with two incoming edges) is the cycle's entrance. Use Floyd's cycle detection: run slow and fast pointers until they meet, reset one to the start, then move both one step at a time. They meet at the duplicate.

**Follow-up:** Know the alternatives and what each breaks: sorting modifies the array, a hash set uses O(n) space, sign-marking modifies the array, and binary search on value counts is O(n log n).

</details>

### 4.5 Bit one-liners

**Problem:** Using bitwise operators, write a one-liner for each: (a) test whether a positive integer n is a power of two; (b) count the 1-bits in n in time proportional to the number of 1-bits; (c) isolate the lowest set bit of x; (d) swap two integers without a temporary variable. When does (d) break?

<details><summary>Hint</summary>

Work out what `n - 1` looks like in binary compared with n.

</details>

<details><summary>Answer</summary>

- **Power of two:** `n > 0 and n & (n - 1) == 0`. A power of two has a single 1-bit, and n − 1 flips it and every bit below it.
- **Count set bits:** repeat `n &= n - 1`, which clears the lowest set bit, once per set bit (Kernighan).
- **Lowest set bit:** `x & -x`, because two's complement −x is ~x + 1.
- **Swap without a temp:** `a ^= b; b ^= a; a ^= b`. It zeroes the value if a and b are the same memory location, and in practice a temp variable is just as fast.

</details>

### 4.6 Majority element

**Problem:** An array of n elements contains one value that appears more than n/2 times. Find it in O(n) time and O(1) extra space. Follow-ups: what if a majority isn't guaranteed? How would you find every value appearing more than n/3 times?

<details><summary>Hint</summary>

Pair each majority element with a different element and cancel both. What's left?

</details>

<details><summary>Answer</summary>

Boyer–Moore voting. Keep a candidate and a count; a matching element increments, a different one decrements, and at zero you adopt the next element.

**Follow-up:** If a majority isn't guaranteed, verify the candidate with a second pass. For "more than n/3", keep two candidates. This is the Misra–Gries idea from teaser 1.3.

</details>

### 4.7 Cycle in a linked list

**Problem:** Given the head of a singly linked list, decide whether it contains a cycle and, if so, return the node where the cycle begins. Use O(1) extra memory (no set of visited nodes) and explain why your method works.

<details><summary>Hint</summary>

Two pointers moving at different speeds inside a loop must eventually meet.

</details>

<details><summary>Answer</summary>

Floyd's tortoise and hare: the slow pointer moves 1 step, the fast moves 2, and they meet only if there's a cycle. To find the start, reset one pointer to the head and step both by 1; they meet at the cycle's entrance.

**Why:** Let a be the distance from the head to the entrance, b the distance from the entrance to the meeting point, and c the cycle length. The fast pointer travelled twice as far, so 2(a + b) = a + b + kc, which gives a = kc − b. Walking a steps from the head and from the meeting point ends at the same node.

</details>

---

## 5. Distributed systems and backend

These are "why did this happen?" puzzles. Name the failure mode first, then the mechanism that fixes it.

### 5.1 Retries created duplicate records

**Problem:** An AI agent calls `POST /records` to create a record. The call times out after 10 seconds, so the agent retries, and the retry returns 201 Created. Later you find two identical records in the database. The API has no idempotency handling and the table has no unique constraint. Explain exactly how the duplicate was created, and design a fix that keeps retries safe.

<details><summary>Hint</summary>

From the client's side, "the request never arrived" and "it succeeded but the response was lost" look identical.

</details>

<details><summary>Answer</summary>

The first write most likely committed after the client gave up (or the response was lost, or an error occurred after commit), so the retry inserted again. Use a stable idempotency key (run ID + step, or a business key; never a new UUID per attempt or the LLM's tool call ID). Store it under a unique constraint in the same transaction as the write, and replay the stored response on repeats. Add a unique constraint on the natural key as a backstop, and have only one layer own retries.

</details>

### 5.2 Exactly-once delivery

**Problem:** Your product manager wants every order event sent from service A to service B through a message queue "exactly once": never lost and never processed twice. Is exactly-once *delivery* possible over an unreliable network? If not, what do you build to get the same effect? What does Kafka's "exactly-once" feature actually guarantee?

<details><summary>Hint</summary>

Think about what happens when B processes a message but its acknowledgement is lost.

</details>

<details><summary>Answer</summary>

Not as a network guarantee. A lost ack forces a choice between redelivery (duplicates) and giving up (loss). What you build is at-least-once delivery plus an idempotent consumer (dedupe by event ID in the same transaction as the side effect), which gives *effectively-once* processing.

**Follow-up:** Kafka's idempotent producer plus transactions gives exactly-once for read–process–write *within Kafka*. Side effects outside Kafka, such as an email or a payment, still need idempotency.

</details>

### 5.3 Two Generals

**Problem:** Two armies camp on hills on opposite sides of a valley. They win only if both attack at the same time; attacking alone means defeat. They communicate only by messengers who must cross the valley, where any messenger may be captured. Can the generals design a protocol that guarantees both commit to the same attack time? What does this imply for networked systems?

<details><summary>Hint</summary>

Suppose a protocol works with a fixed number of messages. What happens if the very last one is lost?

</details>

<details><summary>Answer</summary>

No. Every message needs an acknowledgement to be trusted, and that acknowledgement needs one too, so no finite exchange gives certainty. In practice you design for uncertainty: timeouts, retries, idempotency, and protocols that tolerate one side acting before the other knows.

</details>

### 5.4 Ordering by wall-clock time

**Problem:** Several servers write to the same key, and "last write wins" by comparing each write's wall-clock timestamp. Users report that newer updates are sometimes overwritten by older ones. Why is wall-clock ordering unreliable across machines, and what can you use instead?

<details><summary>Hint</summary>

What happens to a server's clock when NTP corrects it, or when two servers are 50 ms apart?

</details>

<details><summary>Answer</summary>

Clocks drift and skew between machines, and NTP corrections can move them backward, so a later timestamp doesn't mean "happened later." Use a single sequence (a DB sequence or a log offset), Lamport clocks for an order consistent with causality, or vector clocks to detect truly concurrent writes. Hybrid logical clocks combine physical and logical time. Google Spanner's TrueTime instead bounds clock uncertainty and waits it out before commit.

</details>

### 5.5 Design a rate limiter

**Problem:** A public API must limit each API key to 100 requests per minute while allowing short bursts. It runs on 20 stateless servers behind a load balancer. Choose an algorithm, decide where its state lives, specify the response for a client over the limit, and explain the weakness of a simple fixed one-minute counter.

<details><summary>Hint</summary>

Picture a client that sends 100 requests at 12:00:59 and 100 more at 12:01:01.

</details>

<details><summary>Answer</summary>

A token bucket per key: capacity b (say 100), refilled at r = 100/60 tokens per second, where each request takes one token. It allows bursts up to b while holding the long-run rate to r. Over the limit, return 429 with a `Retry-After` header.

**Distributed:** The 20 servers share state in Redis, with check-and-decrement made atomic by a Lua script (or `INCR` plus an expiry per window).

**Fixed window weakness:** That client gets 200 requests in 2 seconds, twice the intended rate. Sliding windows or token buckets avoid it.

</details>

### 5.6 Cache stampede

**Problem:** A homepage reads a value from cache with a 60-second TTL. Recomputing it is a 2-second database query. At 10,000 requests per second, every time the key expires the database falls over. Explain what's happening and give at least three fixes.

<details><summary>Hint</summary>

How many requests arrive during the 2 seconds while the value is being rebuilt, and what does each of them do?

</details>

<details><summary>Answer</summary>

About 20,000 requests miss during the rebuild, and each runs the same query: a thundering herd. Fixes: let only one request rebuild (a per-key lock or single-flight), serve the stale value while it refreshes (stale-while-revalidate), refresh probabilistically before expiry, and add jitter to TTLs so related keys don't expire together.

</details>

### 5.7 Consistent hashing

**Problem:** You shard a cache across N servers with `server = hash(key) mod N`. When you add one server (N → N + 1), nearly every lookup misses and the database is flooded. Why? Design a key-to-server mapping where adding or removing a server moves only a small fraction of keys, while keeping load balanced.

<details><summary>Hint</summary>

With mod N, how many keys keep the same server when N changes?

</details>

<details><summary>Answer</summary>

With `mod N`, going to N + 1 remaps about N/(N + 1) of all keys. Instead put servers and keys on a hash ring; each key goes to the next server clockwise. Adding or removing a server moves only about 1/N of the keys. Use 100 to 200 virtual nodes per server to even out the load.

</details>

### 5.8 Read-your-writes

**Problem:** A user changes their display name, the page reloads, and it still shows the old name; a few seconds later the new name appears. Writes go to a primary database and reads go to asynchronously replicated read replicas. Explain the bug and give ways to guarantee users always see their own writes.

<details><summary>Hint</summary>

Where did the write go, and where did the reload read from?

</details>

<details><summary>Answer</summary>

The read hit a replica that hadn't applied the write yet (replication lag). For read-your-writes, route that user's reads to the primary for a short window after a write, use sticky sessions, or return the write's log position (LSN) and have the read wait until a replica has reached it.

</details>

### 5.9 Two buyers, one item

**Problem:** A store has 1 unit of an item left. Two customers click "Buy" at the same moment. Each request runs: read stock; if stock > 0, write stock = stock − 1 and create an order. Both orders succeed. Explain the race and fix it in SQL.

<details><summary>Hint</summary>

Both requests can read stock = 1 before either writes.

</details>

<details><summary>Answer</summary>

This is a lost update: the check and the write are separate steps. Make them one atomic statement: `UPDATE items SET stock = stock - 1 WHERE id = ? AND stock > 0`, then create the order only if one row was affected. Alternatives are `SELECT … FOR UPDATE` (pessimistic) or a version column with compare-and-set (optimistic).

</details>

### 5.10 Fan-out and tail latency

**Problem:** A search request fans out to 100 backend shards in parallel and waits for all of them. Each shard answers fast 99% of the time and slowly (about 1 second) 1% of the time. What fraction of user requests are slow? How do you reduce that tail?

<details><summary>Hint</summary>

The request is fast only if every one of the 100 shards is fast.

</details>

<details><summary>Answer</summary>

1 − 0.99¹⁰⁰ ≈ 63% of requests wait on at least one slow shard, so one shard's p99 becomes everyone's median. Fixes: hedged requests (send a duplicate after the p95 time and take the first reply), tighter per-call timeouts with partial results, and fewer fan-out hops.

</details>

---

## 6. Estimation

Interviewers care about the right order of magnitude, said out loud. These figures are approximate and rounded for easy mental math; real hardware varies.

| Operation | Approximate latency |
| --- | --- |
| L1 cache reference | 1 ns |
| Main memory reference | 100 ns |
| SSD random read | 20–100 µs |
| Round trip within a datacenter | 0.5 ms |
| Disk (HDD) seek | 10 ms |
| Round trip US to Europe | 150 ms |

| Medium | Approximate sequential throughput |
| --- | --- |
| Memory | 10+ GB/s |
| NVMe SSD | 3–7 GB/s |
| SATA SSD | 0.5 GB/s |
| HDD | 0.1–0.2 GB/s |
| 10 Gbps network | 1.25 GB/s |

**Rules of thumb:** 2¹⁰ ≈ 10³, 2²⁰ ≈ 10⁶, 2³⁰ ≈ 10⁹, 2³² ≈ 4.3 × 10⁹, 2⁴⁰ ≈ 10¹². A day is 86,400 seconds, about 10⁵. So 1 million requests a day is about 12 QPS, and 1 billion a day is about 12,000 QPS. Peak traffic is often 2–3× the average.

### 6.1 KV cache per token

**Problem:** You serve Llama 3 8B in fp16 on one 80 GB GPU. The model has 32 layers, 8 key/value heads (grouped-query attention) and a head dimension of 128. How much memory does the KV cache need per token of context? How many concurrent 8,192-token sequences fit after loading the weights?

<details><summary>Hint</summary>

Every layer stores one key vector and one value vector per KV head for every token.

</details>

<details><summary>Answer</summary>

2 (K and V) × layers × KV heads × head dimension × bytes per value = 2 × 32 × 8 × 128 × 2 = 131,072 bytes, which is 128 KiB per token. An 8K-token sequence needs 1 GiB. The weights take about 16 GB, leaving about 64 GB, so roughly 64 sequences.

**Follow-up:** That limit is why grouped-query attention, KV-cache quantization and paged attention matter.

</details>

### 6.2 Decode speed ceiling

**Problem:** The same 8B fp16 model (about 16 GB of weights) runs on one H100 with about 3.35 TB/s of memory bandwidth and roughly 1,000 TFLOPS of dense fp16 compute. For a single user (batch size 1), what is the theoretical maximum decode speed in tokens per second? Which resource limits it, and how does batching change the picture?

<details><summary>Hint</summary>

Generating one token needs about 2 FLOPs per parameter, but also reading every weight from memory once. Which takes longer?

</details>

<details><summary>Answer</summary>

Reading 16 GB at 3.35 TB/s takes about 4.8 ms, so the ceiling is about 200 tokens per second. The compute needed (about 16 GFLOPs per token) takes only about 0.016 ms, so decode at batch size 1 is memory-bound. Batching reuses each weight read across many sequences, which is why throughput scales with batch size until compute or KV-cache memory runs out.

</details>

### 6.3 Storage for 1 billion records

**Problem:** You need to store 1 billion records of about 100 bytes each in a database with secondary indexes, replicated 3 ways. Roughly how much disk do you provision?

<details><summary>Hint</summary>

Start from raw bytes, then multiply for each kind of overhead.

</details>

<details><summary>Answer</summary>

100 GB raw. Add indexes (often 0.5–1× more) and 3× replication, and plan for about 450–600 GB, plus headroom for growth and compaction.

</details>

### 6.4 QPS for 100 million users

**Problem:** An app has 100 million daily active users, each making about 10 API requests a day. What average and peak requests per second should the backend handle?

<details><summary>Hint</summary>

A day is about 10⁵ seconds.

</details>

<details><summary>Answer</summary>

1 billion requests a day is about 12,000 QPS on average. With peaks at 2–3× the average, plan for 25,000–35,000 QPS.

</details>

---

## 7. Choosing the right data structure

Interviewers phrase these as features. The real work is listing the operations the feature needs and how fast each must be, then picking the structure (often two combined) that makes all of them cheap at once.

| You need | Pick | Typical cost |
| --- | --- | --- |
| Lookup or insert by key | Hash map | O(1) average |
| Sorted keys, floor/ceiling, ranges | Balanced BST, skip list or sorted map | O(log n) |
| The current min or max, repeatedly | Heap (priority queue) | O(log n) push/pop, O(1) peek |
| Recency order with O(1) moves | Hash map + doubly linked list | O(1) |
| Prefix lookups | Trie | O(prefix length) |
| Window max/min, next greater element | Monotonic deque or stack | O(1) amortized |
| Groups that only ever merge | Union-find | Near O(1) amortized |
| Range sums with updates | Fenwick tree or segment tree | O(log n) |
| Dependencies and ordering | DAG + topological sort | O(V + E) |
| Counts over a sliding time window | Circular buffer of time buckets | O(1) |
| Nearby points on a map | Geohash/S2 grid or quadtree | About O(1) per update |
| Approximate membership or counts | Bloom filter, Count-Min, HyperLogLog (section 1) | O(k) |

### 7.1 LRU cache

**Problem:** Design a cache with a fixed capacity of N entries that supports get(key) and put(key, value), both in O(1) time. When the cache is full, put evicts the least recently used entry. An entry counts as used whenever it is read or written.

<details><summary>Hint</summary>

You need fast lookup by key and fast "move to most recent" and "remove oldest". No single structure does both.

</details>

<details><summary>Answer</summary>

A hash map from key to a node in a doubly linked list ordered by recency. get looks up the node and moves it to the head. put updates or inserts at the head; if over capacity, it removes the tail node and its map entry. Everything is O(1). In Python, `OrderedDict` with `move_to_end` and `popitem(last=False)` is exactly this.

**Follow-up:** "LFU instead?" Keep key → node plus frequency → doubly linked list, and track the minimum frequency; still O(1). "Many threads?" Shard the cache with a lock per shard, or use approximate LRU (the CLOCK algorithm, or Redis's sampling of a few keys).

</details>

### 7.2 Running median

**Problem:** Numbers arrive one at a time from a stream. After each arrival, report the median of all numbers seen so far, with each insert faster than O(n). Follow-up: report the median of only the last k numbers.

<details><summary>Hint</summary>

You only ever need the middle. Keep the lower half and the upper half separately, with quick access to the edge of each.

</details>

<details><summary>Answer</summary>

Two heaps: a max-heap for the lower half and a min-heap for the upper half, with sizes differing by at most 1. Insert into one, then rebalance by moving a top element across. The median is the top of the larger heap, or the average of both tops. Inserts are O(log n) and the median is O(1).

**Follow-up:** The sliding-window version needs deletions. Use two heaps with lazy deletion (a hash map of values waiting to be removed), or an order-statistic structure such as a sorted list or indexable skip list, for O(log k) per step.

</details>

### 7.3 Autocomplete

**Problem:** As a user types into a search box, show the 5 most popular past queries that start with the typed prefix. There are 10 million distinct queries with popularity counts, and suggestions must return within a few milliseconds per keystroke.

<details><summary>Hint</summary>

All queries sharing a prefix should live under one place you can reach in time proportional to the prefix length.

</details>

<details><summary>Answer</summary>

A trie keyed by characters, where each node caches the top 5 queries in its subtree (precomputed). A lookup walks the prefix in O(p) and returns the cached list. To save memory, compress chains of single-child nodes (a radix tree). Popularity updates are batched and the trie rebuilt periodically, since suggestions rarely need second-by-second freshness.

**Follow-up:** "Simpler alternative?" A sorted array of queries: binary-search the prefix to find its range, then take the top 5 from that range with a precomputed index.

</details>

### 7.4 Hits in the last 5 minutes

**Problem:** Count hits on an endpoint and answer "how many hits in the last 5 minutes?" at any moment. Hits arrive in timestamp order, up to thousands per second. Memory must stay bounded however much traffic arrives.

<details><summary>Hint</summary>

You don't need each hit, only how many fell in each second.

</details>

<details><summary>Answer</summary>

A circular buffer of 300 slots, one per second, each holding (timestamp, count). On a hit at second t, go to slot t mod 300; if its timestamp isn't t, reset it to (t, 0); then increment. A query sums the slots whose timestamps fall within the last 300 seconds. Memory is fixed at 300 slots. An exact deque of every hit's timestamp also works, but its memory grows with traffic.

**Follow-up:** "Across many servers?" Each server keeps its own buckets and you sum them, or use per-second counters in Redis with a TTL.

</details>

### 7.5 Sliding window maximum

**Problem:** Given an array of n numbers and a window size k, output the maximum of every contiguous window of k elements as the window slides from left to right. Aim for O(n) total time.

<details><summary>Hint</summary>

Once a larger number enters the window, every smaller number before it can never be the maximum again.

</details>

<details><summary>Answer</summary>

A monotonic deque of indices whose values decrease from front to back. For each new element, pop from the back while the back value is ≤ the new one, push the new index, and pop the front if it has left the window. The front is the window's maximum. Each index enters and leaves once, so it's O(n); a heap would be O(n log k).

</details>

### 7.6 Timers and delayed jobs

**Problem:** Build a scheduler where callers register jobs to run at a future time ("send this reminder at 9:00"), can cancel them, and a worker repeatedly asks what is due now. Expect millions of pending jobs.

<details><summary>Hint</summary>

The worker always needs the earliest due time, fast.

</details>

<details><summary>Answer</summary>

A min-heap keyed by due time: O(log n) insert and pop, O(1) peek. Cancel by marking the job cancelled in a hash map and skipping it when it's popped (lazy deletion). For millions of timers with coarse resolution, such as network timeouts, a hierarchical timing wheel gives O(1) insert and cancel; Kafka, Netty and the Linux kernel use one. For durability, use a database table indexed on due_at, or a Redis sorted set scored by timestamp.

</details>

### 7.7 Insert, delete and getRandom in O(1)

**Problem:** Design a set of integers that supports insert(x), remove(x) and getRandom(), which returns a uniformly random current element. Each operation must take O(1) on average.

<details><summary>Hint</summary>

A random pick is easy from an array; lookup by value is easy from a hash map. How do you delete from the middle of an array in O(1)?

</details>

<details><summary>Answer</summary>

An array of values plus a hash map from value to array index. insert appends and records the index. remove looks up the index, moves the last element into that slot, updates its index, and pops the end. getRandom picks a random index.

**Follow-up:** "Allow duplicates?" Map each value to a set of indices.

</details>

### 7.8 Are two users connected?

**Problem:** A social network processes a stream of "A and B became friends" events, with no unfriending. At any moment it must answer whether users X and Y are in the same friend group, meaning connected by any chain of friendships. There are 100 million users and billions of events. Follow-up: how many hops apart are X and Y?

<details><summary>Hint</summary>

You never need the path itself, only whether two users share a group, and groups only ever merge.

</details>

<details><summary>Answer</summary>

Union-find (disjoint set union) with path compression and union by rank or size. Each union and find is near O(1) amortized, and memory is one parent pointer and one rank per user.

**Follow-up:** Hop distance needs the actual graph: use bidirectional BFS from both users, which meets in the middle and explores far fewer nodes than BFS from one side. If unfriending is allowed, union-find no longer works because it can't split groups.

</details>

### 7.9 Game leaderboard

**Problem:** A game with 50 million players needs three operations: update a player's score, show the top 100, and show any player's exact rank ("you're #1,234,567"). Each must be O(log n) or better.

<details><summary>Hint</summary>

A heap gives you the top easily but can't tell you anyone's rank. What keeps everything sorted and can count how many items sit above a value?

</details>

<details><summary>Answer</summary>

An order-statistic sorted structure, such as a skip list or a balanced BST with subtree counts, plus a hash map from player to score. An update removes the old (score, player) entry and inserts the new one in O(log n). The top 100 is a walk from the top, and a rank is a count of entries above, in O(log n). Redis sorted sets (`ZADD`, `ZREVRANGE`, `ZREVRANK`) are exactly this: a skip list plus a hash table. Break ties by (score, time reached).

</details>

### 7.10 Range sums with updates

**Problem:** An array of n numbers receives a mix of update(i, value) and sum(l, r) operations in any order, where sum(l, r) is the total of elements l through r. Both must take O(log n). What changes if there are no updates?

<details><summary>Hint</summary>

Prefix sums make queries O(1), but one update then forces O(n) fixes. Store partial sums over power-of-two ranges instead.

</details>

<details><summary>Answer</summary>

A Fenwick tree (binary indexed tree) gives O(log n) updates and prefix sums; a range sum is prefix(r) − prefix(l − 1). A segment tree is also O(log n) and additionally handles min, max and range updates (with lazy propagation). Without updates, a plain prefix-sum array answers each query in O(1).

</details>

### 7.11 Days until a warmer day

**Problem:** Given a list of daily temperatures, output for each day how many days you'd wait for a warmer one (0 if never). n can be 10⁶, so O(n²) is too slow.

<details><summary>Hint</summary>

Process days left to right and keep the days still waiting for an answer. What order are their temperatures in?

</details>

<details><summary>Answer</summary>

A monotonic stack of indices whose temperatures decrease from bottom to top. For each new day, pop every waiting day that is colder and record the gap, then push today. Each index is pushed and popped once, so it's O(n). The same pattern solves stock span, largest rectangle in a histogram, and trapping rain water.

</details>

### 7.12 Meeting rooms and bookings

**Problem:** (a) Given a list of meetings with start and end times, what is the minimum number of rooms so that no two meetings in the same room overlap? (b) A calendar accepts booking requests one at a time and must reject any request that overlaps an existing booking, checking each in O(log n).

<details><summary>Hint</summary>

(a) Sort by start time; when a meeting begins, which room frees up earliest? (b) A new booking can only clash with its immediate neighbours in time order.

</details>

<details><summary>Answer</summary>

(a) Sort meetings by start and keep a min-heap of end times for rooms in use. For each meeting, if the earliest end is ≤ its start, reuse that room (pop); then push its end. The heap's largest size is the answer, in O(n log n). (b) A sorted map keyed by start time (Java `TreeMap`, Python `sortedcontainers`): check the predecessor (floor) and successor (ceiling) for overlap, in O(log n). For general "what overlaps this interval?" queries, use an interval tree.

</details>

### 7.13 Time-travel key-value store

**Problem:** Design a key-value store where set(key, value, timestamp) records a value at a time, and get(key, timestamp) returns the value as of that time: the latest value set at or before the given timestamp. For each key, timestamps arrive in increasing order.

<details><summary>Hint</summary>

For each key, the history is already sorted by time.

</details>

<details><summary>Answer</summary>

A hash map from key to a list of (timestamp, value), appended in order. get binary-searches for the last timestamp ≤ the query, in O(log h) for h versions. The same idea underlies MVCC version chains in databases.

</details>

### 7.14 Build order

**Problem:** You have n packages and a list of dependencies of the form "A must be built before B". Output a valid build order, or report that none exists. Follow-up: which packages can be built in parallel?

<details><summary>Hint</summary>

A package is ready once everything it depends on is built. Track how many unbuilt dependencies each package still has.

</details>

<details><summary>Answer</summary>

Treat it as a directed graph and topologically sort it with Kahn's algorithm. Compute in-degrees, queue every package with in-degree 0, then repeatedly pop one, output it, and decrement its dependents, queuing any that reach 0. If fewer than n packages come out, there's a cycle. It runs in O(V + E).

**Follow-up:** Each wave of packages that reach in-degree 0 together can build in parallel; the number of waves is the critical-path length.

</details>

### 7.15 First unique item in a stream

**Problem:** Items (characters or user IDs) arrive one at a time. After each arrival, report the earliest-arrived item that has so far appeared exactly once, or none. Each step must be O(1).

<details><summary>Hint</summary>

You need a count per item, plus the arrival order of items that are still unique.

</details>

<details><summary>Answer</summary>

A hash map of counts plus a queue of candidates in arrival order. On arrival, increment the count, and enqueue the item if its count is now 1. Then pop from the front while the front's count is above 1. The front is the answer, in amortized O(1). With a doubly linked list and a map to its nodes, repeats can be removed immediately, as in the LRU cache.

</details>

### 7.16 Nearby drivers

**Problem:** A ride-hailing app tracks 1 million drivers, each sending a location every 4 seconds. When a rider requests a car, return the 10 nearest available drivers within 2 km in under 50 ms. What structure indexes the locations?

<details><summary>Hint</summary>

Scanning every driver is too slow. Divide the map into cells, so a search only looks at the rider's cell and its neighbours.

</details>

<details><summary>Answer</summary>

A grid-based spatial index: geohash, or cell systems such as Uber's H3 hexagons or Google's S2. Map each driver to a cell ID and keep a hash map from cell to the drivers in it. A query checks the rider's cell and its neighbours, computes exact distances, and keeps the 10 nearest with a size-10 heap. Moving a driver between cells is O(1), which matters at about 250,000 location updates per second. A quadtree adapts better to uneven density but costs more to rebalance under constant updates. Redis offers this as `GEOADD` and `GEOSEARCH`, built on geohash and a sorted set.

</details>

---

## Practice log

Log every out-loud attempt. Anything marked Shaky or Missed gets a redo date within 3 days. Result: Nailed it / Shaky / Missed.

| Date | Teaser | Time (min) | Result | Beat missed | Redo on |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
