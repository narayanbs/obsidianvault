In C, **`#error`** is a **preprocessor directive**. It’s used to **intentionally stop compilation** and show a **custom error message**.

Think of it as: *“If this condition happens, refuse to compile and tell the programmer why.”* 🛑

---

### Basic syntax

```c
#error "Your error message here"
```

When the compiler hits this line, it immediately fails and prints the message.

---

### Simple example

```c
#error "This program should not be compiled"
```

Compilation stops with an error like:

```
error: This program should not be compiled
```

---

### Common real-world use (with `#if`)

`#error` is usually paired with conditional compilation.

#### Example: checking a macro

```c
#ifndef MY_CONFIG
#error "MY_CONFIG is not defined"
#endif
```

If `MY_CONFIG` isn’t defined, the compiler stops and tells you exactly what’s wrong.

---

### Example: platform checks

```c
#if defined(_WIN32)
#error "Windows is not supported"
#endif
```

This prevents compiling on unsupported systems.

---

### Example: C standard version check

```c
#if __STDC_VERSION__ < 201112L
#error "C11 or later is required"
#endif
```

---

### Key points

* `#error` runs **before compilation** (preprocessor stage)
* It **forces a compile-time error**
* Useful for:

  * Enforcing configuration rules
  * Catching unsupported platforms
  * Giving clear developer-friendly messages
* It does **not** exist at runtime

---
## `#error` vs `assert()` (important difference)

### `#error`

* **Compile-time**
* Stops the program **before it’s built**
* Used for **configuration / environment checks**

```c
#if !defined(DEBUG)
#error "DEBUG must be defined"
#endif
```

👉 The program will **not compile at all**.

---

### `assert()`

* **Runtime**
* Only triggers **while the program is running**
* Used for **logic checks**

```c
#include <assert.h>

int divide(int a, int b) {
    assert(b != 0);
    return a / b;
}
```

👉 Program compiles fine, but **crashes at runtime** if `b == 0`.

---

### Quick comparison

| Feature           | `#error`               | `assert()`    |
| ----------------- | ---------------------- | ------------- |
| When it runs      | Compile time           | Runtime       |
| Stops compilation | ✅                      | ❌             |
| Can be disabled   | ❌                      | ✅ (`NDEBUG`)  |
| Use case          | Config/platform checks | Bug detection |

---

## Portable `#error` checks (best practice)

### 1️⃣ Check OS

```c
#if !defined(__linux__) && !defined(_WIN32)
#error "Unsupported operating system"
#endif
```

---

### 2️⃣ Check compiler

```c
#ifndef __GNUC__
#error "GCC compiler required"
#endif
```

---

### 3️⃣ Check word size

```c
#if UINTPTR_MAX != 0xFFFFFFFF && UINTPTR_MAX != 0xFFFFFFFFFFFFFFFF
#error "Unsupported pointer size"
#endif
```

---

### 4️⃣ Check feature macros

```c
#ifndef __STDC_NO_THREADS__
#error "C11 threads are required"
#endif
```

---

## Pro tip ⚡ (cleaner alternative)

Some compilers support `#pragma error`, but **`#error` is standard C** and portable—always prefer it.

---

## When you SHOULD use `#error`

✔ Enforcing compiler requirements
✔ Preventing wrong platform builds
✔ Mandatory macro checks
✔ Library configuration errors

## When you should NOT use it

❌ Normal program logic
❌ User input validation
❌ Runtime conditions

---


