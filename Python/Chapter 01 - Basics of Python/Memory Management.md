### Python Memory Management
Python manages memory management automatically, so programmers generally do not explicitly allocate or free individual objects as they do in C using `malloc()` and `free()`.

In standard CPython implementation, memory management involves several related mechanisms :

1. **Object Allocation** ; python creates objects when required. each object has implementation-specific metadata, and storage for its value or state.
2. **Reference Counting** : cpython tracks references to objects. when an object's reference count reaches 0. it can generally be deallocated, subject to implementation details.
3. **Garbage Collection** : cpython also has a cyclic garbage collector that can identify certain unreachable reference cycles that reference counting alone cannot reclaim.
4. **Memory Allocator** : cpython uses memory allocation mechanisms, including specialized allocators for small objects. Reclaimed memory is not necessarily returned to the operating system immediately.

In python, we can use `del` keyword to remove the name binding.
```python
a = [10, 20]
b = a
del a
```
The conceptual sequence will be :
- A list object is created.
- `a` refers to the list.
- `b = a` creates another reference to the same list.
- `del a` removes the name binding for `a`.
- The list remains accessible through `b`

>**Note** : `del` removes a name binding only, it does not forcibly destroys an object. the object will be destroyed only when there is no reference to it. (Reference Counting)

Reference Counting alone cannot reclaim unreachable object. for example :
```python
a = []
a.append(a)
del a
```

The list contains a reference to itself. Even after name a is deleted, a reference cycle exists. This type of unreachable cycles can be detected by CPython's cyclic garbage collector, and it then reclaims the memory.

So, the points to remember are :
- **Reference Counting** primarily tracks references to objects.
- **Cyclic Garbage Collection** handles unreachable reference cycles that reference counting alone cannot detect or reclaim.
- Different python implementations can have different memory management, all the above information is subjected to standard cpython implementation.


---
### Stack, Heap, and Python Objects
In python, all objects and data structures live on a private heap, while the stack is used to store frame objects, local variable references, and execution flow. In languages like cpp, java, etc. - variable hold values directly on stack. But in python a variable inside a stack references or points to a memory location of object residing inside heap.

| Feature          | Stack                                                                        | Heap (private)                                                               |
| ---------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| What it stores ? | Function call blocks (Frames), references to variables, and execution order. | Every single python object (int, str, class, list, dict, etc).               |
| Allocation Style | Fast, Linear, Contiguous memory blocks.                                      | Dynamic, non-contiguous blocks managed by interpreter.                       |
| Lifecycle        | Automatically managed via function entries and exits (LIFO)                  | Managed by `pymalloc` allocator, reference counting, and garbage collection. |
When we execute code like `x = 100`, python creates an integer object (100) on the heap, a reference named x is created in the current function's stack frame on the stack, pointing directly to memory address of `100` on the heap.

To bypass slow OS-level allocation (`malloc`), python uses a specialized memory manager called `pymalloc`. It breaks down memory into highly structured hierarchy :
- **Arenas (256 KB / 1 MB)** : the largest chunks of memory requested directly from the OS. These arenas consists of pools.
- **Pools (4 KB)** : a pool is a 4 KB block composed of uniform blocks. all blocks inside a pool are of same size.
- **Blocks** : The actual slots where objects are stored, `pymalloc` divides blocks into 64 different size classes (e.g 8 Bytes, 16 Bytes, up to 512 Bytes).
	- If an object is <= 512 Bytes : `pymalloc` handles it from matching size-class pool.
	- if an object is > 512 Bytes : python skips `pymalloc` and hands over the allocation to system's raw standard C allocator (`malloc`).

Everything in python is a `PyObject`, which includes a type pointer and reference count. However the python memory manager uses specific strategies :
- **Small integers & Booleans** ; python pre-allocates integers from `-5` to `256` at startup. that means when you assign `y = 10`, python does not allocate new heap memory. It references the pre-existing, static memory address for `10`.
- **Strings** : Short, compile-time strings (like variable names or dictionary keys) are optimized via string interning, ensuring unique strings share a single heap address to save space.
- **List & Tuples** : a list object on the heap is essentially an *array of pointers* to other heap objects. when python list grows, python over-allocates extra slot spaces dynamically to ensure appending items do not trigger a heavy reallocation cycle every time.


---
### Integer Caching
As we discussed earlier, python considers -5 to 256 as small integers, and caches them on startup.
```python
a = 256
b = 256
print(a is b) # True
```

Also,
```python
a = 257
b = 257
print(a is b) # True
```

But when using Interactive mode like REPL or Notebooks,
```python
>>> a = 257
>>> b = 257
>>> a is b
>>> False
```

This happens because 257 is out of caching range, but in script mode the whole script goes as once, and the compiler optimizes the code as it sees that `257` is twice in the code. But in the case of interactive mode, each line is compiled independently, that is why cache out of range occurs.

Same is the case with float values, but none is cached so all scenarios will return false in interactive mode and true in script mode.
```python
a = 1.7
b = 1.7
print(a is b) # True
```
```python
>>> a = 1.7
>>> b = 1.7
>>> a is b
>>> False
```

But in the case of string it returns true in both modes,
```python
a = "hello"
b = "hello"
print(a is b) # true
```
```python
>>> a = "hello"
>>> b = "hello"
>>> a is b
>>> True
```

and for mutable object like lists, dict, set, etc it always returns false in both modes.
```python
a = [1,2,3]
b = [1,2,3]
print(a is b) # False
```
```python
>>> a = [1,2,3]
>>> b = [1,2,3]
>>> a is b
>>> False
```

---
### Object Identity

| Expression | Meaning                                                     |
| ---------- | ----------------------------------------------------------- |
| `a == b`   | Tests whether value of a and b is same                      |
| `a is b`   | Tests whether both names refers to the same object          |
| `id(a)`    | Returns an identity value which is unique for every object. |
Lets understand with an example :
```python
a = [1, 2]
b = [1, 2]
c = a

print(a == b) # returns True
print(a is b) # returns False
print(a is c) # returns True
```

although a and b have similar content but they refer to different objects - Note that mutable object like lists, dictionaries, etc - for all of them new object is created, while for immutable objects like int, str, etc - multiples names points to same object because it is immutable and not going to be changed. But in case of `c = a`, c is refered to same object as that of a.

```python
a = 10
b = 10

print(a==b) # true
print(a is b) # generally true for CPython
```

In this case as `10` is an immutable object and already loaded at python startup, a and b refer to same object.