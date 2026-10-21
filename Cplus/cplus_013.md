# Templates , Perfect Forwarding, move Internals

C++ templates are one of the language's most powerful features. They let you write code once and have the compiler generate versions of it for different types. If you've ever wished you could write a function that works for both `int` and `double` without copying the code, templates solve that problem.

I'll build the idea gradually.

## Step 1: The problem templates solve

Imagine you want a function that returns the larger of two numbers.

Without templates:

```cpp
int max(int a, int b) {
    return (a > b) ? a : b;
}

double max(double a, double b) {
    return (a > b) ? a : b;
}

char max(char a, char b) {
    return (a > b) ? a : b;
}
```

The logic is identical. Only the type changes.

Templates let you write it once.

```cpp
template<typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}
```

Now you can write:

```cpp
int x = max(3, 7);

double y = max(4.5, 2.1);

char c = max('A', 'Z');
```

The compiler automatically creates the appropriate function for each type.

Think of `T` as a placeholder for "some type."

---

## Step 2: What actually happens?

When the compiler sees

```cpp
max(3, 7);
```

it generates something similar to

```cpp
int max(int a, int b) {
    return (a > b) ? a : b;
}
```

When it later sees

```cpp
max(2.5, 7.1);
```

it generates

```cpp
double max(double a, double b) {
    return (a > b) ? a : b;
}
```

This process is called **template instantiation**.

Templates are a compile-time feature.

---

# Step 3: `typename` vs `class`

You'll see both.

```cpp
template<typename T>
```

and

```cpp
template<class T>
```

For template parameters, they mean the same thing.

Most modern code prefers

```cpp
template<typename T>
```

because it's more descriptive.

---

# Step 4: More than one type

Suppose you want two different types.

```cpp
template<typename T, typename U>
void print(T a, U b) {
    std::cout << a << " " << b;
}
```

Usage:

```cpp
print(5, "hello");

print(3.14, true);

print("Age", 20);
```

Here

```
T = int
U = const char*
```

for the first call.

---

# Step 5: Template classes

Templates aren't only for functions.

Suppose you want a class representing a pair.

Without templates:

```cpp
class IntPair {
public:
    int first;
    int second;
};
```

This only works for integers.

Template version:

```cpp
template<typename T>
class Pair {
public:
    T first;
    T second;
};
```

Usage:

```cpp
Pair<int> p1;

Pair<double> p2;

Pair<std::string> p3;
```

Notice that for classes you explicitly specify the type.

```cpp
Pair<int>
```

---

# Step 6: A real example

Suppose we write our own simple container.

```cpp
template<typename T>
class Box {
private:
    T value;

public:
    Box(T v) : value(v) {}

    T get() {
        return value;
    }
};
```

Usage:

```cpp
Box<int> a(10);

Box<double> b(3.14);

Box<std::string> c("hello");

std::cout << a.get();
```

The compiler generates different versions.

```
Box<int>

Box<double>

Box<std::string>
```

---

# Step 7: Why the STL uses templates

Consider `std::vector`.

You write

```cpp
std::vector<int>
```

or

```cpp
std::vector<double>
```

or

```cpp
std::vector<std::string>
```

There isn't a separate implementation for every type.

There's only one template.

Conceptually:

```cpp
template<typename T>
class Vector {
    ...
};
```

Then the compiler creates

```
Vector<int>

Vector<double>

Vector<std::string>
```

This is why almost every Standard Library container is templated.

---

# Step 8: Non-type template parameters

Templates can also accept values.

Example:

```cpp
template<typename T, int N>
class Array {
private:
    T data[N];
};
```

Usage:

```cpp
Array<int, 10> numbers;

Array<double, 5> values;
```

Here

```
T = int
N = 10
```

The size is known at compile time.

---

# Step 9: Default template parameters

```cpp
template<typename T = int>
class Number {
    T value;
};
```

Now you can write

```cpp
Number<> n;
```

which means

```cpp
Number<int>
```

Or

```cpp
Number<double> d;
```

---

# Step 10: Template specialization

Sometimes one type needs different behavior.

General version:

```cpp
template<typename T>
class Printer {
public:
    void print() {
        std::cout << "Generic";
    }
};
```

Special version:

```cpp
template<>
class Printer<bool> {
public:
    void print() {
        std::cout << "True/False Printer";
    }
};
```

Now

```cpp
Printer<int> p1;
```

uses the generic version.

```cpp
Printer<bool> p2;
```

uses the specialized version.

---

# Step 11: Constraints

Templates assume the operations you use are valid.

Consider

```cpp
template<typename T>
T add(T a, T b) {
    return a + b;
}
```

Works:

```cpp
add(2, 3);

add(2.5, 4.6);

add(std::string("Hi"), std::string("!"));
```

Fails:

```cpp
struct Person {};

Person p1, p2;

add(p1, p2);
```

because `Person` has no `operator+`.

The compiler reports an error when it tries to instantiate the template.

---

# Step 12: Concepts (Modern C++20)

Instead of allowing any type, you can specify requirements.

```cpp
template<typename T>
requires std::integral<T>
T square(T x) {
    return x * x;
}
```

Only integral types are accepted.

Examples:

```cpp
square(5);      // OK

square(10L);    // OK

square(3.14);   // Error
```

Concepts make template errors much easier to understand.

---

## What is the use of Non-type parameters? Can't we use a constant instead

That's an excellent question. At first glance, it *does* seem like you could just use a constant. The key difference is **who chooses the constant** and **when it's known**.

Let's compare the two approaches.

### Using a regular constructor parameter

```cpp
class Array {
    int* data;
    int size;

public:
    Array(int n) : size(n) {
        data = new int[n];
    }
};
```

Usage:

```cpp
Array a(10);
Array b(20);
```

Here, `10` and `20` are runtime values. The compiler doesn't know the size when it's compiling the class—it only knows when the object is created.

---

### Using a non-type template parameter

```cpp
template<typename T, int N>
class Array {
    T data[N];
};
```

Usage:

```cpp
Array<int, 10> a;
Array<int, 20> b;
```

Here, `10` and `20` are part of the **type itself**.

These are two different types:

```cpp
Array<int, 10>
Array<int, 20>
```

This distinction has several advantages.

---

## 1. No dynamic allocation

With a constructor parameter:

```cpp
class Array {
    int* data;
};
```

You usually need:

```cpp
data = new int[n];
```

With a template parameter:

```cpp
template<int N>
class Array {
    int data[N];
};
```

The array is stored directly inside the object.

Benefits:

* No `new`
* No `delete`
* Better cache locality
* Faster access

---

## 2. The compiler knows the size

Suppose you write

```cpp
template<int N>
class Array {
    int data[N];
};
```

The compiler knows `N` while generating the code.

It can:

* unroll loops,
* eliminate bounds checks (when possible),
* optimize memory layout,
* compute `sizeof(Array<N>)`.

Example:

```cpp
Array<int, 100> arr;
```

The compiler already knows:

```cpp
sizeof(arr) == 100 * sizeof(int)
```

No runtime calculation is needed.

---

## 3. The size becomes part of the type

This is the biggest conceptual difference.

```cpp
Array<int, 10>
```

and

```cpp
Array<int, 20>
```

are unrelated types.

So this won't compile:

```cpp
Array<int, 10> a;
Array<int, 20> b;

a = b;      // Error
```

because they represent arrays of different fixed sizes.

This lets the compiler catch mistakes.

---

## 4. You can specialize based on the value

Templates can specialize on non-type parameters.

```cpp
template<int N>
class Buffer {
    // General implementation
};

template<>
class Buffer<1> {
    // Optimized implementation for size 1
};
```

You couldn't do that with a constructor argument because constructor arguments aren't known when the class definition is compiled.

---

## 5. Function overloading based on size

```cpp
template<int N>
void process(Array<int, N>& arr) {
    std::cout << N << '\n';
}
```

Now:

```cpp
Array<int, 32> a;
process(a);
```

The compiler automatically knows `N` is `32`.

If `N` were just a member variable:

```cpp
class Array {
    int size;
};
```

the compiler couldn't treat different sizes as different types.

---

## Why not just use `const int`?

Suppose you have

```cpp
const int SIZE = 10;
```

and

```cpp
class Array {
    int data[SIZE];
};
```

This works—but **every** `Array` has size 10.

What if you also want size 20?

You'd need another class:

```cpp
class Array10 { ... };

class Array20 { ... };

class Array50 { ... };
```

Templates let you write **one** class:

```cpp
template<int N>
class Array {
    int data[N];
};
```

and instantiate it as:

```cpp
Array<10>
Array<20>
Array<50>
Array<1000>
```

without duplicating code.

---

### A good mental model

Think of

```cpp
template<int N>
class Array { };
```

as a **class factory**.

When you write:

```cpp
Array<10>
```

the compiler generates a class roughly like:

```cpp
class Array10 {
    int data[10];
};
```

When you write:

```cpp
Array<20>
```

it generates another class:

```cpp
class Array20 {
    int data[20];
};
```

You didn't write two classes—the compiler did it for you.

---

This idea is used throughout modern C++. A familiar example is `std::array`:

```cpp
std::array<int, 5> a;
std::array<int, 100> b;
```

The `5` and `100` are non-type template parameters. Because the size is part of the type, `std::array` stores its elements inline (no heap allocation), and the compiler can optimize many operations based on the known fixed size.

-------------

# Perfect Forwarding in C++

Perfect forwarding is a technique that allows a function template to pass its arguments to another function **while preserving the original value category** (lvalue/rvalue) and cv-qualifiers (`const`, `volatile`).

It is one of the major motivations behind:

* rvalue references (`T&&`)
* reference collapsing
* `std::forward`


One of the most important concepts behind perfect forwarding is the **forwarding reference** (often incorrectly called a "universal reference"). A forwarding reference allows a function template to accept an argument of *any value category* and later pass that argument along while preserving its original properties.

`std::forward` is the tool that makes this possible.

---

# The Problem

Consider a simple wrapper function:

```cpp
void process(int& x)
{
    std::cout << "lvalue\n";
}

void process(int&& x)
{
    std::cout << "rvalue\n";
}

template<typename T>
void wrapper(T value)
{
    process(value);
}
```

Now:

```cpp
int a = 10;

wrapper(a);       // ?
wrapper(20);      // ?
```

You might expect:

```
lvalue
rvalue
```

But the output is:

```
lvalue
lvalue
```

Why?

Because inside `wrapper`, the parameter `value` is a **named variable**.

A named variable is always an **lvalue**, even if the object originally came from an rvalue.

Example:

```cpp
int&& ref = 10;

process(ref); 
```

This calls:

```cpp
process(int&)
```

not:

```cpp
process(int&&)
```

The name `ref` makes it an lvalue expression.

This creates a problem for generic wrapper functions:

* The caller passes an rvalue.
* The wrapper receives it.
* The wrapper loses the information that it was originally an rvalue.

---

# 3. Forwarding References

A forwarding reference is a special form of rvalue reference:

```cpp
template<typename T>
void wrapper(T&& value)
{
}
```

The key rule:

> If `T` is a deduced template parameter, then `T&&` is a forwarding reference.

This allows the compiler to deduce `T` differently depending on what is passed.

Example:

```cpp
int x = 10;

wrapper(x);
wrapper(20);
```

### Case 1: Passing an lvalue

```cpp
wrapper(x);
```

Deduction:

```
T = int&
```

Substitution:

```
T&& becomes int& && 
```

Reference collapsing rules apply:

```
int& &&  -> int&
```

So the function becomes:

```cpp
void wrapper(int& value)
```

---

### Case 2: Passing an rvalue

```cpp
wrapper(20);
```

Deduction:

```
T = int
```

Substitution:

```
T&& becomes int&&
```

The function becomes:

```cpp
void wrapper(int&& value)
```

So a forwarding reference can preserve the original category.

---

# 4. The Remaining Problem

Even with a forwarding reference:

```cpp
template<typename T>
void wrapper(T&& value)
{
    process(value);
}
```

we still have:

```cpp
wrapper(20);
```

Inside:

```cpp
process(value);
```

`value` is named.

Therefore:

```
value -> lvalue
```

The output is still:

```
lvalue
```

The compiler knows the original type, but the expression itself has lost the value category.

This is where `std::forward` comes in.

---

# 5. What `std::forward` Does

`std::forward` restores the original value category.

Example:

```cpp
template<typename T>
void wrapper(T&& value)
{
    process(std::forward<T>(value));
}
```

Now:

```cpp
int x = 10;

wrapper(x);
wrapper(20);
```

Output:

```
lvalue
rvalue
```

---

# 6. How `std::forward` Works

`std::forward` is essentially a conditional cast.

Simplified implementation:

```cpp
template<typename T>
T&& forward(std::remove_reference_t<T>& arg)
{
    return static_cast<T&&>(arg);
}
```

The important part:

```cpp
static_cast<T&&>(arg)
```

The result depends on what `T` was originally deduced as.

---

## Example 1: Lvalue

Call:

```cpp
int x;

wrapper(x);
```

Deduction:

```
T = int&
```

Inside:

```cpp
std::forward<T>(value)
```

becomes:

```cpp
std::forward<int&>(value)
```

Return type:

```
int& &&
```

Reference collapsing:

```
int&
```

Result:

```cpp
process(int&)
```

---

## Example 2: Rvalue

Call:

```cpp
wrapper(10);
```

Deduction:

```
T = int
```

Inside:

```cpp
std::forward<int>(value)
```

Return type:

```
int&&
```

Result:

```cpp
process(int&&)
```
---

# 7. Why Not Just Use `std::move`?

A common misconception is that `std::move` and `std::forward` are interchangeable.

They are not.

## `std::move`

`std::move` always produces an rvalue:

```cpp
std::move(x)
```

Example:

```cpp
int x = 10;

process(std::move(x));
```

Calls:

```
process(int&&)
```

Even though `x` was originally an lvalue.

---

## `std::forward`

`std::forward` preserves the original category:

```cpp
process(std::forward<T>(x));
```

If `x` was originally:

```
lvalue -> remains lvalue
rvalue -> becomes rvalue
```

---

# 8. Real-World Example: Generic Factory Functions

A common use case is forwarding constructor arguments.

Without forwarding:

```cpp
template<typename T, typename Arg>
T create(Arg arg)
{
    return T(arg);
}
```

Problem:

```cpp
std::string s = "hello";

create<std::vector<std::string>>(s);
```

The argument is copied unnecessarily.

With forwarding:

```cpp
template<typename T, typename... Args>
T create(Args&&... args)
{
    return T(std::forward<Args>(args)...);
}
```

Now:

```cpp
create<std::string>("hello");
```

can move temporary objects efficiently.

This is the technique used by:

* `std::make_unique`
* `std::make_shared`
* `std::emplace_back`

---

# 9. Relationship Between Forward References and Perfect Forwarding

The complete pattern is:

```cpp
template<typename T>
void function(T&& arg)
{
    another_function(std::forward<T>(arg));
}
```

This provides:

| Feature              | Purpose                         |
| -------------------- | ------------------------------- |
| `T&&`                | Accept lvalues and rvalues      |
| Template deduction   | Remember original type          |
| Reference collapsing | Convert `T&&` correctly         |
| `std::forward`       | Restore original value category |

Together these enable **perfect forwarding**.

---

# 10. Important Restrictions

Not every `T&&` is a forwarding reference.

### Forwarding reference:

```cpp
template<typename T>
void f(T&& x);
```

Because `T` is deduced.

---

### Not forwarding:

```cpp
void f(std::string&& x);
```

This is simply an rvalue reference.

---

### Also not forwarding:

```cpp
template<typename T>
struct A
{
    void f(T&& x);
};
```

Here `T` is already known when the class is instantiated.

# 11. Summary

Before C++11, wrapper functions often had to create separate overloads:

```cpp
void wrapper(int&);
void wrapper(int&&);
```

Forwarding references solve this by allowing one template function to accept both cases:

```cpp
template<typename T>
void wrapper(T&& value);
```

However, because named variables are always lvalues, `std::forward` is required:

```cpp
std::forward<T>(value);
```

It restores the original value category and enables **perfect forwarding**, allowing generic libraries to:

* avoid unnecessary copies,
* preserve move semantics,
* write efficient factory functions,
* implement utilities such as `make_unique` and `emplace_back`.

In short:

> A forwarding reference preserves *what was passed*.
> `std::forward` preserves *how it should behave when passed onward*.



---
##  Deduction Logic 

This is one of the trickiest parts of C++ templates. The key idea is that type and value category are different things.
Consider
```
std::string s = "hello";
```

The type of `s` is `std::string`. But the expression s is an `lvalue`.

When you write:
~~~
template<typename T>
void wrapper(T&& arg);
~~~
and call:
~~~
wrapper(arg);
~~~
``
the compiler does not just look at the type `std::string`. It also notices that the argument expression s is an `lvalue`.

---

# Reference Collapsing Refresher

There are only **4 rules**:

```cpp
T&  &  -> T&
T&  && -> T&
T&& &  -> T&
T&& && -> T&&
```


A useful way to remember it:

> **If either side is an lvalue reference (`&`), the result is `&`.**
>
> Only `&& + &&` stays `&&`.


Rules:

The important thing:

> Any combination involving an lvalue reference becomes an lvalue reference.

---

# Real Example: emplace_back

Consider:

```cpp
std::vector<std::string> v;

v.emplace_back("hello");
```

A simplified implementation:

```cpp
template<typename... Args>
void emplace_back(Args&&... args)
{
    new(storage)
        T(std::forward<Args>(args)...);
}
```

Suppose:

```cpp
std::string s = "hello";

v.emplace_back(s);
```

Deduction:

```cpp
Args = std::string&
```

Forwarding preserves lvalue.

The copy constructor is used.

---

Now:

```cpp
v.emplace_back(std::string("hello"));
```

Deduction:

```cpp
Args = std::string
```

Forwarding preserves rvalue.

The move constructor is used.

Without perfect forwarding, `emplace_back` would lose this information.

---

# Variadic Templates and Perfect Forwarding

One of the most common patterns:

```cpp
template<typename F, typename... Args>
decltype(auto) invoke(F&& f, Args&&... args)
{
    return std::forward<F>(f)(
        std::forward<Args>(args)...);
}
```

This is essentially what many library utilities do.

Examples:

* `std::make_unique`
* `std::make_shared`
* `std::thread`
* `std::optional::emplace`
* `std::variant::emplace`
* `std::tuple` constructors

---

# Why Not Always Use std::move?

Consider:

```cpp
template<typename T>
void wrapper(T&& arg)
{
    process(std::move(arg));
}
```

Now:

```cpp
std::string s;

wrapper(s);
```

Inside:

```cpp
std::move(arg)
```

forces an rvalue.

The caller's lvalue gets moved from unexpectedly.

This is usually a bug.

Perfect forwarding avoids this.

---

# A Mental Model

Suppose a caller hands you an object:

```cpp
wrapper(argument);
```

Your wrapper has two choices:

### Force moving

```cpp
std::move(argument)
```

Meaning:

> "I don't care how it arrived; I'm treating it as expendable."

---

### Perfect forwarding

```cpp
std::forward<T>(argument)
```

Meaning:

> "I will pass it onward exactly the way I received it."

That's why it is called **perfect forwarding**.

---

# The One Rule to Memorize

Whenever you see:

```cpp
template<typename T>
void f(T&& x)
```

and `T` is deduced,

you should almost immediately ask:

```cpp
std::forward<T>(x)
```

because otherwise the original value category is usually lost.

This pairing—

```cpp
T&&
std::forward<T>()
```

—is the heart of perfect forwarding in modern C++.


-------------------------------------
---------------------------------

# std::move internals


This is the heart of move semantics. The surprising thing is:

> **`std::move` doesn't move anything.**
>
> It is just a cast.

Let's go through it carefully.

---

## The implementation

```cpp
template<typename T>
constexpr std::remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(t);
}
```

Suppose:

```cpp 
std::string a = "hello";
```

and you write:

```cpp 
std::move(a)
```

---

## Step 1: Template deduction

The parameter is:

```cpp
T&& t
```

Since `a` is an lvalue, template deduction for a forwarding reference gives:

```cpp 
T = std::string&
```

So substitute into the function:

```cpp 
std::remove_reference_t<std::string&>&& move(std::string& && t)
```

---

## Step 2: Reference collapsing

```cpp 
std::string& &&
```

collapses to:

```cpp
std::string&
```

So now we have:

```cpp
std::remove_reference_t<std::string&>&& move(std::string& t)
```

---

## Step 3: remove_reference_t

`remove_reference_t<T>` removes `&` and `&&`.

So:

```cpp
std::remove_reference_t<std::string&>
```

becomes:

```cpp id="9j0yt6"
std::string
```

Now the function is effectively:

```cpp
std::string&& move(std::string& t)
```

---

## Step 4: The cast

Return statement:

```cpp 
return static_cast<std::string&&>(t);
```

This converts the lvalue expression `t` into an xvalue (an expiring value).

No object is created.

No data is copied.

No data is moved.

We're just changing how the compiler views the expression.

---

## Visualize it

Before:

```cpp 
std::string a = "hello";
```

Memory:

```text
a ---> "hello"
```

After:

```cpp 
auto&& r = std::move(a);
```

Memory:

```text
a ---> "hello"
 ^
 |
 r
```

Same object.

Different reference type.

---

## Why do we need remove_reference_t?

Imagine it wasn't there:

```cpp
template<typename T>
T&& bad_move(T&& t) {
    return static_cast<T&&>(t);
}
```

Call:

```cpp 
std::string a;
bad_move(a);
```

Deduction:

```cpp 
T = std::string&
```

Return type becomes:

```cpp 
std::string& &&
```

Collapse:

```cpp 
std::string&
```

Oops!

You returned an lvalue reference, not an rvalue reference.

The move capability is lost.

That's why `remove_reference_t` is critical.

It forces:

```cpp 
std::string&&
```

regardless of whether `T` was:

```cpp 
std::string
std::string&
std::string&&
```

---

## What does the cast actually achieve?

Consider:

```cpp 
std::string a = "hello";

std::string b = std::move(a);
```

Without `std::move`:

```cpp 
std::string b = a;
```

Compiler sees:

```cpp id="u8r27k"
a
```

which is an lvalue.

So it chooses:

```cpp id="23lz9o"
string(const string&)
```

(copy constructor)

---

With `std::move`:

```cpp
std::string b = std::move(a);
```

Compiler sees:

```cpp 
std::string&&
```

So it chooses:

```cpp 
string(string&&)
```

(move constructor)

---

## The one-line interpretation

This entire function:

```cpp 
template<typename T>
constexpr std::remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<std::remove_reference_t<T>&&>(t);
}
```

is essentially saying:

> "Take whatever object you give me and treat it as an rvalue, so move constructors and move assignments become eligible."

No moving happens inside `std::move` itself. The move happens later when another function sees that rvalue and chooses a move operation instead of a copy operation.