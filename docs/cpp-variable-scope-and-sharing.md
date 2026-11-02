# Global, local, and shared variables

## The idea

The **scope** of a variable describes where its name can be used in the
program. A variable declared outside functions has namespace scope (often
called global scope when declared at the top level). A variable declared
inside a function or block has local scope and is available only within that
scope.

Scope is different from **sharing**. Threads share access to objects in the
same process when they have a way to refer to those objects. A global object
can be shared by multiple threads, but not every global is necessarily used
by multiple threads. A local variable in a thread's function is normally a
separate object for each function call.

## Example

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int counter = 0;                 // Namespace-scope object
std::mutex counter_mutex;        // One mutex shared by both threads

void increment() {
    int completed = 0;           // A separate local for each call

    for (int i = 0; i < 1000; ++i) {
        std::lock_guard<std::mutex> lock(counter_mutex);
        ++counter;               // Both calls update the same object
        ++completed;             // Each call updates its own local
    }

    std::cout << "This thread completed " << completed << " increments\n";
}

int main() {
    std::thread first(increment);
    std::thread second(increment);
    first.join();
    second.join();

    std::cout << "Total: " << counter << '\n'; // Total: 2000
}
```

`counter` and `counter_mutex` are declared outside the functions, so both
calls to `increment` can refer to the same objects. Each call creates its own
`completed` and loop variable `i`. The mutex protects concurrent access to
`counter`.

If `counter` were instead declared as `int counter = 0;` inside
`increment()`, each call would get a separate counter. Those local counters
would not add up into one shared total.

## Check your understanding

1. Which variables in the example are declared outside all functions?
2. Do both calls to `increment()` use the same `counter`?
3. Does each call to `increment()` have its own `completed` variable?
4. If `counter` were declared locally inside `increment()`, would both
   threads update the same counter?
5. Is every global variable automatically shared between threads?

<details>
<summary>Answer key</summary>

1. `counter` and `counter_mutex`.
2. Yes. They both refer to the same namespace-scope object.
3. Yes. Each function call has its own local variable.
4. No. Each call would have a distinct local counter.
5. No. A global variable is shared only if multiple threads can access and
   actually use that object.

</details>
