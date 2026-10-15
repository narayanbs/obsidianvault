# Lvalues, Rvalues, Prvalues, Xvalues, and Lifetime Extension in C++

**Every expression has three important properties**:

1. **Type** (what kind of value it represents)
2. **Value category** (how it behaves: lvalue, xvalue, prvalue)
3. **Value** (if the expression is evaluated, what result it produces)

Here is exactly how that breaks down for `x = 20` (assuming `x` is an `int`):

### The Breakdown

The assignment operator for built-in types is specified roughly as
```c++
int& operator=(int);
```

* **Value:** `20`
The expression evaluates to the value that was just assigned to the variable.
* **Type:** `int&` (an lvalue reference to `int`)
It doesn't just return a copy of the number 20; it returns the actual memory location of `x`.
* **Value Category**: lvalue

Because the type is a reference (`int&`), the **value category** of this expression is an **lvalue**. This is a fancy C++ term meaning the result of the expression refers to a persistent object in memory that you can take the address of.

---
### A Quick Visual Comparison

To see why this is unique, contrast the **assignment operator** with the **arithmetic operator**:

| Expression | Evaluated Value    | Resulting Type / Category | Can you assign to it?                            |
| ---------- | ------------------ | ------------------------- | ------------------------------------------------ |
| `x = 20`   | `20`               | `int&` (lvalue)           | **Yes** `(x = 20) = 50;` is valid.               |
| `x + 20`   | `40` (if x was 20) | `int` (rvalue)            | **No** `(x + 20) = 50;` causes a compiler error. |

Because `x + 20` returns a temporary, fleeting value (an rvalue), you can't assign anything to it. But because `x = 20` returns the actual variable `x` (an lvalue), it is perfectly ready to receive another assignment.


## Value categories

Understanding C++ value categories is essential for move semantics, perfect forwarding, references, and modern C++ optimization techniques.

The terminology evolved significantly in C++11. Before C++11, programmers mainly talked about **lvalues** and **rvalues**. Modern C++ refines these into a hierarchy:

```
                 Expression
                      |
         +------------+------------+
         |                         |
      glvalue                    rvalue
         |                         |
   +-----+-----+             +-----+-----+
   |           |             |           |
 lvalue      xvalue       prvalue     xvalue
```

More precisely:

* **glvalue** = generalized lvalue
  * lvalue
  * xvalue  (expiring lvalue)
  
* **rvalue**
  * prvalue  (pure rvalue)
  * xvalue (expiring rvalue) 

---

# 1. Lvalues

An **lvalue** is an expression that refers to a persistent object with an identifiable location in memory.

### Examples

```cpp
int x = 10;

x;      // lvalue
++x;    // lvalue
*xptr;  // lvalue
arr[0]; // lvalue
```

### Characteristics

* Has identity (you can take its address).
* Usually persists beyond the current expression.
* Can appear on the left side of assignment.

```cpp
x = 20;   // valid
```

### Why "lvalue"?

Historically, because it could appear on the **left-hand side** of an assignment:

```cpp
x = 5;
```

Although not every lvalue is assignable:

```cpp
const int c = 10;

c = 20;   // error
```

---

# 2. Rvalues

An **rvalue** is a temporary value that does not have a persistent identity.

### Examples

These are temporary values.

```cpp
42
x + y
int foo()
std::string("hello")
```


```cpp
int x = 5;

x + 1 = 10;  // error
```
The result of `x + 1` is an rvalue.


### Characteristics

* Usually temporary.
* Cannot generally have their address taken.
* Often eligible for moving.

---

# 3. Lvalue References (`T&`)

A non-const lvalue reference can bind only to lvalues.

```cpp
int x = 10;

int& ref = x;      // OK

int& ref2 = 42;    // Error
```

Because `42` is an rvalue.

---

# 4. Const Lvalue References (`const T&`)

A const lvalue reference can bind to both lvalues and rvalues.

```cpp
int x = 10;

const int& r1 = x;   // OK
const int& r2 = 42;  // OK

similarly 

void print(const std::string& s) {
    std::cout << s << '\n';
}

print(std::string("hello"));  // temporary object

```
	
This is one of the mechanisms that enables **lifetime extension**.

---

# 5. Rvalue References (`T&&`)

Introduced in C++11.

Can bind to rvalues.

```cpp
int&& r = 42;    // OK

int x = 10;

int&& r2 = x;    // Error
```

### Example

```cpp
std::string&& s = std::string("hello");
```

The temporary string is bound to an rvalue reference.

Rvalue references enable:

* Move semantics
* Perfect forwarding
* Efficient resource transfer

---

**To summarize, C++ uses the rule:**

* __T& (non-const lvalue reference)__ → binds only to lvalues.
* __const T&__ → binds to lvalues and rvalues.
* __T&& (rvalue reference)__ → binds to rvalues.

-----

# 6. Prvalues

**Prvalue** means **pure rvalue**.

A prvalue represents a value that is not yet associated with an object identity.

### Examples

```cpp
42

x + y

std::string("hello")

int foo()
```

```cpp
int x = 2;
int y = 3;

auto z = x + y;
```

The expression `x + y` is a prvalue.

### Key idea

A prvalue is "just a value".

It may later be used to initialize an object.

```cpp
std::string s = std::string("hello");
```

The temporary created by the prvalue initializes `s`.

### Since C++17

Prvalues no longer necessarily create temporary objects immediately.

The standard introduced the idea of **temporary materialization**.

```cpp
T x = T();
```

The object can be constructed directly into `x`.

This is related to guaranteed copy elision.

---

# 7. Xvalues

**Xvalue** means **eXpiring value**.

An xvalue is a glvalue whose resources can be reused.

Think:

> "This object still has identity, but it is about to die."

### Examples

```cpp
std::move(x)
```

```cpp
std::string s = "hello";

std::move(s);
```

The expression:

```cpp
std::move(s)
```

is an xvalue.

### Why?

Because:

* it still refers to `s`
* the object has identity
* we signal that its resources may be stolen

### Another Example

```cpp
T&& foo();

foo(); // xvalue
```

If a function returns an rvalue reference, the call expression is an xvalue.

---

# 8. Difference Between Prvalue and Xvalue

Consider:

```cpp
std::string("hello")
```

This is a **prvalue**.

It is a fresh temporary.

---

Now:

```cpp
std::string s = "hello";

std::move(s)
```

This is an **xvalue**.

It refers to an existing object.

### Summary

| Property           | Prvalue | Xvalue                    |
| ------------------ | ------- | ------------------------- |
| Has identity?      | No      | Yes                       |
| Temporary?         | Usually | Refers to existing object |
| Can be moved from? | Yes     | Yes                       |
| Category           | rvalue  | rvalue + glvalue          |

---

# 9. Move Semantics

Consider:

```cpp
std::vector<int> a(1000000);

std::vector<int> b = a;
```

Copy occurs.

---

With move:

```cpp
std::vector<int> b = std::move(a);
```

`std::move(a)` is an xvalue.

The move constructor can steal resources.

---

# 10. Lifetime Extension

One of the most important topics.

## Temporary Without Extension

```cpp
const std::string* p;

{
    p = &std::string("hello");
}
```

Invalid.

The temporary dies at the end of the full expression.

---

## Extension via Const Reference

```cpp
const std::string& ref = std::string("hello");
```

The temporary's lifetime is extended.

Equivalent lifetime:

```cpp
std::string __temp("hello");
const std::string& ref = __temp;
```

The temporary lives as long as `ref`.

---

### Example

```cpp
const int& x = 42;
```

Normally:

```cpp
42
```

would disappear immediately.

Lifetime extension keeps it alive.

---

## Extension via Rvalue Reference

C++11 added:

```cpp
std::string&& r = std::string("hello");
```

The temporary's lifetime is extended.

The temporary survives as long as `r`.

---

# 11. Cases Where Lifetime Extension Does NOT Happen

This is a common interview topic.

### Returning a Reference

```cpp
const std::string& foo()
{
    return std::string("hello");
}
```

Dangerous.

The temporary is destroyed before the caller receives it.

Reference dangles.

---

### Binding Through Another Function

```cpp
const std::string& identity(const std::string& s)
{
    return s;
}

const std::string& r = identity(std::string("hello"));
```

The temporary is **not** extended through the returned reference.

After initialization, `r` dangles.

---
### Member References

```cpp
struct S
{
    const std::string& ref;
};

S s{std::string("hello")};
```

Historically tricky and compiler-dependent. Modern C++ has special rules, but this remains an area where dangling references can easily occur. Prefer owning the object instead.

-------


