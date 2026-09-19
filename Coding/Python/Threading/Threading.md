---
tags: [coding, python, concurrency, threading]
type: cheatsheet
source: compiled reference (Python 3 threading)
last-verified: 2026-08-27
---

# Threading

## Up
- [[Python]]

Python's `threading` module runs code concurrently within one process. Because of the **GIL** (Global Interpreter Lock), only one thread executes Python bytecode at a time — so threads help with **I/O-bound** work (network, disk, waiting) but not **CPU-bound** work. For CPU parallelism use `multiprocessing` or `concurrent.futures.ProcessPoolExecutor`.

---

## Creating & Starting Threads

```python
import threading, time

def worker(name, delay):
    for i in range(3):
        print(f"{name}: {i}")
        time.sleep(delay)

t = threading.Thread(target=worker, args=("A", 0.5), kwargs={})
t.start()          # begins execution
t.join()           # wait for it to finish (optional timeout=)

# multiple threads
threads = [threading.Thread(target=worker, args=(f"T{i}", 0.1)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()

# daemon thread — dies when the main program exits
t = threading.Thread(target=worker, daemon=True)
```

### Subclassing Thread

```python
class MyThread(threading.Thread):
    def __init__(self, n):
        super().__init__()
        self.n = n
        self.result = None
    def run(self):                 # override run(), NOT start()
        self.result = self.n * 2

t = MyThread(21); t.start(); t.join(); print(t.result)   # 42
```

---

## Locks (mutual exclusion)

```python
lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:                    # acquire/release automatically
        counter += 1              # protected critical section

# manual form
lock.acquire()
try:
    ...
finally:
    lock.release()

rlock = threading.RLock()         # re-entrant: same thread can acquire repeatedly
```

Without a lock, `counter += 1` across threads is a race condition (read-modify-write is not atomic).

---

## Coordination Primitives

```python
# Event — signal between threads
event = threading.Event()
event.wait(timeout=5)   # block until set
event.set(); event.clear(); event.is_set()

# Condition — wait for a state change
cond = threading.Condition()
with cond:
    cond.wait()             # release lock & block until notified
    cond.notify()           # wake one waiter  (notify_all() for all)

# Semaphore — limit concurrent access to N
sem = threading.Semaphore(3)
with sem:                   # at most 3 threads inside at once
    ...

# Barrier — wait until N threads reach the same point
barrier = threading.Barrier(3)
barrier.wait()
```

---

## Thread-Safe Queues (the preferred pattern)

```python
import queue, threading

q = queue.Queue()          # FIFO (also LifoQueue, PriorityQueue)

def producer():
    for i in range(10):
        q.put(i)
    q.put(None)            # sentinel to stop

def consumer():
    while True:
        item = q.get()
        if item is None: break
        print("processing", item)
        q.task_done()

threading.Thread(target=producer).start()
threading.Thread(target=consumer).start()
q.join()                  # block until all items processed
```

`queue.Queue` is internally locked — it's the safest way to hand work between threads without writing your own locking.

---

## concurrent.futures (high-level, recommended)

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch(url):
    import requests
    return requests.get(url, timeout=10).status_code

urls = ["https://example.com"] * 5

with ThreadPoolExecutor(max_workers=8) as ex:
    # map: results in input order
    for code in ex.map(fetch, urls):
        print(code)

    # submit + as_completed: results as they finish
    futures = {ex.submit(fetch, u): u for u in urls}
    for fut in as_completed(futures):
        url = futures[fut]
        try:
            print(url, fut.result())
        except Exception as e:
            print(url, "failed:", e)
```

`ThreadPoolExecutor` manages the pool, collects results, and propagates exceptions via `future.result()` — prefer it over hand-rolled `Thread` objects for most work.

---

## Useful Helpers

```python
threading.current_thread().name
threading.active_count()
threading.get_ident()                       # OS thread id
threading.local()                           # per-thread storage

timer = threading.Timer(5.0, lambda: print("fired"))
timer.start(); timer.cancel()               # run fn after a delay
```

---

## Choosing a Concurrency Model

| Workload | Use |
|---|---|
| I/O-bound (HTTP, files, DB waits), few tasks | `threading` / `ThreadPoolExecutor` |
| I/O-bound, thousands of tasks | `asyncio` (async/await) |
| CPU-bound (number crunching) | `multiprocessing` / `ProcessPoolExecutor` |

- The GIL means threads won't speed up pure-Python CPU work — expect no gain (or a slight loss).
- Always `join()` non-daemon threads, or protect shared state with locks/queues.
- Prefer message-passing (`queue.Queue`) over shared mutable state to avoid deadlocks.
