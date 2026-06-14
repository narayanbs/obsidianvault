# Notes

---
### Note 1

#### Object layout, Static and dynamic binding in inheritances, Virtual table


 **statically bound methods  are bound at compile time, the compiler doesn't need to look at the object to find them. Instead, it hardcodes a direct jump to the function's address in the code segment.

 Let's look at  **offsets**  in the case of  *member variables* and the *vptr*
 
 A `Derived` object is structured so that its `Base` part sits right at the very beginning.

Let’s look at exactly how static vs. dynamic binding works, complete with code and memory layouts.

---

## 1. The Code Example

Let's set up a `Base` and `Derived` class. `Base` has one normal (static) method, one virtual (dynamic) method, and one integer. `Derived` overrides both.

```cpp
class Base {
public:
    int base_data = 10;
    
    void normal_func() {}      // Static binding
    virtual void virt_func() {} // Dynamic binding
};

class Derived : public Base {
public:
    int dev_data = 20;
    
    void normal_func() {}      // Hides Base::normal_func (Static)
    void virt_func() override {} // Overrides Base::virt_func (Dynamic)
};

```

---

## 2. Memory Layout: The Offset Principle

When you instantiate a `Derived` object, the compiler ensures that the **`Base` sub-object is placed at offset 0**.

Because `Base` has a virtual function, it gets a `vptr`. Therefore, the `vptr` is placed at the absolute beginning of the object.

### Memory Layout of `Derived obj;`

| Offset (Bytes) | What is stored here? | Belonging to... |
| --- | --- | --- |
| **0 - 7** | `vptr` (Points to `Derived` Vtable) | `Base` part (at Offset 0) |
| **8 - 11** | `int base_data` | `Base` part |
| **12 - 15** | `int dev_data` | `Derived` part |

Because the `Base` part sits at offset 0, **a pointer/reference to `Base` and a pointer/reference to `Derived` point to the exact same memory address.**

```cpp
Derived dev;
Base* base_ptr = &dev; 
// base_ptr points to the exact same byte address as &dev!

```

---

## 3. How the Compiler Invokes the Methods

Now, let's look at what happens under the hood when you use a `Base&` reference to call these methods.

```cpp
Derived derived_obj;
Base &ref = derived_obj;

```

### Scenario A: Static Binding (`ref.normal_func()`)

The compiler sees that `normal_func()` is not virtual. It doesn't look at the object's memory, its `vptr`, or its offsets. It only looks at the *type* of `ref` (which is `Base`).

It translates `ref.normal_func()` directly into assembly that says:

> *"Jump straight to the hardcoded memory address of `Base::normal_func`."*

The fact that `derived_obj` has its own `normal_func` is completely ignored because static binding happens at compile time.

### Scenario B: Dynamic Binding (`ref.virt_func()`)

The compiler sees that `virt_func()` is virtual. It cannot hardcode an address. Instead, it generates code to look inside the object at runtime:

1. **Go to Offset 0:** It looks at the reference's address (which points to offset 0 of the object).
2. **Dereference the `vptr`:** It reads the 8-byte pointer sitting at offset 0. Because the actual object created was a `Derived` object, this `vptr` points to the **`Derived` Vtable**.
3. **Look up the slot:** It looks up the slot reserved for `virt_func()` in that vtable.
4. **Jump:** The vtable slot contains the address of `Derived::virt_func()`. The program jumps there.

---

## To Summarize Your Offset Intuition

You were entirely right to think about offsets. Because the compiler enforces that a `Derived` object starts with its `Base` components:

* The **`vptr`** is always at offset 0 for both `Base` and `Derived`.
* The **`base_data`** is always at the exact same offset (offset 8) whether the compiler thinks it's looking at a `Base` or a `Derived`.

This uniform layout is the exact secret sauce that allows a `Base` reference to seamlessly interact with a `Derived` object without needing to know how big the `Derived` class actually is!


### But what if Base doesn't have virtual methods and derived has virtual methods? Is this allowed. How is the layout now?

Yes, absolutely! You can 100% have a base class with no virtual methods, and a derived class that introduces virtual methods. It is completely valid C++ code.

However, doing this alters the memory layout in a fascinating way and introduces a massive runtime danger that every C++ developer needs to watch out for.

Here is exactly how the compiler handles this, how it impacts memory, and the "gotcha" you need to avoid.

---

## 1. What the Memory Layout Looks Like

When a base class has no virtual methods, it doesn't have a `vptr`. If the derived class introduces a virtual method, the compiler *must* inject a `vptr` somewhere.

Because the `Base` class parts are usually kept clean and tightly packed at the beginning of the object for slicing and casting purposes, the compiler typically appends the `vptr` right after the base class data, or restructures the layout.

Let's look at this code:

```cpp
class Base {
public:
    int base_data = 10;
    void normal() {} // No virtual methods here!
};

class Derived : public Base {
public:
    int dev_data = 20;
    virtual void new_virt_func() {} // Derived introduces polymorphism
};

```

### The Resulting Memory Layout

Because `Base` has no `vptr`, a pure `Base` object is just 4 bytes (the size of `base_data`). But when you instantiate `Derived`, the compiler injects a `vptr` specifically for the `Derived` parts:

| Offset (Bytes) | What is stored here? | Belonging to... |
| --- | --- | --- |
| **0 - 3** | `int base_data` | `Base` part (at Offset 0) |
| **4 - 11** | `vptr` (Points to `Derived` Vtable) | injected by `Derived` (alignment padding may apply) |
| **12 - 15** | `int dev_data` | `Derived` part |

Notice how different this is from our previous example. **The `vptr` is no longer at offset 0.** ---

## 2. Calling the Virtual Method

If you have a `Derived` reference, calling the virtual function is straightforward:

```cpp
Derived d;
d.new_virt_func(); // Works perfectly. The compiler knows exactly where the vptr is.

```

But what happens if you try to call it through a `Base` reference?

```cpp
Derived d;
Base &ref = d;
// ref.new_virt_func(); // COMPILE ERROR!

```

This fails to compile because the `Base` class has no idea `new_virt_func()` even exists.

---

## 3. The Dangerous "Gotcha": Undefined Behavior

While this inheritance pattern is allowed, it introduces a classic C++ trap involving **object destruction**.

If a base class does not have a virtual method, it means it also does **not** have a `virtual ~Base()` destructor. If you ever delete a `Derived` object through a `Base` pointer, the compiler will perform static binding on the destructor.

```cpp
Base* ptr = new Derived(); // Valid upcast

delete ptr; // WARNING: UNDEFINED BEHAVIOR!

```

### What goes wrong here?

1. The compiler looks at `ptr` and sees its type is `Base*`.
2. Because `Base` has no virtual methods, the compiler uses **static binding** and calls `Base::~Base()`.
3. The `Derived` destructor (`Derived::~Derived()`) is **never called**.
4. Any memory or resources managed exclusively by the `Derived` class will leak, and because the memory layout sizes don't match up perfectly, it often results in a corrupted heap or an immediate crash.

---

## Summary Rule of Thumb

* Can you do it? **Yes.**
* Is the memory layout different? **Yes, the `vptr` is shifted down because the `Base` part doesn't have one.**
* Should you do it? **Only if you are 100% sure you will never delete a derived object via a base pointer.** As a golden rule in C++, if you intend for a class to be used as a base class polymorphically (meaning you'll handle it via base pointers/references), **always give the base class a `virtual` destructor**, even if it has no other virtual methods!

---

### Note 2

A **virtual destructor** is your primary defense mechanism against memory leaks and undefined behavior when using polymorphism in C++.

To understand how they help, it is best to look at a horror story of what happens *without* them, and then see how adding that one single keyword fixes everything.

---

## The Scenario: A Polymorphic Resource Manager

Imagine you have a `Base` class that represents a generic connection, and a `Derived` class that opens a specific network socket and allocates a large buffer on the heap.

```cpp
#include <iostream>

class Base {
public:
    Base() { std::cout << "Base Constructed\n"; }
    ~Base() { std::cout << "Base Destructed\n"; } // NOT VIRTUAL!
};

class Derived : public Base {
private:
    int* large_buffer;
public:
    Derived() { 
        std::cout << "Derived Constructed\n"; 
        large_buffer = new int[1000]; // Allocating heap memory
    }
    ~Derived() { 
        std::cout << "Derived Destructed\n"; 
        delete[] large_buffer; // Cleaning up heap memory
    }
};

```

Now, let's use these classes standard polymorphically—creating a `Derived` object but managing it through a `Base` pointer:

```cpp
int main() {
    Base* ptr = new Derived(); 
    
    std::cout << "--- Cleaning up ---\n";
    delete ptr; 
}

```

---

## 1. What Happens WITHOUT a Virtual Destructor?

If you run the code above, the output will look like this:

```text
Base Constructed
Derived Constructed
--- Cleaning up ---
Base Destructed

```

### Notice what is missing?

`Derived Destructed` was **never printed**. This means `delete[] large_buffer;` never executed. You have just leaked 4,000 bytes of memory.

### Why did this happen?

It comes back to **Static Binding**. Because the destructor in `Base` is not marked `virtual`, the compiler looks at the type of `ptr` (which is `Base*`) and says: *"Okay, at compile-time, I am hardcoding a jump straight to `Base::~Base()`."* The compiler completely ignores the fact that `ptr` actually points to a `Derived` object. The `Derived` destructor is skipped entirely.

---

## 2. How a Virtual Destructor Fixes It

To fix this, we change exactly one line of code in the `Base` class by adding the `virtual` keyword:

```cpp
class Base {
public:
    Base() { std::cout << "Base Constructed\n"; }
    virtual ~Base() { std::cout << "Base Destructed\n"; } // NOW VIRTUAL!
};

```

If we run the exact same `main()` function now, the output changes perfectly:

```text
Base Constructed
Derived Constructed
--- Cleaning up ---
Derived Destructed
Base Destructed

```

### How does it help under the hood?

By making `~Base()` virtual, the compiler enters the destructor's address into the class's **vtable**.

When `delete ptr;` is called:

1. The compiler notices the destructor is `virtual`.
2. It looks at the object's `vptr` at runtime.
3. The `vptr` points to the `Derived` vtable, so it fetches the address for the **`Derived` destructor** first.
4. `Derived::~Derived()` executes, successfully freeing `large_buffer`.
5. Once the derived destructor finishes, it automatically chains upward and calls the `Base` destructor to clean up the rest of the object.

---

## The Golden Rules of Destructors

To keep your code safe, C++ developers rely on two simple rules of thumb regarding destructors:

1. **The Polymorphic Rule:** If a class has *even one* virtual function, it is intended to be used polymorphically. Therefore, you should **always** give it a virtual destructor.
```cpp
virtual ~Base() = default; // If you don't need custom logic, let the compiler make it

```


2. **The Final Rule:** If a class is *not* meant to be inherited from, do not make the destructor virtual (to avoid paying the 8-byte `vptr` penalty). You can mark the class as `final` to prevent anyone from accidentally inheriting from it.

Does seeing how the destruction sequence chains from derived to base clarify why the `virtual` keyword is so vital here?


--------------------------

## Question

~~~
in c++ if we have the following classes. what would be the memory layout

class Base {
public:
    int base_data = 10;
    
    void normal_func() {}      // Static binding
   
};

class Derived : public Base {
public:
    int base_data = 20;
   
};
~~~

For these classes:

```cpp
class Base {
public:
    int base_data = 10;

    void normal_func() {}
};

class Derived : public Base {
public:
    int base_data = 20;
};
```

there is **no virtual function**, so neither class contains a vtable pointer (`vptr`).

### Sizes

On a typical system where `int` is 4 bytes:

```cpp
sizeof(Base)    == 4
sizeof(Derived) == 8
```

because a `Derived` object contains:

1. The complete `Base` subobject.
2. Its own member `base_data`.

### Memory layout

For:

```cpp
Derived d;
```

the memory typically looks like:

```
Address
+------------------+
| Base::base_data  | 10
+------------------+
| Derived::base_data| 20
+------------------+
```

or more explicitly:

```
d
┌─────────────────────┐
│ Base subobject      │
│   base_data = 10    │
├─────────────────────┤
│ Derived member      │
│   base_data = 20    │
└─────────────────────┘
```

### Accessing the members

Because the derived class declares another member with the same name, it **hides** the base member:

```cpp
Derived d;

d.base_data;          // Derived::base_data (20)
d.Base::base_data;    // Base::base_data (10)
```

### Addresses

You can observe the layout:

```cpp
Derived d;

std::cout << &d << '\n';
std::cout << &d.Base::base_data << '\n';
std::cout << &d.base_data << '\n';
```

Typical output (conceptually):

```
0x1000   // &d
0x1000   // Base::base_data
0x1004   // Derived::base_data
```

The base subobject starts at the beginning of the derived object, and the derived member follows it.

### What about `normal_func()`?

```cpp
void normal_func() {}
```

is a non-virtual member function. It is **not stored inside each object**. There is only one copy of the function's machine code in the program's text segment, and objects do not contain any pointer to it.

So the object memory contains only:

```
Base object:
    int base_data

Derived object:
    Base::base_data
    Derived::base_data
```

and no extra storage for `normal_func()`.

## Question

~~~
when we create a normal function like the following
void normal_func() {
base_data = 40;
}

and in main
Derived d{};
d.normal_func();

which base_data is updated
~~~


Given:

```cpp
class Base {
public:
    int base_data = 10;

    void normal_func() {
        base_data = 40;
    }
};

class Derived : public Base {
public:
    int base_data = 20;
};
```

and:

```cpp
int main() {
    Derived d{};
    d.normal_func();
}
```

the **Base::base_data** member is updated.

After the call:

```cpp
d.Base::base_data == 40
d.base_data       == 20
```

### Why?

`normal_func()` is defined inside `Base`, so when the compiler sees:

```cpp
base_data = 40;
```

inside that function, it resolves `base_data` to **Base::base_data** at compile time.

Conceptually, the function behaves like:

```cpp
void Base::normal_func(Base* this) {
    this->Base::base_data = 40;
}
```

When you call:

```cpp
d.normal_func();
```

the `this` pointer points to the `Base` subobject within `d`, and the generated code accesses the `base_data` member at the offset corresponding to `Base::base_data`.

### Visualizing the object

Before:

```text
d
┌─────────────────────┐
│ Base::base_data = 10│
├─────────────────────┤
│ Derived::base_data=20│
└─────────────────────┘
```

After:

```text
d
┌─────────────────────┐
│ Base::base_data = 40│
├─────────────────────┤
│ Derived::base_data=20│
└─────────────────────┘
```

### If the function were virtual?

Even then, unless `Derived` overrides the function, the implementation in `Base` would still execute and update `Base::base_data`.

The important distinction is:

* **Member lookup inside a function body** is based on the class where the function is defined.
* **Virtual dispatch** only decides *which function implementation runs*, not which member name a particular implementation refers to.

So in your example, `Base::base_data` is the one that changes.


### Additional Note

~~~
template <typename T>
struct SimpleUniquePointer {
  SimpleUniquePointer() = default;

  SimpleUniquePointer(T *pointer) : pointer(pointer) {}

  ~SimpleUniquePointer() {
    if (pointer) delete pointer;
  }

  SimpleUniquePointer(const SimpleUniquePointer &) = delete;
  SimpleUniquePointer &operator=(const SimpleUniquePointer &) = delete;

  SimpleUniquePointer(SimpleUniquePointer &&other) noexcept : pointer{other.pointer} {
    other.pointer = nullptr;
  }

  SimpleUniquePointer &operator=(SimpleUniquePointer &&other) noexcept {
    if (pointer) delete pointer;
    pointer = other.pointer;
    other.pointer = nullptr;
    return *this;
  }
};
~~~
Suppose we have the class above, we can instantiate the instance as shown below
~~~
auto ptr_a = SimpleUniquePointer(new int{20});
~~~
In older versions of C++, this would have created a temporary object and then invoked the **move constructor** to transfer it into `ptr_a`.

However, in modern C++ (specifically since **C++17**), a special compiler optimization rule called **Mandatory Copy Elision** (often referred to as _Guaranteed PRValue Baldness_ or _Copy Omission_) kicks in.

### What the Compiler Actually Does

Even though it _looks_ like a temporary is being created and moved, the C++17 standard strictly forbids the compiler from actually creating a temporary here.

Instead, the compiler treats that line exactly as if you had written:

~~~
SimpleUniquePointer ptr_a{new int{20}};
~~~
### How it Works Behind the Scenes

1. **The Conceptual View:** You see a temporary being constructed on the RHS, which is then used to initialize `ptr_a` on the LHS.
    
2. **The Compiler's Reality:** The compiler looks at the target destination (`ptr_a`) and constructs the object **directly inside `ptr_a`'s memory space** from the very beginning.
    

Because of this, **zero moves and zero copies take place**. The move constructor you wrote in your class isn't even called! In fact, this optimization is so powerful that this line would compile perfectly even if you completely deleted (`= delete`) your move constructor.