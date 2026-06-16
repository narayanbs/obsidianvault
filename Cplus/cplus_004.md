# C++ RVO NVRO Copy Ellison

In C++, copying or moving objects can be expensive, especially for large structures or containers. To mitigate this, compilers use optimization techniques known collectively as **Copy Elision**.

**Copy Elision** is a compiler optimization technique where the compiler avoids creating temporary copies of objects, even if the copy/move constructor has side effects.

Two of the most common forms of copy elision are **Return Value Optimization (RVO)** and **Named Return Value Optimization (NRVO)**.

---

## 1. Return Value Optimization (RVO)

RVO occurs when a function returns an **unnamed temporary object (a prvalue)**. Instead of creating a temporary object inside the function and then copying or moving it into the caller's destination variable, the compiler constructs the object directly inside the memory allocated for the caller's variable.

### Example:

C++

```
#include <iostream>

struct MyObject {
    MyObject() { std::cout << "Constructed\n"; }
    MyObject(const MyObject&) { std::cout << "Copied\n"; }
    MyObject(MyObject&&) noexcept { std::cout << "Moved\n"; }
};

MyObject createObject() {
    return MyObject(); // Returning an unnamed temporary (RVO)
}

int main() {
    MyObject obj = createObject();
}
```

### What happens under the hood:

Without RVO, the sequence would be:

1. Construct a temporary `MyObject` inside `createObject()`.
    
2. Copy/Move that temporary into `obj` in `main()`.
    
3. Destruct the temporary.
    

With **RVO**, the compiler secretly passes the address of `obj` from `main()` into `createObject()`. The function then constructs the object **directly inside `obj`'s memory slot**.

**Output (with RVO):**

Plaintext

```
Constructed
```

_(Notice that "Copied" or "Moved" is never printed)._

---

## 2. Named Return Value Optimization (NRVO)

NRVO is similar to RVO, but it applies when the function returns a **named local variable (an lvalue)** instead of an unnamed temporary.

Because the variable has a name, the compiler has to be a bit smarter. It must ensure that the local variable can be completely bypassed and mapped directly to the caller's return destination.

### Example:

C++

```
MyObject createNamedObject() {
    MyObject localObj; // Named local variable
    // Do some work with localObj...
    return localObj;   // NRVO triggers here
}

int main() {
    MyObject obj = createNamedObject();
}
```

### What happens under the hood:

Just like RVO, the compiler allocates space for `obj` in `main()`'s stack frame and passes its address to `createNamedObject()`. Inside the function, `localObj` becomes an alias for that memory.

**Output (with NRVO):**

Plaintext

```
Constructed
```

> ⚠️ **Note on NRVO:** Unlike RVO, NRVO is _not_ guaranteed by the C++ standard. It is highly optimized by modern compilers (GCC, Clang, MSVC), but if your function has multiple complex execution paths with different named variables being returned, the compiler might fail to apply NRVO and fall back to a move or copy.

---

## 3. Copy Elision and C++ Standard Evolution

The relationship between Copy Elision, RVO, and NRVO has evolved significantly over different C++ standards:

|**C++ Standard**|**RVO Status**|**NRVO Status**|
|---|---|---|
|**C++11 / C++14**|**Optional.** Compilers were allowed to do it, but if they didn't, a move/copy constructor had to exist.|**Optional.** Compiler-dependent optimization.|
|**C++17 and later**|**Mandatory.** It is no longer just an "optimization"—the language definition guarantees no copy/move happens for prvalues.|**Optional.** Still a highly-optimized compiler choice.|

Because RVO is **guaranteed** since C++17, you can even return objects that have completely deleted copy and move constructors!

C++

```
struct NonCopyableNonMovable {
    NonCopyableNonMovable() = default;
    NonCopyableNonMovable(const NonCopyableNonMovable&) = delete;
    NonCopyableNonMovable(NonCopyableNonMovable&&) = delete;
};

NonCopyableNonMovable make() {
    return NonCopyableNonMovable(); // Perfect RVO, compiles fine in C++17+
}
```

---

## 💡 A Common Pitfall: Don't explicit `std::move` on returns!

A very common mistake is trying to "help" the compiler by wrapping the return value in `std::move()`.

C++

```
MyObject avoidThis() {
    MyObject localObj;
    return std::move(localObj); // ❌ Pessimization!
}
```

**Why is this bad?** `std::move(localObj)` turns the expression into an rvalue reference(an xvalue `MyObject&&`) . RVO and NRVO specifically look for a _prvalue_ (temporary) or an _lvalue_ (named local variable). By forcing a move, you **completely disable NRVO**. Instead of constructing the object directly in place, the compiler is now forced to construct `localObj` locally and then move it into the destination. Always just return the variable by name!