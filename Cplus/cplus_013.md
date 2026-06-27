# Templates

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

# Mental model

Think of a template as a blueprint rather than a finished object.

This:

```cpp
template<typename T>
class Box {
};
```

is **not** a class yet.

It is a recipe.

When you write

```cpp
Box<int>
```

the compiler uses the recipe to create a real class.

Similarly,

```cpp
template<typename T>
T max(T a, T b);
```

is a recipe for generating functions.

Each unique type produces a separate instantiation.

---

## A good learning path

To build confidence with templates, I recommend practicing in this order:

1. Function templates (`max`, `swap`, `add`).
2. Class templates (`Pair<T>`, `Box<T>`).
3. Multiple template parameters (`Pair<T, U>`).
4. Non-type template parameters (`Array<T, N>`).
5. Template specialization.
6. Variadic templates (functions/classes with a variable number of template arguments).
7. Concepts (C++20) to constrain template parameters.

This progression mirrors how templates are used in real-world C++ code, from simple generic functions to the advanced metaprogramming techniques that underpin much of the Standard Library.

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
