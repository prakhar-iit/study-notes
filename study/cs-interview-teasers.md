# CS Interview Teasers — Working Doc

Oct 28, 2026 · Prakhar

## How to use this doc

These are the teasers that come up again and again in backend, systems and AI-platform interviews. Each one gives the question, the insight that cracks it, the answer, and the follow-up interviewers usually ask next.

- **Practise out loud.** Aim for under 2 minutes per teaser, using the five beats: concept, mechanism, number, failure mode, trade-off.
- **Ask before you solve.** Most teasers hinge on a clarifying question (the alphabet, the memory limit, whether the answer may be approximate). Asking it is part of what's being scored.
- **Size it first.** Do the arithmetic out loud before proposing a design. Numbers you'll reuse are in the estimation section.
- **Keep it growing.** Add new teasers under the matching section as you meet them, and log each attempt in the practice log at the end.

## 1. Big data, small memory

Almost every one of these is solved by one of three moves: shrink the key space into a bitmap, hash-partition to disk so that equal items land together, or accept an approximate answer from a sketch.

### 1.1 Duplicates among 1 billion 5-character strings, 1 GB RAM

**Q:** Find the duplicate strings.

**Insight:** The data is 5 GB, five times the RAM. The alphabet decides everything, because with only 5 characters the number of *possible* strings may be small.

**Answer:** Encode each string as a base-|alphabet| integer and index a bitmap.

- **Alphanumeric:** 62⁵ ≈ 916 M possible strings, so one bitmap is 115 MB. Keep two (`seen`, `dup`), about 230 MB in one sequential pass.
- **Lowercase only:** 26⁵ ≈ 11.9 M possible strings. Duplicates are guaranteed (pigeonhole).
- **Arbitrary bytes:** 256⁵ ≈ 1.1 T possible strings, a 137 GB bitmap. Instead hash-partition into 64 files, so equal strings share a file, then count each file in a hash map of about 250 MB.

**Follow-up:** "What if one bucket is huge?" A count map's memory depends on *distinct* keys, so many copies of one string are harmless. Too many distinct keys means re-splitting that bucket with a new hash seed.

### 1.2 Missing integer among 4 billion 32-bit integers

**Q:** A file holds 4 billion non-negative 32-bit integers. Find one that isn't there, with 1 GB of RAM, then with 10 MB.

**Insight:** There are 2³² ≈ 4.29 billion possible values and only 4 billion in the file, so at least one is missing.

**Answer (1 GB):** A bitmap of 2³² bits is 512 MB. Set a bit for each value, then scan for a zero.

**Answer (10 MB):** Use two passes.

1. Split the range into 2¹⁶ blocks of 2¹⁶ values each, and count how many file values fall in each block. That's 65,536 counters at 4 bytes each, 256 KB.
2. Any block with fewer than 65,536 values has a gap. Re-read the file, bitmapping only that block (8 KB).

**Follow-up:** "What if there are duplicates?" It still works. The block counts sum to 4 billion, which is less than 2³², so some block must have fewer than 2¹⁶ values.

### 1.3 Top-K most frequent queries in a 100 GB log

**Q:** Find the 10 most frequent search queries without loading the log into memory.

**Answer (exact):** Hash-partition by query into N files, so each query lives in exactly one file. Count each file in a hash map, and keep a **min-heap of size K** across all files. That's O(n log K) time.

**Answer (streaming, approximate):** A Count-Min Sketch plus a size-K heap, or the Space-Saving or Misra–Gries algorithms with a fixed number of counters.

**Follow-up:** "Why a min-heap?" The root is the smallest of your current top K, so each new item is compared against it and evicts it in O(log K). "Top K in the last hour?" Keep per-minute count buckets and sum the last 60.

### 1.4 Sort 100 GB with 1 GB of RAM

**Answer:** External merge sort.

1. Read 1 GB at a time, sort it in memory, and write it out as a sorted run. That gives 100 runs.
2. Do one 100-way merge with a min-heap holding the head of each run, giving each run about a 10 MB read buffer.

Total I/O is about 400 GB: read and write twice.

**Follow-up:** "Too many runs for one merge?" Merge in multiple passes. Replacement selection produces runs about twice the memory size on random input, which halves the run count.

### 1.5 Count distinct visitors in a stream of billions

**Insight:** An exact answer needs memory proportional to the distinct count. 1 billion IDs in a hash set is roughly 16 GB.

**Answer:** HyperLogLog. Hash each item, use the first p bits to pick a register, and record the longest run of leading zeros in the remaining bits. Standard error is about 1.04/√m for m registers. Redis's implementation uses 12 KB for about 0.81% error.

**Follow-up:** HLLs merge by taking the per-register max, so daily sketches combine into weekly uniques. Intersections are inaccurate, because subtracting two estimates compounds their errors.

### 1.6 Random sample of k items from a stream of unknown length

**Answer:** Reservoir sampling. Keep the first k items. For item i (counting from 1, i > k), pick a random j from 1 to i, and if j ≤ k, replace slot j. Every item ends up in the sample with probability k/n.

**Why (k = 1):** Item i is kept with probability 1/i and then survives each later item j with probability (j − 1)/j. The product telescopes to 1/n.

**Follow-up:** "Weighted sampling?" Give each item a key u^(1/w), with u uniform in (0, 1), and keep the k largest keys. "Across several machines?" The same keyed approach merges: take the top k keys overall.

### 1.7 Have we seen this URL before? (Bloom filter)

**Answer:** A bit array of m bits and k hash functions. Set k bits on insert, and check all k on lookup. False positives are possible; false negatives are not.

```math
\frac{m}{n} = \frac{-\ln p}{(\ln 2)^2}, \qquad k = \frac{m}{n}\ln 2
```

A 1% false-positive rate needs about 9.6 bits per item and 7 hashes, so 1 billion URLs need about 1.2 GB. A 0.1% rate needs about 14.4 bits per item.

**Follow-up:** "Deletions?" Use a counting Bloom filter or a cuckoo filter. "Where is it used?" LSM-tree stores such as RocksDB and Cassandra use one per SSTable to skip disk reads for keys that aren't there.

## 2. Counting and probability

These reward a clean setup more than clever math: define the outcomes, count them, and check against a tiny case.

### 2.1 Why nC2 = n(n − 1)/2

**Answer:** There are n choices for the first item and n − 1 for the second, giving n(n − 1) *ordered* pairs. Each unordered pair {A, B} was counted twice, as (A, B) and (B, A), so divide by 2.

```math
\binom{n}{2} = \frac{n(n-1)}{2}, \qquad \binom{n}{k} = \frac{n!}{k!\,(n-k)!}
```

**Second view:** Handshakes. Person i shakes hands with the i − 1 people before them, so the total is 0 + 1 + … + (n − 1).

**Why it matters:** Comparing all pairs of 1 billion items is about 5 × 10¹⁷ comparisons, or roughly 16 years at 10⁹ per second. That number is why the big-data teasers above avoid pairwise comparison.

### 2.2 Birthday paradox and hash collisions

**Q:** How many people before two share a birthday with probability above 50%?

**Answer:** 23. The chance that nobody matches is the product of (1 − i/365) for i from 0 to n − 1, which drops below 0.5 at n = 23.

**Engineering version:** Collisions become likely at about √N items, not N. A 50% collision chance arrives after roughly 1.18 × √N items.

- **32-bit hash:** about 77,000 items.
- **64-bit hash:** about 5 billion items.
- **UUID v4 (122 random bits):** about 2.7 × 10¹⁸ items, which is why random UUIDs are safe as IDs.

### 2.3 Expected number of colliding pairs

**Answer:** Put n items into m buckets uniformly at random. Each of the nC2 pairs collides with probability 1/m, so the expected number of colliding pairs is nC2 / m.

**Example:** 10,000 keys with a 32-bit hash give about 5 × 10⁷ / 4.3 × 10⁹ ≈ 0.012 expected collisions.

### 2.4 Monty Hall

**Q:** You pick one of three doors. The host, who knows where the car is, opens a different door with a goat and offers a switch. Should you switch?

**Answer:** Yes. Switching wins 2/3 of the time. Your first pick is right only 1/3 of the time, and switching wins in every case where it was wrong.

**Follow-up:** With 100 doors and 98 opened, switching wins 99/100. That version makes the intuition obvious.

### 2.5 A fair coin from a biased coin

**Answer:** Von Neumann's trick. Flip twice: HT means heads, TH means tails, and HH or TT means start over. HT and TH both have probability p(1 − p), so they're equally likely.

**Follow-up:** Expected flips are 1 / (p(1 − p)), so a very biased coin is slow.

### 2.6 rand7() from rand5()

**Answer:** Rejection sampling. Compute x = 5 × (rand5() − 1) + rand5(), which is uniform on 1 to 25. If x ≤ 21, return ((x − 1) mod 7) + 1; otherwise try again.

**Follow-up:** "Why not (rand5() + rand5()) mod 7?" A sum of two dice isn't uniform; middle values are more likely. Expected rand5() calls are 2 × 25/21 ≈ 2.38.

### 2.7 Coupon collector

**Q:** Requests go to 100 servers uniformly at random. How many requests until every server has received at least one?

**Answer:** n × Hₙ ≈ n ln n + 0.577n. For n = 100 that's about 519.

### 2.8 Expected flips to see two heads in a row

**Answer:** 6. Let E be the expected flips from the start and E_H the expected flips after one head.

- E = 1 + ½ × E_H + ½ × E
- E_H = 1 + ½ × 0 + ½ × E

Solving gives E = 6.

**Follow-up:** "Heads then tails?" Only 4. A failed HH attempt sends you back to the start, while a failed HT attempt (a second H) leaves you one step in.

### 2.9 Shuffle an array uniformly

**Answer:** Fisher–Yates. For i from n − 1 down to 1, swap a[i] with a[j], where j is random in 0 to i. All n! orderings are equally likely.

**Follow-up:** "Why not swap each element with a random index from 0 to n − 1?" That has nⁿ equally likely paths, which isn't divisible by n! for n ≥ 3 (27 paths onto 6 orderings for n = 3), so some orderings must be more likely.

## 3. Classic logic puzzles

The interviewer is grading how you reason: state the bound ("each weighing has 3 outcomes") before the construction, and test a small case aloud.

### 3.1 Two eggs, 100 floors

**Q:** Find the highest safe floor with the fewest drops in the worst case.

**Answer:** 14 drops. If the first egg breaks, the second must scan linearly below it, so each later jump must be one floor shorter. Drop the first egg at 14, 27, 39, 50, 60, 69, 77, 84, 90, 95, 99, 100. You need n(n + 1)/2 ≥ 100, and 14 × 15/2 = 105.

**Follow-up:** "k eggs?" Use DP on moves and eggs: f(m, k) = f(m − 1, k − 1) + f(m − 1, k) + 1 floors are coverable with m moves and k eggs. Find the smallest m with f(m, k) ≥ floors.

### 3.2 25 horses, 5 tracks, no timer

**Q:** Find the 3 fastest horses with the fewest races.

**Answer:** 7 races.

1. Races 1–5: race each group of 5.
2. Race 6: race the 5 group winners. The winner is fastest overall. Call the groups of the 1st, 2nd and 3rd finishers A, B and C.
3. Race 7: the only remaining candidates for places 2 and 3 are A2, A3, B1, B2 and C1. Race them; the top two finishers are 2nd and 3rd overall.

### 3.3 1,000 bottles, one poisoned, one day

**Q:** Poison takes 24 hours to act. How many testers do you need to find the bottle in one day?

**Answer:** 10, since 2¹⁰ = 1,024. Number the bottles in binary; tester i drinks from every bottle whose bit i is 1. The set of testers who die spells the bottle's number.

**Follow-up:** "Two days?" Each tester then has three outcomes (dies on day 1, dies on day 2, survives), so 3⁷ = 2,187 means 7 testers.

### 3.4 8 balls, one heavier, a balance scale

**Answer:** 2 weighings. Weigh 3 against 3. If one side is heavier, weigh 1 against 1 from that side (balanced means it's the third). If balanced, weigh the remaining 2.

**Insight:** Each weighing has 3 outcomes, so k weighings separate up to 3ᵏ candidates. That's 9 balls in 2 weighings.

### 3.5 12 coins, one fake, heavier or lighter

**Answer:** 3 weighings. There are 24 possibilities (12 coins × heavier or lighter), and 3³ = 27 outcomes is enough.

**Opening:** Weigh 4 against 4. If balanced, the fake is among the other 4: weigh 3 of them against 3 known-good coins. If unbalanced, the fake is among the 8, and you know which side each could be heavy or light on. Rotate coins between the pans so that every outcome narrows to 3 or fewer cases.

### 3.6 Two burning ropes, 45 minutes

**Q:** Each rope burns in 60 minutes, but unevenly. Measure 45 minutes.

**Answer:** Light rope A at both ends and rope B at one end. A finishes at 30 minutes; light B's other end then, and it finishes 15 minutes later.

### 3.7 Bridge and torch

**Q:** Four people take 1, 2, 5 and 10 minutes to cross. At most two cross at once, at the slower pace, and the torch must go with them. What's the fastest crossing?

**Answer:** 17 minutes.

1. 1 and 2 cross (2).
2. 1 returns (1).
3. 5 and 10 cross (10).
4. 2 returns (2).
5. 1 and 2 cross (2).

**Insight:** Send the two slowest together, and let the two fastest shuttle the torch. Having 1 escort everyone takes 19.

### 3.8 100 doors

**Q:** On pass k you toggle every k-th door, for k from 1 to 100. Which doors end open?

**Answer:** The 10 perfect squares (1, 4, 9 … 100). A door is toggled once per divisor, and only perfect squares have an odd number of divisors.

## 4. Bit and array tricks

These usually come with an "O(1) extra space" constraint. The tools are XOR (x ^ x = 0, x ^ 0 = x), arithmetic identities, and treating an array as a linked structure.

### 4.1 Missing number from 1 to n

**Answer:** The expected sum is n(n + 1)/2; subtract the actual sum. To avoid overflow, XOR all the indices 1 to n with all the values instead; everything cancels except the missing number.

### 4.2 The one number that appears once (all others twice)

**Answer:** XOR every element. Pairs cancel, leaving the single number. O(n) time, O(1) space.

**Follow-up:** "Others appear three times?" Count each bit position across all numbers mod 3; the leftover bits form the answer.

### 4.3 Two numbers that appear once

**Answer:** XOR everything to get a ^ b. Take its lowest set bit with `x & -x`; a and b differ there. Split all numbers by that bit and XOR each group separately to get a and b.

### 4.4 Find the duplicate: n + 1 values from 1 to n, O(1) space, read-only

**Insight:** Treat the array as a function i → a[i]. The duplicate value is a node with two incoming edges, which is the entrance to a cycle.

**Answer:** Floyd's cycle detection. Run slow and fast pointers until they meet, reset one pointer to the start, then move both one step at a time. They meet at the duplicate.

**Follow-up:** Know the alternatives and what each breaks: sorting modifies the array, a hash set uses O(n) space, sign-marking modifies the array, and binary search on value counts is O(n log n).

### 4.5 Quick bit identities

- **Power of two:** `n > 0 and n & (n - 1) == 0`.
- **Count set bits:** repeat `n &= n - 1`, which clears the lowest set bit, once per set bit (Kernighan).
- **Lowest set bit:** `x & -x`.
- **Swap without a temp:** `a ^= b; b ^= a; a ^= b`. It zeroes the value if a and b are the same memory location, and in practice a temp variable is just as fast.

### 4.6 Majority element (more than n/2)

**Answer:** Boyer–Moore voting. Keep a candidate and a count; a matching element increments, a different one decrements, and at zero you adopt the next element. O(n) time, O(1) space.

**Follow-up:** If a majority isn't guaranteed, verify the candidate with a second pass. For "more than n/3", keep two candidates. This is the Misra–Gries idea from teaser 1.3.

### 4.7 Cycle in a linked list, and where it starts

**Answer:** Floyd's tortoise and hare: the slow pointer moves 1 step, the fast moves 2, and they meet only if there's a cycle. To find the start, reset one pointer to the head and step both by 1; they meet at the cycle's entrance.

**Why:** Let a be the distance from the head to the entrance, b the distance from the entrance to the meeting point, and c the cycle length. The fast pointer travelled twice as far, so 2(a + b) = a + b + kc, which gives a = kc − b. Walking a steps from the head and from the meeting point ends at the same node.

## 5. Distributed systems and backend teasers

These are "why did this happen?" puzzles. Name the failure mode first, then the mechanism that fixes it.

### 5.1 The API said "failed", the agent retried, and now there are duplicates

**Insight:** The first write probably succeeded and only the response was lost, through a timeout, a reset connection, or an error after commit. From the client, "never arrived" and "succeeded but the reply was lost" look identical.

**Answer:** Use a stable idempotency key (run ID + step, or a business key, never a new UUID per attempt or the LLM's tool call ID). Store it under a unique constraint in the same transaction as the write, and replay the stored response on repeats. Add a unique constraint on the natural key as a backstop, and have only one layer own retries.

### 5.2 Can we get exactly-once delivery?

**Answer:** Not as a network guarantee. What you build is at-least-once delivery plus an idempotent consumer, which gives *effectively-once* processing.

**Follow-up:** "Kafka has exactly-once?" Kafka's idempotent producer plus transactions gives exactly-once for read–process–write *within Kafka*. Side effects outside Kafka, such as an email or a payment, still need idempotency.

### 5.3 Two Generals

**Answer:** No finite protocol guarantees both sides agree over a channel that can lose messages, because every acknowledgement itself needs an acknowledgement. The practical consequence is designing for uncertainty: timeouts, retries and idempotency.

### 5.4 Ordering events by wall-clock timestamp

**Insight:** Clocks drift and skew between machines, and NTP corrections can move them backward, so "later timestamp" doesn't mean "happened later."

**Answer:** Use a single sequence (a DB sequence or a log offset), Lamport clocks for an order consistent with causality, or vector clocks to detect concurrent writes. Hybrid logical clocks combine physical and logical time. Google Spanner's TrueTime instead bounds clock uncertainty and waits it out before commit.

### 5.5 Design a rate limiter

**Answer:** A token bucket: capacity b, refilled at r tokens per second, where each request takes one token. It allows bursts up to b while holding the long-run average rate to r. Reject with 429 and a `Retry-After` header.

**Distributed:** Keep the counter in Redis and make check-and-decrement atomic with a Lua script (or `INCR` plus an expiry per window).

**Follow-up:** "Why not a fixed window?" A client can spend its full limit just before a window boundary and again just after it, getting 2× the rate in a short span. Sliding windows or token buckets avoid that.

### 5.6 Cache stampede (thundering herd)

**Q:** A hot key expires, and thousands of requests hit the database at once.

**Answer:** Let only one request rebuild the value (a per-key lock or single-flight), serve the stale value while it refreshes (stale-while-revalidate), refresh probabilistically before expiry, and add jitter to TTLs so that keys don't expire together.

### 5.7 Why consistent hashing?

**Insight:** With `hash(key) mod N`, going from N to N + 1 servers remaps about N/(N + 1) of all keys, nearly everything.

**Answer:** Put servers and keys on a hash ring; each key goes to the next server clockwise. Adding or removing a server moves only about 1/N of the keys. Use 100 to 200 virtual nodes per server to even out the load.

### 5.8 "I updated my profile, refreshed, and saw the old value"

**Insight:** The write went to the primary, and the read hit a replica that hadn't caught up (replication lag).

**Answer:** For read-your-writes, route that user's reads to the primary for a short window after a write, use sticky sessions, or return the write's log position (LSN) and have the read wait until a replica has reached it.

### 5.9 Two buyers, one item left (lost update)

**Q:** Both requests read stock = 1, both decrement, and you oversell.

**Answer:** Make the check and the write one atomic step: `UPDATE items SET stock = stock - 1 WHERE id = ? AND stock > 0`, then check the number of rows affected. Alternatives are `SELECT … FOR UPDATE` (pessimistic) or a version column with compare-and-set (optimistic).

### 5.10 Why fan-out makes tail latency worse

**Answer:** If 1% of calls to one backend are slow, a request that fans out to 100 backends waits on a slow call with probability 1 − 0.99¹⁰⁰ ≈ 63%. Your p99 becomes everyone's median.

**Fix:** Hedged requests (send a duplicate after the p95 time and take the first reply), tighter per-call timeouts, and fewer fan-out hops.

## 6. Estimation: numbers to know

Interviewers care about the right order of magnitude, said out loud. These figures are approximate and from memory, rounded for easy mental math; real hardware varies.

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

**Q:** How much GPU memory does one token of context cost?

**Answer:** 2 (K and V) × layers × KV heads × head dimension × bytes per value. For Llama 3 8B in fp16: 2 × 32 × 8 × 128 × 2 = 131,072 bytes, which is 128 KiB per token. An 8K-token sequence needs 1 GiB.

**Follow-up:** "How many 8K sequences fit on an 80 GB GPU?" Subtract about 16 GB of weights, leaving about 64 GB, so roughly 64 sequences. That's why grouped-query attention, KV-cache quantization and paged attention matter.

### 6.2 Decode speed ceiling

**Q:** What's the most tokens per second one sequence can get from an 8B model on one GPU?

**Answer:** At batch size 1, decode is memory-bound: every token reads all the weights. 16 GB of fp16 weights at an H100's roughly 3.35 TB/s of HBM bandwidth gives an upper bound of about 200 tokens per second. Batching amortizes that read across many sequences, which is why throughput scales with batch size.

### 6.3 Storage for 1 billion records

**Answer:** At 100 bytes each, that's 100 GB raw. Add indexes (often 0.5–1× more) and 3× replication, and plan for about 450–600 GB.

### 6.4 Traffic for 100 million daily users

**Answer:** At 10 requests per user per day, that's 1 billion requests a day, about 12,000 QPS on average and 25,000–35,000 at peak.

## Practice log

Log every out-loud attempt. Anything marked Shaky or Missed gets a redo date within 3 days. Result: Nailed it / Shaky / Missed.

| Date | Teaser | Time (min) | Result | Beat missed | Redo on |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
|  |  |  |  |  |  |
