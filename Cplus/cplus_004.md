# Function Arguments and Return values 

## Handling Temporary arguments

Objects created without a name. They are typically used to pass arguments to functions or return values, and they die at the end of the statement (the semicolon) unless their lifetime is extended by a reference.

```cpp
// Created, passed to print(), and destroyed immediately after print() finishes
print(Person{"narayan"}); 
```

When you pass a temporary object (a prvalue) like `Person{"narayan"}` into a function like `print(...)`, how the function uses it depends entirely on how the function's parameter is defined.

Here are the three ways `print` can accept that temporary, ordered from the most common to the most specialized.

---

#### 3.1 By Constant Reference: `void print(const Person& p)` *(Most Common)*

This is the standard, idiomatic way to pass objects in C++ to avoid unnecessary copying.

```cpp
void print(const Person& p) {
    std::cout << p.name << "\n";
}

// Usage:
print(Person{"narayan"});

```

* **What happens:** As we discussed earlier, the temporary object binds directly to the reference `p`.
* **Lifetime:** The temporary object's lifetime is extended for the entire duration of the `print` function. Once `print` finishes executing and returns, the temporary is immediately destroyed.
* **Performance:** High. Zero copies, zero moves.

---

#### 3.2 By Value: `void print(Person p)`

This creates a brand-new, independent copy or move inside the function.

```cpp
void print(Person p) {
    std::cout << p.name << "\n";
}

// Usage:
print(Person{"narayan"});

```

* **What happens (C++17 and later):** Thanks to **Guaranteed Copy Elision**, the compiler does *not* create a temporary outside the function and copy it in. Instead, it constructs the `Person` object **directly inside the function's parameter memory (`p`)**.
* **What happens (C++11/14):** The temporary is created, and then it is **moved** into `p` using the move constructor (because a temporary is an rvalue).
* **Lifetime:** The object `p` lives until the `print` function hits its closing brace `}`, at which point `p` is destroyed.

---

#### 3.3 By Rvalue Reference: `void print(Person&& p)` *(Advanced)*

This explicitly tells the function: *"I am giving you a temporary object, and you are allowed to steal its resources (move from it) if you want to."*

```cpp
void print(Person&& p) {
    // You can "steal" the string data out of p into another variable
    std::string local_name = std::move(p.name); 
    std::cout << local_name << "\n";
}

// Usage:
print(Person{"narayan"});

```

* **What happens:** The reference `p` binds directly to the temporary object. No copy or move happens *just by passing it*. However, inside the function, you have the explicit right to use `std::move` on `p`.
* **Lifetime:** Just like the `const Person&` case, the temporary lives until the `print` function finishes executing.


This is where it's useful to separate the three cases.

---
# Returning from functions 
## Case 1: Returning a named local variable

```cpp
X f() {
    X x;
    return x;
}
```

### Before C++17

The compiler **may** perform **NRVO** (Named Return Value Optimization).

If NRVO happens:

* no copy
* no move
* `x` is constructed directly in the caller.

If NRVO does **not** happen:

* move if available
* otherwise copy.

### C++17 and later

Exactly the same.

C++17 **did not make NRVO mandatory**.

So:

* NRVO may occur (most optimizing compilers do it)
* if it doesn't, move is preferred over copy.

---

## Case 2: Returning a temporary (unnamed object)

```cpp
X f() {
    return X{};
}
```

### Before C++17

Copy elision was optional.

If the compiler elided:

* no copy
* no move.

Otherwise:

* move if available
* otherwise copy.

### C++17 and later

This changed completely.

The standard says the object is constructed **directly** in the caller.

So

```cpp
return X{};
```

invokes

* no copy constructor
* no move constructor

even if both are deleted.

Example:

```cpp
struct X {
    X() = default;
    X(const X&) = delete;
    X(X&&) = delete;
};

X f() {
    return X{};
}
```

This is perfectly valid in C++17+.

---

## Case 3: Returning a function parameter

```cpp
X f(X x) {
    return x;
}
```

NRVO **cannot** apply here because `x` wasn't created inside the function—it already existed as the parameter object.

Therefore:

* move if available
* otherwise copy

There is no "construct directly in caller" optimization like there is for `return X{}`.

---

## Summary

| Return statement                                    | Pre-C++17                                   | C++17+                                     |
| --------------------------------------------------- | ------------------------------------------- | ------------------------------------------ |
| `return x;` (local variable)                        | NRVO if possible; otherwise move, else copy | Same                                       |
| `return X{};`                                       | Optional elision; otherwise move/copy       | **Guaranteed copy elision** (no move/copy) |
| `return makeX();` where `makeX()` returns a prvalue | Optional elision                            | **Guaranteed copy elision**                |
| `return x;` (parameter)                             | Move, else copy                             | Move, else copy                            |

### The key idea

C++17's "guaranteed copy elision" applies only when the returned expression is a **prvalue**, such as:

```cpp
return X{};
return makeX();   // if makeX() returns by value
```

It **does not** apply to **named objects**, whether they're local variables or function parameters:

```cpp
return x;   // x is named
```

For those, the compiler may perform NRVO (locals only), and if that doesn't happen, it prefers the move constructor over the copy constructor when possible.# C++ RVO NRVO Copy Ellison

In C++, copying or moving objects can be expensive, especially for large structures or containers. To mitigate this, compilers use optimization techniques known collectively as **Copy Elision**.

**Copy Elision** is a compiler optimization technique where the compiler avoids creating temporary copies of objects, even if the copy/move constructor has side effects.

Two of the most common forms of copy elision are **Return Value Optimization (RVO)** and **Named Return Value Optimization (NRVO)**.

---

## 1. Return Value Optimization (RVO)

RVO occurs when a function returns an **unnamed temporary object (a prvalue)**. Instead of creating a temporary object inside the function and then copying or moving it into the caller's destination variable, the compiler constructs the object directly inside the memory allocated for the caller's variable.

### Example:

C++

```cpp
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
    std::cout << "Print the message" << std::endl;
}
```

### What happens under the hood:

Without RVO, the sequence would be:

1. Construct a temporary `MyObject` inside `createObject()`.
2.  Copy/Move that temporary to return value object
3. Destroy the temporary.
4. Copy/Move that return value temporary obj  into `obj` in `main()`.
5. Destroy the return value temporary object.
6.  Print the message
7. destroy obj 
    

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

```cpp
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

```cpp
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

```cpp
MyObject avoidThis() {
    MyObject localObj;
    return std::move(localObj); // ❌ Pessimization!
}
```

**Why is this bad?** `std::move(localObj)` turns the expression into an rvalue reference(an xvalue `MyObject&&`) . RVO and NRVO specifically look for a _prvalue_ (temporary) or an _lvalue_ (named local variable). By forcing a move, you **completely disable NRVO**. Instead of constructing the object directly in place, the compiler is now forced to construct `localObj` locally and then move it into the destination. Always just return the variable by name!