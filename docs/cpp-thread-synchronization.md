# Threads, mutexes, and data races

## The idea

A thread is an independently executing path through a program. Threads in the
same process can access the same objects, so unsynchronized access to shared
data can cause a **data race**. In C++, a data race is undefined behavior: the
program is not guaranteed to produce any particular result.

A mutex provides exclusive access to a protected region. `std::lock_guard`
locks the mutex when it is constructed and unlocks it automatically when it
leaves scope, including when the scope exits because of an exception.

## Example: protect a shared counter

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int counter = 0;
std::mutex counter_mutex;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        std::lock_guard<std::mutex> lock(counter_mutex);
        ++counter;
    }
}

int main() {
    std::thread first(increment);
    std::thread second(increment);

    first.join();
    second.join();

    std::cout << counter << '\n'; // 2000
}
```

Both threads increment the same `counter` 1,000 times. The mutex ensures that
only one thread at a time performs each increment, so no update is lost.
`join()` makes `main` wait for each thread to finish before it reads and prints
the result; it does not protect the increments.

The lock is inside the loop, so it is acquired and released once per
increment. The threads may run in any order; the mutex guarantees exclusive
access, not a particular schedule.

## Without the mutex

Removing the `lock_guard` makes the concurrent increments a data race. The
final value is **not guaranteed** to be 2,000 or any other particular number.
It is incorrect to rely on seeing a particular result from a program with
undefined behavior.

## Check your understanding

1. What does `lock_guard` do when it is constructed and when it leaves scope?
2. Why does the example call `join()` before printing?
3. Does `join()` prevent two threads from accessing `counter` concurrently?
4. What is guaranteed about the final counter value when the mutex is used?
5. What can you conclude about the output if the mutex is removed?

<details>
<summary>Answer key</summary>

1. It locks the mutex at construction and unlocks it at scope exit.
2. It ensures both worker threads finish before `main` reads the counter.
3. No. The mutex protects the increments; `join()` waits for completion.
4. It is 2,000, assuming both threads complete normally.
5. There is a data race and the behavior is undefined; no specific output is
   guaranteed.

</details>
