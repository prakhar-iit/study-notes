# Python Advanced Cheatsheet — Concurrency, Internals & Interview Depth

Companion to `python_cheatsheet.md` (data structures & algorithm templates). This one covers the topics that show up in "senior" or systems-flavored interview rounds: the GIL, threading, multiprocessing, asyncio, decorators, generators, context managers, and the object model. Every runnable example in this file has been executed, not just syntax-checked.

---

## Table of Contents

1. [The GIL — What It Is and Why It Matters](#1-the-gil--what-it-is-and-why-it-matters)
2. [Threading vs Multiprocessing vs Asyncio](#2-threading-vs-multiprocessing-vs-asyncio)
3. [threading Module Essentials](#3-threading-module-essentials)
4. [Synchronization Primitives](#4-synchronization-primitives)
5. [Producer-Consumer with queue.Queue](#5-producer-consumer-with-queuequeue)
6. [Race Conditions & Deadlocks](#6-race-conditions--deadlocks)
7. [multiprocessing Essentials](#7-multiprocessing-essentials)
8. [Sharing State Across Processes](#8-sharing-state-across-processes)
9. [concurrent.futures — Executors](#9-concurrentfutures--executors)
10. [asyncio Fundamentals](#10-asyncio-fundamentals)
11. [asyncio Synchronization & Patterns](#11-asyncio-synchronization--patterns)
12. [Running Blocking Code from Async](#12-running-blocking-code-from-async)
13. [Decorators — Deep Dive](#13-decorators--deep-dive)
14. [Generators & Iterators — Protocol Deep Dive](#14-generators--iterators--protocol-deep-dive)
15. [Context Managers — Deep Dive](#15-context-managers--deep-dive)
16. [Closures & Scope](#16-closures--scope)
17. [*args, **kwargs & Argument Unpacking](#17-args-kwargs--argument-unpacking)
18. [Python Data Model / Dunder Methods](#18-python-data-model--dunder-methods)
19. [Memory Management](#19-memory-management)
20. [property, staticmethod, classmethod & functools](#20-property-staticmethod-classmethod--functools)
21. [Abstract Base Classes & Protocols](#21-abstract-base-classes--protocols)
22. [Mutability, Identity & Thread-Safety of Built-ins](#22-mutability-identity--thread-safety-of-built-ins)
23. [Classic Concurrency Interview Questions](#23-classic-concurrency-interview-questions)
24. [Gotchas & "Explain This Output"](#24-gotchas--explain-this-output)
25. [Decision Cheat Table](#25-decision-cheat-table)

---

## 1. The GIL — What It Is and Why It Matters

The **Global Interpreter Lock** is a single mutex in CPython that allows only **one thread to execute Python bytecode at a time**, even on a multi-core machine. It exists because CPython's memory management (reference counting) isn't thread-safe by default — the GIL is the simplest fix.

Consequences:

- **CPU-bound work does NOT speed up with `threading`** — threads take turns holding the GIL, so N threads doing pure computation run at roughly the same total speed as 1 thread (often slightly *slower*, from lock-switching overhead).
- **I/O-bound work DOES speed up with `threading`** — the GIL is released during blocking I/O (network, disk, `time.sleep`), so other threads can run while one thread waits.
- **`multiprocessing` sidesteps the GIL entirely** — each process has its own interpreter and its own GIL, giving true parallelism for CPU-bound work, at the cost of process-startup overhead and needing explicit data sharing (processes don't share memory by default).
- **`asyncio` sidesteps the GIL differently** — single thread, single GIL holder, but cooperative multitasking: a coroutine explicitly yields control at each `await`, so many I/O-bound tasks interleave without needing multiple threads at all.

```python
# Proof: CPU-bound work — threading gives (almost) no speedup, multiprocessing does.
import time, threading, multiprocessing as mp

def cpu_bound(n):
    x = 0
    for i in range(n):
        x += i * i
    return x

N = 5_000_000

def run_sequential():
    t0 = time.perf_counter()
    cpu_bound(N); cpu_bound(N)
    return time.perf_counter() - t0

def run_threaded():
    t0 = time.perf_counter()
    threads = [threading.Thread(target=cpu_bound, args=(N,)) for _ in range(2)]
    for t in threads: t.start()
    for t in threads: t.join()
    return time.perf_counter() - t0

def run_multiprocess():
    t0 = time.perf_counter()
    with mp.Pool(2) as pool:
        pool.map(cpu_bound, [N, N])
    return time.perf_counter() - t0

if __name__ == "__main__":
    seq = run_sequential()
    thr = run_threaded()
    mpr = run_multiprocess()
    print(f"sequential:     {seq:.2f}s")
    print(f"threaded (2x):  {thr:.2f}s  <- NOT ~2x faster, GIL serializes CPU work")
    print(f"multiproc (2x): {mpr:.2f}s  <- actually parallel, uses 2 cores")
```

Typical output (timings vary by machine, but the *pattern* is the interview point):

```
sequential:     0.85s
threaded (2x):  0.87s   <- no speedup (often even a hair slower)
multiproc (2x): 0.48s   <- real speedup from using a second core
```

## 2. Threading vs Multiprocessing vs Asyncio

| | Best for | Parallelism | Shares memory? | Overhead |
|---|---|---|---|---|
| **threading** | I/O-bound (network, disk, DB calls) | Concurrent, not parallel (GIL) | Yes — same process | Low (threads are cheap) |
| **multiprocessing** | CPU-bound (crunching numbers) | True parallel (separate GILs) | No — must explicitly share (`Value`, `Array`, `Manager`, pipes/queues) | High (process spawn, pickling data) |
| **asyncio** | High-volume I/O-bound (thousands of concurrent connections) | Concurrent, single-threaded (cooperative) | Yes — single thread | Very low, but **one blocking call freezes everything** |

Rule of thumb for interviews: **I/O-bound → threading or asyncio. CPU-bound → multiprocessing.** If you use `threading` for CPU-bound work, that's a red flag interviewers listen for.

## 3. threading Module Essentials

```python
import threading, time

def worker(name, delay):
    time.sleep(delay)          # simulated I/O — GIL is released here
    print(f"{name} done")

# Basic thread lifecycle
t = threading.Thread(target=worker, args=("t1", 0.05))
t.start()          # begin running
t.join()           # block caller until t finishes (join(timeout=...) to cap the wait)
print(t.is_alive())            # False — already finished

# Multiple threads running concurrently
threads = [threading.Thread(target=worker, args=(f"t{i}", 0.02)) for i in range(3)]
for t in threads: t.start()
for t in threads: t.join()     # wait for all — always join what you start

# Daemon threads — killed automatically when the main program exits
# (use for background work you don't need to finish cleanly, e.g. a heartbeat)
bg = threading.Thread(target=worker, args=("bg", 10), daemon=True)
bg.start()          # program can exit without waiting for this thread

# Subclassing Thread (alternative to passing target=)
class Worker(threading.Thread):
    def __init__(self, n):
        super().__init__()
        self.n = n
        self.result = None
    def run(self):                       # override run(), not start()
        self.result = self.n * self.n

w = Worker(7)
w.start(); w.join()
print(w.result)      # 49

# threading.local() — per-thread storage (avoids passing thread-id everywhere)
local_data = threading.local()
def set_and_read():
    local_data.value = threading.get_ident()
    return local_data.value
```

## 4. Synchronization Primitives

```python
import threading, time

# --- Lock: mutual exclusion, only one thread in the critical section at a time ---
counter = 0
lock = threading.Lock()

def increment_safe(n):
    global counter
    for _ in range(n):
        with lock:               # acquires on enter, releases on exit (even on exception)
            counter += 1

threads = [threading.Thread(target=increment_safe, args=(100_000,)) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
print("with lock:", counter)     # always 400000 — correct

# --- RLock: re-entrant lock — same thread can acquire it again without deadlocking itself ---
rlock = threading.RLock()
def outer():
    with rlock:
        inner()                  # a plain Lock here would deadlock the same thread
def inner():
    with rlock:
        pass
outer()

# --- Semaphore: allow up to N concurrent holders (e.g. rate-limit concurrent connections) ---
sem = threading.Semaphore(2)     # at most 2 threads in the section at once
def limited_task(i, log):
    with sem:
        log.append(f"start {i}")
        time.sleep(0.01)
        log.append(f"end {i}")

log = []
threads = [threading.Thread(target=limited_task, args=(i, log)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()

# --- Event: one-way signal from one/many threads to others ---
event = threading.Event()
def waiter(out):
    event.wait()                 # blocks until event.set() is called (optionally with timeout)
    out.append("proceeding")
out = []
t = threading.Thread(target=waiter, args=(out,))
t.start()
time.sleep(0.01)
event.set()                      # release all waiters
t.join()
print(out)                       # ['proceeding']

# --- Condition: wait/notify for producer-consumer style coordination ---
condition = threading.Condition()
items = []
def produce():
    with condition:
        items.append(1)
        condition.notify()       # wake one waiting thread (notify_all() for all)
def consume(out):
    with condition:
        condition.wait_for(lambda: len(items) > 0)   # releases lock while waiting
        out.append(items.pop())

out = []
c = threading.Thread(target=consume, args=(out,))
c.start()
time.sleep(0.01)
produce()
c.join()
print(out)                       # [1]

# --- Barrier: block N threads until all N have arrived, then release them together ---
barrier = threading.Barrier(3)
def phase(out, i):
    out.append(f"{i} before")
    barrier.wait()                # blocks until all 3 threads call wait()
    out.append(f"{i} after")
out = []
threads = [threading.Thread(target=phase, args=(out, i)) for i in range(3)]
for t in threads: t.start()
for t in threads: t.join()
```

## 5. Producer-Consumer with queue.Queue

`queue.Queue` is **thread-safe by design** — no manual locking needed. This is the standard, interview-expected way to hand work between threads.

```python
import threading, queue, time

q = queue.Queue(maxsize=10)      # bounded queue — put() blocks when full (backpressure)
SENTINEL = object()              # unique marker to signal "no more work"

def producer(n):
    for i in range(n):
        q.put(i)                 # blocks if the queue is full
    q.put(SENTINEL)              # tell the consumer we're done

def consumer(out):
    while True:
        item = q.get()           # blocks until an item is available
        if item is SENTINEL:
            q.task_done()
            break
        out.append(item * item)
        q.task_done()            # signal this item is fully processed

out = []
p = threading.Thread(target=producer, args=(20,))
c = threading.Thread(target=consumer, args=(out,))
p.start(); c.start()
p.join(); c.join()
print(len(out), out[:5])         # 20 [0, 1, 4, 9, 16]

# Multiple consumers: q.join() blocks until every put() item has been task_done()'d.
# Common pattern: put one SENTINEL per consumer, or use q.join() then set a stop Event.
```

## 6. Race Conditions & Deadlocks

```python
import threading

# --- Race condition: unsynchronized read-modify-write on shared state ---
# `counter += 1` is really three steps — read, add 1, write back — and a thread can be
# swapped out between any of them. On some CPython builds a plain tight loop like this
# rarely gets preempted mid-increment in practice (the eval-loop's GIL-release checks
# tend to land at loop/call boundaries), so `unsafe_counter += 1` alone may look
# deceptively safe if you only run it a few times. Widening the window with a zero-length
# sleep (which still releases the GIL) makes the interleaving reliably visible — the bug
# is real either way, this just makes it easy to observe:
import time
unsafe_counter = 0
def increment_unsafe(n):
    global unsafe_counter
    for _ in range(n):
        temp = unsafe_counter    # read
        time.sleep(0)            # yield the GIL mid-operation — widens the race window
        unsafe_counter = temp + 1  # write back, possibly clobbering another thread's update

threads = [threading.Thread(target=increment_unsafe, args=(200,)) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
print("without lock:", unsafe_counter)   # reliably << 800 — lost updates from interleaving
# Fix: wrap the increment in a Lock (see Section 4) or use a queue-based single-writer design.
# The same class of bug can happen with the plain `unsafe_counter += 1` above too — it's just
# less likely to surface on a short, fast loop. Never rely on "it passed my test run" for
# concurrency correctness; reason about atomicity, don't just observe it.

# --- Deadlock: two locks acquired in inconsistent order by two threads ---
lock_a, lock_b = threading.Lock(), threading.Lock()

def task_1():
    with lock_a:
        with lock_b:              # thread 1: A then B
            pass

def task_2():
    with lock_b:
        with lock_a:               # thread 2: B then A  -> can deadlock if interleaved badly
            pass
# If task_1 grabs lock_a and task_2 grabs lock_b at the same moment, each then blocks
# waiting for the lock the other already holds. Neither can proceed. Classic deadlock.

# Fix 1 — consistent lock ordering: every thread acquires locks in the SAME global order
def task_1_fixed():
    with lock_a:
        with lock_b:
            pass
def task_2_fixed():
    with lock_a:      # acquire in the same order as task_1: A then B
        with lock_b:
            pass

# Fix 2 — bounded waiting with Lock.acquire(timeout=...): back off and retry instead of blocking forever
def task_with_timeout():
    got_a = lock_a.acquire(timeout=1)
    if not got_a:
        return "gave up"
    try:
        got_b = lock_b.acquire(timeout=1)
        if not got_b:
            return "gave up"
        try:
            pass          # critical section
        finally:
            lock_b.release()
    finally:
        lock_a.release()
```

**Deadlock's four necessary conditions** (interview trivia — breaking any one prevents deadlock): mutual exclusion, hold-and-wait, no preemption, circular wait. Consistent lock ordering breaks *circular wait*, which is the usual fix in practice.

## 7. multiprocessing Essentials

```python
import multiprocessing as mp
import os

def worker(x):
    return x * x

if __name__ == "__main__":          # REQUIRED on Windows/macOS (spawn) — guards against
                                     # re-importing and re-running this module in each child
    # Pool: the go-to for "map this function over a list of inputs across processes"
    with mp.Pool(processes=4) as pool:
        results = pool.map(worker, [1, 2, 3, 4, 5])
        print(results)               # [1, 4, 9, 16, 25]

    # Process: lower-level, one process per task, manual start/join (mirrors threading.Thread)
    def show_pid():
        print("child pid:", os.getpid())
    p = mp.Process(target=show_pid)
    p.start()
    p.join(timeout=3)

    # Getting a return value back needs an explicit channel — Process itself has no .result
    def compute(x, out_queue):
        out_queue.put(x * x)
    q = mp.Queue()
    p = mp.Process(target=compute, args=(9, q))
    p.start()
    p.join()
    print(q.get())                   # 81
```

Key gotchas: arguments and return values must be **picklable** (plain data, not e.g. lambdas, open file handles, or DB connections); each process gets a **copy** of the parent's memory at fork time, so mutating a global in a child does **not** affect the parent; and forgetting the `if __name__ == "__main__":` guard causes infinite process spawning on platforms using `spawn` (Windows, and macOS by default since 3.8).

## 8. Sharing State Across Processes

Processes don't share memory — you must opt in explicitly.

```python
import multiprocessing as mp

if __name__ == "__main__":
    # --- Value / Array: shared memory for simple types, with a built-in lock ---
    counter = mp.Value('i', 0)       # 'i' = signed int; see `array` typecodes for others
    def increment(v):
        with v.get_lock():           # still need to lock compound operations!
            v.value += 1
    procs = [mp.Process(target=increment, args=(counter,)) for _ in range(8)]
    for p in procs: p.start()
    for p in procs: p.join()
    print(counter.value)             # 8

    arr = mp.Array('i', [0, 0, 0])
    def set_val(a, i, v):
        a[i] = v
    p = mp.Process(target=set_val, args=(arr, 1, 99))
    p.start(); p.join()
    print(list(arr))                 # [0, 99, 0]

    # --- Manager: shared Python objects (dict, list) via a proxy — more flexible, slower ---
    with mp.Manager() as manager:
        shared_dict = manager.dict()
        shared_list = manager.list()
        def fill(d, l, i):
            d[i] = i * i
            l.append(i)
        procs = [mp.Process(target=fill, args=(shared_dict, shared_list, i)) for i in range(4)]
        for p in procs: p.start()
        for p in procs: p.join()
        print(dict(shared_dict), list(shared_list))

    # --- Pipe: two-way channel between exactly two processes ---
    parent_conn, child_conn = mp.Pipe()
    def send_data(conn):
        conn.send("hello from child")
        conn.close()
    p = mp.Process(target=send_data, args=(child_conn,))
    p.start()
    print(parent_conn.recv())        # 'hello from child'
    p.join()
```

## 9. concurrent.futures — Executors

The modern, higher-level API — prefer this over raw `threading`/`multiprocessing` unless you need fine-grained control. Same interface for both thread- and process-based parallelism, so swapping one for the other is a one-line change.

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
import time

def io_task(n):
    time.sleep(0.02)
    return n * 2

def cpu_task(n):
    return sum(i * i for i in range(n))

# ThreadPoolExecutor — for I/O-bound work
with ThreadPoolExecutor(max_workers=4) as ex:
    results = list(ex.map(io_task, range(8)))          # preserves input order
    print(results)

# submit() + as_completed() — when you need results as they finish, not in submission order
with ThreadPoolExecutor(max_workers=4) as ex:
    futures = {ex.submit(io_task, n): n for n in range(5)}
    for future in as_completed(futures):
        n = futures[future]
        print(f"input {n} -> {future.result()}")

# Exceptions propagate through .result() — this is how you catch them
def risky(n):
    if n == 3: raise ValueError("bad input")
    return n
with ThreadPoolExecutor(max_workers=2) as ex:
    futures = [ex.submit(risky, n) for n in range(5)]
    for f in futures:
        try:
            print(f.result())
        except ValueError as e:
            print("caught:", e)

# ProcessPoolExecutor — for CPU-bound work (needs the __main__ guard, same as multiprocessing)
if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=4) as ex:
        results = list(ex.map(cpu_task, [100_000] * 4))
```

## 10. asyncio Fundamentals

```python
import asyncio, time

# A coroutine is defined with `async def`; calling it returns a coroutine OBJECT,
# it does not run the body yet — you must await it or schedule it.
async def fetch(name, delay):
    print(f"{name}: start")
    await asyncio.sleep(delay)     # yields control to the event loop — NOT time.sleep!
    print(f"{name}: done")
    return f"{name} result"

async def main():
    # Sequential await — no concurrency, same as calling functions normally
    r1 = await fetch("a", 0.05)

    # Concurrent execution — asyncio.gather runs coroutines interleaved on ONE thread
    results = await asyncio.gather(
        fetch("b", 0.05),
        fetch("c", 0.05),
        fetch("d", 0.05),
    )
    return results

t0 = time.perf_counter()
print(asyncio.run(main()))          # entry point — creates event loop, runs main(), closes loop
print(f"elapsed: {time.perf_counter() - t0:.2f}s")
# b, c, d run concurrently (~0.05s total for those three), not sequentially (~0.15s) —
# because each await asyncio.sleep() yields control back to the loop instead of blocking it.

# Tasks: schedule a coroutine to start running NOW, without awaiting it immediately
async def main2():
    task = asyncio.create_task(fetch("bg", 0.05))   # starts running concurrently
    await asyncio.sleep(0.01)
    print("doing other work while bg runs")
    result = await task                              # now wait for it to finish
    return result
asyncio.run(main2())

# GOTCHA: a blocking call inside a coroutine freezes the ENTIRE event loop —
# no other coroutine can run until it returns, because there's only one thread.
async def bad():
    time.sleep(1)          # WRONG — blocks everything; use `await asyncio.sleep(1)` instead
```

## 11. asyncio Synchronization & Patterns

```python
import asyncio

# --- asyncio.Lock — same idea as threading.Lock, but for coroutines on one thread ---
lock = asyncio.Lock()
counter = 0
async def increment():
    global counter
    async with lock:
        counter += 1

async def main_lock():
    await asyncio.gather(*(increment() for _ in range(100)))
    print(counter)          # 100
asyncio.run(main_lock())

# --- asyncio.Queue — producer/consumer for coroutines ---
async def producer(q, n):
    for i in range(n):
        await q.put(i)
    await q.put(None)        # sentinel

async def consumer(q, out):
    while True:
        item = await q.get()
        if item is None:
            break
        out.append(item * item)

async def main_queue():
    q = asyncio.Queue()
    out = []
    await asyncio.gather(producer(q, 5), consumer(q, out))
    return out
print(asyncio.run(main_queue()))    # [0, 1, 4, 9, 16]

# --- Timeouts and cancellation ---
async def slow():
    await asyncio.sleep(5)

async def main_timeout():
    try:
        await asyncio.wait_for(slow(), timeout=0.05)
    except asyncio.TimeoutError:
        return "timed out"
print(asyncio.run(main_timeout()))   # 'timed out'

# --- gather with exceptions: default raises on first exception, others keep running ---
async def maybe_fail(n):
    if n == 2:
        raise ValueError(f"failed at {n}")
    return n

async def main_exc():
    results = await asyncio.gather(
        *(maybe_fail(i) for i in range(4)), return_exceptions=True
    )
    return results             # exceptions are returned as values, not raised
print(asyncio.run(main_exc()))     # [0, 1, ValueError('failed at 2'), 3]
```

## 12. Running Blocking Code from Async

CPU-bound or legacy blocking calls (a synchronous DB driver, `time.sleep`, heavy computation) must NOT run directly in a coroutine — they freeze the whole event loop. Offload them to a thread or process pool.

```python
import asyncio, time
from concurrent.futures import ProcessPoolExecutor

def blocking_io():
    time.sleep(0.05)             # simulate a blocking call (sync DB driver, file I/O, ...)
    return "io done"

def cpu_heavy(n):
    return sum(i * i for i in range(n))

async def main():
    loop = asyncio.get_running_loop()

    # Default executor (a ThreadPoolExecutor) — good for blocking I/O
    result = await loop.run_in_executor(None, blocking_io)
    print(result)

    # Explicit ProcessPoolExecutor — good for CPU-bound work (bypasses the GIL)
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_heavy, 1_000_000)
        print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

## 13. Decorators — Deep Dive

```python
import functools, time

# A decorator is just a function that takes a function and returns a (usually wrapped) function.
def basic_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@basic_decorator
def greet(name):
    return f"hi {name}"
greet("Sam")

# WITHOUT functools.wraps, the wrapper steals the original function's __name__/__doc__ —
# breaks introspection, debuggers, and other decorators that inspect metadata.
def timing(func):
    @functools.wraps(func)          # copies __name__, __doc__, etc. from func to wrapper
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter()-t0:.4f}s")
        return result
    return wrapper

@timing
def slow_add(a, b):
    """Adds two numbers."""
    return a + b
slow_add(1, 2)
print(slow_add.__name__, slow_add.__doc__)     # 'slow_add' 'Adds two numbers.' — preserved

# Decorator WITH arguments — needs an extra layer of nesting
def retry(times):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exc = None
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exc = e
            raise last_exc
        return wrapper
    return decorator

calls = []
@retry(times=3)
def flaky():
    calls.append(1)
    if len(calls) < 3:
        raise ValueError("not yet")
    return "success"
print(flaky(), len(calls))     # success 3

# Class-based decorator — implements __call__ instead of returning an inner function
class CountCalls:
    def __init__(self, func):
        functools.update_wrapper(self, func)     # same idea as functools.wraps
        self.func = func
        self.calls = 0
    def __call__(self, *args, **kwargs):
        self.calls += 1
        return self.func(*args, **kwargs)

@CountCalls
def ping():
    return "pong"
ping(); ping()
print(ping.calls)              # 2

# Manual memoization decorator (functools.cache/lru_cache does this for you — see Section 20)
def memoize(func):
    cache = {}
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
print(fib(30))                 # instant, thanks to caching
```

## 14. Generators & Iterators — Protocol Deep Dive

```python
# The iterator protocol: __iter__ returns an iterator, __next__ returns the next value
# and raises StopIteration when exhausted. `for` loops call this under the hood.
class CountUp:
    def __init__(self, limit):
        self.limit = limit
        self.n = 0
    def __iter__(self):
        return self                # an iterator is its own iterator
    def __next__(self):
        if self.n >= self.limit:
            raise StopIteration
        self.n += 1
        return self.n

print(list(CountUp(5)))            # [1, 2, 3, 4, 5]

# A generator FUNCTION does all of this for you — `yield` pauses and resumes state automatically.
def count_up(limit):
    n = 0
    while n < limit:
        n += 1
        yield n
print(list(count_up(5)))           # [1, 2, 3, 4, 5]

# Generators are LAZY and STATEFUL — values are produced one at a time, on demand.
# Key interview point: O(1) memory regardless of how many values are produced.
def infinite_evens():
    n = 0
    while True:
        yield n
        n += 2
import itertools
print(list(itertools.islice(infinite_evens(), 5)))    # [0, 2, 4, 6, 8] — never builds the full list

# send() / throw() / close() — two-way communication with a running generator
def echo():
    while True:
        received = yield
        print("got:", received)

gen = echo()
next(gen)               # "prime" the generator — advance to the first yield
gen.send("hello")       # resumes execution, `received` = "hello"
gen.close()              # raises GeneratorExit inside the generator, stops it

# yield from — delegate to a sub-generator (flattens, forwards send/throw/StopIteration value)
def inner():
    yield 1
    yield 2
    return "inner done"
def outer():
    result = yield from inner()
    yield result
print(list(outer()))     # [1, 2, 'inner done']

# Generator expression — like a list comprehension, but lazy (note the parens, not brackets)
squares = (x * x for x in range(1_000_000))   # O(1) memory — nothing computed yet
print(next(squares))                           # 0 — computed on demand
print(sum(x * x for x in range(10)))           # generator expr as a direct function argument
```

## 15. Context Managers — Deep Dive

```python
# Class-based context manager: implement __enter__ and __exit__
class Timer:
    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self                    # what `as x` binds to
    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.elapsed = time.perf_counter() - self.start
        return False                   # False/None => propagate any exception; True => suppress it

with Timer() as t:
    sum(range(1000))
print(f"{t.elapsed:.6f}s")

# __exit__ returning True SWALLOWS the exception — use deliberately, not by accident
class IgnoreErrors:
    def __enter__(self): return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        return exc_type is not None    # suppress any exception that occurred

with IgnoreErrors():
    raise ValueError("this is silently swallowed")
print("execution continues here")

# contextlib.contextmanager — write a context manager as a generator function (much less boilerplate)
from contextlib import contextmanager

@contextmanager
def timer_cm():
    import time
    t0 = time.perf_counter()
    try:
        yield t0                       # code before yield = __enter__, after = __exit__
    finally:
        print(f"{time.perf_counter() - t0:.6f}s")   # finally guarantees cleanup runs even on exception

with timer_cm():
    sum(range(1000))

# contextlib.suppress — shorthand for "ignore this specific exception type"
from contextlib import suppress
with suppress(ZeroDivisionError):
    1 / 0
print("continues normally")

# ExitStack — manage a dynamic/unknown number of context managers, all cleaned up on exit
from contextlib import ExitStack
def process_files(paths):
    with ExitStack() as stack:
        files = [stack.enter_context(open(p)) for p in paths]  # each gets closed on exit, in reverse order
        return [f.read() for f in files]

# Multiple context managers in one `with` (equivalent to nesting, cleaner to read)
with open("/dev/null") as a, open("/dev/null") as b:
    pass
```

## 16. Closures & Scope

```python
# LEGB: Local -> Enclosing -> Global -> Built-in — the order Python resolves a name in.
x = "global"
def outer():
    x = "enclosing"
    def inner():
        x = "local"
        return x
    return inner()

# A closure: an inner function that "remembers" variables from its enclosing scope
def make_counter():
    count = 0
    def increment():
        nonlocal count      # without this, `count += 1` would raise UnboundLocalError
        count += 1
        return count
    return increment

c1 = make_counter()
c2 = make_counter()          # separate closure, separate `count`
print(c1(), c1(), c1(), c2())   # 1 2 3 1

# CLASSIC GOTCHA: late binding in closures created inside a loop.
# All three lambdas share the SAME variable `i` — by the time they're called, the loop has
# finished, so they all see the final value.
fns = [lambda: i for i in range(3)]
print([f() for f in fns])       # [2, 2, 2]  <- NOT [0, 1, 2] as many expect

# Fix 1: default argument captures the value AT DEFINITION time, not call time
fns_fixed = [lambda i=i: i for i in range(3)]
print([f() for f in fns_fixed]) # [0, 1, 2]

# Fix 2: a factory function that creates a new scope per iteration
def make_fn(i):
    return lambda: i
fns_fixed2 = [make_fn(i) for i in range(3)]
print([f() for f in fns_fixed2])  # [0, 1, 2]

# This bites people in real code most often with loops that spawn threads or build callbacks:
# threading.Thread(target=lambda: handle(i)) inside a for loop has the exact same bug.
```

## 17. *args, **kwargs & Argument Unpacking

```python
def f(a, b, *args, c=10, **kwargs):
    # a, b: positional (required)
    # *args: extra positional args, collected as a tuple
    # c: keyword-only (must be passed by name, has a default)
    # **kwargs: extra keyword args, collected as a dict
    print(a, b, args, c, kwargs)

f(1, 2, 3, 4, c=99, d=5)          # 1 2 (3, 4) 99 {'d': 5}

# Unpacking when CALLING a function
def add3(a, b, c): return a + b + c
nums = [1, 2, 3]
print(add3(*nums))                # unpack list/tuple as positional args
d = {"a": 1, "b": 2, "c": 3}
print(add3(**d))                  # unpack dict as keyword args

# Positional-only (/) and keyword-only (*) markers (3.8+)
def g(pos_only, /, normal, *, kw_only):
    return pos_only, normal, kw_only
g(1, 2, kw_only=3)                 # OK
g(1, normal=2, kw_only=3)          # OK
# g(pos_only=1, normal=2, kw_only=3)  # TypeError — pos_only can't be passed by name

# Merging dicts/lists via unpacking (see also Section 5 of the main cheatsheet)
merged = {**{"a": 1}, **{"b": 2}, "c": 3}
combined = [*[1, 2], *[3, 4]]

# Forwarding all arguments — the standard pattern for wrapper functions/decorators
def wrapper(*args, **kwargs):
    return add3(*args, **kwargs)   # pass everything straight through
```

## 18. Python Data Model / Dunder Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):            # unambiguous, for developers: repr(obj), debugger output
        return f"Vector({self.x}, {self.y})"
    def __str__(self):             # readable, for end users: print(obj), str(obj)
        return f"({self.x}, {self.y})"

    def __eq__(self, other):       # ==
        return isinstance(other, Vector) and (self.x, self.y) == (other.x, other.y)
    def __hash__(self):            # needed for use in sets/dict keys IF __eq__ is defined
        return hash((self.x, self.y))
    def __lt__(self, other):       # < (enables sorting)
        return (self.x, self.y) < (other.x, other.y)

    def __add__(self, other):      # + operator overloading
        return Vector(self.x + other.x, self.y + other.y)
    def __len__(self):             # len(obj)
        return 2
    def __getitem__(self, i):      # obj[i] — also makes the object ITERABLE automatically
        return (self.x, self.y)[i]
    def __setitem__(self, i, val):
        if i == 0: self.x = val
        else: self.y = val
    def __contains__(self, val):   # `in` operator
        return val in (self.x, self.y)
    def __call__(self, scale):     # makes instances callable: obj(scale)
        return Vector(self.x * scale, self.y * scale)
    def __bool__(self):            # truthiness: bool(obj), `if obj:`
        return self.x != 0 or self.y != 0

v1, v2 = Vector(1, 2), Vector(3, 4)
print(v1 + v2)          # (4, 6)  <- uses __add__ and __str__
print(v1 == Vector(1, 2))            # True <- __eq__
print(v1 < v2)                       # True <- __lt__
print(len(v1), v1[0], 3 in v1)       # 2 1 False
print(list(v1))                      # [1, 2] <- __getitem__ makes it iterable
print(v1(2))                         # (2, 4) <- __call__
print(bool(Vector(0, 0)))            # False <- __bool__

# Common dunder groups worth knowing by name for interviews:
# Construction:    __init__, __new__ (rarely overridden — __new__ controls allocation, __init__ init)
# Representation:  __repr__, __str__, __format__
# Comparison:      __eq__, __ne__, __lt__, __le__, __gt__, __ge__ (functools.total_ordering fills in the rest)
# Arithmetic:      __add__, __sub__, __mul__, __truediv__, __floordiv__, __mod__, __pow__ (+ __radd__ etc for reflected)
# Container:       __len__, __getitem__, __setitem__, __delitem__, __contains__, __iter__, __next__
# Context manager: __enter__, __exit__
# Callable:        __call__
```

## 19. Memory Management

```python
import sys, gc, weakref

# CPython uses REFERENCE COUNTING as the primary mechanism: an object is freed the
# instant its refcount hits 0 — deterministic, unlike Java/Go's GC.
a = [1, 2, 3]
print(sys.getrefcount(a))     # 2 (the variable `a`, plus the temporary arg to getrefcount)
b = a                          # refcount += 1
del b                          # refcount -= 1

# Reference counting alone can't collect CYCLES (a refers to b, b refers to a).
# CPython's cyclic garbage collector (the `gc` module) handles that separately,
# periodically scanning for unreachable cycles.
class Node:
    def __init__(self): self.other = None
n1, n2 = Node(), Node()
n1.other, n2.other = n2, n1     # a reference cycle
del n1, n2                       # neither refcount hits 0 (they still ref each other) —
gc.collect()                     # the cyclic GC is what actually frees them

# weakref: reference an object WITHOUT increasing its refcount — common for caches
# and observer patterns where you don't want to keep something alive just by watching it.
class Big:
    pass
obj = Big()
ref = weakref.ref(obj)
print(ref() is obj)              # True — call it to get the object (or None if collected)
del obj
print(ref())                     # None — the weakref does not keep it alive

# __slots__: skip the per-instance __dict__, storing attributes in fixed-size slots instead.
# Saves memory (meaningful at scale — millions of instances) and prevents adding new
# attributes ad hoc, at the cost of losing __dict__-based flexibility.
class PointDict:
    def __init__(self, x, y): self.x, self.y = x, y
class PointSlots:
    __slots__ = ("x", "y")
    def __init__(self, x, y): self.x, self.y = x, y

print(sys.getsizeof(PointDict(1, 2).__dict__))   # has a dict — extra overhead
p = PointSlots(1, 2)
# p.z = 3   # AttributeError — __slots__ blocks arbitrary new attributes

# Small int caching / string interning — why `is` can look like it "works" for ==
x, y = 256, 256
print(x is y)           # True — small ints -5..256 are ALWAYS cached/interned by CPython

# GOTCHA WITHIN A GOTCHA: literals written in the same statement/function are also
# frequently deduplicated by the compiler's constant-folding — independent of the
# -5..256 cache — so this is misleading if you're trying to test "large int caching":
x, y = 1000, 1000
print(x is y)            # True on CPython — NOT because 1000 is cached, but because the
                          # compiler reused one constant object for both literals in this code

# To see genuinely separate runtime-allocated large ints (no shared literal, no small-int
# cache), force them to be parsed/constructed dynamically instead of written as literals:
x = int("1000")
y = int("1000")
print(x is y)             # False — two distinct int objects this time

# Takeaway: NEVER rely on `is` for value equality — both directions of this example are
# CPython implementation details (the -5..256 cache AND same-code-object constant folding),
# not language guarantees. Use `==` for value comparison; reserve `is` for None/singletons.
```

## 20. property, staticmethod, classmethod & functools

```python
import functools

class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property                      # access like an attribute: circle.radius (no parens)
    def radius(self):
        return self._radius

    @radius.setter                 # enables circle.radius = 5 with validation
    def radius(self, value):
        if value < 0:
            raise ValueError("radius must be non-negative")
        self._radius = value

    @property                      # computed/read-only property — no setter defined
    def area(self):
        return 3.14159 * self._radius ** 2

    @staticmethod                  # no `self`/`cls` — just a regular function namespaced in the class
    def unit_circle():
        return Circle(1)

    @classmethod                   # receives the class, not an instance — common for alternate constructors
    def from_diameter(cls, diameter):
        return cls(diameter / 2)

c = Circle(2)
print(c.radius, c.area)            # 2 3.14159*4
c.radius = 5                       # calls the setter
print(Circle.unit_circle().radius) # 1
print(Circle.from_diameter(10).radius)  # 5.0

# Interview distinction: instance method (self, sees instance state) vs classmethod
# (cls, sees/creates class-level state, inherited correctly by subclasses) vs
# staticmethod (neither — just lives in the class's namespace for organization).
class Base:
    @classmethod
    def make(cls):
        return cls()               # `cls` is whichever subclass actually called this
class Derived(Base):
    pass
print(type(Derived.make()))        # <class '__main__.Derived'> — NOT Base

# functools.lru_cache / cache — memoization without writing it by hand (see Section 13's manual version)
@functools.lru_cache(maxsize=None)      # maxsize=None == functools.cache (3.9+)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
print(fib(35))
print(fib.cache_info())            # CacheInfo(hits=..., misses=..., maxsize=None, currsize=...)

# functools.singledispatch — function overloading by argument TYPE (Python has no native overloading)
@functools.singledispatch
def describe(x):
    return f"generic: {x}"
@describe.register
def _(x: int):
    return f"int: {x}"
@describe.register
def _(x: list):
    return f"list of {len(x)}"
print(describe(5), describe([1,2,3]), describe("s"))
```

## 21. Abstract Base Classes & Protocols

```python
from abc import ABC, abstractmethod

# ABC: enforce that subclasses implement certain methods — fails at INSTANTIATION time
# if they don't, not just at call time. Use when you want to define a contract.
class Shape(ABC):
    @abstractmethod
    def area(self): ...
    @abstractmethod
    def perimeter(self): ...

class Square(Shape):
    def __init__(self, s): self.s = s
    def area(self): return self.s ** 2
    def perimeter(self): return 4 * self.s

# Shape()          # TypeError — can't instantiate an abstract class directly
sq = Square(4)
print(sq.area())    # 16

# Python generally prefers DUCK TYPING ("if it walks like a duck and quacks like a duck")
# over rigid inheritance hierarchies — you often don't need ABC at all, just call the method
# and let it raise AttributeError/TypeError naturally (EAFP — see Section 24).

# typing.Protocol (3.8+): STRUCTURAL typing — "anything with this shape counts",
# without requiring explicit inheritance. Great for interfaces in type-hinted code.
from typing import Protocol, runtime_checkable

@runtime_checkable
class HasArea(Protocol):
    def area(self) -> float: ...

class Circle:                       # note: does NOT inherit from HasArea
    def __init__(self, r): self.r = r
    def area(self): return 3.14159 * self.r ** 2

def total_area(shapes: list) -> float:
    return sum(s.area() for s in shapes)

print(isinstance(Circle(2), HasArea))   # True — structural match, no inheritance needed
print(total_area([Square(2), Circle(2)]))
```

## 22. Mutability, Identity & Thread-Safety of Built-ins

```python
# `is` checks IDENTITY (same object in memory); `==` checks VALUE EQUALITY.
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b, a is b)          # True False — same value, different objects

# Simple built-in operations (a single bytecode op) are effectively atomic under the GIL —
# e.g. `list.append`, a single dict assignment `d[k] = v`, `x = y` — but COMPOUND operations
# are NOT atomic, even though they look like one line:
counter = 0
counter += 1        # actually 3 steps: LOAD counter, ADD 1, STORE counter — a thread
                     # can be interrupted between any of these (this is exactly Section 6's race condition)

shared_list = []
shared_list.append(1)     # this specific call IS atomic — safe to do from multiple threads
                           # without a lock (CPython implementation detail, not a language guarantee —
                           # don't design around it; use a Lock or Queue for anything non-trivial)

# Mutable default argument gotcha (also in the main cheatsheet, Section 1) is really an
# identity/mutability issue: the default is created ONCE, at function-definition time,
# and every call without that arg shares the SAME object.
def bad(acc=[]):
    acc.append(1)
    return acc
print(bad(), bad())    # [1] [1, 1] — same list, not fresh each call!
```

## 23. Classic Concurrency Interview Questions

```python
import threading

# --- "Print numbers alternately using two threads" ---
def print_alternating():
    turn = threading.Event()      # controls whose turn it is
    turn.set()                     # thread A starts
    out = []
    def a():
        for i in range(1, 10, 2):
            turn.wait(); turn.clear()
            out.append(i)
    def b():
        for i in range(2, 10, 2):
            while turn.is_set(): pass   # busy-wait for simplicity (a Condition is cleaner — see below)
            out.append(i)
            turn.set()
    ta, tb = threading.Thread(target=a), threading.Thread(target=b)
    ta.start(); tb.start()
    ta.join(); tb.join()
    return out
print(print_alternating())         # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Cleaner version with Condition (no busy-waiting):
def print_alternating_condition():
    cond = threading.Condition()
    state = {"turn": "a"}
    out = []
    def a():
        for i in range(1, 10, 2):
            with cond:
                cond.wait_for(lambda: state["turn"] == "a")
                out.append(i)
                state["turn"] = "b"
                cond.notify_all()
    def b():
        for i in range(2, 10, 2):
            with cond:
                cond.wait_for(lambda: state["turn"] == "b")
                out.append(i)
                state["turn"] = "a"
                cond.notify_all()
    ta, tb = threading.Thread(target=a), threading.Thread(target=b)
    ta.start(); tb.start()
    ta.join(); tb.join()
    return out
print(print_alternating_condition())

# --- "Implement a thread-safe bounded blocking queue from scratch" (what queue.Queue does internally) ---
class BoundedBlockingQueue:
    def __init__(self, capacity):
        self.capacity = capacity
        self.items = []
        self.cond = threading.Condition()

    def put(self, item):
        with self.cond:
            self.cond.wait_for(lambda: len(self.items) < self.capacity)
            self.items.append(item)
            self.cond.notify_all()        # wake anyone waiting to get()

    def get(self):
        with self.cond:
            self.cond.wait_for(lambda: len(self.items) > 0)
            item = self.items.pop(0)
            self.cond.notify_all()        # wake anyone waiting to put()
            return item

bq = BoundedBlockingQueue(2)
out = []
def producer():
    for i in range(5): bq.put(i)
def consumer():
    for _ in range(5): out.append(bq.get())
tp, tc = threading.Thread(target=producer), threading.Thread(target=consumer)
tp.start(); tc.start()
tp.join(); tc.join()
print(out)          # [0, 1, 2, 3, 4]

# --- Rate limiter (Semaphore-based, bounds concurrent access; token bucket is the time-based cousin) ---
class RateLimiter:
    def __init__(self, max_concurrent):
        self.sem = threading.Semaphore(max_concurrent)
    def __enter__(self):
        self.sem.acquire()
    def __exit__(self, *exc):
        self.sem.release()

limiter = RateLimiter(2)
def call_api(i, log):
    with limiter:
        log.append(f"calling {i}")
log = []
threads = [threading.Thread(target=call_api, args=(i, log)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()
```

## 24. Gotchas & "Explain This Output"

```python
# 1. Threads don't speed up CPU-bound code (Section 1) — the #1 concurrency interview trap.

# 2. Mutable default arguments (main cheatsheet Section 1 / Section 22 here) — created once,
#    shared across all calls that don't override it.

# 3. Late-binding closures in loops (Section 16) — all closures share the loop variable's FINAL value.

# 4. multiprocessing on spawn platforms without `if __name__ == "__main__":` —
#    re-imports and re-executes the whole module in each child -> infinite recursion of process spawning.

# 5. Forgetting `await` — this does NOT run the coroutine, it just creates an object:
async def foo(): return 42
async def caller():
    result = foo()          # BUG: result is a coroutine object, not 42
    # print(result)          # <coroutine object foo at 0x...>  (also raises "coroutine was never awaited")
    result = await foo()    # correct: result is 42
    return result

# 6. Blocking the event loop with a synchronous call inside an async function (Section 10/12) —
#    freezes ALL other coroutines, not just the current one, because there's only one thread.

# 7. `except Exception:` swallowing KeyboardInterrupt/SystemExit is avoided automatically —
#    those inherit from BaseException, not Exception — but a bare `except:` catches everything,
#    including Ctrl+C and sys.exit(). Prefer `except Exception:` unless you truly mean everything.

# 8. Modifying a list/dict while iterating over it — a classic non-concurrency footgun that
#    LOOKS like a race condition but isn't: it's undefined behavior even single-threaded.
items = [1, 2, 3, 4]
# for x in items:
#     if x % 2 == 0:
#         items.remove(x)     # BUG: mutating while iterating skips elements
# Fix: iterate over a copy, or build a new list.
items = [x for x in items if x % 2 != 0]

# 9. `is` vs `==` for small ints/interned strings "accidentally working" (Section 19/22) —
#    don't rely on CPython implementation details; use `==` for value comparison, `is` only for None/singletons.
```

## 25. Decision Cheat Table

| Situation | Use |
|---|---|
| Waiting on network calls, disk I/O, DB queries (blocking libraries) | `threading` (simple) or `asyncio` (if you need thousands of concurrent connections) |
| Waiting on network calls with an async-native library (aiohttp, asyncpg) | `asyncio` |
| Crunching numbers, image/data processing, anything CPU-bound | `multiprocessing` or `ProcessPoolExecutor` |
| Need a quick "run this over a list in parallel" | `concurrent.futures` (`ThreadPoolExecutor` for I/O, `ProcessPoolExecutor` for CPU) |
| Need shared mutable state across threads | `threading.Lock` + shared object, or `queue.Queue` (avoids needing a lock at all) |
| Need shared mutable state across processes | `multiprocessing.Value`/`Array` (simple types) or `Manager` (dicts/lists) |
| Need to memoize a pure function | `functools.lru_cache` / `functools.cache` |
| Need to enforce a subclass contract | `abc.ABC` + `@abstractmethod` |
| Need structural "duck typing" with type-checker support | `typing.Protocol` |
| Building a large number of small objects, care about memory | `__slots__` |
| Need a caching structure that shouldn't keep objects alive | `weakref` |

---

*Companion to `python_cheatsheet.md`. Python 3.10+ assumed for syntax (a few features noted inline need 3.8/3.9).*
