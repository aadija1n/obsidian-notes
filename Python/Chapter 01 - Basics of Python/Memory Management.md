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

| Feature | Stack | Heap  |
| ------- | ----- | ----- |
|         |       |       |
