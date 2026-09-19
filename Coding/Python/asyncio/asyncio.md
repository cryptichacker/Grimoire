---
tags: [coding, python, asyncio, concurrency, async]
type: cheatsheet
source: compiled reference (Python asyncio)
last-verified: 2026-09-03
---

# asyncio

## Up
- [[Python]]

`asyncio` is Python's standard library for **single-threaded concurrency** using an event loop and `async`/`await`. It excels at **I/O-bound** work with many concurrent operations (thousands of network calls) — where [[Threading]] hits overhead. It does **not** speed up CPU-bound work (still one thread, bound by the GIL).

---

## Coroutines & Running

```python
import asyncio

async def main():                 # 'async def' defines a coroutine
    print("start")
    await asyncio.sleep(1)        # 'await' yields control while waiting
    print("done")

asyncio.run(main())               # entry point — creates loop, runs, closes
```

- Calling `main()` returns a **coroutine object** — it does nothing until awaited or scheduled.
- `await` only works inside `async def`. You can await coroutines, Tasks, and Futures ("awaitables").

---

## Concurrency: Tasks & gather

```python
async def fetch(n):
    await asyncio.sleep(n)
    return n

async def main():
    # run concurrently, collect results in order
    results = await asyncio.gather(fetch(1), fetch(2), fetch(3))   # ~2s, not 6s

    # create_task schedules immediately (concurrent)
    t = asyncio.create_task(fetch(5))
    ...                                   # other work runs while t is in flight
    val = await t

    # TaskGroup (3.11+) — structured concurrency, auto-awaits + cancels on error
    async with asyncio.TaskGroup() as tg:
        tg.create_task(fetch(1))
        tg.create_task(fetch(2))

asyncio.run(main())
```

- **`gather`** runs awaitables concurrently, returns a list of results (`return_exceptions=True` to collect errors instead of raising).
- **`create_task`** turns a coroutine into a running Task on the loop.
- **`as_completed`** yields results as they finish: `for coro in asyncio.as_completed(tasks): r = await coro`.

---

## Timeouts & Cancellation

```python
try:
    await asyncio.wait_for(fetch(10), timeout=2)   # cancels on timeout
except asyncio.TimeoutError:
    print("too slow")

async with asyncio.timeout(2):        # 3.11+ context-manager form
    await fetch(10)

task.cancel()                          # request cancellation → raises CancelledError inside
```

---

## Synchronization Primitives

```python
lock = asyncio.Lock()
async with lock: ...                   # mutual exclusion

sem = asyncio.Semaphore(10)            # cap concurrency (e.g. 10 in-flight requests)
async with sem: await fetch(...)

event = asyncio.Event(); await event.wait(); event.set()

q = asyncio.Queue()
await q.put(item); item = await q.get(); q.task_done(); await q.join()
```

`asyncio.Semaphore` is the idiom for **rate-limiting** concurrent I/O.

---

## Async Iterators & Context Managers

```python
async def stream():
    for i in range(3):
        await asyncio.sleep(0.1)
        yield i                        # async generator

async for x in stream(): print(x)

class Conn:
    async def __aenter__(self): ...; return self
    async def __aexit__(self, *exc): ...
async with Conn() as c: ...
```

---

## Running Blocking Code (don't block the loop!)

```python
# CPU-bound or blocking sync call → offload to a thread/process
result = await asyncio.to_thread(blocking_fn, arg)          # 3.9+
# or: loop.run_in_executor(None, blocking_fn, arg)
```

**Never call blocking functions** (`time.sleep`, sync `requests`, heavy CPU) directly in a coroutine — they freeze the whole loop. Use `await asyncio.sleep`, async libraries (httpx/aiohttp/asyncpg), or `to_thread`.

---

## Ecosystem

| Need | Async library |
|---|---|
| HTTP client | `httpx` (async), `aiohttp` |
| Postgres | `asyncpg`, SQLAlchemy async |
| Redis | `redis.asyncio` |
| Web frameworks | [[FastAPI]], Starlette, aiohttp |
| Task orchestration | `anyio` (works across asyncio/trio) |

---

## asyncio vs Threading vs Multiprocessing

| Workload | Use |
|---|---|
| I/O-bound, thousands of tasks | **asyncio** |
| I/O-bound, few tasks / blocking libs | [[Threading]] |
| CPU-bound | `multiprocessing` / `ProcessPoolExecutor` |

---

## Tips
- One `asyncio.run()` per program entry; don't create multiple loops.
- Prefer **TaskGroup** (3.11+) for structured concurrency — it cancels siblings on error.
- Bound concurrency with a **Semaphore**; unbounded `gather` over 10k tasks can exhaust sockets.
- A forgotten `await` is the #1 bug — a coroutine you never await just... doesn't run (and warns).
- Debug with `asyncio.run(main(), debug=True)` to catch slow callbacks and un-awaited coroutines.
