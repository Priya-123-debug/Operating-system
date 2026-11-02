# C++ program memory layout

## The idea

C++ does not require programs to use a particular physical memory layout.
However, a common implementation model describes memory using regions such as
the **text/code**, **data**, **stack**, and **heap**. These names are useful
for learning, but actual placement depends on the compiler, operating system,
and executable format.

- **Text/code:** commonly contains executable instructions.
- **Initialized data:** commonly contains initialized objects with static
  storage duration, such as a global initialized to a non-zero value.
- **BSS (uninitialized data):** a common executable-format region for
  zero-initialized static-storage objects.
- **Stack:** commonly used for function calls and automatic local variables.
- **Heap / dynamic storage:** storage requested dynamically, for example
  with `new`. C++ describes this as dynamic storage duration; “heap” is the
  usual implementation term.

The C++ standard specifies object lifetimes and storage durations, not that a
particular variable must occupy a named hardware memory region.

## Example

```cpp
#include <iostream>

int initialized_global = 10; // Static storage duration
int zero_initialized_global; // Static storage duration; initialized to zero

void inspect_storage() {
    int local = 20;                  // Automatic storage duration
    static int persistent_local = 30; // Static storage duration
    int* pointer = new int(40);      // Pointer is local; allocated int is dynamic

    std::cout << initialized_global << ' '
              << zero_initialized_global << ' '
              << local << ' '
              << persistent_local << ' '
              << *pointer << '\n';

    delete pointer; // Release the dynamically allocated int
}

int main() {
    inspect_storage();
}
```

In the common layout model, the global objects and `persistent_local` are
associated with data storage (often initialized data or BSS). `local` and
`pointer` have automatic storage duration and are commonly placed on the
stack. The `int` created by `new` has dynamic storage duration and is commonly
placed on the heap. The pointer and the object it points to are distinct
objects with different storage durations.

`static` on a local variable changes its storage duration: the variable
continues to exist between function calls, even though its name is visible
only inside that function.

## Check your understanding

1. What is the difference between the local pointer `pointer` and the
   dynamically allocated integer it points to?
2. Which objects in the example have static storage duration?
3. Where are automatic local variables commonly stored in the simplified
   memory model?
4. Does the C++ standard guarantee that a global variable is physically
   stored in a region called the “data segment”?
5. Why does the example call `delete pointer`?

<details>
<summary>Answer key</summary>

1. The pointer itself is a local automatic object; the allocated integer is a
   separate object with dynamic storage duration.
2. `initialized_global`, `zero_initialized_global`, and
   `persistent_local`.
3. On the stack in the common implementation model.
4. No. The standard specifies storage duration and lifetime, not a physical
   segment layout.
5. To release the memory obtained with `new`.

</details>
