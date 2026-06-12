In C++, `const` is a keyword used to make something **read-only** after it has been initialized. It helps prevent accidental modification and makes code easier to understand and maintain.

## 1. Constant Variables

```cpp
const int age = 25;
```

After initialization, you cannot change `age`:

```cpp
age = 30;  // Error
```

### Why use it?

* Prevents accidental changes.
* Makes intent clear.
* Can help the compiler optimize code.

---

## 2. Const with Pointers

### Pointer to a const value

```cpp
const int x = 10;
const int* ptr = &x;
```

or

```cpp
int const* ptr = &x;
```

You can change the pointer, but not the value through the pointer:

```cpp
int y = 20;
ptr = &y;      // OK
*ptr = 30;     // Error
```

---

### Const pointer

```cpp
int x = 10;
int* const ptr = &x;
```

You cannot change the pointer itself:

```cpp
ptr = nullptr; // Error
```

But you can modify the value:

```cpp
*ptr = 20;     // OK
```

---

### Const pointer to const value

```cpp
const int x = 10;
const int* const ptr = &x;
```

Neither the pointer nor the value can be changed.

---

## 3. Const Function Parameters

Passing by const reference avoids copying and prevents modification.

```cpp
void print(const std::string& name)
{
    std::cout << name << '\n';
}
```

Inside the function:

```cpp
name += "!";   // Error
```

Benefits:

* No copy is made.
* The function cannot modify the original object.

---

## 4. Const Member Functions

A member function marked `const` promises not to modify the object's state.

```cpp
class Person
{
    std::string name;

public:
    std::string getName() const
    {
        return name;
    }
};
```

The compiler treats `this` as a pointer to a constant object inside the function.

Trying to modify data members:

```cpp
name = "Bob";  // Error
```

---

## 5. Const Objects

```cpp
const Person p;
```

Only `const` member functions can be called:

```cpp
p.getName();   // OK if getName() is const
p.setName();   // Error
```

---

## 6. Const Return Values

```cpp
const int getValue()
{
    return 42;
}
```

This is usually unnecessary for primitive types and is less common in modern C++.

---

## 7. Const References

```cpp
int x = 10;
const int& ref = x;
```

You cannot modify `x` through `ref`:

```cpp
ref = 20; // Error
```

But if `x` changes elsewhere:

```cpp
x = 30;
std::cout << ref; // 30
```

---

## 8. Constexpr vs Const

`const` means the value cannot be modified after initialization.

```cpp
const int x = 10;
```

`constexpr` means the value is known at compile time.

```cpp
constexpr int x = 10;
```

Example:

```cpp
constexpr int square(int n)
{
    return n * n;
}

constexpr int result = square(5);
```

`result` is computed during compilation.

---

## Reading Complex Const Declarations

A useful rule is: **read from the variable name outward**.

```cpp
const int* p;
```

`p` is a pointer to a const int.

```cpp
int* const p;
```

`p` is a const pointer to an int.

```cpp
const int* const p;
```

`p` is a const pointer to a const int.

---

## Best Practices

✅ Use `const` whenever a value should not change.

```cpp
const double PI = 3.1415926535;
```

✅ Pass large objects by `const` reference.

```cpp
void process(const std::vector<int>& data);
```

✅ Mark member functions `const` when they don't modify the object.

```cpp
int size() const;
```

✅ Prefer `constexpr` when the value can be determined at compile time.

---

### Quick Summary

| Declaration            | Meaning                             |
| ---------------------- | ----------------------------------- |
| `const int x = 5;`     | `x` cannot be modified              |
| `const int* p;`        | Pointer to const data               |
| `int* const p;`        | Const pointer                       |
| `const int* const p;`  | Const pointer to const data         |
| `void f(const T& t)`   | Read-only reference parameter       |
| `int get() const`      | Member function won't modify object |
| `constexpr int x = 5;` | Compile-time constant               |

A good mental model is: **`const` protects whatever is immediately to its left (or right if nothing is on the left).** This rule helps decode most C++ `const` declarations.

-----------------------------

## constexpr

`constexpr` is a C++ keyword that tells the compiler that a value, function, or object **can be evaluated at compile time** when given compile-time inputs.

It was introduced in C++11 and became much more powerful in C++14, C++17, and C++20.

---

# Why use `constexpr`?

Without `constexpr`:

```cpp
int square(int x) {
    return x * x;
}

int a = square(5);
```

`square(5)` is evaluated at runtime.

With `constexpr`:

```cpp
constexpr int square(int x) {
    return x * x;
}

constexpr int a = square(5);
```

The compiler computes `a = 25` during compilation.

---

# `const` vs `constexpr`

```cpp
const int x = 10;
```

`x` cannot be modified, but it is not necessarily a compile-time constant.

```cpp
constexpr int y = 10;
```

`y` is guaranteed to be a compile-time constant.

Example:

```cpp
int n = 5;

const int a = n;      // OK
constexpr int b = n;  // Error
```

`n` is not known until runtime, so `b` cannot be `constexpr`.

---

# Constexpr Variables

```cpp
constexpr double PI = 3.141592653589793;
```

The compiler knows the value during compilation.

Useful for:

```cpp
int arr[10];

constexpr int SIZE = 10;
int arr2[SIZE];
```

---

# Constexpr Functions

A `constexpr` function can be executed:

* At compile time if arguments are constant.
* At runtime if arguments are not constant.

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}
```

Compile-time evaluation:

```cpp
constexpr int result = add(2, 3);
```

Runtime evaluation:

```cpp
int x;
std::cin >> x;

int result = add(x, 3);
```

The same function works in both situations.

---

# Example: Factorial

```cpp
constexpr int factorial(int n)
{
    return n <= 1 ? 1 : n * factorial(n - 1);
}

constexpr int f = factorial(5);
```

The compiler computes:

```cpp
f = 120
```

during compilation.

---

# Using `constexpr` for Array Sizes

```cpp
constexpr int size()
{
    return 100;
}

int arr[size()];
```

Since `size()` is known at compile time, this is valid.

---

# Constexpr Objects

```cpp
class Point
{
public:
    int x;
    int y;

    constexpr Point(int a, int b)
        : x(a), y(b) {}
};

constexpr Point p(3, 4);
```

The object is created at compile time.

---

# Constexpr Member Functions

```cpp
class Point
{
public:
    int x;

    constexpr int getX() const
    {
        return x;
    }
};
```

If the object is `constexpr`, the function can be evaluated at compile time.

---

# C++11 vs Modern C++

### C++11

Very restrictive:

```cpp
constexpr int square(int x)
{
    return x * x;   // single return statement only
}
```

### C++14 and later

Much more flexible:

```cpp
constexpr int square(int x)
{
    int result = x * x;
    return result;
}
```

Loops and local variables are allowed.

```cpp
constexpr int sum(int n)
{
    int total = 0;

    for (int i = 1; i <= n; ++i)
        total += i;

    return total;
}
```

---

# `constexpr` and `if`

Since C++17, you can use `if constexpr`.

```cpp
template<typename T>
void print(T value)
{
    if constexpr (std::is_integral_v<T>)
    {
        std::cout << "Integer\n";
    }
    else
    {
        std::cout << "Not Integer\n";
    }
}
```

The compiler removes the unused branch during compilation.

---

# `constexpr` vs `consteval`

C++20 introduced `consteval`.

```cpp
consteval int square(int x)
{
    return x * x;
}
```

This **must** be evaluated at compile time.

```cpp
constexpr int a = square(5); // OK

int n = 5;
int b = square(n);           // Error
```

By contrast:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

can run at either compile time or runtime.

---

# Visual Summary

```cpp
const int x = 10;
```

* Read-only.
* May be compile-time or runtime.

```cpp
constexpr int x = 10;
```

* Read-only.
* Must be compile-time evaluable.

```cpp
consteval int f(int x)
{
    return x * x;
}
```

* Must execute at compile time.

---

## The Key Idea

Think of `constexpr` as saying:

> "This code is eligible to be executed by the compiler before the program even runs."

For example:

```cpp
constexpr int cube(int x)
{
    return x * x * x;
}

constexpr int value = cube(4);
```

The compiler effectively replaces it with:

```cpp
constexpr int value = 64;
```

before the executable is generated.
