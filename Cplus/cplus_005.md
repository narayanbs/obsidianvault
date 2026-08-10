# Initialization 

Here's the cheat sheet I'd memorize.

Assuming:

```cpp
struct Person {
    std::string name;
    int age;
};
```
With no constructor

| Syntax                             | What it does in C++14                                                                                         |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `Person p;`                        | Default-initialization; `name=""`, `age` indeterminate                                                        |
| `Person p{};`                      | Value/empty-list initialization; `name=""`, `age=0`                                                           |
| `Person p = {};`                   | Copy-list initialization; effectively `name=""`, `age=0`                                                      |
| `Person p{"john", 44};`            | Direct-list aggregate initialization                                                                          |
| `Person p = {"john", 44};`         | Copy-list aggregate initialization; **no Person temporary**                                                   |
| `Person p{"john"};`                | Aggregate initialization; `age=0`                                                                             |
| `Person p = {"john"};`             | Same idea; `age=0`                                                                                            |
| `Person p = Person{"john", 44};`   | Temporary `Person` then initialization from it; C++14 copy/move may be visible with `-fno-elide-constructors` |
| `Person p2 = p1;`                  | Copy construction                                                                                             |
| `Person p2(p1);`                   | Copy construction                                                                                             |
| `Person p2{p1};`                   | Copy construction                                                                                             |
| `Person p2 = std::move(p1);`       | Move construction                                                                                             |
| `p2 = p1;`                         | Copy assignment                                                                                               |
| `p2 = std::move(p1);`              | Move assignment                                                                                               |
| `new Person{"john", 44}`           | Dynamically allocate + aggregate-initialize                                                                   |
| `Person people[3]{};`              | Value-initialize all 3                                                                                        |
| `Person p("john", 44);`            | **Error in C++14**; works for aggregates in C++20                                                             |
| `Person p{.name="john", .age=44};` | **Error in C++14**; designated initialization is C++20                                                        |

Now assuming

```cpp
struct Person {
    std::string name;
    int age;

    Person(std::string n, int a) : name(std::move(n)), age(a) {}
};
```
with constructor

| Syntax                             | What it does in C++14                                                                                            |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `Person p;`                        | **Error** — no default constructor                                                                               |
| `Person p{};`                      | **Error** — no default constructor                                                                               |
| `Person p = {};`                   | **Error** — no default constructor                                                                               |
| `Person p{"john", 44};`            | Calls `Person(std::string, int)` directly                                                                        |
| `Person p = {"john", 44};`         | Calls `Person(std::string, int)` via copy-list-initialization; **no `Person` temporary**                         |
| `Person p{"john"};`                | **Error** — no matching constructor                                                                              |
| `Person p = {"john"};`             | **Error** — no matching constructor                                                                              |
| `Person p = Person{"john", 44};`   | Constructs a `Person` temporary, then initializes `p`; copy/move may be visible with `-fno-elide-constructors`   |
| `Person p2 = p1;`                  | Copy construction                                                                                                |
| `Person p2(p1);`                   | Copy construction                                                                                                |
| `Person p2{p1};`                   | Usually copy construction, **if no constructor makes this a better match**                                       |
| `Person p2 = std::move(p1);`       | Move construction                                                                                                |
| `p2 = p1;`                         | Copy assignment                                                                                                  |
| `p2 = std::move(p1);`              | Move assignment                                                                                                  |
| `new Person{"john", 44}`           | Dynamically allocate + call `Person(std::string, int)`                                                           |
| `Person people[3]{};`              | **Error** — requires a default constructor                                                                       |
| `Person p("john", 44);`            | Calls `Person(std::string, int)` directly                                                                        |
| `Person p{.name="john", .age=44};` | **Error in C++14**; designated initialization is C++20, and constructor-based initialization is different anyway |
# C++ Initialization trivia

```cpp
struct A {
    A(int) {}
};

int main() {
    A a1{20};      // direct-list initialization
    A a2 = 30;     // copy initialization
    A a3 = {40};   // copy-list initialization
}
```

All three are forms of **initialization**, not assignment.

### `A a1{20};`

This is **direct-list initialization**.

The compiler looks for a constructor that accepts the contents of the braces.

```cpp
A(int)
```

matches, so it constructs `a1`.

---

### `A a2 = 30;`

This is **copy initialization**.

Conceptually, the compiler does something like:

```cpp
A temp(30);
A a2 = temp;
```

In reality, thanks to copy elision, no temporary is created.

---

### `A a3 = {40};`

This is **copy-list initialization**.

The compiler first interprets `{40}` as an initializer list for constructing an `A`.

Since `A` has no `std::initializer_list` constructor, it falls back to ordinary constructors.

It finds

```cpp
A(int)
```

which can be called with `40`, so it's equivalent to

```cpp
A a3 = A{40};
```

Again, no extra object is created in practice.

---

## Why isn't `{40}` an `initializer_list`?

A common misconception is that braces always mean `std::initializer_list`.

They don't.

Braces simply mean **list initialization**. The compiler tries constructors in roughly this order:

1. Constructors taking `std::initializer_list<T>`
2. Otherwise, ordinary constructors that can accept the elements

For example:

```cpp
struct A {
    A(int) {}
};

A a{5};      // calls A(int)
```

because there is no `initializer_list` constructor.

But:

```cpp
struct B {
    B(std::initializer_list<int>) {}
    B(int) {}
};

B b{5};      // calls initializer_list constructor
```

because `initializer_list` constructors are preferred during list initialization.

---

## Why does `explicit` matter?

Now change your class:

```cpp
struct A {
    explicit A(int) {}
};

A a1{20};      // OK
A a2 = 30;     // Error
A a3 = {40};   // Error
```

Both `a2` and `a3` are forms of **copy initialization**, so they are **not allowed to use an `explicit` constructor**.

---

So, although `A a2 = 30;` and `A a3 = {40};` look similar, they are different kinds of initialization:

* `A a2 = 30;` → **copy initialization**
* `A a3 = {40};` → **copy-list initialization**

Both are allowed to call a non-`explicit` constructor like `A(int)`.

-------------

# List Initialization

In C++, **list initialization** means initializing an object using **curly braces `{}`**.

It was introduced in **C++11** and is now a very common way to initialize objects.

### 1. Basic idea

```cpp
int x{10};
```

Here, `x` is initialized to `10` using a braced initializer.

You can also initialize multiple values:

```cpp
int arr[]{1, 2, 3, 4};
```

And classes:

```cpp
struct Point {
    int x;
    int y;
};

Point p{10, 20};
```

The important thing is the **`{}`**. That's what makes it *list initialization*.

---

## Direct-list vs copy-list initialization

There are two forms of list initialization.

### Direct-list initialization

```cpp
int x{10};

Point p{10, 20};
```

The general form is:

```cpp
Type variable{values};
```

This is called **direct-list-initialization**.

---

### Copy-list initialization

```cpp
int x = {10};

Point p = {10, 20};
```

The general form is:

```cpp
Type variable = {values};
```

This is called **copy-list-initialization**.

So the simplest distinction is:

```cpp
int x{10};       // direct-list
int y = {10};    // copy-list
```

Both initialize an `int` to `10`, but the initialization rules are not exactly the same.

---

## Why does C++ have both?

The distinction becomes important when **constructors and implicit conversions** are involved.

For example:

```cpp
class Number {
public:
    Number(int x) {}
    explicit Number(double x) {}
};
```

With direct-list initialization:

```cpp
Number n1{10};       // OK
Number n2{10.5};     // OK
```

With copy-list initialization:

```cpp
Number n3 = {10};    // OK
Number n4 = {10.5};  // ERROR
```

Why?

`Number(double)` is marked `explicit`. Copy-list-initialization does not allow an explicit constructor to be used in this situation.

This is one reason the distinction matters.

---

# The really important feature: narrowing conversions

List initialization also has a special safety rule: **narrowing conversions are generally prohibited**.

For example:

```cpp
int x{3.14};     // ERROR
```

because `3.14` → `int` loses information.

But:

```cpp
int x = 3.14;    // allowed (x becomes 3)
```

So braces can catch certain accidental conversions at compile time.

This is one of the major reasons modern C++ programmers often prefer:

```cpp
int x{10};
```

over:

```cpp
int x = 10;
```

---

# One more thing: `std::initializer_list`

There is another important concept associated with `{}`.

Consider:

```cpp
std::vector<int> v{1, 2, 3};
```

The braces can be used to construct a container from an **initializer list**.

For example:

```cpp
std::vector<int> v{1, 2, 3, 4};
```

Conceptually, `{1, 2, 3, 4}` represents a list of values that can be used by a constructor.

This becomes particularly interesting because a class can have an `std::initializer_list` constructor, and **list initialization gives those constructors special treatment**.

For example:

```cpp
class MyClass {
public:
    MyClass(int x) {
        // ...
    }

    MyClass(std::initializer_list<int> values) {
        // ...
    }
};

MyClass obj{10};
```

Here, the `initializer_list` constructor can be preferred over the ordinary `int` constructor.

That's why you'll sometimes hear people say:

> "Be careful with brace initialization because `initializer_list` constructors can change overload resolution."

---

## A useful mental model

When you see these:

```cpp
T x(value);       // direct initialization
T x = value;      // copy initialization

T x{value};       // direct-list initialization
T x = {value};    // copy-list initialization
```

Think of them as four related but distinct initialization forms:

| Syntax           | Name                           |
| ---------------- | ------------------------------ |
| `T x(value);`    | Direct initialization          |
| `T x = value;`   | Copy initialization            |
| `T x{value};`    | **Direct-list initialization** |
| `T x = {value};` | **Copy-list initialization**   |

The **`{}`** is what makes the last two *list initialization*.


If you're learning C++ constructors, the next thing worth understanding is **why `T x{10}` can behave differently from `T x(10)`**, especially with `std::initializer_list`.


> **`T x{...}` and `T x(...)` can select different constructors.**

Let's build this up carefully.

### 1. Parentheses: ordinary direct initialization

Suppose we have:

```cpp
class Box {
public:
    Box(int x) {
        std::cout << "int constructor\n";
    }

    Box(std::initializer_list<int> values) {
        std::cout << "initializer_list constructor\n";
    }
};
```

Now:

```cpp
Box a(10);
```

prints:

```text
int constructor
```

Because `()` performs **direct initialization**, and the `int` constructor is the natural match.

---

### 2. Braces: list initialization

Now:

```cpp
Box b{10};
```

You might expect the same thing.

But it can print:

```text
initializer_list constructor
```

Why?

Because **list initialization gives `std::initializer_list` constructors special priority during overload resolution**.

So:

```cpp
Box a(10);   // prefers ordinary constructor
Box b{10};   // initializer_list constructor gets special treatment
```

This is one of the most important differences between `()` and `{}` in C++.

---

### 3. Why does `std::vector` make this especially confusing?

Consider:

```cpp
std::vector<int> a(5);
```

This means:

> Create a vector containing **5 integers**, initially zero-initialized.

So:

```cpp
a.size() == 5
```

But:

```cpp
std::vector<int> b{5};
```

means:

> Create a vector containing **one integer whose value is 5**.

So:

```cpp
b.size() == 1
b[0] == 5
```

This happens because `vector` has an `initializer_list` constructor.

Conceptually:

```text
vector<int> a(5)

        ↓

       5
       ↓
[ 0, 0, 0, 0, 0 ]


vector<int> b{5}

        ↓

initializer_list<int>{5}
        ↓
[ 5 ]
```

This is a classic C++ example.

---

### 4. Multiple values

Now:

```cpp
std::vector<int> v{10, 20, 30};
```

Clearly, the intent is:

```text
[10, 20, 30]
```

because the `initializer_list` constructor accepts the three integers.

Compare:

```cpp
std::vector<int> v1(10, 20);
```

This means:

> Create 10 integers, each initialized to 20.

So:

```text
[20, 20, 20, 20, 20, 20, 20, 20, 20, 20]
```

Whereas:

```cpp
std::vector<int> v2{10, 20};
```

means:

```text
[10, 20]
```

This is why you should **not blindly replace parentheses with braces**.

---

# 5. The initialization hierarchy to remember

For now, I'd recommend remembering this:

```cpp
T x(value);
```

means:

> **Direct initialization using parentheses.**

And:

```cpp
T x{value};
```

means:

> **Direct-list initialization using braces.**

When braces are involved, C++ gives `std::initializer_list` constructors special consideration.

So:

```cpp
T x(10);   // ordinary overload resolution
T x{10};   // initializer_list gets special priority
```

---

# 6. What about `T x = {10}`?

This is **copy-list initialization**:

```cpp
T x = {10};
```

It is still list initialization, so `initializer_list` constructors receive special treatment.

But there is an additional difference:

**explicit constructors cannot be used for copy-list initialization.**

For example:

```cpp
class A {
public:
    explicit A(int x) {}
};
```

Then:

```cpp
A a{10};       // OK: direct-list initialization
A b = {10};    // ERROR: explicit constructor
```

So:

```text
A a{10};
     ↑
direct-list
     ↓
explicit constructors can participate


A b = {10};
       ↑
copy-list
       ↓
explicit constructors cannot be used
```

---

# 7. One final distinction: `{}` is not always `initializer_list`

This is important.

Consider:

```cpp
struct Point {
    int x;
    int y;
};

Point p{10, 20};
```

There isn't necessarily an `initializer_list` here.

`Point` can simply be **aggregate-initialized**:

```text
Point
 ├── x = 10
 └── y = 20
```

So `{}` is a general **list-initialization syntax**. `std::initializer_list` is just one of the mechanisms that can participate in it.

That's a subtle but important distinction:

> **List initialization ≠ initializer_list.**

`initializer_list` is a library type:

```cpp
std::initializer_list<T>
```

while **list initialization** is a language feature involving `{}`.

---

### The mental model I'd use

When you see:

```cpp
T x(...);
```

think:

**"Call/select a constructor using parentheses."**

When you see:

```cpp
T x{...};
```

think:

**"Perform list initialization; check initializer-list constructors first."**

And remember:

```cpp
std::vector<int> a(5);   // 5 elements
std::vector<int> b{5};   // 1 element: 5
```

That one example captures a surprisingly large part of why this topic matters.


## Explicit  keyword -- More details

In C++, the `explicit` keyword is used on constructors (and conversion operators since C++11) to **prevent unintended implicit conversions**.

### Without `explicit`

Consider this class:

```cpp
#include <iostream>
using namespace std;

class Distance {
public:
    Distance(int meters) {
        cout << "Distance: " << meters << " meters\n";
    }
};

void printDistance(Distance d) {
    cout << "Printing distance\n";
}

int main() {
    Distance d = 10;      // Implicit conversion from int to Distance
    printDistance(20);    // Also implicitly converts int to Distance
}
```

**Output:**

```
Distance: 10 meters
Distance: 20 meters
Printing distance
```

Here, the compiler automatically converts an `int` into a `Distance` object because the constructor takes a single argument.

---

### With `explicit`

```cpp
#include <iostream>
using namespace std;

class Distance {
public:
    explicit Distance(int meters) {
        cout << "Distance: " << meters << " meters\n";
    }
};

void printDistance(Distance d) {}

int main() {
    Distance d = 10;      // ❌ Error
    printDistance(20);    // ❌ Error

    Distance d2(10);      // ✅ OK
    Distance d3{20};      // ✅ OK
    printDistance(Distance(30)); // ✅ OK
}
```

Now, the compiler **requires you to explicitly create the object** instead of silently converting an `int`.

---

## Why is this useful?

Suppose you have:

```cpp
class String {
public:
    explicit String(int size);
};
```

Without `explicit`, this would compile:

```cpp
String s = 100;
```

But what does `100` mean?

* A string with capacity 100?
* A string containing `"100"`?
* Something else?

Making the constructor `explicit` forces the programmer to write:

```cpp
String s(100);
```

which makes the intention clear.

---

## Rule of thumb

Use `explicit` for **single-parameter constructors** (or constructors that can be called with one argument due to default parameters) unless you intentionally want implicit conversions.

```cpp
class A {
public:
    explicit A(int x);      // Good practice
};

class B {
public:
    B(int x);               // Allows implicit conversion
};
```

---

## Constructors affected by `explicit`

It applies to constructors that can be used as conversion constructors:

```cpp
class Example {
public:
    explicit Example(int x);
    explicit Example(int x, int y = 0); // Also behaves like a single-argument constructor
};
```

---

## Summary

| Without `explicit`                | With `explicit`              |
| --------------------------------- | ---------------------------- |
| Allows implicit conversion        | Prevents implicit conversion |
| `A a = 5;` ✅                      | `A a = 5;` ❌                 |
| `func(5);` if `func(A)` exists ✅  | `func(5);` ❌                 |
| Convenient but can introduce bugs | Safer and clearer            |

In modern C++, it's generally recommended to mark constructors that are intended only for object creation—not implicit conversion—as `explicit`. This makes code more predictable and helps prevent accidental conversions.


