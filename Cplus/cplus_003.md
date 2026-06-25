# C++ copy and move use cases

In C++, understanding exactly *when* these special member functions are called is the key to mastering resource management (and avoiding accidental performance hits).

Here is the breakdown of the exact scenarios that trigger the copy and move constructors, as well as the copy and move assignment operators.

---

## 1. Copy Constructor

The copy constructor creates a **new object** as a copy of an **existing object** of the same type. It operates on uninitialized memory.

### Triggers:

* **Direct or Copy Initialization:** Creating a new object directly from an existing one.
```cpp
MyClass obj1;
MyClass obj2 = obj1; // Triggers copy constructor
MyClass obj3(obj1);  // Triggers copy constructor

```


* **Passing by Value:** Passing an object into a function by value (not by reference).
```cpp
void printObject(MyClass obj) { /* ... */ }

MyClass mainObj;
printObject(mainObj); // Triggers copy constructor to create 'obj'

```


* **Returning by Value (Pre-C++17 / No Elision):** Returning an object from a function by value. *Note: Modern compilers often optimize this away via Copy Elision/NRVO, but conceptually, it relies on the copy constructor.*
```cpp
MyClass createObject() {
    MyClass localObj;
    return localObj; // Can trigger copy constructor (if not elided)
}

```


* **Catching Exceptions by Value:** 
```cpp
try {
throw MyClass();
} catch (MyClass ex) { // Triggers copy constructor
// ...
}
```

---

## 2. Copy Assignment Operator (`operator=`)

Unlike the constructor, the copy assignment operator copies values from an existing object into an **already initialized, existing object**.

### Triggers:

* **Assigning one existing variable to another:**
```cpp
MyClass obj1;
MyClass obj2; 
// Both objects already exist
obj2 = obj1; // Triggers copy assignment operator
```



> ⚠️ **Common Gotcha:** `MyClass obj2 = obj1;` looks like an assignment because of the `=` sign, but because a new object (`obj2`) is being created, it calls the **copy constructor**, not the assignment operator.

---

## 3. Move Constructor

Introduced in C++11, the move constructor creates a **new object** by "stealing" resources from a temporary or explicitly moved object (**rvalue**), leaving the source in a valid but unspecified state.

### Triggers:

* **Initialization with a Temporary (Rvalue):**
```cpp
MyClass obj = MyClass(); // Triggers move constructor (if not elided by compiler)
```


* **Using `std::move`:** Explicitly casting an lvalue to an rvalue reference to force a move.
```cpp
MyClass obj1;
MyClass obj2 = std::move(obj1); // Triggers move constructor
```


* **Returning a Local Object by Value:** If the compiler cannot perform Copy Elision, it will automatically attempt to use the move constructor to return the local variable.
```cpp
MyClass makeObject() {
    MyClass localObj;
    return localObj; // Automatically moved if not elided
}

```



---

## 4. Move Assignment Operator (`operator=`)

The move assignment operator transfers resources from a temporary or explicitly moved object (**rvalue**) into an **already initialized, existing object**.

### Triggers:

* **Assigning a Temporary to an Existing Object:**
```cpp
MyClass obj;
obj = MyClass(); // Triggers move assignment (temporary is destroyed right after)
```


* **Assigning via `std::move` to an Existing Object:**
```cpp
MyClass obj1;
MyClass obj2;
// Both objects already exist
obj2 = std::move(obj1); // Triggers move assignment

```



---

## Quick Reference Summary

| Special Member Function | Target Object Status | Source Object Type       | Example Syntax              |
| ----------------------- | -------------------- | ------------------------ | --------------------------- |
| **Copy Constructor**    | Being Created (New)  | Lvalue (Existing/Named)  | `MyClass a = b;`            |
| **Move Constructor**    | Being Created (New)  | Rvalue (Temporary/Moved) | `MyClass a = std::move(b);` |
| **Copy Assignment**     | Already Initialized  | Lvalue (Existing/Named)  | `a = b;`                    |
| **Move Assignment**     | Already Initialized  | Rvalue (Temporary/Moved) | `a = std::move(b);`         |

## When should we use default move constructors / assignment

Use a **defaulted move constructor and move assignment operator whenever the compiler-generated behavior is exactly what you want**.

The compiler-generated move operations perform a member-wise move:

* movable class-type members are moved
* primitive/scalar members are copied
* base classes are moved
* arrays are moved element-by-element

For many classes, that's perfect.

### Good candidate for `= default`

```cpp id="h6ck6x"
class Widget {
    std::string name;
    std::vector<int> data;
    std::unique_ptr<Foo> foo;

public:
    Widget(Widget&&) noexcept = default;
    Widget& operator=(Widget&&) noexcept = default;
};
```

The generated move operations will:

* move the `std::string`
* move the `std::vector`
* transfer ownership of the `std::unique_ptr`

No custom code is needed.

---

### When you need custom move operations

#### 1. You manage a raw resource

```cpp id="t3sfxh"
class DynamicArray {
    int* ptr;
    std::size_t size;
};
```

The compiler-generated move constructor would just copy the pointer value:

```cpp id="vwdk0w"
ptr = other.ptr;
```

leaving both objects owning the same allocation, which causes double deletion.

You need custom move logic:

```cpp id="5yq57v"
DynamicArray(DynamicArray&& other) noexcept
    : ptr(other.ptr), size(other.size)
{
    other.ptr = nullptr;
    other.size = 0;
}
```

(or better, use `std::unique_ptr<int[]>`).

---

#### 2. The moved-from object needs special handling

Suppose you wrap a file descriptor:

```cpp id="ijwzwv"
class File {
    int fd;
};
```

After moving, you must invalidate the source:

```cpp id="qu0x25"
other.fd = -1;
```

The default move won't do that because `int` is merely copied.

---

#### 3. Some members shouldn't be moved

Occasionally you want behavior different from member-wise move.

For example:

```cpp id="1uq5lf"
class Cache {
    std::vector<int> data;
    Statistics stats;
};
```

You might want to move `data` but reset `stats` instead of moving it.

That requires a custom implementation.

---

### A practical rule

If your class follows the **Rule of Zero** (uses RAII types like `std::string`, `std::vector`, `std::unique_ptr`, etc., and doesn't manually manage resources), then:

```cpp id="n08ggm"
class MyClass {
public:
    MyClass(MyClass&&) noexcept = default;
    MyClass& operator=(MyClass&&) noexcept = default;
};
```

or often you don't even need to declare them at all.

If your class owns raw resources (`new`, file handles, sockets, mutex ownership, etc.), you'll often need custom move operations or, better yet, wrap those resources in types that already implement correct move semantics.

A useful checklist:

**Use `= default` if**

* every member already has correct move semantics,
* member-wise move is exactly what you want,
* no special cleanup/reset of the source is needed.

**Write custom move operations if**

* you own raw resources,
* the source object must be explicitly invalidated/reset,
* you need behavior different from simple member-wise moves.

## Inside move constructor/assignment, when should you copy and when should you use std::move

That's a much more subtle question.

When you're implementing a move constructor/assignment, the real question is:

> **Is the member itself responsible for owning/managing a resource?**
>
> * If yes, usually move it (`std::move`).
> * If no, and you're manually stealing the resource at a lower level, just copy the handle and null/reset the source.

### Example 1: Raw pointer ownership — don't use `std::move`

Suppose you're implementing your own smart pointer:

```cpp
class UniquePtr {
    int* ptr;

public:
    UniquePtr(UniquePtr&& other)
        : ptr(other.ptr)          // copy pointer value
    {
        other.ptr = nullptr;
    }
};
```

Here `ptr` is just an address.

You are manually transferring ownership:

1. Copy the address.
2. Null out the source.

Using `std::move` adds nothing:

```cpp
ptr(std::move(other.ptr)) // same as copy for raw pointers
```

because moving a raw pointer is identical to copying it.

---

### Example 2: `std::unique_ptr` member — use `std::move`

```cpp
class Widget {
    std::unique_ptr<int> ptr;

public:
    Widget(Widget&& other)
        : ptr(std::move(other.ptr))
    {}
};
```

Now the member itself knows how to transfer ownership.

Without `std::move`:

```cpp
: ptr(other.ptr)   // ERROR
```

`std::unique_ptr` is non-copyable.

The move constructor of `unique_ptr` performs the ownership transfer.

Note:: **Here  std::move itself does not transfer ownership. It merely enables the move operation to be selected**

Suppose:
```cpp
std::unique_ptr<int> p1(new int(42));
std::unique_ptr<int> p2;

p2 = std::move(p1);
```

`std::move(p1)` is essentially:

```cpp
static_cast<std::unique_ptr<int>&&>(p1)
```

It does not:
* move memory
* transfer ownership
* modify p1
by itself.

so what actually transfers ownership?

After the cast, overload resolution sees an rvalue:
```cpp
p2 = std::move(p1);
```
and chooses unique_ptr's move assignment operator:
```cpp
unique_ptr& operator=(unique_ptr&& other);
```

Inside that function, ownership transfer is implemented.

---

### Example 3: `std::string` member — use `std::move`

```cpp
class Person {
    std::string name;

public:
    Person(Person&& other)
        : name(std::move(other.name))
    {}
};
```

Without `std::move`:

```cpp
: name(other.name)
```

you get a copy.

The string already knows how to efficiently steal its internal buffer, so let it do that.

---

### Example 4: Resource managed manually

Imagine a vector-like class:

```cpp
class MyVector {
    int* data;
    size_t size;

public:
    MyVector(MyVector&& other)
        : data(other.data),
          size(other.size)
    {
        other.data = nullptr;
        other.size = 0;
    }
};
```

This is the classic move implementation.

No `std::move` needed because you're transferring the underlying resource yourself.

---

### Example 5: Mixed members

```cpp
class Buffer {
    int* raw_data;
    std::string name;
    std::vector<int> cache;

public:
    Buffer(Buffer&& other)
        : raw_data(other.raw_data),      // manual steal
          name(std::move(other.name)),   // delegate move
          cache(std::move(other.cache))  // delegate move
    {
        other.raw_data = nullptr;
    }
};
```

Notice the pattern:

* `raw_data`: you own the resource directly → copy handle + null source.
* `name`: `std::string` already implements move → use `std::move`.
* `cache`: `std::vector` already implements move → use `std::move`.

---

### Move assignment follows the same idea

Manual resource management:

```cpp
MyVector& operator=(MyVector&& other)
{
    delete[] data;

    data = other.data;
    size = other.size;

    other.data = nullptr;
    other.size = 0;

    return *this;
}
```

Managed members:

```cpp
Widget& operator=(Widget&& other)
{
    ptr = std::move(other.ptr);
    return *this;
}
```

---

### A useful mental model

Ask yourself:

**Am I moving the resource itself, or am I moving an object that manages the resource?**

#### Resource itself

```cpp
int* ptr;
FILE* file;
SocketHandle handle;
```

Usually:

```cpp
ptr = other.ptr;
other.ptr = nullptr;
```

No `std::move` needed.

#### Object managing a resource

```cpp
std::string
std::vector
std::unique_ptr
std::fstream
MyCustomRAIIClass
```

Usually:

```cpp
member = std::move(other.member);
```

because that object's move operations already know how to transfer ownership correctly.

That's why in well-written modern C++, explicit pointer stealing becomes rare. Most move constructors end up looking like:

```cpp
MyClass(MyClass&& other)
    : str(std::move(other.str))
    , vec(std::move(other.vec))
    , ptr(std::move(other.ptr))
{}
```

because the member types already implement the resource transfer logic for you.
