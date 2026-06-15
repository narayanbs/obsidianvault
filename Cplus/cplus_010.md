# C++ Smart Pointers

Smart pointers are C++ library classes that manage dynamically allocated memory automatically. They help prevent common memory management problems such as:

* Memory leaks
* Double deletion
* Dangling pointers
* Exception-safety issues

Smart pointers are defined in the C++ Standard Library header:

```cpp
#include <memory>
```

The three primary smart pointers are:

1. `std::unique_ptr`
2. `std::shared_ptr`
3. `std::weak_ptr`

---

# Understanding `std::unique_ptr`

`std::unique_ptr` is one of the most important smart pointers in modern C++. It provides **exclusive ownership** of a dynamically allocated object and automatically releases the memory when the pointer goes out of scope.

## Why do we need it?

Before smart pointers, memory management was often done manually:

```cpp
void process()
{
    int* ptr = new int(42);

    // use ptr

    delete ptr;
}
```

Problems:

* Forgetting `delete` causes memory leaks.
* Exceptions can skip cleanup code.
* Ownership is often unclear.

Smart pointers solve these issues using the **RAII (Resource Acquisition Is Initialization)** principle.

---

# Basic Usage

```cpp
#include <iostream>
#include <memory>

int main()
{
    std::unique_ptr<int> ptr = std::make_unique<int>(42);

    std::cout << *ptr << std::endl;
}
```

Output:

```text
42
```

When `ptr` goes out of scope, the memory is automatically freed.

Equivalent manual code:

```cpp
int* ptr = new int(42);
delete ptr;
```

but much safer.

Note: 
```
std::unique_ptr<int> ptr = std::make_unique<int>(42); 
```
is the same as
```
std::unique_ptr<int> ptr(new int{42});

or

std::unique_ptr<int> ptr{new int{42}};
```
but make_unique is preferred, because it performs the allocation and construction inside a single function call, where as  the other one first evaluates new int{42} and then constructs the unique_ptr.

---

# Ownership Model

A `unique_ptr` owns exactly one object.

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(10);
```

At this point:

```text
p1 ---> 10
```

No other `unique_ptr` can own the same object.

---

# Copying Is Not Allowed

This will not compile:

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(10);

std::unique_ptr<int> p2 = p1;  // ERROR
```

Why?

Because both pointers would think they own the same memory.

The compiler prevents this.

---

# Moving Ownership

Ownership can be transferred using `std::move`.

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(10);

std::unique_ptr<int> p2 = std::move(p1);
```

After the move:

```text
p1 ---> nullptr

p2 ---> 10
```

Example:

```cpp
#include <iostream>
#include <memory>

int main()
{
    auto p1 = std::make_unique<int>(100);

    auto p2 = std::move(p1);

    if (!p1)
        std::cout << "p1 is empty\n";

    std::cout << *p2 << std::endl;
}
```

Output:

```text
p1 is empty
100
```

---

# Managing User-Defined Objects

```cpp
#include <iostream>
#include <memory>

class Employee
{
public:
    Employee()
    {
        std::cout << "Constructed\n";
    }

    ~Employee()
    {
        std::cout << "Destroyed\n";
    }

    void work()
    {
        std::cout << "Working\n";
    }
};

int main()
{
    std::unique_ptr<Employee> emp =
        std::make_unique<Employee>();

    emp->work();
}
```

Output:

```text
Constructed
Working
Destroyed
```

Notice that the destructor is automatically called.

---

# Passing `unique_ptr` to Functions

There are three common patterns.

## 1. Function Uses but Does Not Take Ownership

Pass by reference.

```cpp
void printValue(const std::unique_ptr<int>& ptr)
{
    std::cout << *ptr << std::endl;
}
```

Usage:

```cpp
auto ptr = std::make_unique<int>(42);
printValue(ptr);
```

Ownership remains with the caller.

---

## 2. Function Takes Ownership

Pass by value and move.

```cpp
void process(std::unique_ptr<int> ptr)
{
    std::cout << *ptr << std::endl;
}
```

Usage:

```cpp
auto ptr = std::make_unique<int>(42);

process(std::move(ptr));
```

After this:

```cpp
ptr == nullptr
```

Ownership transferred to `process()`.

---

## 3. Function Returns Ownership

```cpp
std::unique_ptr<int> createValue()
{
    return std::make_unique<int>(50);
}
```

Usage:

```cpp
auto ptr = createValue();
```

Note:
This works because createValue() returns a temporary rvalue and hence invokes the move constructor

This is a very common pattern in modern C++. 


---

# Arrays with `unique_ptr`

For arrays:

```cpp
std::unique_ptr<int[]> arr =
    std::make_unique<int[]>(5);
```

Assign values:

```cpp
arr[0] = 10;
arr[1] = 20;
```

Access:

```cpp
std::cout << arr[1];
```

No need for:

```cpp
delete[] arr;
```

The smart pointer handles it automatically.

---

# Using `get()`

Sometimes legacy APIs require a raw pointer.

```cpp
auto ptr = std::make_unique<int>(10);

int* raw = ptr.get();
```

Important:

```cpp
delete raw;   // NEVER DO THIS
```

`unique_ptr` still owns the memory.

`get()` only exposes the raw pointer temporarily.

---

# Releasing Ownership

`release()` gives up ownership without deleting the object.

```cpp
auto ptr = std::make_unique<int>(10);

int* raw = ptr.release();
```

After this:

```text
ptr ---> nullptr
raw ---> 10
```

Now you must manually delete:

```cpp
delete raw;
```

Use sparingly.

---

# Resetting a `unique_ptr`

```cpp
auto ptr = std::make_unique<int>(10);

ptr.reset();
```

The managed object is immediately destroyed.

You can also replace it:

```cpp
ptr.reset(new int(20));
```

Better:

```cpp
ptr = std::make_unique<int>(20);
```

---

# Custom Deleters

Sometimes resources are not freed with `delete`.

Example: closing a file.

```cpp
#include <cstdio>
#include <memory>

auto deleter = [](FILE* file)
{
    if (file)
        fclose(file);
};

std::unique_ptr<FILE, decltype(deleter)> fptr(fopen("data.txt", "r"), deleter);
```

When `file` goes out of scope:

```cpp
fclose(file);
```

deleter is automatically called.

This makes `unique_ptr` useful for managing:

* Files
* Sockets
* Database connections
* Mutexes
* OS handles

not just heap memory.

---

# Polymorphism Example

```cpp
class Shape
{
public:
    virtual ~Shape() = default;
    virtual void draw() = 0;
};

class Circle : public Shape
{
public:
    void draw() override
    {
        std::cout << "Circle\n";
    }
};
```

```cpp
std::unique_ptr<Shape> shape =
    std::make_unique<Circle>();

shape->draw();
```

Output:

```text
Circle
```

This is widely used in factory patterns.

---

# Factory Pattern Example

```cpp
class Logger
{
public:
    void log()
    {
        std::cout << "Logging\n";
    }
};

std::unique_ptr<Logger> createLogger()
{
    return std::make_unique<Logger>();
}
```

Usage:

```cpp
auto logger = createLogger();
logger->log();
```

This clearly communicates:

> "The caller becomes the sole owner."

---

# Common Mistakes

## Mistake 1: Using `new` directly

Less preferred:

```cpp
std::unique_ptr<int> ptr(new int(5));
```

Preferred:

```cpp
auto ptr = std::make_unique<int>(5);
```

Benefits:

* Exception safe
* Cleaner syntax
* Less repetition

---

## Mistake 2: Copying

```cpp
auto p2 = p1;  // ERROR
```

Use:

```cpp
auto p2 = std::move(p1);
```

---

## Mistake 3: Manually deleting managed memory

```cpp
delete ptr.get();  // WRONG
```

This causes undefined behavior.

---

## Mistake 4: Using moved-from pointers

```cpp
auto p2 = std::move(p1);

*p1 = 10;  // WRONG
```

After moving, assume:

```cpp
p1 == nullptr
```

until reassigned.

---

# When Should You Use `unique_ptr`?

Use `std::unique_ptr` when:

✅ One owner exists
✅ Ownership transfer is explicit
✅ Automatic cleanup is desired
✅ Building factories or resource wrappers

Avoid it when:

❌ Multiple objects must share ownership

In that case, use `std::shared_ptr`.

---

# Summary

`std::unique_ptr` provides:

* Automatic resource cleanup
* Exclusive ownership
* Move-only semantics
* Exception safety
* Support for custom deleters
* Efficient memory management (almost zero overhead compared to raw pointers)

A useful rule of thumb in modern C++ is:

> Prefer stack allocation first. If dynamic allocation is needed and ownership is unique, use `std::unique_ptr`. Use raw pointers only for non-owning references.


---

# Understanding `std::shared_ptr`

If `std::unique_ptr` is about **exclusive ownership**, then `std::shared_ptr` is about **shared ownership**.

A `shared_ptr` allows multiple pointers to own the same object. The object is automatically destroyed when the **last owner** releases it.


## Why Do We Need It?

Consider a situation where several parts of a program need access to the same object.

For example:

```text
GameEngine
    |
    +-- Renderer
    |
    +-- PhysicsSystem
    |
    +-- AI System
```

All three systems may need access to the same game object.

With `unique_ptr`, only one owner can exist.

With `shared_ptr`, ownership can be shared safely.

---

# Basic Usage

```cpp
#include <iostream>
#include <memory>

int main()
{
    std::shared_ptr<int> ptr =
        std::make_shared<int>(42);

    std::cout << *ptr << std::endl;
}
```

Output:

```text
42
```

The memory is automatically released when the last `shared_ptr` owning it is destroyed.

---

# Reference Counting

The key idea behind `shared_ptr` is **reference counting**.

```cpp
auto p1 = std::make_shared<int>(10);
```

Internally:

```text
Object: 10
Reference Count = 1
```

Now copy it:

```cpp
auto p2 = p1;
```

```text
Object: 10
Reference Count = 2
```

Another copy:

```cpp
auto p3 = p1;
```

```text
Object: 10
Reference Count = 3
```

When one pointer goes away:

```text
Reference Count = 2
```

When the count reaches zero:

```text
Reference Count = 0
```

the object is automatically destroyed.

---

# Example

```cpp
#include <iostream>
#include <memory>

int main()
{
    auto p1 = std::make_shared<int>(100);

    {
        auto p2 = p1;

        std::cout << p1.use_count() << '\n';
    }

    std::cout << p1.use_count() << '\n';
}
```

Output:

```text
2
1
```

Explanation:

* `p1` owns the object.
* `p2` shares ownership.
* When `p2` goes out of scope, the count decreases.

---

# Visualizing Ownership

```cpp
auto p1 = std::make_shared<int>(50);
auto p2 = p1;
auto p3 = p1;
```

Memory layout:

```text
        +----------------+
p1 ---->|                |
p2 ---->|   int(50)      |
p3 ---->|                |
        +----------------+

Reference Count = 3
```

All three pointers own the same object.

# Copying Is Allowed

Unlike `unique_ptr`:

```cpp
auto p1 = std::make_shared<int>(10);

auto p2 = p1;
```

This is perfectly valid.

The ownership is shared.

---

# Moving a `shared_ptr`

Moving transfers ownership from one pointer object to another.

```cpp
auto p1 = std::make_shared<int>(10);

auto p2 = std::move(p1);
```

Result:

```text
p1 -> nullptr
p2 -> owns object
```

Reference count remains unchanged. (it is still 1, because ownership is moved not copied)

# Control Block

The **control block** is the hidden object that makes `std::shared_ptr` work.

When you write:

```cpp
auto p = std::make_shared<int>(10);
```

there are actually **two things** involved:

1. The managed object (`int` containing `10`)
2. The control block (metadata used for ownership management)

A simplified picture:

```text
p
│
▼
+------------------+
| control block    |
|------------------|
| shared count = 1 |
| weak count = 0   |
| deleter          |
| allocator        |
| ptr to int       |
+------------------+
          │
          ▼
       +-----+
       | 10  |
       +-----+
```

---

## What's inside the control block?

Typically:

### 1. Shared ownership count

Tracks how many `shared_ptr`s own the object.

```cpp
auto p1 = std::make_shared<int>(10);
auto p2 = p1;
auto p3 = p1;
```

Control block:

```text
shared count = 3
```

When a `shared_ptr` is destroyed:

```cpp
p2.reset();
```

count becomes:

```text
shared count = 2
```

When it reaches zero:

```text
shared count = 0
```

the managed object is deleted.

---

### 2. Weak ownership count

Used by `std::weak_ptr`.

```cpp
auto sp = std::make_shared<int>(10);
std::weak_ptr<int> wp = sp;
```

Now:

```text
shared count = 1
weak count = 1
```

The object is destroyed when the shared count reaches 0.

The control block itself remains alive until both counts are 0.

---

### 3. Deleter

For custom deletion logic.

```cpp
auto p = std::shared_ptr<FILE>(
    fopen("a.txt", "r"),
    fclose
);
```

The control block stores the deleter (`fclose`) so it knows how to destroy the resource.

---

### 4. Allocator information

If custom allocators are used, the control block stores what's needed to free memory correctly.

---

## Why do copies increase the count?

```cpp
auto p1 = std::make_shared<int>(10);
auto p2 = p1;
```

Both pointers refer to the same control block:

```text
       p1
        │
        ▼
     control block
        ▲
        │
       p2
```

The count in that control block becomes 2.

---

## Why does move not increase the count?

```cpp
auto p2 = std::move(p1);
```

The control block isn't duplicated.

Before:

```text
p1 ──► control block
count = 1
```

After:

```text
p1 = nullptr

p2 ──► control block
count = 1
```

Ownership is transferred, not added.

---

## `make_shared` vs `shared_ptr(new T)`

### `make_shared`

```cpp
auto p = std::make_shared<int>(10);
```

Usually allocates:

```text
+----------------------+
| control block + int  |
+----------------------+
```

**One allocation**

---

### Direct construction

```cpp
auto p = std::shared_ptr<int>(new int(10));
```

Typically allocates:

```text
+---------------+     +-----+
| control block | --> | 10  |
+---------------+     +-----+
```

**Two allocations**

That's why `make_shared` is usually faster and more memory efficient.

---

## What happens when the last owner disappears?

```cpp
auto p1 = std::make_shared<int>(10);
auto p2 = p1;

p1.reset();
p2.reset();
```

Sequence:

```text
shared count: 2 -> 1 -> 0
```

When it reaches 0:

1. `int` object is destroyed.
2. If no `weak_ptr`s exist, control block is also destroyed.

If weak pointers still exist:

```text
shared count = 0
weak count = 1
```

then:

```text
object destroyed
control block kept alive
```

until the last `weak_ptr` goes away.

---

A useful mental model is:

```text
shared_ptr
    |
    +--> control block
             |
             +--> reference counts
             +--> deleter
             +--> allocator info
             +--> managed object (or pointer to it)
```

Every `shared_ptr` that refers to the same object shares **the same control block**, and the reference count lives there—not inside the `shared_ptr` object itself.



---

# Managing User-Defined Objects

```cpp
#include <iostream>
#include <memory>

class Employee
{
public:
    Employee()
    {
        std::cout << "Constructed\n";
    }

    ~Employee()
    {
        std::cout << "Destroyed\n";
    }
};

int main()
{
    auto emp1 = std::make_shared<Employee>();

    {
        auto emp2 = emp1;
        auto emp3 = emp1;

        std::cout << emp1.use_count() << '\n';
    }

    std::cout << emp1.use_count() << '\n';
}
```

Output:

```text
Constructed
3
1
Destroyed
```

The object is destroyed only when the final owner disappears.

---

# Passing `shared_ptr` to Functions

There are multiple ownership semantics.

---

## 1. Read-Only Access

```cpp
void print(const std::shared_ptr<int>& ptr)
{
    std::cout << *ptr << '\n';
}
```

Usage:

```cpp
auto ptr = std::make_shared<int>(42);

print(ptr);
```

No new owner is created.

---

## 2. Share Ownership

```cpp
void store(std::shared_ptr<int> ptr)
{
    // stores ptr somewhere
}
```

Usage:

```cpp
store(ptr);
```

Reference count increases.

The function now becomes an owner.

---

## 3. Use Raw Pointer or Reference

If ownership isn't needed:

```cpp
void print(const int& value)
{
    std::cout << value;
}
```

Usage:

```cpp
print(*ptr);
```

Often preferable.

---

# Returning `shared_ptr`

```cpp
std::shared_ptr<int> createValue()
{
    return std::make_shared<int>(100);
}
```

Usage:

```cpp
auto ptr = createValue();
```

The caller receives shared ownership.

---

# `make_shared` vs `new`

Avoid:

```cpp
std::shared_ptr<int> ptr(new int(10));
```

Prefer:

```cpp
auto ptr = std::make_shared<int>(10);
```

Benefits:

### Single Allocation

`make_shared` typically allocates:

```text
Control Block + Object
```

in one memory allocation.

Using `new` usually requires:

```text
Allocation 1 -> Object
Allocation 2 -> Control Block
```

So `make_shared` is generally:

* Faster
* More cache friendly
* Exception safe

---

# Resetting a `shared_ptr`

```cpp
auto ptr = std::make_shared<int>(10);

ptr.reset();
```

Ownership is released.

If this was the last owner:

```text
Reference Count -> 0
Object destroyed
```

---

# Custom Deleters

```cpp
auto deleter = [](FILE* file)
{
    if (file)
        fclose(file);
};

std::shared_ptr<FILE> file(
    fopen("data.txt", "r"),
    deleter);
```

When the last owner disappears:

```cpp
fclose(file);
```

is automatically called.

---

# Polymorphism Example

```cpp
class Shape
{
public:
    virtual ~Shape() = default;
    virtual void draw() = 0;
};

class Circle : public Shape
{
public:
    void draw() override
    {
        std::cout << "Circle\n";
    }
};
```

```cpp
std::shared_ptr<Shape> shape =
    std::make_shared<Circle>();

shape->draw();
```

Output:

```text
Circle
```

Very common in frameworks and plugin systems.

---

# The Biggest Problem: Circular References

Consider:

```cpp
class B;

class A
{
public:
    std::shared_ptr<B> b;
};

class B
{
public:
    std::shared_ptr<A> a;
};
```

Usage:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

Visual:

```text
A --> B
^     |
|     v
+-----+
```

Reference counts:

```text
A count = 1
B count = 1
```

After assigning:

```text
A count = 2
B count = 2
```

Even when:

```cpp
a.reset();
b.reset();
```

the objects still point to each other.

Counts never reach zero.

Result:

```text
Memory Leak
```

This is called a **reference cycle**.

---

# Solving Cycles with `weak_ptr`

One side should not own the other.

```cpp
class B;

class A
{
public:
    std::shared_ptr<B> b;
};

class B
{
public:
    std::weak_ptr<A> a;
};
```

Now:

```text
A owns B
B merely observes A
```

No cycle exists.

Memory is released correctly.

---

# When Should You Use `shared_ptr`?

Good use cases:

✅ Shared ownership is genuinely required

✅ Object lifetime is difficult to predict

✅ Publisher/subscriber systems

✅ Scene graphs

✅ Resource caches

✅ GUI frameworks

---

# When Should You Avoid It?

Avoid `shared_ptr` when:

❌ One clear owner exists

Use:

```cpp
std::unique_ptr
```

instead.

`unique_ptr` is:

* Smaller
* Faster
* No reference counting
* No atomic counter updates

A common modern C++ guideline is:

> Prefer `unique_ptr` by default. Use `shared_ptr` only when multiple owners are truly needed.

---

# Common Mistakes

## Mistake 1: Using `shared_ptr` Everywhere

Bad:

```cpp
std::shared_ptr<Car> car;
```

just because "it's safer."

Ask:

> Do multiple owners actually exist?

If not:

```cpp
std::unique_ptr<Car>
```

is better.

---

## Mistake 2: Creating Multiple `shared_ptr`s from the Same Raw Pointer

Wrong:

```cpp
int* p = new int(10);

std::shared_ptr<int> a(p);
std::shared_ptr<int> b(p);
```

Now there are **two independent control blocks**.

Both will attempt:

```cpp
delete p;
```

Result:

```text
Double Delete
Undefined Behavior
```

Correct:

```cpp
auto a = std::make_shared<int>(10);

auto b = a;
```

---

## Mistake 3: Ignoring Cycles

```cpp
shared_ptr<A>
shared_ptr<B>
```

pointing at each other is a classic leak.

Use `weak_ptr` to break cycles.

---

# Comparison: `unique_ptr` vs `shared_ptr`

| Feature            | unique_ptr | shared_ptr          |
| ------------------ | ---------- | ------------------- |
| Ownership          | Exclusive  | Shared              |
| Copyable           | No         | Yes                 |
| Movable            | Yes        | Yes                 |
| Reference Counting | No         | Yes                 |
| Overhead           | Very low   | Higher              |
| Default Choice     | Yes        | No                  |
| Supports Cycles    | N/A        | Can leak via cycles |

---

# Summary

`std::shared_ptr` provides:

* Shared ownership of an object
* Automatic lifetime management
* Reference counting
* Copyable semantics
* Support for polymorphism and resource sharing

The key rule is:

> Use `std::unique_ptr` when there is a single owner. Use `std::shared_ptr` only when multiple independent owners must keep an object alive. Use `std::weak_ptr` to observe shared objects without extending their lifetime and to break ownership cycles.


Perfect. The natural progression is:


---

# Understanding `std::weak_ptr`

```text
Raw Pointer
    ↓
unique_ptr   (one owner)
    ↓
shared_ptr   (multiple owners)
    ↓
weak_ptr     (non-owner observer)
```

Most developers understand `unique_ptr` quickly. `shared_ptr` takes a bit longer. `weak_ptr` is usually where smart pointers finally "click" because it solves the biggest problem introduced by `shared_ptr`: **ownership cycles**.

## What Is a `weak_ptr`?

A `weak_ptr` is a smart pointer that:

* Refers to an object managed by a `shared_ptr`
* Does **not** own the object
* Does **not** increase the reference count
* Can determine whether the object still exists

Think of it as:

> "I know about this object, but I am not responsible for keeping it alive."

---

# Why Do We Need It?

Suppose we have a `shared_ptr`.

```cpp
auto sp = std::make_shared<int>(42);
```

Now create another shared owner:

```cpp
auto sp2 = sp;
```

Reference count:

```text
Count = 2
```

Every `shared_ptr` extends the lifetime of the object.

Sometimes this is undesirable.

You may want to:

* Observe an object
* Check whether it still exists
* Access it if alive

without becoming an owner.

That's exactly what `weak_ptr` provides.

---

# Basic Usage

```cpp
#include <memory>

auto shared = std::make_shared<int>(100);

std::weak_ptr<int> weak = shared;
```

Memory state:

```text
shared ---> int(100)

weak -----> int(100)
```

Reference counts:

```text
Shared Count = 1
Weak Count   = 1
```

Notice:

```cpp
shared.use_count();
```

returns:

```text
1
```

The weak pointer does not contribute to ownership.

---

# Why Can't We Dereference a `weak_ptr`?

This does not compile:

```cpp
std::weak_ptr<int> weak = shared;

*weak;          // ERROR
weak->foo();    // ERROR
```

Why?

Because the object might already have been destroyed.

The compiler forces you to check.

---

# Accessing the Object with `lock()`

To use the object, convert the `weak_ptr` into a temporary `shared_ptr`.

```cpp
if (auto ptr = weak.lock())
{
    std::cout << *ptr << '\n';
}
```

`lock()` returns:

```text
shared_ptr to object   (if alive)

or

nullptr                (if destroyed)
```

This guarantees safe access.

---

# Example

```cpp
#include <iostream>
#include <memory>

int main()
{
    std::weak_ptr<int> weak;

    {
        auto shared = std::make_shared<int>(42);

        weak = shared;

        if (auto ptr = weak.lock())
        {
            std::cout << *ptr << '\n';
        }
    }

    if (auto ptr = weak.lock())
    {
        std::cout << *ptr << '\n';
    }
    else
    {
        std::cout << "Object no longer exists\n";
    }
}
```

Output:

```text
42
Object no longer exists
```

---

# Checking Expiration

A weak pointer can tell whether its object has been destroyed.

```cpp
weak.expired()
```

Example:

```cpp
if (weak.expired())
{
    std::cout << "Object is gone\n";
}
```

Equivalent to:

```cpp
weak.lock() == nullptr
```

although `lock()` is usually preferred because it avoids race conditions in multithreaded code.

---

# The Classic Shared Pointer Cycle Problem

Consider:

```cpp
class Person;
```

```cpp
class Person
{
public:
    std::shared_ptr<Person> partner;
};
```

Usage:

```cpp
auto alice = std::make_shared<Person>();
auto bob   = std::make_shared<Person>();

alice->partner = bob;
bob->partner   = alice;
```

Visual:

```text
Alice -----> Bob
  ^           |
  |           |
  +-----------+
```

Reference counts:

```text
Alice = 2
Bob   = 2
```

Now:

```cpp
alice.reset();
bob.reset();
```

What happens?

```text
Alice count = 1
Bob count = 1
```

Neither reaches zero.

Neither is destroyed.

Memory leak.

---

# Solving the Cycle with `weak_ptr`

One side should merely observe.

```cpp
class Person
{
public:
    std::weak_ptr<Person> partner;
};
```

Usage:

```cpp
auto alice = std::make_shared<Person>();
auto bob   = std::make_shared<Person>();

alice->partner = bob;
bob->partner   = alice;
```

Now:

```text
Alice count = 1
Bob count = 1
```

The weak references don't affect ownership.

When:

```cpp
alice.reset();
bob.reset();
```

both objects are destroyed correctly.

---

# Real-World Example: Parent-Child Trees

Consider a tree structure.

```text
Root
 ├── Child1
 └── Child2
```

A child often needs to know its parent.

Naive approach:

```cpp
class Node
{
public:
    std::shared_ptr<Node> parent;
    std::vector<std::shared_ptr<Node>> children;
};
```

Problem:

```text
Parent owns child
Child owns parent
```

Cycle.

Better:

```cpp
class Node
{
public:
    std::weak_ptr<Node> parent;
    std::vector<std::shared_ptr<Node>> children;
};
```

Ownership:

```text
Parent ---> Child
Child ----> Parent (weak)
```

No leak.

This is one of the most common uses of `weak_ptr`.

---

# Observer Pattern Example

Imagine a GUI button.

```cpp
class Button
{
};
```

Several systems may want to observe it.

```cpp
std::weak_ptr<Button> buttonRef;
```

The observer can safely check:

```cpp
if (auto btn = buttonRef.lock())
{
    btn->click();
}
```

without extending the button's lifetime.

---

# Understanding the Control Block

Recall from `shared_ptr`:

```text
+--------------------+
| Shared Count       |
| Weak Count         |
| Deleter            |
+--------------------+
```

Example:

```cpp
auto sp = std::make_shared<int>(10);

std::weak_ptr<int> w1 = sp;
std::weak_ptr<int> w2 = sp;
```

Counts:

```text
Shared Count = 1
Weak Count   = 2
```

Destroy `sp`:

```cpp
sp.reset();
```

Now:

```text
Shared Count = 0
Weak Count   = 2
Object destroyed
Control block remains
```

Why?

Because the weak pointers still need a way to determine:

```cpp
expired()
lock()
```

Once all weak pointers disappear:

```text
Shared Count = 0
Weak Count   = 0
```

the control block is finally destroyed.

---

# `use_count()`

A weak pointer can inspect ownership.

```cpp
weak.use_count()
```

Example:

```cpp
auto sp1 = std::make_shared<int>(10);
auto sp2 = sp1;

std::weak_ptr<int> weak = sp1;

std::cout << weak.use_count();
```

Output:

```text
2
```

Only shared owners are counted.

Weak pointers are ignored.

---

# Common Mistakes

## Mistake 1: Treating `weak_ptr` Like a Raw Pointer

Wrong:

```cpp
if (!weak.expired())
{
    // use object somehow
}
```

The object could disappear immediately afterward.

Correct:

```cpp
if (auto ptr = weak.lock())
{
    // safe
}
```

---

## Mistake 2: Using `shared_ptr` Everywhere

Bad:

```cpp
class Child
{
    std::shared_ptr<Parent> parent;
};
```

Often:

```cpp
std::weak_ptr<Parent>
```

is the correct design.

---

## Mistake 3: Forgetting That `lock()` Creates Ownership

```cpp
auto ptr = weak.lock();
```

This temporarily increases the shared count.

```text
Shared Count +1
```

until `ptr` goes out of scope.

---

# Typical Use Cases

## Parent/Child Relationships

```cpp
Parent -> shared_ptr<Child>

Child -> weak_ptr<Parent>
```

---

## Observer Pattern

```cpp
Observer -> weak_ptr<Subject>
```

---

## Caches

Objects can disappear naturally.

```cpp
std::weak_ptr<Texture>
```

lets you know whether a texture is still loaded.

---

## Event Systems

Listeners often hold weak references to publishers.

---

# Comparison of Smart Pointers

| Feature            | unique_ptr | shared_ptr | weak_ptr |
| ------------------ | ---------- | ---------- | -------- |
| Owns Object        | Yes        | Yes        | No       |
| Copyable           | No         | Yes        | Yes      |
| Movable            | Yes        | Yes        | Yes      |
| Reference Count    | No         | Yes        | No       |
| Keeps Object Alive | Yes        | Yes        | No       |
| Can Break Cycles   | N/A        | No         | Yes      |
| Direct Dereference | Yes        | Yes        | No       |

---

# Mental Model

Think of the three smart pointers as different relationships:

```text
unique_ptr
==========
"I am the sole owner."


shared_ptr
==========
"We jointly own it."


weak_ptr
========
"I know it exists,
but I don't own it."
```

That's the essence of `weak_ptr`: it gives you a safe way to **observe a shared object without participating in its ownership**, making it the primary tool for breaking reference cycles and modeling non-owning relationships in modern C++.











