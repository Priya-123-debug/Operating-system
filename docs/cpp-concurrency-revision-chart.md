# C++ Concurrency and OS Concepts

## Start here: choose the concept

| If the problem is… | Think… | Main purpose |
|---|---|---|
| Threads update shared data at the same time | **Race condition** | Protect or safely update shared state |
| Threads wait on resources held by one another | **Deadlock** | Break the circular wait |
| One thread keeps missing out on a resource | **Starvation** | Improve fairness |
| Threads keep reacting but do no useful work | **Livelock** | Change the retry protocol |
| Only one thread may enter a critical section | **Mutex** | Enforce exclusive access |
| A fixed number of resources is available | **Semaphore** | Count available permits |
| A thread must sleep until something becomes true | **Condition variable** | Wait efficiently for a state change |

Use the sections below to understand why each answer fits.

## 1. Find the concurrency risk

Ask these questions in order:

1. **Can multiple threads or processes run at the same time?**
   - If no, there is no concurrency between them to analyze.
2. **Do they access the same data or resource?**
   - If no, there is no race on that data. You may still need synchronization
     for ordering or other shared resources.
3. **Can they access shared data concurrently, with at least one writing?**
   - If yes, protect the shared state with a mutex or use an appropriate
     atomic operation.
4. **Are they waiting for resources held by one another?**
   - If yes, check for a circular wait and the other deadlock conditions.

### Race condition and data race are related, but different

- A **race condition** is a broad term: the result depends on timing or the
  order in which concurrent operations happen.
- A **data race** in C++ is a specific unsynchronized conflict: concurrent
  access to the same memory location, at least one access writes, and the
  accesses are not properly ordered. A C++ data race causes **undefined
  behavior**.

For example, two threads executing `++counter` on the same ordinary integer
without synchronization can have a data race. Do not rely on any particular
final value.

## 2. Recognize the four common failure modes

### Race condition

**What happens:** The result depends on the timing or interleaving of
operations.

**Example:** Two threads increment the same counter without synchronization.

**Typical response:** Protect the shared state with a mutex or use an
appropriate atomic operation.

### Deadlock

**What happens:** Threads wait indefinitely for resources held by one another.

**Example:** Thread A holds lock A and waits for lock B, while thread B holds
lock B and waits for lock A.

**Typical response:** Acquire locks in a consistent order, or redesign resource
acquisition. Timeouts may help with recovery, but do not by themselves prove
that a design is deadlock-free.

### Starvation

**What happens:** One thread waits indefinitely because other threads
repeatedly get the resource first, while the system may still make progress.

**Typical response:** Use fair scheduling or fair resource allocation where
available.

### Livelock

**What happens:** Threads keep responding to one another, but make no useful
progress.

**Example:** Two workers repeatedly back off and retry at the same time.

**Typical response:** Add randomized backoff, retry limits, or a protocol that
breaks the repeated interaction.

## 3. Deadlock checklist: the Coffman conditions

All four conditions must hold at the same time for deadlock to be possible:

1. **Mutual exclusion** — A resource can be held by only one thread at a time.
2. **Hold and wait** — A thread holds one resource while waiting for another.
3. **No preemption** — A resource cannot be forcibly taken from its holder.
4. **Circular wait** — Threads form a cycle, each waiting for a resource held
   by the next.

> **Remember:** All four conditions make deadlock possible, not inevitable.
> Prevent deadlock by ensuring at least one condition cannot hold. A common
> strategy is to acquire locks in one consistent order, preventing circular
> wait.

## 4. Choose the right synchronization tool

### Mutex — protect shared state

Use a mutex when only one thread at a time should access a critical section.

```cpp
#include <mutex>

std::mutex counter_mutex;
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(counter_mutex);
    ++counter;
}
```

- A mutex has ownership: the thread that locks it should unlock it.
- `std::lock_guard` unlocks automatically when it leaves scope.
- Use the same mutex consistently whenever accessing the protected state.

### Semaphore — count available permits

Use a counting semaphore when a limited number of threads may use a resource
at once. C++20 provides `std::counting_semaphore` in `<semaphore>`.

```cpp
#include <semaphore>

std::counting_semaphore<3> permits(3);

void use_resource() {
    permits.acquire(); // Wait for one of the three permits.
    // Use the resource.
    permits.release();
}
```

The initial count of `3` allows up to three acquisitions before another
thread must wait. A semaphore does not enforce mutex-style ownership: a
different thread can release a permit. In production code, ensure permits
are released on every exit path.

### Condition variable — wait for a condition

Use a condition variable when a thread should sleep until a condition becomes
true, such as a queue receiving an item. Protect the condition with a mutex
and always wait with a predicate.

```cpp
#include <condition_variable>
#include <mutex>

std::mutex mutex;
std::condition_variable condition;
bool ready = false;

void wait_until_ready() {
    std::unique_lock<std::mutex> lock(mutex);
    condition.wait(lock, [] { return ready; });
}

void make_ready() {
    {
        std::lock_guard<std::mutex> lock(mutex);
        ready = true;
    }
    condition.notify_one();
}
```

`wait()` releases the mutex while the thread sleeps and reacquires it before
returning. The predicate rechecks the condition, including after a spurious
wakeup.

### One-line comparison

- **Mutex:** one owner at a time; protects shared state.
- **Semaphore:** a count of permits; limits concurrent access to resources.
- **Condition variable:** waits for a condition to become true.

## 5. Quick memory-layout review

This is a common simplified model, **not** a physical layout guaranteed by
the C++ standard.

| Common region | Commonly contains | Example |
|---|---|---|
| **Stack** | Function calls and automatic local variables. Each thread typically has its own stack. | `int local = 5;` inside a function |
| **Heap / dynamic storage** | Dynamically allocated objects. | `int* p = new int(10);` |
| **Initialized data** | Initialized objects with static storage duration. | `int global = 10;` |
| **BSS** | Zero-initialized static-storage objects in common executable formats. | `int zero_global;` |
| **Text / code** | Compiled machine instructions for functions. | A compiled function |

In `int* p = new int(10);`, the pointer `p` is a separate object from the
integer it points to. The pointer may be an automatic local commonly placed
on the stack; the allocated integer has dynamic storage duration and is
commonly placed on the heap. Actual placement depends on the compiler,
operating system, and executable format.

## 6. Practice: match the situation

Try answering before opening the key.

1. Two threads increment the same global counter without synchronization.
   - A. Deadlock
   - B. Race condition
   - C. Starvation
2. Thread A holds M1 and waits for M2; thread B holds M2 and waits for M1.
   - A. Race condition
   - B. Deadlock
   - C. Livelock
3. Only one thread should modify a linked list at a time.
   - A. Mutex
   - B. Counting semaphore
   - C. Condition variable only
4. A pool has exactly five resources available to concurrent threads.
   - A. Mutex only
   - B. Counting semaphore
   - C. Global variable only
5. A consumer should sleep until a producer adds an item to a queue.
   - A. Condition variable
   - B. Busy waiting
   - C. Deadlock
6. A thread never gets a lock because other threads continually acquire it
   first.
   - A. Starvation
   - B. Race condition
   - C. Segmentation fault

<details>
<summary>Show answers</summary>

1. **B — Race condition.** Unsynchronized concurrent updates are unsafe; a
   C++ data race has undefined behavior.
2. **B — Deadlock.** The threads form a circular wait.
3. **A — Mutex.** It provides exclusive access to the linked list.
4. **B — Counting semaphore.** Five permits can represent five resources.
5. **A — Condition variable.** The consumer waits efficiently, then checks
   the queue condition again.
6. **A — Starvation.** The thread is repeatedly denied access while others
   make progress.

</details>

## Final five rules

1. **Shared data + unsynchronized concurrent writes** → race-condition risk.
2. **Threads waiting on one another in a resource cycle** → possible
   deadlock.
3. **One thread at a time in a critical section** → mutex.
4. **A limited number of resource permits** → semaphore.
5. **Wait until a condition becomes true** → condition variable.

A mutex does not automatically prevent every deadlock. A semaphore does not
automatically make every shared-data operation race-free. Choose the tool
that matches the problem and protect shared state consistently.
